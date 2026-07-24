# Install

Dieses Dokument beschreibt, wie das Infrastruktur-Fundament und die
Runtime von PixelHermes entstanden sind, damit alles jederzeit
reproduzierbar ist.

## Sprint 1 — Foundation

**Keine Software installiert, keine Systemkonfiguration verändert** —
ausschließlich das Git-Repository wurde aufgebaut.

### Voraussetzungen

- Debian 13
- Git

### Ausgangspunkt

Bestehendes GitHub-Repository `Huy3ko/pixelhermes`, geklont nach
`~/companion`, mit einem initialen `README.md`.

### Schritte

```bash
cd ~/companion

# Verzeichnisstruktur
mkdir -p docs configs scripts systemd templates assets backups ADR

# .gitignore sowie Kerndokumentation anlegen:
# README.md, ROADMAP.md, ARCHITECTURE.md, INSTALL.md, CHANGELOG.md,
# PROJECT_STRUCTURE.md, ADR/ (siehe Repo-Inhalt)

git add .
git commit -m "chore(structure): scaffold infrastructure directories"
git commit -m "docs: complete core documentation for Sprint 1"
git commit -m "docs(adr): establish architecture decision record process"

git push origin main
```

### Ergebnis

Ausschließlich ein versioniertes Git-Repository mit vollständiger
Kerndokumentation und der dokumentierten Verzeichnisstruktur — keine
laufenden Services, keine Benutzer, keine Datenbanken, kein Workspace,
keine Agenten.

## Sprint 2 — Hermes Architecture Research

Reine Recherche, keine System- oder Repository-Struktur-Änderung
außerhalb von `docs/hermes/`. Siehe
[docs/hermes/OVERVIEW.md](docs/hermes/OVERVIEW.md).

## Sprint 3 — Runtime Foundation

Erste **echten** Systemänderungen: Laufzeit-Abhängigkeiten und
Grundstruktur, auf der Hermes später installiert wird. Weiterhin
**keine** Installation von Hermes, Hermes Workspace, OpenWebUI, mem0
oder Humalike; **keine** systemd-Services, **keine** Workspaces, **keine**
SQLite-Datenbanken.

### Node.js (offizielle LTS, nicht die Debian-Paketversion)

Installiert über das offizielle NodeSource-Repository (Active LTS zum
Zeitpunkt der Installation: Node.js 24):

```bash
curl -fsSL https://deb.nodesource.com/setup_24.x | sudo -E bash -
sudo apt-get install -y nodejs
```

Ergebnis: `node v24.18.0`, `npm 11.16.0`, installiert unter
`/usr/bin/node` (NodeSource-Repository, nicht das Debian-Bookworm/Trixie-
`nodejs`-Paket).

### Python

System-Python bereits vorhanden (`python3 --version` → `Python
3.13.5`). Ergänzt über `apt`:

```bash
sudo apt-get install -y python3-venv python3-pip
```

Zusätzlich der offizielle Python-Paketmanager **uv**, systemweit
installiert (nicht nur für den aktuellen Nutzer), damit er später auch
für die Companion-Benutzer verfügbar ist:

```bash
curl -LsSf https://astral.sh/uv/install.sh | sudo env UV_INSTALL_DIR="/usr/local/bin" sh
```

Ergebnis: `uv 0.11.32`, `uvx 0.11.32` unter `/usr/local/bin/`.

### Basispakete

Geprüft, bereits vorhanden: `git`, `curl`, `wget`, `sqlite3`,
`build-essential`, `ca-certificates`, `jq`, `tree`, `htop`, `zip`,
`unzip`. Keine weitere Installation nötig.

### Companion-Systemstruktur (real angelegt)

```bash
sudo mkdir -p /opt/companion/{skills,tools,templates,shared} \
  /etc/companion \
  /srv/companion \
  /var/log/companion \
  /var/backups/companion
sudo chown -R root:root /opt/companion /etc/companion /srv/companion \
  /var/log/companion /var/backups/companion
sudo chmod 755 /opt/companion /etc/companion /srv/companion \
  /var/log/companion /var/backups/companion
```

### Companion-Benutzer

