# OpenWebUI ↔ Hermes Agent API Connection (Phase 8.1)

Real, verified connection of an unmodified, upstream OpenWebUI
installation to `hermes_hugo`'s Hermes Agent via Hermes' own
OpenAI-compatible API server. Every claim below is either a literal
command/log/API-response result captured during this work, or a direct
source-code citation.

## Premise correction (researched before touching anything)

The original brief for this phase assumed a separate "Hermes Agent
API" distinct from a "generic OpenAI-compatible API." That distinction
doesn't exist. Confirmed from Hermes' own source
(`gateway/platforms/api_server.py`): `POST /v1/chat/completions` is
handled by `_handle_chat_completions()` → `_run_agent()`, which
instantiates `from run_agent import AIAgent` — **the exact same
`AIAgent` class used by the CLI and every gateway platform adapter
(Telegram, Discord, ...)**. A source comment states outright: *"The
API server creates a server-side Hermes AIAgent."* The OpenAI-shaped
endpoint **is** the official, capability-preserving Hermes Agent API —
Hermes was deliberately built this way specifically so OpenAI-format
frontends (Open WebUI, LobeChat, LibreChat, ...) can connect without
custom integration code. See the architecture-audit discussion earlier
in this phase for the full comparison against OpenClaw's equivalent
design.

## Architecture

```
Hugo (browser/API client)
   │
   ▼
OpenWebUI (companion-openwebui.service, user "openwebui")
   127.0.0.1:8080, unmodified upstream pip install
   │  OpenAI-format HTTP, Bearer API_SERVER_KEY
   ▼
Hermes API server (gateway/platforms/api_server.py)
   127.0.0.1:8642, part of hermes-gateway.service (user "hermes_hugo")
   │
   ▼
AIAgent (same class as CLI/Telegram/Discord — full tools, memory, skills)
   │
   ▼
Grok (api.x.ai) — same provider as CLI use
```

## Required Hermes configuration

`API_SERVER_ENABLED` and `API_SERVER_KEY` are **environment variables**
read directly via `os.getenv()`/`getenv()` in
`gateway/config.py`/`gateway/platforms/api_server.py` — **not**
`config.yaml` keys, confirmed from source after `hermes config set`
produced the same "unrecognized custom key" warning seen earlier with
`HONCHO_BASE_URL` (Sprint 7). Written directly to `~/.hermes/.env`:
```
API_SERVER_ENABLED=true
API_SERVER_KEY=<generated, hex32>
```
No other Hermes settings were touched — Grok, Exa, Honcho, and all
prior config remain exactly as Sprint 6/7 left them.

The API server only runs while the **gateway** process is running
(`api_server.py` is a gateway *platform adapter*, structurally
identical to the Telegram/Discord adapters). For OpenWebUI to have a
persistent, working connection, the gateway needs to run continuously
— installed via Hermes' own official mechanism, not a hand-rolled
unit:
```bash
sudo loginctl enable-linger hermes_hugo   # required: systemd --user needs a session/linger
hermes gateway install                     # writes ~/.config/systemd/user/hermes-gateway.service
systemctl --user enable --now hermes-gateway.service
```
This is the same `hermes gateway install` documented since Sprint 4's
research (`docs/hermes/DEPLOYMENT.md`) — no custom systemd unit was
authored for Hermes itself, consistent with "Upstream First."

## Required OpenWebUI configuration

