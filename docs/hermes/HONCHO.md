# Honcho — Self-Hosted Memory Provider (Sprint 7)

> **Historisch / vollständig entfernt (Phase X).** Dieses Dokument
> beschreibt den Sprint-7-Zustand als Nachweis der damaligen
> Installation und Verifikation. Honcho ist seit Phase X komplett
> deinstalliert (Pakete, Datenbank, systemd-Dienste, Systembenutzer) —
> aktiver Memory-Provider ist `mnemosyne-hermes`, siehe
> [MNEMOSYNE.md](MNEMOSYNE.md) und
> [ADR 0008](../../ADR/0008-mnemosyne-replaces-honcho.md).

Real, self-hosted installation of Honcho (plastic-labs/honcho) as
Hermes' external memory provider for `hermes_hugo`, plus a local
embedding server it depends on. Everything below is either a literal
command/log/DB-query result captured during this installation, or a
direct source-code citation — no guessing. See
[ADR 0003](../../ADR/0003-shared-services-under-opt-companion.md) and
[ADR 0004](../../ADR/0004-local-embedding-server-for-honcho.md) for the
architecture decisions behind this setup.

## What Honcho is (from its own docs, confirmed in Sprint 2 research)

An AI-native memory backend that maintains a running model of the
user via dialectic reasoning over past conversations, replacing
Hermes' built-in file-based `MEMORY.md`/`USER.md` (the two are
mutually exclusive — selecting Honcho as the provider displaces the
default, per `docs/hermes/MEMORY.md`).

## Architecture

```
hermes_hugo (Hermes CLI)
   │  honcho-ai Python SDK, HTTP to 127.0.0.1:8000
   ▼
companion-honcho-api.service (FastAPI, user "honcho")
   │                                    │
   │ SQL (psycopg)                      │ enqueue derivation jobs
   ▼                                    ▼
PostgreSQL 17 + pgvector          companion-honcho-deriver.service
(db "honcho", role "honcho")      (background worker, user "honcho")
   ▲                                    │
   │ pgvector storage                   │ HTTP, OpenAI-compatible
   └────────────────────────────────────┤
                                         ▼
                          companion-embeddings.service
                          (llama-server, user "embeddings",
                           nomic-embed-text-v1.5, 127.0.0.1:8081)

Honcho's own text-generation (deriver/dialectic/summary/dream)
→ HTTPS → api.x.ai (reuses the existing hermes_hugo XAI_API_KEY,
  openai-transport + base_url override — not a new provider)
```

Redis is installed and running (`redis-server.service`, Debian
package default) but Honcho's own cache client logged `Cache disabled,
using in-memory cache` at startup — real observation: Redis is present
per the official self-hosting docs' "production" recommendation, but
Honcho did not end up using it as a cache backend at this config;
not investigated further (not required for the stated goal).

## Installation (all commands actually run)

### System packages

```bash
sudo apt-get install -y postgresql postgresql-17-pgvector redis-server
```
Result: PostgreSQL 17.10, pgvector 0.8.0, Redis 8.0.2 — all via Debian
13's own repositories, both started automatically as systemd services
(standard package behavior).

### Dedicated system user + database

```bash
sudo useradd -r -m -d /opt/companion/honcho -s /usr/sbin/nologin -c "Honcho memory server" honcho
sudo -u postgres psql -c "CREATE ROLE honcho WITH LOGIN PASSWORD '<generated>';"
sudo -u postgres psql -c "CREATE DATABASE honcho OWNER honcho;"
sudo -u postgres psql -d honcho -c "CREATE EXTENSION IF NOT EXISTS vector;"
```

### Honcho application

```bash
sudo -u honcho -H bash -lc 'cd ~ && git clone https://github.com/plastic-labs/honcho.git'
sudo -u honcho -H bash -lc 'cd ~/honcho && uv sync'   # 95 packages, reused the system-wide uv from Sprint 3
```

### Configuration (`~/honcho/.env`, written via `hermes`-style
non-interactive file write, no secrets echoed to the terminal)

Key choices, each justified in [ADR 0004](../../ADR/0004-local-embedding-server-for-honcho.md):

```
DB_CONNECTION_URI=postgresql+psycopg://honcho:<password>@localhost:5432/honcho
AUTH_USE_AUTH=false                          # loopback-only service, no public exposure