Zwei unprivilegierte Linux-Benutzer, Home-Verzeichnis bewusst unter
`/srv/companion/<name>` statt `/home/<name>` (Begründung:
[ADR 0002](ADR/0002-companion-user-home-under-srv.md)), Passwort-Login
gesperrt (Zugriff vorerst nur über `sudo -u <name>`):

```bash
sudo useradd -m -d /srv/companion/hermes_hugo -s /bin/bash \
  -c "Companion Agent hermes_hugo" hermes_hugo
sudo useradd -m -d /srv/companion/hermes_christiane -s /bin/bash \
  -c "Companion Agent hermes_christiane" hermes_christiane
sudo passwd -l hermes_hugo
sudo passwd -l hermes_christiane
```

Ergebnis: `hermes_hugo` (UID 1001), `hermes_christiane` (UID 1002),
Home-Verzeichnisse `700`, kein Hermes, kein systemd-Service, kein
Workspace, keine Datenbank — ausschließlich Benutzer und leere
Home-Verzeichnisse.

## Sprint 4 — Native Hermes Installation

Erste echte Hermes-Installation, ausschließlich für `hermes_hugo`.
`hermes_christiane` folgt erst nach erfolgreichem Abschluss. Kein
systemd-Service, kein Workspace-Umbau, keine zusätzlichen Skills, kein
MCP, kein mem0/Humalike, kein Modell-Provider konfiguriert. Vollständige
Beobachtungen inkl. aller Abweichungen von der Doku-Recherche aus
Sprint 2: [docs/hermes/INSTALLATION.md](docs/hermes/INSTALLATION.md).

### Installation (als hermes_hugo, offizieller Installer)

```bash
sudo -u hermes_hugo -H bash -lc 'cd ~ && curl -fsSL https://hermes-agent.nousresearch.com/install.sh | bash'
```

Nicht-interaktiv fehlgeschlagene, vom Installer selbst vorgeschlagene
Nacharbeiten (beide erfordern Root, wurden separat nachgeholt):

```bash
sudo apt-get install -y ripgrep ffmpeg
sudo bash -c 'cd /srv/companion/hermes_hugo/.hermes/hermes-agent && npx playwright install-deps chromium'
```

Playwright/Chromium blieb trotzdem nicht funktionsfähig (`playwright:
not found`) — nicht weiter verfolgt, da Browser-Tools außerhalb des
Phase-4-Scopes (CLI/API Server/Gateway/Profil) liegen.

### Config-Migration (kein API-Key, keine Anmeldedaten)

```bash
sudo -u hermes_hugo -H bash -lc 'cd ~/.hermes/hermes-agent && source venv/bin/activate && uv pip install -e ".[web,pty]"'
sudo -u hermes_hugo -H bash -lc 'cd ~ && hermes config migrate'
```

Das `[web,pty]`-Install fügte real **keine** neuen Pakete hinzu — bereits
Teil der Standardinstallation (Abweichung von der Doku-Vermutung aus
Sprint 2, siehe [docs/hermes/INSTALLATION.md](docs/hermes/INSTALLATION.md)).
`config migrate` hob das generierte `config.yaml` ohne Abfrage von
API-Keys von Schema-Version 0 auf 33.

### Verifikation (alle Schritte real ausgeführt, danach sauber gestoppt)

```bash
sudo -u hermes_hugo -H bash -lc 'hermes --version'
sudo -u hermes_hugo -H bash -lc 'hermes doctor'
sudo -u hermes_hugo -H bash -lc 'hermes profile list'
sudo -u hermes_hugo -H bash -lc 'hermes config check'
sudo -u hermes_hugo -H bash -lc 'nohup hermes serve --host 127.0.0.1 --port 9119 > /tmp/hermes_serve.log 2>&1 &'
curl http://127.0.0.1:9119/api/status
sudo -u hermes_hugo -H bash -lc 'hermes serve --stop'
sudo -u hermes_hugo -H bash -lc 'nohup hermes gateway run > /tmp/hermes_gateway.log 2>&1 &'
sudo -u hermes_hugo -H bash -lc 'hermes gateway status'
sudo -u hermes_hugo -H bash -lc 'hermes gateway stop'
sudo -u hermes_hugo -H bash -lc 'hermes skills list'
sudo -u hermes_hugo -H bash -lc 'hermes mcp list'
sudo -u hermes_hugo -H bash -lc 'hermes mcp catalog'
```

