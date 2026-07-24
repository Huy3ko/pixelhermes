# PixelHermes

Eigene, private Infrastruktur für einen selbst gehosteten, orchestrierten
Assistenz-Stack. Dieses Repository ist die Single Source of Truth für das
gesamte PixelHermes-Projekt: Struktur, Dokumentation, Konfiguration und
später die Infrastructure-as-Code-Definitionen.

## Status

**Sprint 1: Foundation** — es existiert ausschließlich das
Infrastruktur-Fundament dieses Repositories.

Ausdrücklich **nicht** Teil dieses Sprints:

- Hermes, Hermes Workspace, OpenWebUI, mem0, Humalike, MCP, Skills
- Workspaces, Agenten, Benutzer, Datenbanken
- jegliche Anwendungen oder Services
- jegliche Änderungen am Linux-System

Details siehe [ROADMAP.md](ROADMAP.md).

## Architekturprinzipien

Siehe [ARCHITECTURE.md](ARCHITECTURE.md) für die vollständige
Begründung. Kurzfassung:

- Upstream First
- Git First
- Infrastructure as Code
- Workspace First
- Self Hosted
- Minimalistisch
- Eine Runtime
- Keine unnötigen Abhängigkeiten
- Python als Tool-Runtime
- Hermes orchestriert — LLM denkt — Tools arbeiten
- Dokumente bleiben lokal

## Repository-Struktur

Siehe [PROJECT_STRUCTURE.md](PROJECT_STRUCTURE.md) für die vollständige,
kommentierte Übersicht.

```
pixelhermes/
├── README.md
├── ROADMAP.md
├── ARCHITECTURE.md
├── INSTALL.md
├── CHANGELOG.md
├── PROJECT_STRUCTURE.md
├── ADR/            Architecture Decision Records
├── docs/
├── configs/
├── scripts/
├── systemd/
├── templates/
├── assets/
└── backups/
```

Die spätere Linux-Systemstruktur (`/opt/companion`, `/srv/companion`,
`/etc/companion`, `/var/log/companion`, `/var/backups/companion`) ist in
[ARCHITECTURE.md](ARCHITECTURE.md) geplant, aber in diesem Sprint noch
nicht angelegt.

## Workspace

Die Workspace-Struktur wird bewusst **nicht** in diesem Sprint
definiert. Sie wird erst nach der Installation von Hermes Workspace
analysiert und anschließend möglichst nah am Upstream übernommen (siehe
[ARCHITECTURE.md](ARCHITECTURE.md), "Workspace First").

## Weiterführende Dokumente

- [ROADMAP.md](ROADMAP.md) — Sprintplan
- [ARCHITECTURE.md](ARCHITECTURE.md) — Zielarchitektur & Prinzipien
- [PROJECT_STRUCTURE.md](PROJECT_STRUCTURE.md) — Verzeichnisübersicht
- [INSTALL.md](INSTALL.md) — nachvollziehbare Aufbauschritte
- [CHANGELOG.md](CHANGELOG.md) — Änderungshistorie
- [ADR/](ADR/) — Architecture Decision Records