# Text-generation: reuses the existing xAI/Grok credential
LLM_OPENAI_API_KEY=<hermes_hugo's XAI_API_KEY>
DERIVER_MODEL_CONFIG__TRANSPORT=openai
DERIVER_MODEL_CONFIG__MODEL=grok-build-0.1
DERIVER_MODEL_CONFIG__OVERRIDES__BASE_URL=https://api.x.ai/v1
# (same pattern for DIALECTIC_LEVELS__*, SUMMARY_MODEL_CONFIG, DREAM_*)

# Embeddings: fully self-hosted, local llama.cpp server
EMBEDDING_MODEL_CONFIG__TRANSPORT=openai
EMBEDDING_MODEL_CONFIG__MODEL=nomic-embed-text-v1.5
EMBEDDING_MODEL_CONFIG__OVERRIDES__BASE_URL=http://127.0.0.1:8081/v1
EMBEDDING_MODEL_CONFIG__OVERRIDES__API_KEY_ENV=EMBEDDING_API_KEY
EMBEDDING_API_KEY=<local llama-server key>
EMBEDDING_VECTOR_DIMENSIONS=768
VECTOR_STORE_TYPE=pgvector
```

Verified config actually loads: `python -c "from src.config import
settings; print(settings.DERIVER.MODEL_CONFIG)"` → confirmed
`base_url='https://api.x.ai/v1'` on the resolved runtime object, not
just the raw file.

### Migrations — real, non-trivial failure and fix

