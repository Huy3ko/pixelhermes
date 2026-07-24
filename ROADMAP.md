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

## Sprint 2 — Hermes Architecture Research (abgeschlossen)

Ziel: Hermes Agent (Nous Research) anhand der offiziellen Dokumentation
vollständig verstehen, bevor irgendetwas installiert wird. Keine
Installation, keine Konfiguration, keine Workspace-Anpassungen, keine
Agenten, keine Benutzer — ausschließlich Analyse und Dokumentation.

- [x] Systematische Recherche über Core, Runtime, Skills, MCP,
      Deployment und Modelle anhand der offiziellen Doku
      (`hermes-agent.nousresearch.com/docs/`, `github.com/NousResearch/hermes-agent`)
- [x] `docs/hermes/` mit OVERVIEW, INSTALLATION, ARCHITECTURE, WORKSPACE,
      SKILLS, MCP, MEMORY, PERSONAS, DEPLOYMENT, MODELS, BEST_PRACTICES,
      OPEN_QUESTIONS angelegt
- [x] Für jedes Kapitel PixelHermes-Mapping dokumentiert (was Hermes
      übernimmt, was wir nicht selbst bauen müssen, was direkt passt,
      sinnvolle spätere Erweiterungen, was unverändert übernommen wird)
- [x] Offene, in der offiziellen Doku nicht geklärte Punkte gesammelt
      (`docs/hermes/OPEN_QUESTIONS.md`)

Erkenntnis: die zuvor als "erst nach Installation zu klärende"
Workspace-Struktur (`templates/` bzw. ein künftiges `workspace/`) ist
bereits vollständig dokumentiert — `~/.hermes/`. PixelHermes muss diese
Struktur nicht selbst entwerfen, siehe
[docs/hermes/WORKSPACE.md](docs/hermes/WORKSPACE.md).

## Sprint 3+ — geplant (noch nicht begonnen)

Grobe, unverbindliche Reihenfolge — Details folgen jeweils im Sprint:

- Modell-/Provider-Entscheidung (ADR)
- Installation von Hermes Agent nach `docs/hermes/INSTALLATION.md`
- OpenWebUI
- mem0-Alternative (falls über Hermes' eingebaute Provider-Plugins
  hinaus benötigt — siehe `docs/hermes/MEMORY.md`)
- Humalike
- Skills, MCP-Server (bereits durch Hermes abgedeckt, siehe
  `docs/hermes/SKILLS.md`, `docs/hermes/MCP.md` — hier nur produktive
  Nutzung/Konfiguration)
- Systemverzeichnisse, Systembenutzer, Services

Diese Liste ist eine Absichtserklärung, kein Zeitplan.
