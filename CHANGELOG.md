# Changelog

Format angelehnt an [Keep a Changelog](https://keepachangelog.com/).
Commits folgen [Conventional Commits](https://www.conventionalcommits.org/).

## [Unreleased] - 2026-07-27 - Google Workspace Skill recherchiert

### Added

- `docs/hermes/GOOGLE_WORKSPACE.md`: vollständige Recherche zum
  offiziellen `google-workspace`-Skill (bereits einer der 65 builtin
  Skills, kein MCP, kein zusätzliches Plugin) — OAuth-/PKCE-Flow,
  Redirect-URI `http://localhost:1`, sechs benötigte Google-Cloud-APIs,
  dienstspezifische Scopes, Token-Ablage unter `~/.hermes/`
- `OVERVIEW.md`, `ROADMAP.md`, `README.md` um Verweis ergänzt

### Explicitly not done

- Google-Cloud-OAuth-Client-Erstellung, Registrierung auf dem realen
  `hermes_hugo`-Host, Login-Link-Erzeugung, Autorisierung,
  Funktionsverifikation — erfordert eine Google-Kontoanmeldung des
  Nutzers und realen Host-Zugriff, beides außerhalb dieser
  Repository-only-Sitzung. Kein fingierter Login-Link ausgegeben.

Details: [docs/hermes/GOOGLE_WORKSPACE.md](docs/hermes/GOOGLE_WORKSPACE.md).

## [Phase Y] - 2026-07-25 - Honcho vollständig entfernt

### Removed

- systemd-Units `companion-honcho-api.service`, `companion-honcho-deriver.service`
- PostgreSQL-Datenbank `honcho` + Rolle `honcho` (`DROP DATABASE`, `DROP ROLE`)
- Systembenutzer/-gruppe `honcho`, Home-Verzeichnis `/opt/companion/honcho`
  (~1,2 GB: Git-Checkout, uv-venv, uv-Cache) — vorher als tar-Backup
  gesichert (Konfiguration/Code, ohne venv/Cache)
- `~/.hermes/honcho.json`, inaktive `HONCHO_API_KEY`-Template-Zeile in
  `~/.hermes/.env`

### Kept (bewusst nicht entfernt)

- PostgreSQL- und Redis-Serverpakete selbst (auf Nutzerentscheidung hin
  installiert gelassen, ohne Honcho-Daten) — nur Honchos Datenbank/
  Rolle wurde entfernt
- Hermes' eigenes gebündeltes `plugins/memory/honcho`-Plugin und die
  `honcho_ai`-Bibliothek in Hermes' venv — Teil der Hermes-Distribution,
  nicht unsere Installation
- Mnemosynes eigenes `core/importers/honcho.py` — Teil des
  Mnemosyne-Pakets

### Verified

- Vollständiges Audit vor jeder Löschung bestätigte exklusive
  Honcho-Zugehörigkeit jeder entfernten Ressource; Docker war nie
  installiert, kein Cron-Subsystem vorhanden, keine lokalen
  TinyLlama/`ctransformers`-Pakete in Honchos venv gefunden
- Nach der Entfernung: keine Honcho-Prozesse/-Services/-User/-DB/
  -Config/-Plugins; `hermes memory status` → `mnemosyne`; Mnemosyne,
  Workspace, OpenWebUI, Tracing, Skills real erneut getestet

Details: [docs/hermes/MNEMOSYNE.md](docs/hermes/MNEMOSYNE.md#honcho-entfernung-phase-y),
[ADR 0008 Update](ADR/0008-mnemosyne-replaces-honcho.md#update-phase-y-honcho-vollständig-entfernt).

## [Phase X] - 2026-07-25 - Mnemosyne ersetzt Honcho

### Added

- `mnemosyne-hermes` (Drittanbieter, `AxDSan/mnemosyne`, PyPI) offiziell
  installiert (`pip install`, eigene `entry_points`-Registrierung),
  `memory.provider` auf `mnemosyne` umgestellt — [ADR 0008](ADR/0008-mnemosyne-replaces-honcho.md),
  [docs/hermes/MNEMOSYNE.md](docs/hermes/MNEMOSYNE.md)
- `companion-tracing`-Plugin um eine `mnemosyne`-Tool-Kategorie
  erweitert (`mnemosyne_call_count` im Request-Summary)
- Alle 12 geforderten Ende-zu-Ende-Tests real durchgeführt und
  dokumentiert (Speichern, Cross-Session-Recall, Mehrfach-Fakten,
  Update, Workspace, OpenWebUI, Grok, Exa, Skills, Datei,
  Session-Historie)
- Realer Performance-Vergleich Mnemosyne vs. Honcho über das
  Tracing-Plugin und direkte systemd-/Prozess-Messung (RAM, CPU, Tool-
  und LLM-Latenz)

### Changed

- Honcho (`companion-honcho-api`/`-deriver`) gestoppt und `disabled` —
  **nicht deinstalliert**, vollständige Entfernung ist eine eigene,
  nachfolgende Phase

### Verified

- Duplicate-Registration-Frage (zwei `hermes plugins list`-Einträge)
  per Quellcode-Analyse (`plugins/memory/__init__.py`) und empirisch
  (Tool-Call-Trace + direkte SQLite-Abfrage der
  `working_memory`-Tabelle) geklärt: Hooks feuern nur einmal
- Kein offizieller Honcho→Mnemosyne-Migrationsweg vorhanden (Mnemosynes
  eigene `comparison.md` erwähnt Honcho nicht) — dokumentiert, keine
  Eigenentwicklung gebaut, Mnemosynes Speicher startete leer

### Known limitations

- Recall-Inkonsistenz bei Multi-Marker-Sammelabfragen trotz bestätigt
  vorhandenem Datenbankeintrag (siehe MNEMOSYNE.md)
- Exa/`web_search` weiterhin nicht zuverlässig aufgerufen — bereits
  vorher bekannt, keine Regression durch diese Migration

## [Tracing] - 2026-07-25 - Request/Step Observability

Instrumentierung und Messung only — keine Optimierungen, keine
Prompt-/Provider-/Memory-/Config-Änderungen.

### Added

- Rein additives Hermes-Plugin `companion-tracing`
  (`~/.hermes/plugins/companion-tracing/`, keine Hermes-Kerndatei
  verändert) misst Request-ID, Session-ID, Schrittdauern (Tool-Calls
  klassifiziert nach Honcho/Exa/Workspace/Skills, LLM-Aufrufe mit
  Hermes' eigenen exakten `api_duration`/`usage`-Werten) über die
  offiziellen Plugin-Hooks
- Geprüft und verworfen: Hermes' gebündeltes Langfuse-Plugin nutzt
  dieselben Hooks, verlangt aber Docker zum Self-Hosting — Konflikt
  mit dem projektweiten Docker-Verbot ([ADR 0007](ADR/0007-local-tracing-plugin-not-langfuse.md))
- JSON-Lines-Traces + lesbare Zusammenfassung unter
  `~/.hermes/logs/tracing/`
- Real getestet an drei unterschiedlichen Anfragetypen (Tool-Call,
  Honcho-lastige Anfrage über 3 LLM-Runden, Super-Hermes-Skill) — dabei
  einen echten Bug im eigenen Plugin-Code gefunden und vor der
  Dokumentation behoben (verwaister State-Eintrag durch
  session_id/turn_id-Key-Mismatch)
- Realer Fund: ein einzelner `honcho_reasoning`-Aufruf dominierte eine
  gemessene Anfrage (23,6s von 54,6s Gesamtdauer) stärker als jeder
  einzelne LLM-Call
- `docs/hermes/TRACING.md` neu, mit ehrlicher Abgrenzung: TTFT,
  Streaming-Dauer, RAG, Skill-Selection und Tool-Planning sind ohne
  Hermes-Kernpatch nicht separat messbar — als solche dokumentiert,
  nicht vorgetäuscht

## [Phase 8.1] - 2026-07-25 - OpenWebUI ↔ Hermes Agent API

### Added

- Hermes API-Server aktiviert für `hermes_hugo` (`API_SERVER_ENABLED`,
  `API_SERVER_KEY` — echte Env-Vars, direkt in `.env` geschrieben, kein
  `config.yaml`-Schlüssel, gleicher Fehlerort wie `HONCHO_BASE_URL` in
  Sprint 7)
- Hermes-Gateway erstmals dauerhaft betrieben, offiziell über `hermes
  gateway install` (`systemd --user` + `loginctl enable-linger`), nicht
  nur testweise gestartet und gestoppt
- OpenWebUI unverändert per offiziellem `pip install open-webui`
  installiert (dedizierter Systembenutzer `openwebui`,
  `/opt/companion/openwebui/`, Python 3.11 via `uv venv` — 3.13 wird
  von OpenWebUI nicht unterstützt), als systemd-Service, gebunden an
  `127.0.0.1:8080`
- Einziger Benutzer (Hugo) per offiziellem Headless-Env-Var-Pfad
  angelegt; Signup danach automatisch deaktiviert (OpenWebUI-eigenes
  Verhalten)
- OpenWebUI über den generischen "OpenAI"-Verbindungstyp mit Hermes'
  `/v1/chat/completions` verbunden — offiziell dokumentierter
  OpenWebUI-Hermes-Integrationsweg, keine Kompatibilitätsschicht
- Ende-zu-Ende real verifiziert: identischer Agent-Pfad für direkten
  Hermes-Zugriff und OpenWebUI-vermittelten Zugriff (Prompt-Token-
  Parität, `platform=api_server` in Hermes' `agent.log`)
- `docs/hermes/OPENWEBUI.md` neu; `ADR 0006` (OpenWebUI über Hermes'
  OpenAI-Endpoint statt einer nicht existierenden separaten "Agent
  API"); Top-Level-Docs aktualisiert

### Corrected premise (research before implementation)

- Die ursprüngliche Annahme einer von der generischen OpenAI-API
  getrennten "Hermes Agent API" wurde per Quellcode-Prüfung widerlegt:
  `/v1/chat/completions` instanziiert dieselbe `AIAgent`-Klasse wie
  CLI und Gateway-Plattformen — es gibt nur eine API. Vergleich mit
  OpenClaws äquivalentem Design bestätigte dasselbe Muster dort.

### Not included (bewusst, laut Vorgabe)

- Caddy, HTTPS, DuckDNS, Reverse-Proxy
- Mehrbenutzerbetrieb in OpenWebUI (nur Hugo)
- Feature-Tests durch OpenWebUI (Uploads, Tools, Memory, Workspace,
  Sessions, Skills, Curator) — nächste Phase

## [Sprint 7] - 2026-07-24 - Companion Foundation

Erste Erweiterung über reines Hermes hinaus: ein selbstgehosteter
Companion-Stack (Honcho + lokaler Embedding-Server), plus geprüfte
(teils abgelehnte) Drittanbieter-Erweiterungen.

### Added

- PostgreSQL 17 + pgvector, Redis installiert (Debian-Pakete)
- Dedizierte Systembenutzer `honcho`, `embeddings` unter
  `/opt/companion/` — erste geteilte Infrastrukturdienste des Projekts
  ([ADR 0003](ADR/0003-shared-services-under-opt-companion.md))
- Lokaler, vollständig selbstgehosteter Embedding-Server
  (`llama.cpp` + `nomic-embed-text-v1.5`, 768-dim, API-Key-geschützt,
  systemd-Service `companion-embeddings.service`) — OpenAI und Google
  Gemini als Cloud-Alternativen geprüft und bewusst verworfen
  ([ADR 0004](ADR/0004-local-embedding-server-for-honcho.md))
- Honcho selbstgehostet (`plastic-labs/honcho`), Text-Generierung über
  den bestehenden xAI/Grok-Key, Embeddings über den lokalen Server;
  zwei systemd-Services (`companion-honcho-api`,
  `companion-honcho-deriver`); Datenbank-Migration inkl. einem realen,
  dokumentierten Vektordimensions-Fehler und dessen offiziellem Fix
- Hermes mit Honcho verbunden (`memory.provider: honcho`,
  `~/.hermes/honcho.json`); Ende-zu-Ende verifiziert inkl.
  **Cross-Session-Recall** (neue Session erinnert sich korrekt an eine
  in einer früheren Session gespeicherte Nutzerpräferenz)
- `docs/hermes/HONCHO.md`, `docs/hermes/COMPANION_STACK.md` neu;
  `docs/hermes/OVERVIEW.md`, `ARCHITECTURE.md`, `INSTALL.md`,
  `README.md` aktualisiert

### Also added — evaluated and installed under a precisified policy

- **Humalike** — offiziell dokumentiertes Hermes-Plugin, aber
  abhängig von einem kostenpflichtigen externen Cloud-Dienst; bewusst
  abgelehnt, widerspricht "Self Hosted". Nichts installiert.
- **Super Hermes** (`Cranot/super-hermes`) — Drittanbieter-Skill-Paket
  (5 Skills, 7 Prisms), kein offizielles Nous-Research-Projekt.
  Vollständig geprüft (Inhalt, kein Core-File-Zugriff), manuell
  installiert (kein `install.sh` ausgeführt), real getestet
  (`/prism-scan`, 19 echte Tool-Calls), Entfernbarkeit real verifiziert.
  Führte zur Präzisierung der "nur offizielle Erweiterungen"-Regel
  ([ADR 0005](ADR/0005-third-party-extension-policy.md))
- **Hermes Agent Self-Evolution** (`NousResearch/hermes-agent-self-
  evolution`) — als echtes offizielles Repo verifiziert (GitHub-API,
  nicht nur URL). Installiert, konfiguriert, per Quellcode-Analyse
  bestätigt kostenloser `--dry-run` ausgeführt (0 API-Calls, 0 Dateien
  erzeugt) — kein echter, kostenpflichtiger Optimierungslauf ($2-10
  laut README)

### Not included (bewusst)

- OpenWebUI, mem0
- Weitere Cloud-Provider (OpenRouter, Ollama, OpenAI, Google Gemini)
- Hermes-Installation für `hermes_christiane`
- Personas, eigene Hermes-Workspace-Konfiguration

## [Sprint 6] - 2026-07-24 - Productive Runtime

Sprint 5 (geplantes internes Hermes-Audit) wurde übersprungen — Plan
verworfen, nicht ausgeführt, keine Artefakte.

### Added

- Grok (xAI) als einziges LLM konfiguriert für `hermes_hugo`
  (`XAI_API_KEY`, `provider: xai`, `grok-build-0.1`) — real verifiziert:
  Chat, Coding, Tool-Calling, alles per `agent.log` bestätigt
- Exa als einzige Suchmaschine konfiguriert (`EXA_API_KEY`,
  `web.backend`/`web.search_backend`/`web.extract_backend: exa`)
- Reale Workspace-Aufgaben in `~/hermes-notes/`: Git-Repo mit README
  und Commits, ein Rollout-Planungsdokument für `hermes_christiane`,
  eine werkzeugfreie Reasoning-Aufgabe
- Session-Verwaltung real getestet (list/export/optimize-storage/repair
  funktionierend; `session_search` bestätigt korrekt)
- Curator-Status/-Konfiguration beobachtet (nichts ausgelöst/verändert)
- `docs/hermes/PRODUCTIVE_RUNTIME.md`, `docs/hermes/ASSESSMENT.md` neu
  angelegt; `docs/hermes/MODELS.md`, `docs/hermes/OVERVIEW.md`
  aktualisiert; `ARCHITECTURE.md`, `INSTALL.md`, `ROADMAP.md`
  aktualisiert

### Found (real, unresolved — dokumentiert statt "gefixt")

- **Exa/`web_search` wird vom Modell nicht real aufgerufen** trotz
  korrekter Konfiguration — es erzeugt stattdessen plausible, aber
  erfundene "Tool-Ergebnisse". Dreifach reproduziert (zwei Modelle),
  per `agent.log` (`tool_turns=0`) zweifelsfrei belegt; kein
  Hermes-eigenes Beispiel im Quellcode als Ursache gefunden
- `hermes sessions archive --older-than` traf in mehreren Tests nie,
  obwohl sichtlich ältere Sessions vorhanden waren
- Realer, unbehandelter Absturz (`PermissionError` auf `.git`-Suche),
  wenn Hermes aus einem für den Zielbenutzer unlesbaren
  Arbeitsverzeichnis aufgerufen wird
- Drei unterschiedliche, nicht in Einklang zu bringende Skill-Zählungen
  (69 Installer-Log, 65 `skills list`, 68 `curator status`) auf
  derselben unveränderten Installation

### Not included (bewusst)

- OpenWebUI, OpenRouter, Ollama, mem0, Humalike
- Zusätzliche Skills, konfigurierte MCP-Server, systemd-Services
- Personas, eigene Memory-/Workspace-Konfiguration, eigene Datenbanken
- Hermes-Installation für `hermes_christiane`

## [Sprint 4] - 2026-07-24 - Native Hermes Installation

Erste installierte Anwendung im Projekt — ausschließlich für den
Benutzer `hermes_hugo`, `hermes_christiane` folgt erst nach
erfolgreichem Abschluss.

### Added

- Hermes Agent nativ installiert für `hermes_hugo` über den offiziellen
  Installer (`hermes-agent.nousresearch.com/install.sh`), Version 0.19.0
  (2026.7.20), Install-Methode `git`
- `ripgrep`, `ffmpeg` systemweit nachinstalliert (vom Hermes-Installer
  selbst als fehlend gemeldet)
- Config auf Schema-Version 33 migriert (`hermes config migrate`), ohne
  API-Keys/Anmeldedaten einzutragen
- CLI, API Server (`hermes serve`) und Gateway (`hermes gateway run`)
  jeweils manuell gestartet, verifiziert (Prozess läuft, HTTP 200 auf
  `/api/status`, Profil- und Config-Erkennung über API bestätigt) und
  wieder sauber gestoppt
- 65 standardmäßige builtin-Skills und der Stand des offiziellen
  MCP-Katalogs (0 konfigurierte Server, 4 verfügbare) beobachtet und
  dokumentiert, ohne etwas davon zu verändern
- `docs/hermes/` um reale Beobachtungen ergänzt (INSTALLATION,
  WORKSPACE, SKILLS, MCP, ARCHITECTURE, OPEN_QUESTIONS), klar getrennt
  von der Doku-Recherche aus Sprint 2 — inklusive mehrerer entdeckter
  Abweichungen zwischen offizieller Doku und realem Verhalten (u. a.
  `web`/`pty`-Extras bereits im Standardinstall enthalten, unbekannte
  Toolset-Referenzen `hermes-teams`/`hermes-google_chat` im
  Default-Config-Template, Skill-Zähldiskrepanz 69 vs. 65)
- `INSTALL.md`, `ARCHITECTURE.md` um die tatsächlich durchgeführten
  Schritte und Verifikationsergebnisse ergänzt

### Not included (bewusst)

- Modell-/Provider-Konfiguration (kein API-Key hinterlegt)
- Zusätzliche Skills, konfigurierte MCP-Server
- systemd-Service (Gateway/API Server nur manuell gestartet und wieder
  gestoppt)
- Workspace-Anpassungen jeder Art (nur beobachtet, nicht verändert)
- Playwright/Chromium-Browser-Tools (Installation fehlgeschlagen, nicht
  weiter verfolgt — außerhalb des Phase-4-Scopes)
- Hermes-Installation für `hermes_christiane`

## [Sprint 3] - 2026-07-24 - Runtime Foundation

Erste echten Änderungen am Server selbst (bisher nur am Repository).

### Added

- Node.js 24.x (Active LTS) über das offizielle NodeSource-Repository
  installiert (`v24.18.0`), bewusst nicht das Debian-Paket
- `python3-venv`, `python3-pip` installiert; System-Python 3.13.5
  bestätigt
- `uv` 0.11.32 systemweit unter `/usr/local/bin` installiert (offizieller
  Installer)
- Basispakete geprüft (`git`, `curl`, `wget`, `sqlite3`,
  `build-essential`, `ca-certificates`, `jq`, `tree`, `htop`, `zip`,
  `unzip`) — bereits vollständig vorhanden
- Reale Systemstruktur angelegt: `/opt/companion/{skills,tools,templates,shared}`,
  `/etc/companion/`, `/srv/companion/`, `/var/log/companion/`,
  `/var/backups/companion/`
- Zwei Linux-Benutzer angelegt: `hermes_hugo` (UID 1001),
  `hermes_christiane` (UID 1002), Home-Verzeichnis unter
  `/srv/companion/<name>`, Passwort-Login gesperrt
- [ADR 0002](ADR/0002-companion-user-home-under-srv.md): Home-Verzeichnis
  der Companion-Benutzer bewusst unter `/srv/companion/` statt `/home/`
- `INSTALL.md` und `ARCHITECTURE.md` um die tatsächlich durchgeführten
  Schritte und die reale Systemstruktur ergänzt

### Not included (bewusst)

- Hermes, Hermes Workspace, OpenWebUI, mem0, Humalike
- systemd-Services, Hermes-Workspaces, Personas, Memory-Engine, Skills
- SQLite-Datenbanken (Paket `sqlite3` ist nur das CLI-Tool, keine
  angelegte Datenbank)

## [Sprint 2] - 2026-07-24 - Hermes Architecture Research

### Added

- `docs/hermes/` mit vollständiger, quellenbelegter Recherche zur
  offiziellen Hermes-Agent-Dokumentation (Nous Research):
  `OVERVIEW.md`, `INSTALLATION.md`, `ARCHITECTURE.md`, `WORKSPACE.md`,
  `SKILLS.md`, `MCP.md`, `MEMORY.md`, `PERSONAS.md`, `DEPLOYMENT.md`,
  `MODELS.md`, `BEST_PRACTICES.md`, `OPEN_QUESTIONS.md`
- Für jedes Kapitel ein PixelHermes-Mapping (was Hermes bereits
  übernimmt, was nicht selbst entwickelt werden muss, was direkt passt,
  sinnvolle spätere Erweiterungen, was unverändert übernommen wird)
- Alle in der offiziellen Doku nicht geklärten Punkte gesammelt in
  `docs/hermes/OPEN_QUESTIONS.md`, statt als Vermutung dargestellt

### Not included (bewusst)

- Installation, Konfiguration oder Ausführung von Hermes
- Workspace-Anpassungen, Agenten, Benutzer, Services, Datenbanken
- Jegliche Änderungen am Linux-System

## [Sprint 1] - 2026-07-24 - Foundation

### Added

- `.gitignore` für Secrets, Logs und lokale Backup-Archive
- Verzeichnisstruktur: `docs/`, `configs/`, `scripts/`, `systemd/`,
  `templates/`, `assets/`, `backups/`, `ADR/`
- Kerndokumentation: `README.md` (vervollständigt), `ROADMAP.md`,
  `ARCHITECTURE.md`, `INSTALL.md`, `PROJECT_STRUCTURE.md`,
  `CHANGELOG.md`
- Architekturprinzipien dokumentiert: Upstream First, Git First,
  Infrastructure as Code, Workspace First, Self Hosted, Minimalistisch,
  Eine Runtime, Keine unnötigen Abhängigkeiten, Python als Tool-Runtime,
  "Hermes orchestriert – LLM denkt – Tools arbeiten", Dokumente bleiben
  lokal
- Geplante Linux-Systemstruktur dokumentiert (`/opt/companion`,
  `/srv/companion`, `/etc/companion`, `/var/log/companion`,
  `/var/backups/companion`) — bewusst nicht angelegt
- Architecture-Decision-Record-Prozess unter `ADR/` etabliert

### Not included (bewusst)

- Hermes, Hermes Workspace, OpenWebUI, mem0, Humalike, MCP, Skills
- Workspaces, Agenten, Benutzer, Datenbanken
- Anwendungen, Services, Linux-Systemänderungen
