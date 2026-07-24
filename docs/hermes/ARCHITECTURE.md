# Hermes Agent — Architecture (Core)

Deckt die Core-Themen Configuration, CLI, API Server, Gateway, Profiles
und Sessions ab. Quellen wie jeweils angegeben; alle aus
`hermes-agent.nousresearch.com/docs/` bzw. dem GitHub-Repo
`NousResearch/hermes-agent`.

## Configuration

Quelle: [Configuration](https://hermes-agent.nousresearch.com/docs/user-guide/configuration),
[Environment Variables](https://hermes-agent.nousresearch.com/docs/reference/environment-variables)

Alles lebt unter `~/.hermes/` (= `HERMES_HOME`):

- `config.yaml` — Haupteinstellungen (Modell, Terminal, TTS, Kompression, ...)
- `.env` — API-Keys/Secrets
- `auth.json` — OAuth-Provider-Credentials
- `SOUL.md` — Agenten-Identität (siehe [PERSONAS.md](PERSONAS.md))
- `memories/` — persistentes Gedächtnis (siehe [MEMORY.md](MEMORY.md))
- `skills/` — Agent-erstellte Skills (siehe [SKILLS.md](SKILLS.md))
- `sessions/` — Gateway-Sitzungsdaten

**Präzedenz (höchste zuerst):** CLI-Argumente → `config.yaml` → `.env` →
eingebaute Defaults.

**Grundregel:** Secrets → `.env`, Verhaltenseinstellungen → `config.yaml`.
Nur `${VAR_NAME}`-Syntax wird in `config.yaml` expandiert, kein `$VAR`.

**CLI-Verwaltung:** `hermes config` (Anzeige), `edit`, `get KEY`,
`set KEY VALUE`, `check` (fehlende Optionen erkennen), `migrate`
(interaktiv neue Optionen ergänzen), `path`, `env-path`, `show`. Über
`hermes config set` gesetzte API-Keys landen automatisch in `.env`.

**Terminal-Backend-spezifische Config:** Docker
(`terminal.docker_image`, `terminal.docker_volumes`, ...), SSH
(`TERMINAL_SSH_HOST`, `TERMINAL_SSH_USER`, `TERMINAL_SSH_PORT`), Local
(keine Sonderkonfiguration).

**Enterprise/Organisation:** Administratoren können Konfigurationswerte
über einen "managed scope" unveränderlich vorgeben.

**Nicht dokumentiert:** kein einzelnes, vollständiges kanonisches
`config.yaml`-Beispiel mit allen Top-Level-Keys an einer Stelle — die
Doku verweist stattdessen auf ein Dashboard-Formular mit "150+
Konfigurationsfeldern".

## CLI

Quelle: [CLI](https://hermes-agent.nousresearch.com/docs/user-guide/cli),
[CLI Commands Reference](https://hermes-agent.nousresearch.com/docs/reference/cli-commands)

Die CLI ist ein vollwertiges Terminal-UI (TUI) mit Mehrzeilen-Editor,
Slash-Command-Autovervollständigung, Verlauf und Streaming-Tool-Output.

**Grundlegender Aufruf:** `hermes` (interaktive Sitzung), `hermes chat -q
"..."` (Einzelabfrage), `--tui`/`--cli`/`--dev`, `hermes -z "prompt"`
(reiner One-Shot).

**Modell/Provider:** `hermes chat --model "..."`, `--provider
nous|openrouter`.

**Sessions:** `--continue`/`-c`, `--resume <id>`/`-r`.

**Umfangreicher Befehlsraum** (Auszug, vollständig in der Referenz):
Chat, Gateway (`hermes gateway run|start|stop|restart|status|...`), LSP,
Setup, Messaging, Secrets/Auth, Config/Migrate, Security/Diagnostics
(`hermes security audit`, `hermes doctor`), Daten (`hermes backup`,
`import`, `sessions`, `logs`), Scheduling (`hermes cron`, `kanban`,
`project`), Skills/Extensions (`hermes skills`, `bundles`, `curator`,
`moa`, `fallback`), Plugins/Integrationen (`hermes plugins`, `acp`,
`mcp`, `memory`, `hooks`), Server (`hermes serve`, `hermes dashboard`),
Profile (`hermes profile ...`).

**Slash-Commands (Auszug):** `/help`, `/model`, `/tools`, `/skills
browse`, `/background <prompt>`, `/voice on`, `/reasoning high`,
`/title`, `/status`, `/sessions`, `/personality <name>`, `/compress`,
`/new`, `/handoff <platform>`, `/rollback`.

## API Server

Quelle: [Web Dashboard](https://hermes-agent.nousresearch.com/docs/user-guide/features/web-dashboard),
[CLI Commands Reference](https://hermes-agent.nousresearch.com/docs/reference/cli-commands),
[Architecture](https://hermes-agent.nousresearch.com/docs/developer-guide/architecture)

Hermes exponiert einen HTTP-Server über **`hermes dashboard`** (Browser
öffnet sich automatisch) bzw. **`hermes serve`** (headless, gleiche
Flags: `--host`, `--port`, `--insecure`, `--skip-build`, `--stop`,
`--status`).

- Standard-Bind: `http://127.0.0.1:9119`.
- Seiten: Status, Chat (eingebettetes TUI über WebSocket/xterm.js),
  Config-Editor, API Keys, Sessions, Skills, MCP Servers, Channels, Cron
  Jobs, Analytics.
- REST-Endpunkte (dokumentiert): `GET/PUT /api/config`, `GET/PUT/DELETE
  /api/env`, `GET /api/status`, `/api/sessions`, `/api/logs`,
  `/api/analytics/usage`, plus undetaillierte administrative Routen für
  MCP, Channels, Webhooks, Pairing.
- **Auth-Modell:** Auth ist nur nötig, wenn der Bind-Host NICHT
  `127.0.0.1`/`::1`/`localhost`/`0.0.0.0` ist und `--insecure` nicht
  gesetzt ist. Providers: Nous-OAuth, Basic-Auth
  (`HERMES_DASHBOARD_BASIC_AUTH_*`), selbstgehostetes OIDC. **Fail
  closed:** ein Nicht-Loopback-Dashboard startet ohne konfigurierten
  Auth-Provider gar nicht erst.
- **Voraussetzung:** Standardinstallation enthält keine HTTP/PTY-Deps —
  `uv pip install -e ".[web,pty]"` nötig.
- Multi-Profil: das Dashboard läuft auf Maschinenebene und verwaltet
  alle Profile über einen Sidebar-Switcher / `?profile=<name>`.

**Nicht dokumentiert:** exakter Verhaltensunterschied `hermes serve` vs.
`hermes dashboard` jenseits von "headless"; vollständige Liste der
"administrativen Routen".

## Gateway

Quelle: [Messaging](https://hermes-agent.nousresearch.com/docs/user-guide/messaging),
[Gateway Internals](https://hermes-agent.nousresearch.com/docs/developer-guide/gateway-internals)

"The gateway is a single background process that connects to all your
configured platforms, handles sessions, runs cron jobs, and delivers
voice messages." Unterstützt 25+ Plattformen (Telegram, Discord, Slack,
WhatsApp, Signal, SMS, E-Mail, Teams, Google Chat, DingTalk, Feishu,
Matrix, u. a.).

**Nachrichtenfluss:** Plattform-Adapter normalisieren Events zu
`MessageEvent` → Concurrency-Guards → Session-Auflösung via Routing-Key
`agent:main:{platform}:{chat_type}:{chat_id}` → eine `AIAgent`-Instanz
pro Konversation. Gateway liest `config.yaml` direkt (anders als die CLI
mit hartkodierten Defaults) — kann zu Verhaltensabweichungen führen,
falls ein Config-Key beim Nutzer fehlt.

**Autorisierung (Reihenfolge, freizügigste zuerst):**
Plattform-Allow-all-Flag → Plattform-Allowlist → DM-Pairing-Codes →
globales Allow-all → Default-Deny.

**Verwaltung:** `hermes gateway setup|run|start|stop|restart|status|...`.
Diensteinrichtung plattformspezifisch: systemd (Linux), launchd (macOS).

**Multi-Profil-Gateways:** jedes Profil betreibt einen eigenen,
unabhängigen Gateway-Prozess mit eigenen Bot-Tokens; "Token-Locks"
verhindern Konflikte bei geteilten Bot-Tokens.

## Profiles

Quelle: [Profiles](https://hermes-agent.nousresearch.com/docs/user-guide/profiles),
[Architecture](https://hermes-agent.nousresearch.com/docs/developer-guide/architecture)

Ein Profil ist "a separate Hermes home directory" — mehrere unabhängige
Agenten auf einer Maschine, jeweils mit eigenem `config.yaml`, `.env`,
`SOUL.md`, Sessions, Memory, Logs, Cron-Jobs, Gateway-State und eigener
State-Datenbank. Der Default-Profil ist wörtlich `~/.hermes` selbst.

**Erstellung:** `hermes profile create <name>` (leer), `--clone`
(Config/.env/SOUL.md/Skills, frische Sessions/Memory), `--clone-all`
(vollständiger Zustand außer Verlauf/Backups), `--clone-from <quelle>`.

**Nutzung:** Auto-generierter Befehlsalias `~/.local/bin/<name>`
(`coder chat`), explizit `hermes -p coder chat`, sticky Default `hermes
profile use coder`.

**Isolations-Caveat (explizit dokumentiert):** Profile isolieren
**Zustand** (Config, Memory, Sessions), aber **keine Dateisystem-
Sandbox** — Agenten behalten normalen Dateisystemzugriff des OS-Users.

**Verwaltungsbefehle:** `list`, `show`, `rename`, `export`, `import`,
`delete`, `alias`, `install <source>` (Profil-Distribution), `update`,
`info`.

## Sessions

Quelle: [Sessions](https://hermes-agent.nousresearch.com/docs/user-guide/sessions),
[Session Storage](https://hermes-agent.nousresearch.com/docs/developer-guide/session-storage)

Eine Session ist ein vollständiger Konversationsthread auf einer
beliebigen Plattform. **Speicherung:** SQLite-Datenbank `~/.hermes/state.db`
im **WAL-Modus** (mehrere Leser, ein Schreiber) — löste frühere
Pro-Session-JSONL-Dateien ab. Sechs Schemakomponenten: Sessions,
Messages, zwei FTS5-Volltext-Tabellen (Standard + Trigram für
CJK-Substring-Suche), State-Metadaten, Schema-Versionierung (aktuell
Version 21).

**Lebenszyklus:** Auto-Titel nach 3–7 Wörtern; `/compress` erzeugt eine
Fortsetzungssession (Lineage über `parent_session_id`); Gateway-
Auto-Reset-Policies `idle`/`daily`/`both`; optionales Auto-Pruning
(`sessions.auto_prune`, Default 90 Tage).

**Suche:** eingebautes `session_search`-Tool mit FTS5-Volltextsuche,
drei Modi (Discovery, Scroll, Browse) — keine Zusammenfassung durch das
LLM nötig.

**Nicht dokumentiert:** exaktes DDL/Schema aller sechs Komponenten;
genaue Trigger-Bedingungen für Trigram- vs. Standard-FTS5-Index.

---

## PixelHermes-Mapping

**Was übernimmt Hermes bereits?** CLI, API-Server/Dashboard,
Messaging-Gateway, Multi-Profil-System und Session-Persistenz sind
vollständig fertige, dokumentierte Subsysteme — inklusive Auth-Modell
für den Dashboard-Zugriff und Autorisierungslogik für den Gateway.

**Was müssen wir NICHT selbst entwickeln?** Keine eigene REST-API,
keinen eigenen Session-Store, kein eigenes Multi-Tenant-/Multi-Profil-
Konzept, kein eigenes Messaging-Gateway. Das deckt sich mit dem in
[README.md](../../README.md) genannten Prinzip "Hermes orchestriert".

**Was passt direkt zu PixelHermes?** Das Profile-Konzept korrespondiert
mit den in einem früheren Sprint angedachten "Agenten"
(`hermes_hugo`/`hermes_christiane` aus einem früheren Konzeptentwurf) —
diese sollten als Hermes-**Profile** modelliert werden, nicht als
eigene Systembenutzer/Services (siehe [ARCHITECTURE.md](../../ARCHITECTURE.md),
"Workspace First"). Die geplanten Systempfade `/etc/companion`,
`/var/log/companion` passen konzeptionell zu `config.yaml`/`.env` bzw.
`~/.hermes/logs/`, sollten aber wegen der expliziten
Profil-Verzeichniskonvention (`~/.hermes/`) nicht 1:1 dorthin gespiegelt
werden — dazu mehr in [DEPLOYMENT.md](DEPLOYMENT.md).

**Welche Erweiterungen wären später sinnvoll?** Ein schlanker Reverse
Proxy vor `hermes dashboard` für Fernzugriff (siehe
[DEPLOYMENT.md](DEPLOYMENT.md)); ein PixelHermes-Wrapper-Skript, das
`hermes profile list` in eine für PixelHermes lesbare Übersicht bringt.

**Welche Komponenten sollten wir bewusst unverändert übernehmen?** CLI,
Dashboard/API-Server, Gateway-Implementierung, Session-Storage-Engine
und das Profile-System vollständig unverändert — jede Abweichung
erfordert laut "Upstream First" ein [ADR](../../ADR/).