```bash
sudo -u honcho -H bash -lc 'cd ~/honcho && uv run alembic upgrade head'
```
Ran cleanly (creates `app_id`/`user_id` columns, indexes, an HNSW
index on `documents.embedding` at the **default 1536 dimensions** —
OpenAI's `text-embedding-3-small` size, not our 768-dim local model).

Starting the API server then failed for real, with a clear error from
Honcho's own startup validation:
```
src.startup.embedding_validator.StartupValidationError: public.documents.embedding
dim (1536) does not match EMBEDDING_VECTOR_DIMENSIONS (768). Run
`uv run python scripts/configure_embeddings.py` or fix EMBEDDING_VECTOR_DIMENSIONS.
```
Fixed exactly as instructed (safe: no data existed yet, script refuses
if any non-null embeddings are present):
```bash
sudo -u honcho -H bash -lc 'cd ~/honcho && uv run python scripts/configure_embeddings.py --yes'
```
Output confirmed: dropped and recreated the HNSW indexes on
`documents.embedding` and `message_embeddings.embedding` at
`vector(768)`. This is the official, documented recovery path for a
non-default embedding dimension — not a workaround.

### systemd services

Two units, both `User=honcho`, both loading the same `.env` via
`EnvironmentFile=`:

- `companion-honcho-api.service` — `fastapi run --host 127.0.0.1 --port
  8000 src/main.py` (the actual production command from Honcho's own
  `Dockerfile`/`docker/entrypoint.sh`, not the `fastapi dev` command
  shown in the generic self-hosting guide, which is dev-only).
- `companion-honcho-deriver.service` — `python -m src.deriver`.

Both `enable --now`d, both confirmed `active`.

### Embedding server (`companion-embeddings.service`)

Built from source (`ggml-org/llama.cpp`, `cmake --build build --target
llama-server`, ~2 minutes on this VPS's 6 vCPUs). Model:
`nomic-embed-text-v1.5.Q8_0.gguf` (146 MB) from the official
`nomic-ai/nomic-embed-text-v1.5-GGUF` Hugging Face repo. Started as
`--embedding --pooling mean`, `127.0.0.1:8081`, protected with
`--api-key` (a real, generated local key — the default startup
otherwise logs "CORS is set to allow all origins ('*') and no API key
is set ... this can be a security risk", which was fixed rather than
ignored).

Verified directly:
```
$ curl -X POST http://127.0.0.1:8081/v1/embeddings -d '{"input":"...", "model":"nomic-embed-text-v1.5"}'
→ 768-dim vector, OpenAI-compatible {object, data[].embedding, usage} shape
$ curl (no Authorization header) → HTTP 401
$ curl (with Bearer key)         → HTTP 200
```

## Connecting Hermes to Honcho

```bash
sudo -u honcho -H ...                                      # (Honcho side, above)
sudo -u hermes_hugo -H bash -lc 'hermes config set memory.provider honcho'
```
Plus a hand-written `~/.hermes/honcho.json` (the CLI's `hermes memory
setup honcho` wizard is interactive-only; the JSON schema and its
resolution order — `$HERMES_HOME/honcho.json` then `~/.hermes/honcho.json`
— are confirmed directly from
`plugins/memory/honcho/client.py` in the Hermes source, not guessed):
```json
{
  "baseUrl": "http://127.0.0.1:8000",
  "hosts": {
    "hermes": { "enabled": true, "aiPeer": "hermes", "peerName": "hermes_hugo", "workspace": "hermes" }
  }
}
```
Also required: `uv pip install honcho-ai` inside Hermes' own venv (the
Python SDK the plugin imports — not installed by the base Hermes
installer, confirmed via `ModuleNotFoundError` before installing it).

An earlier attempt to wire this via `hermes config set HONCHO_BASE_URL
...` was real but wrong — Hermes accepted it as an unrecognized custom
key ("saved anyway, but Hermes may not read it") rather than the
plugin's actual expected `honcho.json`/env-fallback mechanism; removed
once the correct approach was confirmed from source.

`hermes memory status` before the fix: `Status: not available ✗,
Missing: HONCHO_API_KEY`. After writing `honcho.json` correctly:
`Status: available ✓` — real before/after confirmation, not assumed.

## Verification (real, end-to-end)

1. `hermes chat -q "My name is Hugo and I prefer terse, technical
   answers..."` → `honcho_profile` tool fired (visible in the CLI's
   tool-call trace, distinct from the earlier Grok/Exa finding where no
   tool call happened at all — this one is real).
2. Confirmed directly in Postgres, not just trusted from the CLI:
   ```sql
   SELECT id, name FROM peers;        -- "hermes" (AI), "hermes_hugo" (user)
   SELECT count(*) FROM queue;        -- 1 (a real derivation job enqueued)
   SELECT count(*), count(embedding) FROM documents;  -- 1, 1 (real 768-dim embedding stored)
   ```
3. **Cross-session semantic retrieval, the actual point of Honcho:** a
   brand new session (`hermes chat -q "What do you know about my
   communication preferences?"`) triggered `honcho_profile` →
   `honcho_context` → `honcho_search` (with a real semantic query
   string, "communication preferences OR terse OR fluff OR technical
   answers") and correctly answered from the prior session's stored
   fact. This is the load-bearing proof that the whole pipeline —
   Hermes → Honcho API → Postgres → deriver → local embedding server →
   pgvector → retrieval — actually works, not just that each piece
   starts.
4. `hermes doctor` afterwards: `Memory Provider: ✓ Honcho connected
   workspace=hermes mode=hybrid freq=async`, and all previously-working
   tool categories (`web`, `x_search`, `terminal`, `file`,
   `session_search`, `skills`, ...) still show ✓ — Grok/Exa/Workspace
   unaffected by this change.

## Data classification

| Location | Contents | Critical? |
|---|---|---|
| PostgreSQL `honcho` DB (`/var/lib/postgresql/17/main`) | peers, sessions, messages, derived documents+embeddings, deriver queue | Critical — this *is* Honcho's memory; no built-in Hermes fallback data exists once Honcho is the active provider |
| `/opt/companion/honcho/honcho/.env` | DB credentials, xAI key reference, embedding server key | Critical, secrets — not in git, root/honcho-readable only |
| `/opt/companion/embeddings/models/*.gguf` | the embedding model weights | Reproducible — re-downloadable from the official Hugging Face repo, not unique data |
| `~/.hermes/honcho.json` (hermes_hugo) | connection config only, no secrets (AUTH_USE_AUTH=false) | Reproducible — trivially rewritten |

## Known limitations / not investigated further

- Redis is installed but Honcho logged "Cache disabled, using
  in-memory cache" — not pursued, since it wasn't required for the
  stated goal (a working self-hosted memory provider).
- No backup procedure yet for the `honcho` Postgres database — out of
  scope for this sprint, worth a future ADR once real memory
  accumulates.