Ergebnisse (Details und vollständige Rohausgaben:
[docs/hermes/ARCHITECTURE.md](docs/hermes/ARCHITECTURE.md),
[docs/hermes/SKILLS.md](docs/hermes/SKILLS.md),
[docs/hermes/MCP.md](docs/hermes/MCP.md)):

| Prüfung | Ergebnis |
|---|---|
| Hermes startet | `Hermes Agent v0.19.0 (2026.7.20)` |
| CLI funktioniert | `doctor`/`config check`/`profile list` fehlerfrei |
| Profil wird erkannt | genau ein Profil `◆default` |
| Konfiguration lädt korrekt | Config-Version 33, Pfade über API bestätigt |
| API erreichbar | `HTTP 200` auf `/api/status`, `hermes_home`/`config_path` korrekt |
| Gateway startet | läuft manuell, meldet sich selbst als "not a system service" |
| Standard-Skills | 65 aktivierte builtin-Skills über 13 Kategorien, 0 Hub/lokal |
| Standard-MCP-Server | 0 konfiguriert; Katalog zeigt 4 verfügbare (blender, linear, n8n, unreal-engine) |

Kein Prozess läuft nach der Verifikation dauerhaft weiter (kein
systemd — wie für diese Phase vorgegeben).

### Ergebnis

Ein funktionierender, unveränderter Hermes-Agent für `hermes_hugo`
(CLI, API Server, Gateway, Default-Profil) — kein Modell-Provider, kein
zusätzlicher Skill, kein MCP-Server, kein systemd-Service, kein
Workspace-Umbau. `hermes_christiane` weiterhin nur Konto + leeres Home.

## Sprint 6 — Productive Runtime

Erste produktive Provider-Konfiguration, ausschließlich für
`hermes_hugo`. Genau zwei Provider: Grok (xAI) als einziges LLM, Exa
als einzige Suche. Kein OpenRouter, kein Ollama, kein OpenWebUI, kein
mem0, kein Humalike, keine zusätzlichen Skills, kein MCP-Server, kein
systemd, keine weiteren Benutzer. (Sprint 5 — ein geplantes internes
Hermes-Audit — wurde übersprungen; der geschriebene Plan wurde
verworfen, nicht ausgeführt.)

### Grok konfigurieren

```bash
sudo -u hermes_hugo -H bash -lc 'hermes config set XAI_API_KEY "<key>"'
sudo -u hermes_hugo -H bash -lc 'hermes config set model.provider xai'
sudo -u hermes_hugo -H bash -lc 'hermes config set model.default grok-build-0.1'
```

Verifiziert: Chat, Tool-Calling (terminal), Coding (Datei schreiben,
ausführen, auf echten Fehler reagieren) — alle real getestet und im
`agent.log` bestätigt.

### Exa konfigurieren

```bash
sudo -u hermes_hugo -H bash -lc 'hermes config set EXA_API_KEY "<key>"'
sudo -u hermes_hugo -H bash -lc 'hermes config set web.backend exa'
sudo -u hermes_hugo -H bash -lc 'hermes config set web.search_backend exa'
sudo -u hermes_hugo -H bash -lc 'hermes config set web.extract_backend exa'
```

**Kritischer, ungelöster Fund:** trotz korrekter Konfiguration (Key
gesetzt, `web`-Toolset laut `hermes doctor` verfügbar) ruft das Modell
das `web_search`-Tool in drei unabhängigen, expliziten Tests nicht real
auf — es erzeugt stattdessen plausibel aussehende, aber erfundene
"Tool-Ergebnisse". Belegt über `~/.hermes/logs/agent.log`
(`tool_turns=0` bei jedem Versuch). Nicht als Hermes-eigenes
Beispiel im Quellcode gefunden (gezielt gegrept). Ursache nicht
identifiziert — nicht weiterverfolgt, um Hermes nicht zu verändern.
Volle Beweisführung:
[docs/hermes/PRODUCTIVE_RUNTIME.md](docs/hermes/PRODUCTIVE_RUNTIME.md).

