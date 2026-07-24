# Changelog

Format angelehnt an [Keep a Changelog](https://keepachangelog.com/).
Commits folgen [Conventional Commits](https://www.conventionalcommits.org/).

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
