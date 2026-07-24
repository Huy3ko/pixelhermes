# Changelog

Format angelehnt an [Keep a Changelog](https://keepachangelog.com/).
Commits folgen [Conventional Commits](https://www.conventionalcommits.org/).

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
