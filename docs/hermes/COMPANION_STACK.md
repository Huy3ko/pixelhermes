# Companion Stack — Architecture Overview (Sprint 7)

The full, real, currently-running stack for `hermes_hugo`, as actually
installed and verified — not a target/aspirational diagram. See
[PRODUCTIVE_RUNTIME.md](PRODUCTIVE_RUNTIME.md) for the Grok/Exa layer
(Sprint 6) and [HONCHO.md](HONCHO.md) for the memory/embedding layer
(Sprint 7) for full evidence behind each claim here.

## Components and responsibilities

| Component | Responsibility | Runs as | Official/Upstream status |
|---|---|---|---|
| **Hermes Agent** | Orchestration: agent loop, tool dispatch, workspace, sessions, skills, CLI/API/gateway | `hermes_hugo` | Official (Nous Research), unmodified |
| **Grok (xAI)** | Sole LLM — chat, reasoning, tool-calling, coding | external API (`api.x.ai`) | Official Hermes-supported provider |
| **Exa** | Sole web search backend (`web.backend: exa`) | external API | Official Hermes-supported provider — **see PRODUCTIVE_RUNTIME.md for the unresolved finding that Grok does not reliably invoke it** |
| **Honcho** | External memory provider — cross-session user modeling, dialectic reasoning, semantic recall (replaces built-in MEMORY.md/USER.md) | `honcho` (self-hosted) | Official Hermes memory-provider plugin; Honcho itself is a separate open-source project (plastic-labs), self-hosted here |
| **PostgreSQL 17 + pgvector** | Honcho's persistent storage (peers, sessions, messages, derived documents, embeddings) | `postgres` (Debian package) | Official Honcho dependency |
| **Redis** | Installed per Honcho's production recommendation; not currently used as an active cache (see HONCHO.md) | `redis` (Debian package) | Official Honcho dependency |
| **Local embedding server (llama.cpp)** | Generates embeddings for Honcho's semantic search — fully self-hosted, no cloud embedding provider | `embeddings` (dedicated) | llama.cpp is official upstream (ggml-org), already Hermes-documented as a supported local runtime |

## Data flow

```
User (CLI)
   │
   ▼
Hermes Agent (hermes_hugo)
   ├─→ Grok (api.x.ai)          — chat/reasoning/tool-calls
   ├─→ Exa (via web_search tool) — search (see open finding: not reliably invoked)
   ├─→ Workspace (~/.hermes/)    — sessions, skills, config, logs (local, per-user)
   └─→ Honcho (127.0.0.1:8000)   — memory read/write
             │
             ├─→ PostgreSQL (honcho DB) — durable storage
             ├─→ background deriver     — async reasoning over new messages
             │        └─→ Grok (api.x.ai) — Honcho's own reasoning calls
             │        └─→ local embedding server (127.0.0.1:8081) — vectorization
             └─→ pgvector — semantic search index
```

## Isolation boundaries (real, OS-level — not just logical)

- **Per-Companion-agent isolation** (established Sprint 3, [ADR 0002](../../ADR/0002-companion-user-home-under-srv.md)):
  each Hermes agent (`hermes_hugo`, future `hermes_christiane`) is a
  separate Linux user with its own `~/.hermes/` — no shared filesystem
  access between agents.
- **Shared-service isolation** (new, Sprint 7, [ADR 0003](../../ADR/0003-shared-services-under-opt-companion.md)):
  Honcho and the embedding server each run as their own dedicated,
  unprivileged system user under `/opt/companion/`, with their own
  systemd unit — neither can read `hermes_hugo`'s home directory, and
  `hermes_hugo` cannot read theirs (verified: `hermes_hugo`'s own
  self-investigation in Sprint 6 hit a real permission wall trying to
  access another Companion user's home the same way).
- **Network isolation:** every shared service binds to `127.0.0.1`
  only — Honcho's API, the embedding server, and Postgres/Redis are
  not reachable from outside this host.
- **Multi-agent readiness:** Honcho and the embedding server are
  deliberately *not* tied to `hermes_hugo` specifically — a future
  `hermes_christiane` installation can point at the same
  `127.0.0.1:8000` / `127.0.0.1:8081` endpoints (different Honcho
  `workspace`/`peerName` per agent in each agent's own `honcho.json`)
  without duplicating Postgres, Redis, or the embedding model.

## What stays completely separate (never touches Hermes core)

- Grok and Exa configuration lives entirely in `hermes_hugo`'s own
  `config.yaml`/`.env` — untouched by adding Honcho.
- Honcho's own text-generation reuses the *value* of `hermes_hugo`'s
  `XAI_API_KEY` (copied into Honcho's separate `.env`), not a shared
  credential store — revoking or rotating Hermes' key does not
  automatically propagate, and vice versa. This is a real, documented
  tradeoff, not an oversight.
- No Hermes core file was modified anywhere in this sprint — every
  change was either `hermes config set`, a JSON config file in
  Hermes' own documented location, or entirely outside `~/.hermes/`
  (the new services).

## Not yet part of this stack

Explicitly out of scope for Sprint 7 per the phase brief and
subsequent user decisions during the sprint:

- Humalike (rejected — depends on a paid external API, conflicts with
  "fully self-hosted")
- Cloud embedding providers (OpenAI, Google Gemini — both considered
  and explicitly rejected in favor of the self-hosted llama.cpp path,
  see [ADR 0004](../../ADR/0004-local-embedding-server-for-honcho.md))
- `hermes_christiane` (account exists since Sprint 3; no Hermes
  installation, no Honcho workspace yet)
- Multi-user/OpenWebUI/Persona layer (explicitly deferred to later
  sprints per the phase brief)
