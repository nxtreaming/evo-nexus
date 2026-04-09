# Channels Reference — Construindo Channels Customizados

Referência técnica para construir um MCP server que empurra webhooks, alertas e mensagens de chat para uma sessão Claude Code.

> Para usar channels já prontos (Telegram, Discord, iMessage), veja o [Guia de Channels](channels.md).

## Visão Geral

Um channel é um MCP server que roda na mesma máquina que o Claude Code. O Claude Code o inicia como subprocesso e comunica via stdio. Seu channel é a ponte entre sistemas externos e a sessão:

- **Chat platforms** (Telegram, Discord): seu plugin roda localmente e faz polling da API da plataforma. Quando alguém envia uma DM para o bot, o plugin recebe e encaminha para Claude.
- **Webhooks** (CI, monitoring): seu server escuta em uma porta HTTP local. Sistemas externos fazem POST nessa porta, e seu server empurra o payload para Claude.

## Requisitos

- [`@modelcontextprotocol/sdk`](https://www.npmjs.com/package/@modelcontextprotocol/sdk) (MCP SDK)
- Runtime Node.js-compatível: [Bun](https://bun.sh), [Node](https://nodejs.org) ou [Deno](https://deno.com)

Seu server precisa:

1. Declarar a capability `claude/channel` para que Claude Code registre um listener de notificações
2. Emitir eventos `notifications/claude/channel` quando algo acontecer
3. Conectar via [stdio transport](https://modelcontextprotocol.io/docs/concepts/transports#standard-io)

## Exemplo: Webhook Receiver

Um server single-file que escuta HTTP requests e encaminha para a sessão Claude Code.

### 1. Criar o Projeto

```bash
mkdir webhook-channel && cd webhook-channel
bun add @modelcontextprotocol/sdk
```

### 2. Escrever o Server

```ts
// webhook.ts
#!/usr/bin/env bun
import { Server } from '@modelcontextprotocol/sdk/server/index.js'
import { StdioServerTransport } from '@modelcontextprotocol/sdk/server/stdio.js'

const mcp = new Server(
  { name: 'webhook', version: '0.0.1' },
  {
    capabilities: { experimental: { 'claude/channel': {} } },
    instructions: 'Events from the webhook channel arrive as <channel source="webhook" ...>. Read them and act, no reply expected.',
  },
)

await mcp.connect(new StdioServerTransport())

Bun.serve({
  port: 8788,
  hostname: '127.0.0.1',
  async fetch(req) {
    const body = await req.text()
    await mcp.notification({
      method: 'notifications/claude/channel',
      params: {
        content: body,
        meta: { path: new URL(req.url).pathname, method: req.method },
      },
    })
    return new Response('ok')
  },
})
```

### 3. Registrar no MCP Config

```json
// .mcp.json
{
  "mcpServers": {
    "webhook": { "command": "bun", "args": ["./webhook.ts"] }
  }
}
```

### 4. Testar

```bash
# Terminal 1: iniciar Claude Code com flag de desenvolvimento
claude --dangerously-load-development-channels server:webhook

# Terminal 2: simular um webhook
curl -X POST localhost:8788 -d "build failed on main: https://ci.example.com/run/1234"
```

O payload chega como:

```
<channel source="webhook" path="/" method="POST">build failed on main: https://ci.example.com/run/1234</channel>
```

## Server Options

| Campo | Tipo | Descrição |
|-------|------|-----------|
| `capabilities.experimental['claude/channel']` | `object` | **Obrigatório.** Sempre `{}`. Registra o listener de notificações |
| `capabilities.experimental['claude/channel/permission']` | `object` | Opcional. Declara que o channel pode receber permission relay |
| `capabilities.tools` | `object` | Apenas bidirecional. Sempre `{}`. Habilita tool discovery |
| `instructions` | `string` | Recomendado. Adicionado ao system prompt do Claude |

## Notification Format

Emitir `notifications/claude/channel` com dois params:

| Campo | Tipo | Descrição |
|-------|------|-----------|
| `content` | `string` | O corpo do evento. Entregue como body da tag `<channel>` |
| `meta` | `Record<string, string>` | Opcional. Cada entry vira um atributo na tag `<channel>` |

```ts
await mcp.notification({
  method: 'notifications/claude/channel',
  params: {
    content: 'build failed on main',
    meta: { severity: 'high', run_id: '1234' },
  },
})
```

Resultado no contexto do Claude:

```
<channel source="your-channel" severity="high" run_id="1234">
build failed on main
</channel>
```

**Nota sobre meta keys:** apenas letras, dígitos e underscores. Keys com hífens são silenciosamente descartadas.

## Expose a Reply Tool (Bidirecional)

Para channels bidirecionais (chat bridges), exponha um MCP tool padrão:

```ts
import { ListToolsRequestSchema, CallToolRequestSchema } from '@modelcontextprotocol/sdk/types.js'

// Tool discovery
mcp.setRequestHandler(ListToolsRequestSchema, async () => ({
  tools: [{
    name: 'reply',
    description: 'Send a message back over this channel',
    inputSchema: {
      type: 'object',
      properties: {
        chat_id: { type: 'string', description: 'The conversation to reply in' },
        text: { type: 'string', description: 'The message to send' },
      },
      required: ['chat_id', 'text'],
    },
  }],
}))

// Tool handler
mcp.setRequestHandler(CallToolRequestSchema, async req => {
  if (req.params.name === 'reply') {
    const { chat_id, text } = req.params.arguments as { chat_id: string; text: string }
    // Enviar para a plataforma de chat
    await sendToYourPlatform(chat_id, text)
    return { content: [{ type: 'text', text: 'sent' }] }
  }
  throw new Error(`unknown tool: ${req.params.name}`)
})
```

Atualize as `instructions` para que Claude saiba usar o tool:

```ts
instructions: 'Messages arrive as <channel source="..." chat_id="...">. Reply with the reply tool, passing the chat_id from the tag.'
```

## Gate Inbound Messages (Segurança)

Um channel sem gate é um vetor de prompt injection. Sempre verifique o remetente contra uma allowlist **antes** de emitir:

```ts
const allowed = new Set(loadAllowlist())

// No handler de mensagens:
if (!allowed.has(message.from.id)) {
  return  // descarte silenciosamente
}
await mcp.notification({ ... })
```

**Importante:** faça gate no ID do remetente (`message.from.id`), não no chat/room (`message.chat.id`). Em grupos, qualquer pessoa no room poderia injetar mensagens.

## Permission Relay

> Requer Claude Code v2.1.81+

Channels bidirecionais podem encaminhar prompts de permissão para aprovação remota.

### Como Funciona

1. Claude Code gera um request ID e notifica seu server
2. Seu server encaminha o prompt para o chat
3. O usuário remoto responde com `yes <id>` ou `no <id>`
4. Seu handler parseia a resposta e emite um verdict

O dialog local do terminal permanece aberto — a primeira resposta (local ou remota) é aplicada.

### Implementação

**1. Declarar a capability:**

```ts
capabilities: {
  experimental: {
    'claude/channel': {},
    'claude/channel/permission': {},  // opt-in para relay
  },
  tools: {},
},
```

**2. Handler para incoming request:**

```ts
import { z } from 'zod'

const PermissionRequestSchema = z.object({
  method: z.literal('notifications/claude/channel/permission_request'),
  params: z.object({
    request_id: z.string(),
    tool_name: z.string(),
    description: z.string(),
    input_preview: z.string(),
  }),
})

mcp.setNotificationHandler(PermissionRequestSchema, async ({ params }) => {
  send(
    `Claude wants to run ${params.tool_name}: ${params.description}\n\n` +
    `Reply "yes ${params.request_id}" or "no ${params.request_id}"`,
  )
})
```

**3. Interceptar verdict no inbound handler:**

```ts
const PERMISSION_REPLY_RE = /^\s*(y|yes|n|no)\s+([a-km-z]{5})\s*$/i

// No handler de mensagens inbound:
const m = PERMISSION_REPLY_RE.exec(message.text)
if (m) {
  await mcp.notification({
    method: 'notifications/claude/channel/permission',
    params: {
      request_id: m[2].toLowerCase(),
      behavior: m[1].toLowerCase().startsWith('y') ? 'allow' : 'deny',
    },
  })
  return  // não encaminhe como chat
}
```

### Permission Request Fields

| Campo | Descrição |
|-------|-----------|
| `request_id` | 5 letras minúsculas (sem `l`). Inclua no prompt para ser ecoado na resposta |
| `tool_name` | Nome do tool (ex: `Bash`, `Write`) |
| `description` | Resumo legível do que o tool call faz |
| `input_preview` | Argumentos do tool como JSON, truncado em 200 chars |

## Empacotar como Plugin

Para tornar seu channel instalável e compartilhável:

1. Empacote como [plugin](https://code.claude.com/docs/en/plugins)
2. Publique em um [marketplace](https://code.claude.com/docs/en/plugin-marketplaces)
3. Usuários instalam com `/plugin install` e habilitam com `--channels plugin:<name>@<marketplace>`

Channels publicados em marketplaces próprios ainda precisam de `--dangerously-load-development-channels` até serem adicionados à allowlist oficial ou à `allowedChannelPlugins` da organização.

## Testar Durante o Research Preview

Custom channels não estão na allowlist aprovada. Use a flag de desenvolvimento:

```bash
# Plugin em desenvolvimento
claude --dangerously-load-development-channels plugin:yourplugin@yourmarketplace

# Server direto do .mcp.json
claude --dangerously-load-development-channels server:webhook
```

O bypass é por-entry. A policy `channelsEnabled` da organização ainda se aplica.

## Referências

- [Guia de Channels](channels.md) — setup dos channels oficiais (Telegram, Discord, iMessage)
- [Implementações funcionais](https://github.com/anthropics/claude-plugins-official/tree/main/external_plugins) — código completo com pairing, reply tools e file attachments
- [MCP](https://modelcontextprotocol.io) — protocolo base que channel servers implementam
