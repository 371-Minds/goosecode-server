# AGENTS.md — Agent-to-Agent Handoff Guide

> **Goosecode Server — Heavy-Lifting Engineering Swarm**
>
> This document is the authoritative handoff reference for every AI agent (Goose, Automaton
> sub-agents, or any MCP-compatible tool) operating inside this environment.  Read it before
> taking any action.

---

## 1. Architecture Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                    Goosecode Swarm Platform                     │
│                                                                 │
│  ┌────────────┐   orchestrates   ┌──────────────────────────┐  │
│  │ Automaton  │ ───────────────► │  Goose Engineering Swarm │  │
│  │ (CTO node) │                  │  (parallel sub-agents)   │  │
│  └────────────┘                  └──────────┬───────────────┘  │
│                                             │ runs inside       │
│  ┌──────────────────────────────────────────▼───────────────┐  │
│  │              Agent Sandbox (microVM / container)          │  │
│  │                                                           │  │
│  │  shell-mcp ──► bash   ──► workspace files               │  │
│  │  goose-api ──► tmux   ──► VS Code Server (port 8080)     │  │
│  │                                                           │  │
│  └───────────────────────────────────────────────────────────┘  │
│                                                                 │
│  Data / Memory Layer (Biological Stack)                         │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────────────┐  │
│  │ClickHouse│ │  Redis   │ │ ChromaDB │ │     Ceramic      │  │
│  │ (Titan)  │ │ (Nerves) │ │(Instincts│ │   (Audit trail)  │  │
│  └──────────┘ └──────────┘ └──────────┘ └──────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
```

### Deployment targets
| Target | Distro | Primary use |
|--------|--------|-------------|
| Akash Network | `goose-akash` | Long-running batch jobs, DB hosting |
| Vercel Edge | `goose-vercel` | Next.js SSR + Supergateway routing |
| Mindport | `goose-mindport` | Windows-friendly Portless routing + DNS workflows |
| io.net / Render | `goosecode-server` (default) | General engineering tasks |
| Local dev | `docker-compose.yml` | Full stack with all services |

---

## 2. Goose Configuration

### Default model
```yaml
# ~/.config/goose/config.yaml
GOOSE_PROVIDER: openai
GOOSE_MODEL: o3-mini-2025-01-31
```
Override with environment variables: `GOOSE_PROVIDER`, `GOOSE_MODEL`.

### Session IDs
| Variable | Purpose |
|----------|---------|
| `GOOSE_SESSION_ID` | Named session (must be filesystem-safe, no spaces) |
| `GOOSE_RESUME_SESSION` | `true` to resume; `false` (default) starts fresh |

### Session logs
Stored at `/home/coder/.local/share/goose/sessions/` as JSONL files.  Each line is a
`LogEntry` with `role`, `content`, and optional `tool_use`/`tool_result` blocks.

---

## 3. Agent Entry Points

### 3.1 Goose Terminal API (REST + SSE)
Base URL: `http://localhost:8000`  
Auth header: `X-API-Key: <PASSWORD>`  
Swagger docs: `http://localhost:8000/docs`

| Method | Path | Description |
|--------|------|-------------|
| POST | `/api/terminal/send` | Send a raw tmux keystroke / command |
| GET | `/api/terminal/sessions` | List active tmux sessions |
| GET | `/api/sessions` | List Goose session log files |
| GET | `/api/sessions/{id}` | Retrieve full session log |
| GET | `/api/sessions/latest/id` | ID of the most recent session |
| POST | `/api/stream` | SSE stream — send prompt and stream reply |

#### SSE event sequence
```
command_sent → session_identified → initial_state → update … → conversation_complete
```
`update` events carry new assistant text as `data.new_content`.

### 3.2 shell-mcp (Sandboxed Shell)
Config: `mcp/shell-mcp.json`

The shell-mcp server ([supercorp-ai/shell-mcp](https://github.com/supercorp-ai/shell-mcp))
exposes MCP tools that execute shell commands **inside the sandbox** — meaning destructive
commands like `rm -rf /` only destroy the ephemeral microVM, not the host.

Available MCP tools exposed:
- `shell_execute` — run a single bash command
- `shell_exec_background` — run command in background, return PID
- `shell_read_file` / `shell_write_file` — file I/O helpers
- `shell_list_dir` — directory listing

Connect via stdio or HTTP MCP transport (see `mcp/shell-mcp.json`).

### 3.3 VS Code Server
URL: `http://localhost:8080`  
Password: `$PASSWORD` (default: `talktomegoose`)

---

## 4. Custom Distros

### goose-akash
`distros/goose-akash/`

Adds on top of the base image:
- Akash CLI (`akash`) — deploy SDL templates
- ipfs CLI — for decentralized storage
- sqlite3 — for local agent state
- Pre-loaded SDL templates at `/opt/akash/sdls/`
- System prompt focused on decentralized cloud engineering

Build: `docker build -t goose-akash -f distros/goose-akash/Dockerfile .`

### goose-vercel
`distros/goose-vercel/`

Adds on top of the base image:
- Node.js LTS + pnpm + Vercel CLI
- Expo CLI + EAS CLI — React Native builds
- Next.js scaffolding helpers
- System prompt focused on full-stack web + mobile

Build: `docker build -t goose-vercel -f distros/goose-vercel/Dockerfile .`

### goose-mindport
`distros/goose-mindport/`

Adds on top of the base image:
- Node.js LTS
- Global `portless` CLI exposed as `mindport`
- Shallow checkout of `vercel-labs/portless` under `/opt/integrations/mindport`
- System prompt focused on Windows-friendly local routing and DNS-provider workflows

Build: `docker build -t goose-mindport -f distros/goose-mindport/Dockerfile .`

### AI runtime distros

The following distros extend the base image with shallow runtime checkouts under
`/opt/integrations/` and workflow-specific system prompts:

- `goose-picollm` → `Picovoice/picollm`
- `goose-mii` → `deepspeedai/DeepSpeed-MII`
- `goose-mnn` → `alibaba/MNN`
- `goose-ryzenai` → `k-rks/RyzenAI-SW`
- `goose-exllamav3` → `turboderp-org/exllamav3`
- `goose-envminds` → `371-Minds/envminds`

Build example: `docker build -t goose-picollm -f distros/goose-picollm/Dockerfile .`

---

## 5. Data & Memory Layer (Biological Stack)

Each component maps to a biological memory analogy:

| Service | Analogy | Port | Use case |
|---------|---------|------|---------|
| ClickHouse | Titan (long-term memory) | 8123 / 9000 | Analytics, event logs, agent traces |
| SQLite | Working memory | (file) | Per-agent relational state |
| Redis | Nerves (fast signals) | 6379 | Pub/sub, ephemeral cache, agent comms |
| ChromaDB | Instincts (vector memory) | 8001 | Embeddings, semantic search |
| Ceramic | Audit trail (immutable) | 7007 | Verifiable, decentralized data |

Akash SDL deployment templates: `akash/sdls/`

Local development: start all services with `docker compose up`.

### Connecting from Goose / agents

Use the environment variables injected by `docker-compose.yml` or `.env`:
```bash
CLICKHOUSE_URL=http://clickhouse:8123
REDIS_URL=redis://redis:6379
CHROMADB_URL=http://chromadb:8001
SQLITE_PATH=/workspace/.db/agent_state.sqlite
CERAMIC_URL=http://ceramic:7007
```

---

## 6. Orchestration via Automaton

[371-Minds/automaton](https://github.com/371-Minds/automaton) acts as the CTO orchestrator.
It dispatches engineering sub-agents to this swarm via the Goose Terminal API.

### Typical workflow
```
Automaton CTO
  └─► POST /api/stream { "command": "implement feature X", "session_id": "feature-x" }
        └─► Goose sub-agent (inside microVM sandbox)
              ├─► shell-mcp: edit files, run tests, commit
              ├─► writes logs → ClickHouse (via CLICKHOUSE_URL)
              ├─► caches context → Redis (REDIS_URL)
              └─► stores embeddings → ChromaDB (CHROMADB_URL)
```

### Session namespacing
Use `GOOSE_SESSION_ID` with the Automaton task ID so that multiple sub-agents can run in
parallel without colliding:
```bash
GOOSE_SESSION_ID=automaton-task-<TASK_ID>
```

---

## 7. Expo + Vercel Deployment Pipeline

The `goose-vercel` distro manages the full frontend deployment loop:

```
Write once (Expo / Next.js)
  ├─► Web: Vercel Edge (vercel deploy --prod)
  ├─► iOS:  Expo EAS (eas build --platform ios)
  └─► Android: Expo EAS (eas build --platform android)
```

### Key environment variables
```bash
VERCEL_TOKEN=         # Vercel deploy token
EXPO_TOKEN=           # Expo EAS token
EAS_PROJECT_ID=       # Expo project ID
```

---

## 8. CI/CD Pipeline

`.github/workflows/ci.yml` automates:
1. **Build** — Docker image for each distro
2. **Lint** — shellcheck on all `.sh` files
3. **Integration test** — spin up container, call `/api/stream`, verify response
4. **Push** — tagged images to GHCR on main branch

---

## 9. Security Model

| Risk | Mitigation |
|------|-----------|
| Destructive agent command (e.g., `rm -rf /`) | Runs inside disposable microVM — host is unaffected |
| Secret leakage | Secrets injected via env vars only; never written to workspace files |
| Unauthenticated API access | All endpoints require `X-API-Key` header |
| Port exposure | By default, ports 8080 and 8000 bind to `127.0.0.1` only in production configs |
| Image supply chain | Base images pinned; Dependabot alerts enabled |

---

## 10. Handoff Checklist (Agent → Agent)

When passing a task to a new agent instance:

- [ ] Record the current `GOOSE_SESSION_ID` — pass it as `session_id` to resume context.
- [ ] Snapshot key agent state to SQLite (`SQLITE_PATH`) before handing off.
- [ ] Push any in-progress embeddings to ChromaDB so the next agent can semantic-search context.
- [ ] Write a Ceramic record with the task summary and outcome hash.
- [ ] Set `GOOSE_RESUME_SESSION=true` in the next agent's environment.
- [ ] Include relevant ClickHouse log entries (query `agent_logs` table) in the handoff message.
- [ ] Verify no tmux sessions are orphaned: `GET /api/terminal/sessions`.

---

## 11. File Map (Quick Reference)

```
goosecode-server/
├── Dockerfile                  # Base goosecode-server image
├── entrypoint.sh               # Container startup — Goose + API + VS Code
├── run.sh                      # Local Docker orchestration wrapper
├── github-setup.sh             # Git / GitHub CLI configuration
├── install-goose.sh            # Goose binary installation
├── update-goose-api.sh         # Update Goose API in running container
├── .env.example                # All supported environment variables
├── AGENTS.md                   # ← YOU ARE HERE
│
├── distros/
│   ├── goose-akash/
│   │   ├── Dockerfile          # Akash-specific image
│   │   └── system-prompt.md    # Akash engineering system prompt
│   ├── goose-envminds/
│   │   ├── Dockerfile          # envminds integration image
│   │   └── system-prompt.md    # envminds workflow system prompt
│   ├── goose-exllamav3/
│   │   ├── Dockerfile          # exllamav3 integration image
│   │   └── system-prompt.md    # exllamav3 workflow system prompt
│   ├── goose-mindport/
│   │   ├── Dockerfile          # Mindport / Portless integration image
│   │   └── system-prompt.md    # Mindport workflow system prompt
│   ├── goose-mii/
│   │   ├── Dockerfile          # DeepSpeed-MII integration image
│   │   └── system-prompt.md    # MII workflow system prompt
│   ├── goose-mnn/
│   │   ├── Dockerfile          # MNN integration image
│   │   └── system-prompt.md    # MNN workflow system prompt
│   ├── goose-picollm/
│   │   ├── Dockerfile          # picoLLM integration image
│   │   └── system-prompt.md    # picoLLM workflow system prompt
│   ├── goose-ryzenai/
│   │   ├── Dockerfile          # RyzenAI-SW integration image
│   │   └── system-prompt.md    # RyzenAI workflow system prompt
│   └── goose-vercel/
│       ├── Dockerfile          # Vercel/Expo-specific image
│       └── system-prompt.md    # Full-stack system prompt
│
├── akash/
│   └── sdls/
│       ├── clickhouse.yaml     # ClickHouse Akash SDL
│       ├── redis.yaml          # Redis Akash SDL
│       ├── chromadb.yaml       # ChromaDB Akash SDL
│       └── ceramic.yaml        # Ceramic Akash SDL
│
├── mcp/
│   └── shell-mcp.json          # shell-mcp MCP server config
│
├── docker-compose.yml          # Full local stack
│
├── goose-api/
│   ├── main.py                 # FastAPI REST + SSE server
│   ├── requirements.txt        # Python dependencies
│   └── examples/               # Client example scripts
│
└── .github/
    └── workflows/
        └── ci.yml              # CI/CD pipeline
```

---

*Last updated by Goosecode Swarm bootstrap agent — update this file whenever the architecture changes.*
