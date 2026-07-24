# PixelHermes

Eigene, private Infrastruktur für einen selbst gehosteten, orchestrierten
Assistenz-Stack. Dieses Repository ist die Single Source of Truth für das
gesamte PixelHermes-Projekt: Struktur, Dokumentation, Konfiguration und
später die Infrastructure-as-Code-Definitionen.

## Status

**Sprint 4: Native Hermes Installation** — Repository-Fundament
(Sprint 1), Hermes-Architektur-Recherche (Sprint 2, siehe
[docs/hermes/](docs/hermes/)), reale Linux-Runtime (Sprint 3) und ein
nativ installierter, unveränderter Hermes-Agent für `hermes_hugo`
(CLI, API Server, Gateway, Default-Profil) sind vorhanden.

Ausdrücklich **weiterhin nicht** installiert/angelegt:

- OpenWebUI, mem0, Humalike
- Modell-/Provider-Credentials, Personas, Memory-Konfiguration
- Zusätzliche (Nicht-Hermes-eigene) Skills, konfigurierte MCP-Server
- systemd-Services
- Alles für `hermes_christiane` außer Konto und leerem Home-Verzeichnis

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

Die Linux-Systemstruktur (`/opt/companion`, `/srv/companion`,
`/etc/companion`, `/var/log/companion`, `/var/backups/companion`) ist
seit Sprint 3 real angelegt — siehe
[ARCHITECTURE.md](ARCHITECTURE.md#systemstruktur) für Details und
[INSTALL.md](INSTALL.md) für die genauen Schritte.

## Workspace

Wird nicht im Repository nachgebaut. Hermes bringt seine eigene
Workspace-Konvention mit (`~/.hermes/`), siehe
[docs/hermes/WORKSPACE.md](docs/hermes/WORKSPACE.md) — dort seit
Sprint 4 inklusive der real beobachteten Struktur unter
`/srv/companion/hermes_hugo/.hermes/`. Bewusst nicht verändert oder
erweitert, nur beobachtet. `/srv/companion/hermes_christiane/` hat
weiterhin keinen Workspace-Inhalt.

## Weiterführende Dokumente

- [ROADMAP.md](ROADMAP.md) — Sprintplan
- [ARCHITECTURE.md](ARCHITECTURE.md) — Zielarchitektur & Prinzipien
- [PROJECT_STRUCTURE.md](PROJECT_STRUCTURE.md) — Verzeichnisübersicht
- [INSTALL.md](INSTALL.md) — nachvollziehbare Aufbauschritte
- [CHANGELOG.md](CHANGELOG.md) — Änderungshistorie
- [ADR/](ADR/) — Architecture Decision Records