### Reale Arbeitsaufgaben (nicht künstlich)

In `~/hermes-notes/`: echtes `git init` + `README.md` + Commit (8
Tool-Calls); ein Planungsdokument `ROLLOUT_NOTES.md` für den
`hermes_christiane`-Rollout (107 Tool-Calls über 4 Minuten — der Agent
untersuchte dabei selbstständig das System, u. a. einen echten,
gescheiterten Versuch, auf `hermes_christiane` zuzugreifen); eine
werkzeugfreie Reasoning-Aufgabe zu systemd user- vs. system-Services.

### Sessions, Curator

`hermes sessions list/export/optimize-storage/repair` funktionierten
korrekt; `session_search` fand eine frühere Session korrekt wieder;
`sessions archive --older-than` traf in mehreren Tests nie, obwohl
sichtlich ältere Sessions vorhanden waren — realer, ungeklärter Befund.
Ein realer, unbehandelter Absturz trat auf, als ein Befehl aus einem
für `hermes_hugo` unlesbaren Arbeitsverzeichnis lief (Git-Kontext-
Erkennung fängt Permission-Fehler nicht ab). Curator: nur beobachtet
(`hermes curator status`), nichts ausgelöst — 0 Läufe, Trigger-
Bedingungen (7 Tage/2h Leerlauf) in dieser Sitzung nicht erfüllbar.

### Ergebnis

Produktive Grok+Exa-Konfiguration für `hermes_hugo`, größtenteils
funktionierend — mit einem zentralen, offenen Problem (Exa-Suche wird
nicht zuverlässig genutzt) und mehreren kleineren realen Funden.
Vollständige Bewertung: [docs/hermes/ASSESSMENT.md](docs/hermes/ASSESSMENT.md).

## Sprint 7 — Companion Foundation (Honcho, self-hosted)

