# PixelHermes

Eigene, private Infrastruktur für einen selbst gehosteten, orchestrierten
Assistenz-Stack. Dieses Repository ist die Single Source of Truth für das
gesamte PixelHermes-Projekt: Struktur, Dokumentation, Konfiguration und
später die Infrastructure-as-Code-Definitionen.

## Status

**Phase Y: Honcho vollständig entfernt** — Repository-Fundament
(Sprint 1), Hermes-Architektur-Recherche (Sprint 2, siehe
[docs/hermes/](docs/hermes/)), reale Linux-Runtime (Sprint 3), ein
nativ installierter Hermes-Agent für `hermes_hugo` (Sprint 4),
produktiv konfiguriert mit Grok als LLM und Exa als Suche (Sprint 6),
ein selbstgehosteter Companion-Stack (Sprint 7, mittlerweile größtenteils
historisch — siehe unten), Phase 8.1 ein unverändertes, offizielles
OpenWebUI über Hermes' eigenen OpenAI-kompatiblen API-Server verbunden
([docs/hermes/OPENWEBUI.md](docs/hermes/OPENWEBUI.md)), ein eigenes
Tracing-Plugin (Phase 8.2-ish, siehe
[docs/hermes/TRACING.md](docs/hermes/TRACING.md)) und seit Phase X
`mnemosyne-hermes` als aktiver Memory-Provider anstelle von Honcho —
real getestet (12 Ende-zu-Ende-Tests) und mit gemessenem
Performance-Vergleich. Honcho wurde seit Phase Y **vollständig
entfernt** (Pakete, Datenbank, systemd-Dienste, Systembenutzer) — nur
PostgreSQL/Redis-Serverpakete selbst blieben (leer, ohne Honcho-Daten)
auf Nutzerentscheidung hin installiert. Details: [docs/hermes/MNEMOSYNE.md](docs/hermes/MNEMOSYNE.md)
und [ADR 0008](ADR/0008-mnemosyne-replaces-honcho.md).

Ausdrücklich **weiterhin nicht** eingerichtet:

- mem0
- Humalike (geprüft, bewusst abgelehnt — kostenpflichtiger externer
  Cloud-Dienst, widerspricht "Self Hosted")
- Weitere Cloud-LLM-/Such-/Embedding-Provider (kein OpenRouter, kein
  Ollama, kein OpenAI, kein Google Gemini)
- Zusätzliche (Nicht-Hermes-eigene) Skills, konfigurierte MCP-Server
- Personas, eigene Hermes-Workspace-Konfiguration
- Caddy, HTTPS, DuckDNS, Reverse-Proxy, Mehrbenutzerbetrieb in
  OpenWebUI
- Feature-Tests durch OpenWebUI (Uploads, Tools, Memory, Workspace,
  Sessions, Skills, Curator) — nächste Phase
- Hermes-Installation für `hermes_christiane` (Konto existiert bereits;
  Mnemosyne/Embedding-Server sind bereits für spätere Mitnutzung ausgelegt)

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
