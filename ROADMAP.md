# Roadmap

## Sprint 1 — Foundation (aktuell)

Ziel: ausschließlich das Infrastruktur-Fundament dieses Repositories.

- [x] Bestehende Repository-Struktur geprüft und erweitert
- [x] `.gitignore` angelegt
- [x] Kerndokumentation: `README.md`, `ROADMAP.md`, `ARCHITECTURE.md`,
      `INSTALL.md`, `CHANGELOG.md`, `PROJECT_STRUCTURE.md`
- [x] `ADR/` mit Architecture-Decision-Record-Prozess angelegt
- [x] Verzeichnisstruktur `docs/`, `configs/`, `scripts/`, `systemd/`,
      `templates/`, `assets/`, `backups/`
- [x] Architekturprinzipien dokumentiert (Upstream First, Git First,
      Infrastructure as Code, Workspace First, Self Hosted,
      Minimalistisch, Eine Runtime, Python als Tool-Runtime, ...)
- [x] Spätere Linux-Systemstruktur dokumentiert (nicht angelegt)

Ausdrücklich nicht Teil dieses Sprints: Hermes, Hermes Workspace,
OpenWebUI, mem0, Humalike, MCP, Skills, Workspaces, Agenten, Benutzer,
Datenbanken, Services, Linux-Systemänderungen.

## Sprint 2 — geplant (noch nicht begonnen)

Möglicher Zuschnitt (wird beim Start des Sprints konkretisiert):

- Installation von Hermes Workspace
- Analyse der Upstream-Workspace-Struktur, Übernahme ins Repo
  (`templates/` bzw. neues `workspace/`-Verzeichnis)

## Sprint 3+ — geplant (noch nicht begonnen)

Grobe, unverbindliche Reihenfolge — Details folgen jeweils im Sprint:

- Hermes (Orchestrierung)
- OpenWebUI
- mem0
- Humalike
- MCP
- Skills
- Systemverzeichnisse, Systembenutzer, Services

Diese Liste ist eine Absichtserklärung, kein Zeitplan.
