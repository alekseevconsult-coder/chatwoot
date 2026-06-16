# Omni-Chat-AI

A production-oriented, OSS-first **Unified Chat Panel with AI agents**: connect every messenger
(web widget, Telegram bot + personal Telegram, personal Viber, WhatsApp, Instagram DM) into one
inbox, let **AI agents** handle customers (support / consultation / warranty / sales), hand off
cleanly to **human managers**, integrate **KeyCRM**, and **self-evaluate & self-learn** — with
**no OpenAI lock-in** (Claude primary behind a model gateway).

## Start here
- **Architecture:** [`docs/architecture.md`](docs/architecture.md)
- **Product spec:** [`docs/prd/PRD.md`](docs/prd/PRD.md)
- **Decisions:** [`docs/adr/`](docs/adr/)
- **Knowledge:** [`docs/knowledge/`](docs/knowledge/) · **Workflows:** [`docs/workflows/`](docs/workflows/)
- **Claude Code setup:** [`CLAUDE.md`](CLAUDE.md), [`.claude/agents/`](.claude/agents), [`.claude/commands/`](.claude/commands)

## Stack (all OSS unless noted)
Chatwoot CE (inbox/handoff) · LiteLLM (model gateway → Claude) · LangGraph + Pydantic AI
(multi-agent) · LlamaIndex + Qdrant (RAG) · KeyCRM (CRM) · Langfuse + Ragas + Promptfoo (eval) ·
E-Chat.tech / Evolution API (personal-account connectors).

## One-click deploy (any cloud)

Recommended host: **Hetzner CPX31** (4 vCPU / 8 GB RAM, ~$8.50/mo), Ubuntu 24.04 — runs the whole
stack for a fraction of a per-service PaaS. The repo is **private**, so the server needs a GitHub
token (a fine-grained PAT with read access; revoke it after).

**Option A — paste at server creation (truly one click).** Paste [`cloud-init.yaml`](cloud-init.yaml)
into the provider's **User data / Cloud-init** box (fill `GITHUB_TOKEN`, and `DOMAIN`/`ACME_EMAIL`
for HTTPS), then click *Create*. The server installs Docker and the whole stack automatically.

**Option B — a few commands over SSH (no token in metadata):**
```bash
curl -fsSL https://get.docker.com | sh
git clone --branch claude/omni-chat-ai-stack-7ydEC \
  https://github.com/alekseevconsult-coder/chatwoot-artem.git   # prompts for username + PAT
cd chatwoot-artem/omni-chat-ai
./deploy.sh                                  # or: DOMAIN=example.com ACME_EMAIL=you@example.com ./deploy.sh
```

**Option C — Coolify:** install Coolify on a VPS and import this `docker-compose.yml` (dashboard UX).

**Option D — Render (managed PaaS):** a [`render.yaml`](../render.yaml) Blueprint deploys the core
stack — see [`docs/deploy-render.md`](docs/deploy-render.md). (Netlify can't host this stack;
Railway works too.)

When it finishes, open **`https://panel.<your-domain>/admin`** (or `http://<server-ip>:8080/admin`),
create your admin account, and paste your API keys under Settings. That's it.

## Quick start (local / already have Docker)
```bash
./deploy.sh                   # generates secrets, brings the whole stack up
# or public HTTPS:  DOMAIN=example.com ACME_EMAIL=you@example.com ./deploy.sh
```
Then open the **admin panel** at `http://localhost:8080/admin`:
1. Create your administrator account (first-run wizard).
2. Under **Settings**, paste your API keys — Anthropic, KeyCRM, Telegram, Langfuse. Each has a
   **Test connection** button, and the dashboard shows live green/red status.

No `.env` editing: every key is entered in the panel and stored **encrypted** in Postgres. The
Anthropic key is registered with LiteLLM at runtime, so it takes effect with no restart.

- Admin panel: `http://localhost:8080/admin` · Chatwoot inbox: `:3000` · Langfuse: `:3001`

When `DOMAIN` is set, a **Caddy** service is added automatically and issues/renews Let's Encrypt
certificates — Chatwoot at `https://DOMAIN`, the admin panel at `https://panel.DOMAIN`. No manual
TLS steps.

## AI service
```bash
cd ai-service
uv pip install -e ".[dev]"
pytest                         # smoke tests
uvicorn app.main:app --reload --port 8080
```

## Layout
```
omni-chat-ai/
├── install.sh              # one-line remote bootstrap (installs Docker, clones, deploys)
├── cloud-init.yaml         # paste into a provider's user-data for one-click at server creation
├── deploy.sh               # generates secrets, brings the stack up, adds Caddy/HTTPS if DOMAIN set
├── docker-compose.yml      # full stack (Caddy reverse proxy under the `tls` profile)
├── litellm/config.yaml     # provider-agnostic model routing (Claude primary)
├── ai-service/             # FastAPI + admin panel + LangGraph supervisor + agents + tools
├── docs/architecture.md    # reference architecture
├── docs/prd/PRD.md         # product requirements
├── docs/adr/               # architecture decision records
├── docs/knowledge/         # integration knowledge (Chatwoot, KeyCRM, channels)
├── docs/workflows/         # operational workflows (handoff)
├── CLAUDE.md               # Claude Code project memory / rules
└── .claude/                # subagents + slash-command workflows
```

> Status: turnkey core. Ships a polished **admin panel** (encrypted settings store, first-run
> wizard, live integration status, per-key Test connection), **runtime LLM key registration**
> with LiteLLM (no restarts, no keys in files), a **four-agent supervisor** (router →
> support / consultation / warranty / sales) with grounded KeyCRM tools and clean Chatwoot
> handoff, Langfuse tracing, and a **one-command deploy**. RAG (Qdrant) and Chatwoot channel
> auto-provisioning wire in next. Extend specialists and channels via `.claude/commands/`.
