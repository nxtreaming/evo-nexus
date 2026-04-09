# Channels — Comunicação Bidirecional com Claude Code

Channels permitem enviar mensagens para uma sessão Claude Code em execução e receber respostas de volta — como uma ponte de chat entre plataformas externas e o seu terminal.

> **Channels vs Integrações:** Integrações (API clients, MCP servers) são ferramentas que Claude chama sob demanda durante uma tarefa. Channels são o inverso: sistemas externos que **empurram** eventos para dentro da sessão. Você pode ter ambos — por exemplo, Discord como integração (community monitoring via @pulse) E como channel (chat bidirecional com Claude).

## Channels Disponíveis

| Channel | Plataforma | Tipo | Make Command |
|---------|-----------|------|-------------|
| **Telegram** | Mobile/Desktop | Bot (polling) | `make telegram` |
| **Discord** | Desktop/Mobile | Bot (gateway) | `make discord-channel` |
| **iMessage** | macOS only | Nativo (Messages.app) | `make imessage` |

## Pré-requisitos

- **Claude Code v2.1.80+** (verificar com `claude --version`)
- **Bun** instalado (verificar com `bun --version`, instalar em [bun.sh](https://bun.sh))
- **Login via claude.ai** (Console e API key não suportam channels)
- **Plugin marketplace** configurado: `/plugin marketplace add anthropics/claude-plugins-official`

## Setup: Telegram

O Telegram já está configurado neste workspace. Documentação completa em [docs/integrations/telegram.md](../integrations/telegram.md).

### 1. Criar um Bot

1. Abra o Telegram e envie mensagem para [@BotFather](https://t.me/BotFather)
2. Envie `/newbot` e siga as instruções
3. Copie o **Bot Token** fornecido

### 2. Instalar o Plugin

No Claude Code:

```
/plugin install telegram@claude-plugins-official
```

Se o plugin não for encontrado, atualize o marketplace:

```
/plugin marketplace update claude-plugins-official
```

Depois recarregue:

```
/reload-plugins
```

### 3. Configurar o Token

```
/telegram:configure <seu_token>
```

Isso salva em `~/.claude/channels/telegram/.env`. Alternativamente, defina `TELEGRAM_BOT_TOKEN` no ambiente do shell.

### 4. Iniciar o Channel

**Manual (foreground):**

```bash
claude --channels plugin:telegram@claude-plugins-official
```

**Background (recomendado):**

```bash
make telegram
```

Isso inicia o bot em uma sessão `screen` em background com `--dangerously-skip-permissions`.

### 5. Parear sua Conta

1. No Telegram, envie qualquer mensagem para o seu bot
2. O bot responde com um **código de pareamento**
3. No Claude Code, execute:

```
/telegram:access pair <código>
```

4. Restrinja o acesso apenas à sua conta:

```
/telegram:access policy allowlist
```

### Gerenciar

| Comando | O que faz |
|---------|----------|
| `make telegram` | Iniciar bot em background |
| `make telegram-stop` | Parar o bot |
| `make telegram-attach` | Conectar ao terminal (Ctrl+A D para desconectar) |

---

## Setup: Discord Channel

> **Nota:** Este é o Discord **Channel** (chat bidirecional com Claude Code), diferente da integração Discord API usada pelo @pulse para community monitoring. São independentes e podem coexistir.

### 1. Criar um Bot Discord

1. Acesse o [Discord Developer Portal](https://discord.com/developers/applications)
2. Clique em **New Application** e dê um nome
3. Na seção **Bot**, clique em **Reset Token** e copie o token

### 2. Habilitar Message Content Intent

Nas configurações do bot, em **Privileged Gateway Intents**, habilite:

- **Message Content Intent** (obrigatório)

### 3. Convidar o Bot para o Servidor

Vá em **OAuth2 > URL Generator**. Selecione o scope `bot` e habilite estas permissões:

- View Channels
- Send Messages
- Send Messages in Threads
- Read Message History
- Attach Files
- Add Reactions

Abra a URL gerada para adicionar o bot ao seu servidor.

### 4. Instalar o Plugin

No Claude Code:

```
/plugin install discord@claude-plugins-official
```

Se não encontrar, atualize o marketplace:

```
/plugin marketplace update claude-plugins-official
```

Depois recarregue:

```
/reload-plugins
```

### 5. Configurar o Token

```
/discord:configure <seu_token>
```

Salva em `~/.claude/channels/discord/.env`. Alternativamente, defina `DISCORD_BOT_TOKEN` no ambiente.

### 6. Iniciar o Channel

**Manual (foreground):**

```bash
claude --channels plugin:discord@claude-plugins-official
```

**Background (recomendado):**

```bash
make discord-channel
```

### 7. Parear sua Conta

1. Envie uma DM para o seu bot no Discord
2. O bot responde com um **código de pareamento**
3. No Claude Code:

```
/discord:access pair <código>
```

4. Restrinja o acesso:

```
/discord:access policy allowlist
```

### Gerenciar

| Comando | O que faz |
|---------|----------|
| `make discord-channel` | Iniciar channel em background |
| `make discord-channel-stop` | Parar o channel |
| `make discord-channel-attach` | Conectar ao terminal (Ctrl+A D para desconectar) |

---

## Setup: iMessage

> **Requisito:** macOS apenas. Lê o banco de dados do Messages.app diretamente e envia respostas via AppleScript. Não precisa de bot token nem serviço externo.

### 1. Conceder Full Disk Access

O banco do Messages (`~/Library/Messages/chat.db`) é protegido pelo macOS. Na primeira leitura, o macOS solicita permissão — clique em **Allow**.

Se o prompt não aparecer ou você clicou em "Don't Allow", conceda manualmente:

**System Settings > Privacy & Security > Full Disk Access** → adicione seu terminal (Terminal, iTerm, etc.)

### 2. Instalar o Plugin

No Claude Code:

```
/plugin install imessage@claude-plugins-official
```

Se não encontrar:

```
/plugin marketplace update claude-plugins-official
```

### 3. Iniciar o Channel

**Manual (foreground):**

```bash
claude --channels plugin:imessage@claude-plugins-official
```

**Background (recomendado):**

```bash
make imessage
```

### 4. Testar — Envie uma Mensagem para Si Mesmo

Abra o Messages em qualquer dispositivo com seu Apple ID e envie uma mensagem **para si mesmo**. Self-chat funciona automaticamente sem configuração de acesso.

> Na primeira resposta do Claude, o macOS pede permissão de Automação para o terminal controlar o Messages. Clique **OK**.

### 5. Permitir Outros Remetentes

Por padrão, apenas suas próprias mensagens passam. Para permitir outro contato:

```
/imessage:access allow +5531999999999
```

Handles aceitos: números com código do país (`+55...`) ou emails Apple ID.

### Gerenciar

| Comando | O que faz |
|---------|----------|
| `make imessage` | Iniciar channel em background |
| `make imessage-stop` | Parar o channel |
| `make imessage-attach` | Conectar ao terminal (Ctrl+A D para desconectar) |

---

## Múltiplos Channels

Você pode rodar vários channels simultaneamente. Cada um roda em sua própria sessão `screen`:

```bash
make telegram
make discord-channel
make imessage
```

Ou em uma única sessão Claude Code (foreground):

```bash
claude --channels plugin:telegram@claude-plugins-official plugin:discord@claude-plugins-official plugin:imessage@claude-plugins-official
```

## Segurança

- Cada channel mantém uma **allowlist de remetentes** — apenas IDs aprovados podem enviar mensagens
- Telegram e Discord usam **pairing**: o remetente envia uma mensagem, recebe um código, e você aprova no Claude Code
- iMessage permite self-chat automaticamente; outros contatos são adicionados por handle
- **Permission relay**: channels bidirecionais podem encaminhar prompts de permissão para que você aprove/negue remotamente (ex: aprovar um `Bash` command pelo Telegram)
- Estar no `.mcp.json` não é suficiente — o server também precisa ser nomeado em `--channels`

## Enterprise / Team

Em planos Team e Enterprise, channels são **desabilitados por padrão**. O admin controla via:

| Setting | Função |
|---------|--------|
| `channelsEnabled` | Switch master. Deve ser `true` para qualquer channel funcionar |
| `allowedChannelPlugins` | Quais plugins podem registrar como channel (substitui a lista padrão da Anthropic) |

Configurar em: **claude.ai > Admin settings > Claude Code > Channels**

## Channels Customizados

Você pode construir seus próprios channels para sistemas que não têm plugin ainda (CI/CD, monitoring, webhooks). Veja o [Channels Reference](channels-reference.md) para a documentação técnica completa.

## Troubleshooting

**Bot não responde no Telegram/Discord:**
- Verifique se o Claude Code está rodando com `--channels` (ou via `make telegram` / `make discord-channel`)
- Use `make telegram-attach` ou `make discord-channel-attach` para ver o terminal

**"Plugin not found":**
- Execute `/plugin marketplace update claude-plugins-official`
- Depois `/reload-plugins`

**"Blocked by org policy":**
- Admin precisa habilitar `channelsEnabled` nas managed settings

**iMessage — "authorization denied":**
- Conceda Full Disk Access ao terminal em System Settings > Privacy & Security

**Mensagens não chegam:**
- Confirme que fez o pairing (`/telegram:access pair`, `/discord:access pair`)
- Confirme que a policy é allowlist (`/telegram:access policy allowlist`)

**Permission prompts travam a sessão:**
- Channels com permission relay encaminham os prompts. Responda pelo chat ou no terminal local
- Para uso autônomo: `--dangerously-skip-permissions` (apenas em ambientes confiáveis)