Ziel: den Companion-Stack um einen selbstgehosteten externen
Memory-Provider erweitern. Humalike wurde geprüft, dann bewusst
abgelehnt (kostenpflichtiger externer Cloud-Dienst, widerspricht "Self
Hosted"). Vollständige Beweisführung:
[docs/hermes/HONCHO.md](docs/hermes/HONCHO.md),
[docs/hermes/COMPANION_STACK.md](docs/hermes/COMPANION_STACK.md).

### System-Pakete und Systembenutzer

```bash
sudo apt-get install -y postgresql postgresql-17-pgvector redis-server
sudo useradd -r -m -d /opt/companion/honcho -s /usr/sbin/nologin honcho
sudo useradd -r -m -d /opt/companion/embeddings -s /usr/sbin/nologin embeddings
sudo -u postgres psql -c "CREATE ROLE honcho WITH LOGIN PASSWORD '<generated>';"
sudo -u postgres psql -c "CREATE DATABASE honcho OWNER honcho;"
sudo -u postgres psql -d honcho -c "CREATE EXTENSION IF NOT EXISTS vector;"
```

### Lokaler Embedding-Server (llama.cpp, statt Cloud-Provider)

Begründung: Honcho unterstützt nur `openai`/`gemini` als
Embedding-Transport; Grok bietet keine Embeddings. OpenAI und Gemini
wurden als Cloud-Optionen explizit erwogen und abgelehnt — siehe
[ADR 0004](ADR/0004-local-embedding-server-for-honcho.md).

```bash
sudo -u embeddings -H bash -lc 'git clone https://github.com/ggml-org/llama.cpp.git ~/llama.cpp'
sudo -u embeddings -H bash -lc 'cd ~/llama.cpp && cmake -B build -DCMAKE_BUILD_TYPE=Release -DGGML_NATIVE=ON -DLLAMA_CURL=OFF && cmake --build build --target llama-server -j$(nproc)'
sudo -u embeddings -H bash -lc 'curl -fL -o ~/models/nomic-embed-text-v1.5.Q8_0.gguf "https://huggingface.co/nomic-ai/nomic-embed-text-v1.5-GGUF/resolve/main/nomic-embed-text-v1.5.Q8_0.gguf"'
# systemd unit companion-embeddings.service: --embedding --pooling mean --host 127.0.0.1 --port 8081 --api-key <generated>
```
Verifiziert: `curl -X POST http://127.0.0.1:8081/v1/embeddings ...` →
echter 768-dim-Vektor, OpenAI-kompatibles Antwortformat; ohne
Authorization-Header → `HTTP 401`.

### Honcho (self-hosted)

```bash
sudo -u honcho -H bash -lc 'git clone https://github.com/plastic-labs/honcho.git ~/honcho'
sudo -u honcho -H bash -lc 'cd ~/honcho && uv sync'
# .env: DB_CONNECTION_URI (lokales Postgres), Text-Gen über bestehenden
# XAI_API_KEY (base_url=api.x.ai), Embeddings über lokalen llama-server
sudo -u honcho -H bash -lc 'cd ~/honcho && uv run alembic upgrade head'
```
Realer, dokumentierter Fehler beim ersten Start (Standard-Vektor-
dimension 1536 statt unserer 768) — offizieller Fix angewendet:
```bash
sudo -u honcho -H bash -lc 'cd ~/honcho && uv run python scripts/configure_embeddings.py --yes'
```
Danach zwei systemd-Units (`companion-honcho-api.service`,
`companion-honcho-deriver.service`), beide `enable --now`, beide
`active` bestätigt. `curl http://127.0.0.1:8000/health` → `{"status":"ok"}`.

### Hermes mit Honcho verbinden

```bash
sudo -u hermes_hugo -H bash -lc 'cd ~/.hermes/hermes-agent && source venv/bin/activate && uv pip install honcho-ai'
sudo -u hermes_hugo -H bash -lc 'hermes config set memory.provider honcho'
# ~/.hermes/honcho.json manuell geschrieben (Format aus dem Hermes-Quellcode
# bestätigt, nicht geraten): {"baseUrl": "http://127.0.0.1:8000", "hosts": {...}}
```
`hermes memory status` vorher: `Status: not available ✗`. Danach:
`Status: available ✓`.

### Verifikation (real, Ende-zu-Ende)

- Reale Nachricht mit persönlicher Präferenz → `honcho_profile`-Tool
  feuerte real; Zeilen in Postgres (`peers`, `queue`, `documents` mit
  echtem 768-dim-Embedding) per SQL bestätigt, nicht nur der
  CLI-Ausgabe geglaubt.
- **Neue Session** fragte nach den Präferenzen → `honcho_search` mit
  echter semantischer Query, korrekte Antwort aus der vorherigen
  Session — der eigentliche Zweck von Honcho, real bestätigt.
- `hermes doctor` danach: `Memory Provider: ✓ Honcho connected
  workspace=hermes mode=hybrid freq=async`; alle zuvor funktionierenden
  Tool-Kategorien (Grok, `web`, `x_search`, Workspace, Sessions)
  weiterhin ✓ — keine Regression durch die Erweiterung.

### Super Hermes und Self-Evolution (geprüft, third-party)

Zwei zusätzliche, im Sprintverlauf angefragte Erweiterungen wurden
recherchiert und bewusst nicht blind installiert:

- **Super Hermes** (`Cranot/super-hermes`) — kein offizielles Nous-
  Research-Projekt, sondern ein Drittanbieter-Skill-Paket. Nach
  Rücksprache: Policy erweitert (Drittanbieter erlaubt, wenn
  Kern-Dateien unangetastet bleiben, dokumentierte Erweiterungs-
  mechanismen genutzt werden, und die Erweiterung vollständig
  entfernbar ist) — manuelle Installation (ohne `install.sh`) folgt
  als eigener Schritt.
- **Hermes Agent Self-Evolution** (`NousResearch/hermes-agent-self-
  evolution`) — verifiziert als echtes, offizielles NousResearch-Repo
  (per GitHub-API-Abfrage bestätigt, nicht nur der URL geglaubt).
  Kostet laut README $2-10 pro Optimierungslauf — Installation/
  Konfiguration ohne echten Optimierungslauf folgt als eigener
  Schritt.

## Nächste Schritte

Ursache der Exa/`web_search`-Nichtnutzung klären (ggf. Upstream-Issue).
`hermes_christiane` auf denselben Stack (Grok, Exa, Honcho, lokale
Embeddings) bringen. Siehe [ROADMAP.md](ROADMAP.md).
