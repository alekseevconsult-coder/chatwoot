# Deploy on Render (Blueprint)

Render is the most declarative managed-PaaS option for this stack: a single
[`render.yaml`](../../render.yaml) Blueprint (at the repo root) defines the core services —
managed Postgres + Redis, Qdrant, LiteLLM, Chatwoot (web + worker), and the AI service / admin
panel. (Langfuse/ClickHouse and Evolution are omitted to keep the footprint small; add later.)

> Netlify can't run this stack (no containers/databases). Render and Railway can. For the
> simplest path overall, the VPS one-liner / `cloud-init.yaml` in `omni-chat-ai/` is still the
> most reliable — see the README.

## Steps
1. Push this repo to your GitHub/GitLab (already done — it's on `develop`).
2. Render Dashboard → **New → Blueprint** → select this repo. Render reads `render.yaml`,
   provisions the services, and **auto-generates** the secrets (`APP_SECRET_KEY`,
   `LITELLM_MASTER_KEY`, `SECRET_KEY_BASE`, DB credentials).
3. Wait for the first deploy (Chatwoot runs migrations on boot; ~3–5 min).
4. **Set the two URLs** Render couldn't know in advance (Dashboard → each service → Environment):
   - `chatwoot` service → `FRONTEND_URL` = that service's URL (e.g. `https://chatwoot-xxxx.onrender.com`)
   - `ai-service` service → `PUBLIC_BASE_URL` = that service's URL (e.g. `https://ai-service-xxxx.onrender.com`)
   Save → both redeploy.
5. Open **`https://ai-service-xxxx.onrender.com/admin`**, create your admin account, and enter
   your Anthropic + KeyCRM keys under **Settings** (Test connection). Then **Channels** →
   create the website widget / connect Telegram.

## Notes & known checks (first-deploy)
- **Cost:** ~6 paid services + a Postgres instance. Render's free tier won't run all of them.
- **Shared database:** Chatwoot, LiteLLM, and the AI service share the one managed Postgres
  database (different table names; no collisions). Give them separate DB instances later if you
  prefer stronger isolation.
- **Private networking:** services reach each other by name (`http://litellm:4000`,
  `http://qdrant:6333`, `http://chatwoot:3000`). If your Render region/account uses a different
  private hostname scheme, update `LITELLM_BASE_URL` / `QDRANT_URL` / `CHATWOOT_BASE_URL` on the
  `ai-service` accordingly.
- **Chatwoot boot command:** the Blueprint runs `rails db:chatwoot_prepare` then `rails s`. If the
  image's entrypoint already prepares the DB, you can simplify `dockerCommand` to just the server.
- This Blueprint is validated against Render's spec but **not deploy-tested** — treat the first
  deploy as a shakeout. If you connect the **Render MCP server** to this Claude session (or share
  Render API access), I can create and debug the services live until it's green.

## Railway (alternative)
Railway also works and supports cross-service variable references (`${{Postgres.DATABASE_URL}}`).
There's no single-file equivalent to a Blueprint button, so you'd add services from the repo and
wire variables in its dashboard. Ask and I'll provide a per-service Railway setup guide.