Installed via the official, documented pip method (Python 3.11 — 3.13
is explicitly unsupported by OpenWebUI, so a dedicated `uv venv
--python 3.11` was used rather than the system's 3.13):
```bash
uv venv --python 3.11
source .venv/bin/activate
uv pip install open-webui
```

Configuration is entirely environment-variable driven (no manual UI
click-through needed — none was possible in this headless
environment, but this is also the officially documented headless
setup path, not a workaround):
```
PORT=8080
DATA_DIR=/opt/companion/openwebui/data
WEBUI_SECRET_KEY=<generated>
WEBUI_ADMIN_EMAIL=hugo@companion.local
WEBUI_ADMIN_PASSWORD=<generated>
WEBUI_ADMIN_NAME=Hugo
OPENAI_API_BASE_URL=http://127.0.0.1:8642/v1
OPENAI_API_KEY=<Hermes' API_SERVER_KEY>
ENABLE_OLLAMA_API=false
WEBUI_AUTH=true
```
`WEBUI_ADMIN_EMAIL`/`WEBUI_ADMIN_PASSWORD`/`WEBUI_ADMIN_NAME` are
OpenWebUI's own documented mechanism for headless first-admin creation
on a fresh install (no users yet). Real, observed side effect: this
**also auto-satisfies "single user only"** — OpenWebUI's own code sets
`ENABLE_SIGNUP = False` the moment the first user exists, with no
extra configuration. Verified via `GET /api/config` after startup:
`"enable_signup": false`.

`ENABLE_OLLAMA_API=false` was set deliberately — OpenWebUI otherwise
also probes for a local Ollama instance by default, which doesn't
exist here and isn't wanted (this project's established boundary:
"kein Ollama").

## Network exposure (deviation from initial default, corrected)

`open-webui serve` defaults to `--host 0.0.0.0`. Real observation: port
8080 was not actually internet-reachable even at that default, because
`ufw` (already active, default-deny) only allows SSH/80/443 — but
relying solely on the firewall would be inconsistent with every other
service in this stack (Honcho, embedding server, Hermes' own dashboard
default all bind loopback-only). Corrected to `--host 127.0.0.1` in
the systemd unit. No Caddy, no HTTPS, no DuckDNS, no reverse proxy was
configured, per the phase brief — access today is loopback-only (e.g.
via SSH port-forward), with no further exposure work done.

## Real end-to-end verification performed

1. **Hermes side, direct baseline** (no OpenWebUI involved):
   `GET /v1/models` → `{"id": "hermes-agent", ...}`; `POST
   /v1/chat/completions` with `"Reply with exactly:
   API_SERVER_DIRECT_OK"` → correct reply, `usage.prompt_tokens:
   18334` (a large figure consistent with genuine system-prompt/tool
   schema injection, not a toy echo endpoint).
2. **OpenWebUI's own model list** (`GET /api/models`, authenticated as
   Hugo): shows `"id": "hermes-agent"`, `"connection_type":
   "external"` — the connection is live and recognized.
3. **Full round-trip through OpenWebUI's own chat backend** (`POST
   /api/chat/completions`, not Hermes directly): `"Reply with exactly:
   OPENWEBUI_TO_HERMES_OK"` → correct reply, `usage.prompt_tokens:
   18338` (matches the direct-baseline figure almost exactly — strong
   evidence this is the same real backend, not a mock).
4. **Cross-confirmed from Hermes' own `agent.log`**, not just trusted
   from HTTP responses: both requests appear as real conversation
   turns with `platform=api_server`, `model=grok-build-0.1
   provider=xai` — genuine agent turns, same logging shape as every
   other verified invocation path in this project (CLI, gateway).

No incompatibilities were encountered during the connection process
itself — the officially documented integration (OpenWebUI's generic
"OpenAI" connection type pointed at Hermes' `/v1` endpoint) worked
exactly as documented, no compatibility shims needed. The only real
friction points were both configuration-location mistakes on the
Hermes side (`API_SERVER_ENABLED`/`API_SERVER_KEY` needing `.env`, not
`config.yaml` — the same category of error as `HONCHO_BASE_URL` in
Sprint 7), not integration incompatibilities.

## Explicitly not done in this phase (per the brief)

- No feature testing beyond the plain-text round trip above — uploads,
  tools, memory (Honcho), workspace, sessions, Super Hermes skills,
  and Curator are all deferred to the next validation phase, even
  though the architecture confirms they're reachable via the same
  `AIAgent` instance.
- No Caddy, HTTPS, DuckDNS, or reverse proxy.
- No second OpenWebUI user — signup is closed, only Hugo exists.
