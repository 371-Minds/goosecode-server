# Goosecode Server — Heavy-Lifting Engineering Swarm

A containerized VS Code server environment with integrated [Goose AI coding agent](https://github.com/block/goose), purpose-built as the **heavy-lifting engineering swarm** for solo-developer and multi-agent teams.  Goose handles the actual file editing, terminal commands, and debugging for your Expo/Next.js apps — all executing safely inside an isolated microVM so that a destructive hallucination only destroys a disposable container, not your host machine.

<div align="center">
  <img src="./static/img/logo.png" alt="Goose AI + VS Code Server" width="400">
</div>

[![License: Apache 2.0](https://img.shields.io/badge/License-Apache_2.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)
[![Docker](https://img.shields.io/badge/Docker-Ready-2496ED?logo=docker)](https://www.docker.com/)
[![VS Code](https://img.shields.io/badge/VS_Code-Server-007ACC?logo=visualstudiocode)](https://code.visualstudio.com/)
[![OpenAI](https://img.shields.io/badge/Powered_by-OpenAI-412991?logo=openai)](https://openai.com)
[![CI](https://github.com/371-Minds/goosecode-server/actions/workflows/ci.yml/badge.svg)](https://github.com/371-Minds/goosecode-server/actions/workflows/ci.yml)

## Features

- **Browser-Based Development**: Access VS Code directly from your browser
- **Goose AI Agent**: Pre-installed and configured [Goose AI Coding agent](https://github.com/block/goose)
- **Shared Terminal Session**: The same Goose session is visible in all browser windows
- **Goose Terminal API**: REST API for sending commands to the terminal and retrieving session logs
- **Streaming Conversations**: Real-time streaming of Goose AI conversations using Server-Sent Events (SSE)
- **Material Design**: Dark theme with Material icons for a beautiful coding experience
- **Secure MicroVM Sandbox**: [shell-mcp](https://github.com/supercorp-ai/shell-mcp) provides sandboxed shell execution — destructive commands only affect the disposable container
- **Secure Environment**: Password-protected VS Code Server instance
- **Git Integration**: Git pre-installed and ready for repository operations
- **Persistent Configuration**: Environment variables and configuration preserved between sessions (Unless workspace is deleted)
- **Custom Distros**: Deployment- and runtime-specific `goose-*` images for Akash, Vercel, Mindport, picoLLM, MII, MNN, RyzenAI, exllamav3, and envminds
- **Biological Data Stack**: ClickHouse, Redis, ChromaDB, Ceramic — one `docker compose up` away
- **Automaton Orchestration**: Integrates with [371-Minds/automaton](https://github.com/371-Minds/automaton) for automated CI/CD pipelines
- **Universal Frontend Edge**: Build once for Web (Next.js + Vercel), iOS, and Android (Expo EAS)

## Architecture

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
│  │  shell-mcp ──► bash ──► workspace files                  │  │
│  │  goose-api ──► tmux ──► VS Code Server (:8080)           │  │
│  └───────────────────────────────────────────────────────────┘  │
│                                                                 │
│  Data / Memory Layer (Biological Stack)                         │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────────────┐  │
│  │ClickHouse│ │  Redis   │ │ ChromaDB │ │     Ceramic      │  │
│  │ (Titan)  │ │ (Nerves) │ │(Instincts│ │  (Audit trail)   │  │
│  └──────────┘ └──────────┘ └──────────┘ └──────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
```

## Quick Start

### Option A — Full swarm stack (recommended)

Starts VS Code + Goose AI + all Biological Stack databases with a single command.

1. **Clone and configure**
   ```bash
   git clone https://github.com/371-Minds/goosecode-server.git
   cd goosecode-server
   cp .env.example .env
   # Edit .env — at minimum set OPENAI_API_KEY
   ```

2. **Start the full stack**
   ```bash
   docker compose up
   ```

3. **Access**
   - VS Code Server: http://localhost:8080 (password from `.env`, default: `talktomegoose`)
   - Goose API + Swagger: http://localhost:8000/docs

### Option B — Goose only (lightweight)

```bash
cp .env.example .env
chmod +x run.sh
./run.sh
```

<div align="center">
  <img src="./static/img/screenshot.png" alt="VS Code Server Screenshot" width="600">
  <p><i>Example of the VS Code interface in browser</i></p>
</div>

## Custom Distros

Build a Goose distribution pre-packed with the exact CLIs and system prompts for your target platform.

### goose-akash — Akash Network deployments

Adds the Akash CLI, IPFS CLI, SQLite, and pre-loaded SDL templates.  System prompt is tuned for decentralised cloud engineering.

```bash
# Build base first
docker build -t goosecode-server:latest .

# Build distro
docker build -t goose-akash \
  --build-arg BASE_IMAGE=goosecode-server:latest \
  -f distros/goose-akash/Dockerfile \
  .
```

Extra environment variables:
| Variable | Description |
|----------|-------------|
| `AKASH_KEY_NAME` | Wallet key name |
| `AKASH_NODE` | Akash RPC endpoint |
| `AKASH_CHAIN_ID` | Chain ID (default: `akashnet-2`) |

### goose-vercel — Vercel Edge + Expo deployments

Adds Node.js LTS, pnpm, Vercel CLI, Expo CLI, and EAS CLI.  System prompt is tuned for full-stack Web + Mobile engineering.

```bash
docker build -t goose-vercel \
  --build-arg BASE_IMAGE=goosecode-server:latest \
  -f distros/goose-vercel/Dockerfile \
  .
```

Extra environment variables:
| Variable | Description |
|----------|-------------|
| `VERCEL_TOKEN` | Vercel deploy token |
| `EXPO_TOKEN` | Expo EAS token |
| `EAS_PROJECT_ID` | Expo project ID |
| `NEXT_APP_DIR` | Next.js app path in workspace (default: `web`) |
| `EXPO_APP_DIR` | Expo app path in workspace (default: `mobile`) |

### goose-mindport — Windows-friendly local routing with Mindport

Packages the upstream [`vercel-labs/portless`](https://github.com/vercel-labs/portless) CLI for
local routing and exposes it as both `portless` and `mindport`. The distro also keeps a shallow
checkout at `/opt/integrations/mindport` so agents can inspect upstream behavior while they work.

```bash
docker build -t goose-mindport \
  --build-arg BASE_IMAGE=goosecode-server:latest \
  -f distros/goose-mindport/Dockerfile \
  .
```

Extra environment variables:
| Variable | Description |
|----------|-------------|
| `MINDPORT_TLD` | Preferred development TLD (default: `localhost`) |
| `MINDPORT_PROXY_PORT` | Proxy port used by `mindport` / `portless` (default: `1355`) |
| `MINDPORT_DNS_PROVIDER` | Preferred DNS provider identifier |
| `OPENPROVIDER_USERNAME` / `OPENPROVIDER_PASSWORD` | Openprovider DNS credentials |
| `PORKBUN_API_KEY` / `PORKBUN_SECRET_KEY` | Porkbun DNS credentials |
| `NAMECHEAP_API_USER` / `NAMECHEAP_API_KEY` / `NAMECHEAP_USERNAME` / `NAMECHEAP_CLIENT_IP` | Namecheap DNS credentials |
| `FREENAME_API_KEY` / `FREENAME_API_SECRET` | Freename DNS credentials |

The Goose API exposes the packaged Mindport metadata at `/api/mindport` and lists supported DNS
integrations at `/api/dns/providers` without returning any stored secrets.

### AI runtime distros

These distros extend the base Goosecode Server image with shallow checkouts of the requested
runtime projects under `/opt/integrations`, plus system prompts tuned for each workflow.

| Distro | Integrated project | Runtime checkout path |
|--------|--------------------|-----------------------|
| `goose-picollm` | [Picovoice/picollm](https://github.com/Picovoice/picollm) | `/opt/integrations/picollm` |
| `goose-mii` | [deepspeedai/DeepSpeed-MII](https://github.com/deepspeedai/DeepSpeed-MII) | `/opt/integrations/mii` |
| `goose-mnn` | [alibaba/MNN](https://github.com/alibaba/MNN) | `/opt/integrations/mnn` |
| `goose-ryzenai` | [k-rks/RyzenAI-SW](https://github.com/k-rks/RyzenAI-SW) | `/opt/integrations/ryzenai` |
| `goose-exllamav3` | [turboderp-org/exllamav3](https://github.com/turboderp-org/exllamav3) | `/opt/integrations/exllamav3` |
| `goose-envminds` | [371-Minds/envminds](https://github.com/371-Minds/envminds) | `/opt/integrations/envminds` |

Example build commands:

```bash
docker build -t goose-picollm \
  --build-arg BASE_IMAGE=goosecode-server:latest \
  -f distros/goose-picollm/Dockerfile \
  .

docker build -t goose-mii \
  --build-arg BASE_IMAGE=goosecode-server:latest \
  -f distros/goose-mii/Dockerfile \
  .
```

## Biological Data Stack

Deploy a specialised, multi-tiered data stack that mimics biological memory systems.

| Service | Analogy | Port | Use case |
|---------|---------|------|---------|
| **ClickHouse** | Titan (long-term memory) | 8123 | Analytics, event logs, agent traces |
| **Redis** | Nerves (fast signals) | 6379 | Pub/sub, ephemeral cache, agent comms |
| **ChromaDB** | Instincts (vector memory) | 8001 | Embeddings, semantic search |
| **Ceramic** | Audit trail (immutable) | 7007 | Verifiable, decentralised data |
| **SQLite** | Working memory | (file) | Per-agent relational state |

All services start with `docker compose up`.  Akash SDL deployment templates for each service are in `akash/sdls/`.

```bash
# Deploy ClickHouse to Akash
akash tx deployment create akash/sdls/clickhouse.yaml --from $AKASH_KEY_NAME
```

## Universal Frontend Edge (Expo + Vercel + Automaton)

Build once, deploy everywhere.  Use the `goose-vercel` distro for the full pipeline:

```
Write once (Expo / Next.js)
  ├─► Web: Vercel Edge     vercel deploy --prod
  ├─► iOS:  Expo EAS       eas build --platform ios
  └─► Android: Expo EAS   eas build --platform android
```

## Automaton Orchestration

[371-Minds/automaton](https://github.com/371-Minds/automaton) acts as the CTO node, dispatching engineering sub-agents to this swarm via the Goose Terminal API.

```bash
# Send a task to Goose via the API
curl -X POST http://localhost:8000/api/stream \
  -H "X-API-Key: $PASSWORD" \
  -H "Content-Type: application/json" \
  -d '{"command": "implement feature X", "session_id": "automaton-task-123"}'
```

Sub-agents use `GOOSE_SESSION_ID=automaton-task-<TASK_ID>` for parallel execution without session collisions.

See [AGENTS.md](AGENTS.md) for the full agent-to-agent handoff protocol.

## Secure MicroVM Sandbox (shell-mcp)

[shell-mcp](https://github.com/supercorp-ai/shell-mcp) is configured at `mcp/shell-mcp.json` and exposes MCP tools that execute shell commands **inside the sandbox**.  If an agent hallucinates a destructive command (`rm -rf /`), it only destroys the disposable VM — not your host machine.

To connect Goose to shell-mcp, add it to your Goose MCP configuration:

```yaml
# ~/.config/goose/config.yaml
extensions:
  shell:
    type: stdio
    cmd: npx
    args: ["-y", "@supercorp-ai/shell-mcp"]
    env:
      ALLOWED_COMMANDS: "bash,sh,python3,node,git"
      WORKING_DIR: /workspace
      SANDBOX_MODE: "true"
```

## Using the run.sh Script

The `run.sh` script provides a convenient way to manage your Goosecode Server container. It handles building the image, starting/stopping the container, and passing environment variables.

### Basic Usage

```bash
./run.sh
```

### Advanced Options

You can pass environment variables and configuration options directly to the script:

```bash
./run.sh --openai-key=your_api_key_here --password=your_password --port=8888
```

#### Available Options

| Option | Description | Default |
|--------|-------------|---------|
| `--rebuild` | Force rebuild of the Docker image | - |
| `--port=VALUE` | Host port to map to container | 8080 |
| `--image=VALUE` | Custom Docker image name | goosecode-server |
| `--container=VALUE` | Custom container name | goosecode-server |
| `--openai-key=VALUE` | OpenAI API key | From .env |
| `--password=VALUE` | VS Code Server password | From .env or "talktomegoose" |
| `--github-token=VALUE` | GitHub token | From .env |
| `--git-user=VALUE` | Git user name | From .env or "PlatOps AI" |
| `--git-email=VALUE` | Git user email | From .env or "hello@platops.ai" |
| `--no-terminal-sharing` | Disable shared terminal feature | Sharing enabled by default |
| `--no-goose-api` | Disable the Goose Terminal API | API enabled by default |
| `--api-port=VALUE` | Custom port for the Goose API | 8000 |
| `--goose-session=VALUE` | Specify a Goose session ID | Random session ID |
| `--resume-session` | Resume an existing Goose session | Start new session |

### Environment Variables Priority

1. Command-line arguments (highest priority)
2. Variables from `.env` file
3. Default values (lowest priority)

## Using Goose AI Assistant

Goosecode Server automatically starts a shared Goose AI terminal session when you launch the container. This means:

- The same Goose session is visible in all browser windows
- Multiple users can see and interact with the same conversation
- The session persists even when browser windows are closed

### Accessing the Shared Goose Session

When you open VS Code in your browser:

1. A terminal with Goose AI should open automatically
2. If not, open a terminal in VS Code and run one of these commands:
   ```bash
   # Full interactive mode (default)
   ~/shared-goose.sh
   
   # View-only mode (can't type, just watch)
   ~/goose-view.sh
   ```

3. Each new terminal creates a unique session linked to the shared content
4. Start interacting with Goose by typing your questions or instructions

### Using Environment Variables for Session IDs

You can specify a custom session ID using environment variables:

1. Set the `GOOSE_SESSION_ID` environment variable in your `.env` file or via the `--goose-session` flag:
   ```bash
   # In .env file
   GOOSE_SESSION_ID=my-project-session
   
   # Or when running the container
   ./run.sh --goose-session=my-project-session
   ```

2. To resume an existing session, set the `GOOSE_RESUME_SESSION` environment variable:
   ```bash
   # In .env file
   GOOSE_SESSION_ID=my-project-session
   GOOSE_RESUME_SESSION=true
   
   # Or when running the container
   ./run.sh --goose-session=my-project-session --resume-session
   ```

These variables allow you to maintain consistent session IDs across container restarts or share specific sessions with team members.

### Goose Configuration

Goose is pre-configured with the following settings:

```yaml
GOOSE_PROVIDER: openai
extensions:
  developer:
    enabled: true
    name: developer
    type: builtin
GOOSE_MODE: auto
GOOSE_MODEL: o3-mini-2025-01-31
OPENAI_BASE_PATH: v1/chat/completions
OPENAI_HOST: https://api.openai.com
```

To view your current configuration:
```bash
cat ~/.config/goose/config.yaml
```

To modify your configuration:
```bash
goose configure
```

## UI Customization

The VS Code Server instance comes pre-configured with:

- **Dark Theme**: Easy on the eyes for long coding sessions
- **Material Icon Theme**: Beautiful file and folder icons
- **Material Product Icons**: Enhanced VS Code UI icons
- **Custom Colors**: Optimized color scheme for code readability

## Docker Container Management

### Building the Container
```bash
docker build -t goosecode-server .
```

### Running the Container Manually
```bash
docker run -d -p 8080:8080 -p 8000:8000 --name goosecode-server --env-file .env goosecode-server
```

### Managing the Container
```bash
# Stop the container
docker stop goosecode-server

# Start an existing container
docker start goosecode-server

# Remove the container
docker rm goosecode-server

# View container logs
docker logs goosecode-server

# Access container shell
docker exec -it goosecode-server bash
```

### Customizing Port or Password
```bash
docker run -d -p 8888:8080 -e PASSWORD="your-secure-password" --name goosecode-server --env-file .env goosecode-server
```

## Goose Terminal API

The Goosecode Server includes an HTTP API for interacting with the terminal and accessing Goose conversation logs.

### API Features:

- Send commands to the tmux terminal
- List active tmux sessions
- Retrieve Goose conversation logs
- Stream conversation updates in real-time using Server-Sent Events

The API is enabled by default when starting the container and runs on port 8000.

- Swagger documentation: http://localhost:8000/docs
- API base URL: http://localhost:8000/api/

### Command-line options:

```bash
# Disable the Goose API
./run.sh --no-goose-api

# Change the API port
./run.sh --api-port=9000
```

For more details about the API, including examples and the streaming client, see the [Goose API README](goose-api/README.md).

## Troubleshooting

### Goose AI Issues

| Issue | Solution |
|-------|----------|
| Goose not found | Ensure the installation was successful with `which goose` |
| Configuration errors | Run `goose configure` to set up the agent manually |
| API key issues | Verify your OpenAI API key is correctly set in the `.env` file or passed via command line |
| Shared session not working | Run `~/shared-goose.sh` to connect to the shared session |
| Scrolling affecting other clients | Each window should have its own session; check the status bar for your session name |
| Need view-only access | Run `~/goose-view.sh` for read-only mode |

### Container Issues

| Issue | Solution |
|-------|----------|
| Port conflicts | Change the port mapping using `--port=VALUE` option |
| Permission issues | Container uses the `coder` user; use `sudo` for privileged operations |
| Performance issues | Adjust Docker resource allocation in Docker Desktop settings |
| Environment variables not working | Check priority order: command line > .env file > defaults |
| API not accessible | Ensure port 8000 is published with `-p 8000:8000` and the API is enabled |

---

Built with ❤️ for developers
