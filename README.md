# personal-llm-proxy

Local dev stack to run **Langfuse (self-host)** + **LiteLLM Proxy** together, so you can:

- send OpenAI-compatible requests to LiteLLM (`http://localhost:4000`)
- see traces in Langfuse (`http://localhost:3000`)

## Repo contents

- `docker-compose.yml`: Langfuse full stack + `litellm-proxy`
- `docker-compose.yml.litellm`: LiteLLM-only compose (Langfuse runs elsewhere)
- `litellm_config.yaml`: LiteLLM Proxy config (enables `langfuse_otel` callback)
- `.env.example`: template (copy to `.env`)
- `AGENTS.md`: guide for AI agents working in this repo
- `langfuse-litellm-integrationn.md`: reference doc for the integration

## Quickstart (Langfuse + LiteLLM)

### 0) Prerequisites

- Docker
- Docker Compose
  - Prefer `docker compose`, but if it’s unavailable use `docker-compose`.

### 1) Create `.env`

```bash
cp .env.example .env
```

Fill at least:

- `OPENAI_API_KEY`
- `LITELLM_MASTER_KEY`
- `LANGFUSE_PUBLIC_KEY` / `LANGFUSE_SECRET_KEY`

### 2) Start the stack

```bash
# Preferred
# docker compose up

# Fallback
docker-compose up
```

### 3) Open UIs

- Langfuse UI: http://localhost:3000
- LiteLLM Proxy: http://localhost:4000

### 4) Send a test request to LiteLLM

The config ships with sample model names (see `litellm_config.yaml`).

- `gpt-4o-mini` (OpenAI)
- `mimo-v2-flash-free` (OpenRouter: `xiaomi/mimo-v2-flash:free`)
- `xiaomi/mimo-v2-flash:free` (same as above, kept to match OpenRouter model name)

```bash
curl -X POST "http://localhost:4000/v1/chat/completions" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer ${LITELLM_MASTER_KEY}" \
  -d '{
    "model": "gpt-4o-mini",
    "messages": [
      {"role": "user", "content": "Say hello in one sentence."}
    ]
  }'
```

Then check Langfuse for the trace.

## LiteLLM-only mode

If you already have Langfuse running somewhere else:

```bash
# Preferred
# docker compose -f docker-compose.yml.litellm up

# Fallback
docker-compose -f docker-compose.yml.litellm up
```

Set `LANGFUSE_OTEL_HOST` in `.env` to your Langfuse base URL (example: `http://localhost:3000`).

## How LiteLLM → Langfuse works (important)

This repo uses LiteLLM’s callback integration:

- `litellm_settings.callbacks: [langfuse_otel]`

Required environment variables:

- `LANGFUSE_PUBLIC_KEY`
- `LANGFUSE_SECRET_KEY`
- `LANGFUSE_OTEL_HOST`

`LANGFUSE_OTEL_HOST` must be a **base URL only** (scheme + host + optional port).
LiteLLM appends `/api/public/otel` internally.

Examples:

- In the same compose network: `LANGFUSE_OTEL_HOST=http://langfuse-web:3000`
- From host machine: `LANGFUSE_OTEL_HOST=http://localhost:3000`

Notes:

- Langfuse OTEL ingest supports **OTLP HTTP/protobuf**, not gRPC.
- Include the scheme (`http://` / `https://`). If you omit it, LiteLLM may assume HTTPS.

## Security notes

- Never commit `.env` (it’s gitignored).
- `docker-compose.yml` contains convenient local defaults (many marked `# CHANGEME`).
  Do not expose this stack to the internet without changing secrets and tightening ports/firewall rules.

## References

- `langfuse-litellm-integrationn.md` (reference doc copied from upstream docs)
- LiteLLM Langfuse OTEL integration docs: https://docs.litellm.ai/docs/observability/langfuse_otel_integration
