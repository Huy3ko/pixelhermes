# Changelog

Format angelehnt an [Keep a Changelog](https://keepachangelog.com/).
Commits folgen [Conventional Commits](https://www.conventionalcommits.org/).

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
