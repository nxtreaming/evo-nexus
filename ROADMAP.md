# OpenClaude Roadmap

> Unofficial toolkit for Claude Code — AI-powered business operating system.
>
> This roadmap is updated regularly. Want to vote or suggest? [Open a discussion](https://github.com/EvolutionAPI/open-claude/discussions) or [create an issue](https://github.com/EvolutionAPI/open-claude/issues).

---

## Legend

| Symbol | Meaning |
|--------|---------|
| `[ ]` | Not started |
| `[x]` | Done |
| `⚠️` | Breaking change |
| `🔥` | High priority |
| `💡` | Needs design discussion first |

---

## v0.4 — Foundation & Stability

> Fix, secure, and improve what already exists before growing.

### Skills

- [ ] 🔥 **Evolution product skills** — dedicated skills for Evolution API, Evolution Go, and Evo CRM (deploy, config, monitoring, troubleshooting, docs)
- [x] **Version indicator & update alerts** — show current version in dashboard sidebar, alert when new GitHub releases are available.

### Developer Experience

- [x] 🔥 **CLI installer** — `npx @evoapi/open-claude` — clones repo, installs deps, runs interactive setup wizard.
- [ ] **Full Docker install** — single `docker-compose up` that brings up dashboard, scheduler, and all services.
- [x] **Update checker** — dashboard checks GitHub releases and shows upgrade notification.

### Dashboard UX

- [ ] **Sidebar reorganization** — current sidebar is too long; group by domain, collapse sections, add search.
- [ ] **Active agent visualization** — use Claude Code hooks to display running agents in real-time (polished UI for demos).

---

## v0.5 — Extensibility & Ecosystem

> Make OpenClaude composable and self-extending.

### Agent System

- [ ] 🔥 **Complete existing agents** — ensure all 9 core agents have consistent memory, skills, and routines:
  - [ ] **Clawdia (Ops)** — generalize personal references, generic usage docs
  - [ ] **Flux (Finance)** — adapter pattern for ERPs beyond Omie
  - [ ] **Atlas (Projects)** — generic sprint templates (not hardcoded to Linear/GitHub)
  - [ ] **Pulse (Community)** — adapter for platforms beyond Discord/WhatsApp
  - [ ] **Pixel (Social)** — native scheduling, post approval workflow
  - [ ] **Sage (Strategy)** — scenario planning, board reporting, business canvas
  - [ ] **Nex (Sales)** — structured funnel, forecasting, automated follow-ups, CRM adapter
  - [ ] **Mentor (Courses)** — real platform, module generation, student tracking
  - [ ] **Kai (Personal)** — generalize (remove hardcoded personal data)
- [ ] 🔥 **New business agents** — expand functional coverage:
  - [ ] **Marketing Agent** — orchestrate existing `mkt-*` skills, attribution, budget, full funnel
  - [ ] **HR / People Agent** — onboarding, 1:1s, performance reviews, hiring pipeline
  - [ ] **Customer Success Agent** — health score, churn prediction, NPS/CSAT, client onboarding
  - [ ] **Legal / Compliance Agent** — contracts, renewals, GDPR/LGPD, compliance checklists
  - [ ] **Product Agent** — discovery, feature prioritization (RICE/ICE), PLG metrics, feedback loop
  - [ ] **Data / BI Agent** — cross-area consolidated dashboard, unified KPIs, alerts, trend analysis
- [ ] 💡 **Custom agents** — define spec and UX for user-created agents: naming, memory isolation, skill scope, onboarding
- [ ] 💡 **Help agent** — agent that answers questions about the workspace using its own documentation (RAG)

### Routines & Scheduling

- [ ] 🔥 **Trigger registry** — define and manage named triggers (webhook, cron, event-based) that invoke skills or routines
- [ ] **Non-recurrent scheduled actions** — one-off scheduled tasks (e.g., "post this on LinkedIn Friday at 10am") without creating a full routine
- [ ] **Agent-less routines** — allow purely systematic cron operations without agent context (lighter/faster)

### Integrations

- [ ] **Complete Obsidian integration** — finish `obs-*` skills: bidirectional sync, canvas, bases, CLI

### Import / Export

- [ ] **Backup system** — export workspace state as ZIP (agents, skills, routines, memory, config); import to restore. Support local, git, and cloud bucket targets.
- [ ] **Install via ZIP** — install skills, routines, and agents through the dashboard UI by uploading a ZIP. Includes malware and prompt injection scanning.

---

## v1.0 — Community & Growth

> Community adoption, discoverability, and self-sustaining ecosystem.

### Community & Docs

- [x] 🔥 **Public roadmap** — this file. Community input welcome via [discussions](https://github.com/EvolutionAPI/open-claude/discussions).
- [ ] **Telegram & Discord channels** — activate community channels, document in README and docs site.
- [ ] **In-app tutorials** — contextual tutorials surfaced inside the dashboard, not just external docs.
- [ ] **Resume Claude sessions in chat** — list active/resumable Claude sessions in dashboard chat with `--resume` support.

### Development

- [ ] **Testing framework** — define and implement test strategy for skills, routines, and agent behaviors; prevent regressions.

---

## Future / Research

> Items that need more design, exploration, or external dependencies.

- [ ] 💡 **Project naming** — evaluate whether "OpenClaude" remains the right brand as multi-LLM support grows. Candidates: Evo Agents OS, Evo Runtime, Evo Orchestrator.
- [ ] 💡 **Fully open source stack** — research open LLM models (Mistral, Llama, Qwen) and open agent runtimes (CrewAI, LangGraph) that could complement or replace Claude Code dependency.
- [ ] 💡 **DeepAgents compatibility** — investigate whether DeepAgents paradigm is compatible with OpenClaude's agent+skill architecture.
- [ ] 💡 **Ecosystem diagram** — canonical visual of how all components interact: agents, skills, routines, dashboard, MCPs, integrations.

---

## Contributing

Want to help? Pick any `[ ]` item and:

1. Check [open issues](https://github.com/EvolutionAPI/open-claude/issues)
2. Read [CONTRIBUTING.md](CONTRIBUTING.md)
3. For `💡` items, open a [discussion](https://github.com/EvolutionAPI/open-claude/discussions) first — design is still open

---

*Last updated: 2026-04-09 — [Evolution Foundation](https://evolutionfoundation.com.br)*
