# Roadmap

## Sprint 1 — Foundation (abgeschlossen)

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

## Sprint 3 — Runtime Foundation (abgeschlossen)

Ziel: die Linux-Runtime vorbereiten, auf der Hermes später installiert
wird — weiterhin ohne Hermes, Hermes Workspace, OpenWebUI, mem0 oder
Humalike zu installieren.

- [x] Node.js 24.x (Active LTS) über das offizielle NodeSource-Repository
      installiert (nicht die Debian-Paketversion)
- [x] Python geprüft (System-Python 3.13.5), `python3-venv`/`python3-pip`
      installiert, `uv` systemweit installiert
- [x] Basispakete geprüft (`git`, `curl`, `wget`, `sqlite3`,
      `build-essential`, `ca-certificates`, `jq`, `tree`, `htop`, `zip`,
      `unzip`) — bereits vollständig vorhanden
- [x] Reale Systemstruktur angelegt: `/opt/companion/{skills,tools,templates,shared}`,
      `/etc/companion/`, `/srv/companion/`, `/var/log/companion/`,
      `/var/backups/companion/`
- [x] Zwei Companion-Benutzer angelegt: `hermes_hugo`, `hermes_christiane`
      (Home unter `/srv/companion/`, Passwort-Login gesperrt,
      Begründung: [ADR 0002](ADR/0002-companion-user-home-under-srv.md))
- [x] `INSTALL.md`, `ARCHITECTURE.md`, `CHANGELOG.md`, `README.md` auf
      den tatsächlichen Stand gebracht

Ausdrücklich nicht Teil dieses Sprints: Hermes, Hermes Workspace,
OpenWebUI, mem0, Humalike, systemd-Services, Workspaces, Personas,
Memory-Engine, Skills, SQLite-Datenbanken.

## Sprint 4 — Native Hermes Installation (abgeschlossen)

Ziel: genau ein produktionsfähiger Hermes-Agent, ausschließlich für
`hermes_hugo`. `hermes_christiane` folgt erst nach erfolgreichem
Abschluss. Nur CLI, API Server, Gateway, Profil — keine Desktop-Version,
keine experimentellen Features, kein systemd, kein Workspace-Umbau,
keine zusätzlichen Skills, kein MCP, kein mem0/Humalike.

- [x] Hermes Agent nativ installiert für `hermes_hugo` über den
      offiziellen Installer (v0.19.0, Install-Methode `git`)
- [x] `ripgrep`, `ffmpeg` nachinstalliert (vom Installer selbst
      vorgeschlagen); Playwright/Chromium-Setup versucht, aber
      fehlgeschlagen und bewusst nicht weiter verfolgt (außerhalb des
      Scopes CLI/API Server/Gateway/Profil)
- [x] Config auf Schema-Version 33 migriert, ohne API-Keys/Credentials
- [x] CLI, API Server und Gateway manuell verifiziert (siehe
      [ARCHITECTURE.md](ARCHITECTURE.md#hermes-installation-sprint-4)),
      danach sauber gestoppt — kein systemd-Service
- [x] Standard-Skills (65 builtin) und Standard-MCP-Katalog (0
      konfiguriert, 4 verfügbar) beobachtet, nicht verändert
- [x] Workspace-Struktur ausschließlich beobachtet, nicht angepasst
- [x] `docs/hermes/` um reale Beobachtungen ergänzt, klar getrennt von
      der Doku-Recherche aus Sprint 2 (inkl. mehrerer entdeckter
      Doku-vs-Realität-Abweichungen, siehe
      [docs/hermes/OPEN_QUESTIONS.md](docs/hermes/OPEN_QUESTIONS.md))
- [x] `INSTALL.md`, `ARCHITECTURE.md`, `CHANGELOG.md` aktualisiert

Ausdrücklich nicht Teil dieses Sprints: Modell-/Provider-Konfiguration,
Personas, Memory-Engine (mem0/Humalike), zusätzliche Skills, MCP-Server,
systemd, Hermes-Installation für `hermes_christiane`.

## Sprint 5+ — geplant (noch nicht begonnen)

Grobe, unverbindliche Reihenfolge — Details folgen jeweils im Sprint:

- Modell-/Provider-Entscheidung (ADR) für `hermes_hugo`
- Hermes-Installation für `hermes_christiane` nach demselben Muster
- OpenWebUI
- mem0-Alternative (falls über Hermes' eingebaute Provider-Plugins
  hinaus benötigt — siehe `docs/hermes/MEMORY.md`)
- Humalike
- Produktive Skills-/MCP-Nutzung (bereits durch Hermes abgedeckt, siehe
  `docs/hermes/SKILLS.md`, `docs/hermes/MCP.md`)
- systemd-Services (`hermes gateway install`), sobald ein Profil/Nutzer
  produktiv laufen soll

Diese Liste ist eine Absichtserklärung, kein Zeitplan.
