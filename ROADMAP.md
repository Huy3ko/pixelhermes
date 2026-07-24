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

## Sprint 5 — Hermes Internal Audit (übersprungen)

Ein umfassendes, rein lesendes internes Hermes-Audit (Runtime, Workspace,
SQLite, Sessions, Skills, MCP, Provider-Layer, Datenfluss,
Architekturbewertung) wurde als Plan ausgearbeitet, aber vom Nutzer
zugunsten von Sprint 6 (Produktivbetrieb) verworfen — der Plan wurde
nie ausgeführt. Kein Code, keine Doku aus diesem Sprint entstanden.

## Sprint 6 — Productive Runtime (abgeschlossen)

Ziel: Hermes erstmals produktiv betreiben, ausschließlich für
`hermes_hugo`. Minimalistische Architektur: Grok (xAI) als einziges
LLM, Exa als einzige Suche, ansonsten ausschließlich native
Hermes-Mechanismen (Workspace, Memory, Curator, Session Search, Skills).
Kein OpenWebUI, kein OpenRouter, kein Ollama, kein mem0, kein Humalike,
keine zusätzlichen Skills, keine weiteren Benutzer, keine eigene
Personas-/Workspace-Struktur, keine eigenen Datenbanken.

- [x] Grok konfiguriert (`XAI_API_KEY`, `provider: xai`,
      `grok-build-0.1`) und real verifiziert: Chat, Coding, Tool-Calls
      (terminal/file) — alle über `agent.log` bestätigt
- [x] Exa konfiguriert (`EXA_API_KEY`, `web.backend`/`web.search_backend`/
      `web.extract_backend: exa`) — **kritischer, ungelöster Fund:**
      `web_search` wird vom Modell trotz korrekter Konfiguration nicht
      real aufgerufen (dreifach reproduziert, per Log widerlegt, kein
      Hermes-eigenes Beispiel als Ursache gefunden)
- [x] Reale Workspace-Aufgaben durchgeführt (Git-Repo + Commits,
      Planungsdokument, werkzeugfreie Reasoning-Aufgabe) — keine
      künstlichen Tests
- [x] Sessions dokumentiert: List/Search/Export/Optimize/Repair
      funktionieren; `archive --older-than` zeigte einen realen
      Nichttreffer-Befund; ein realer, unbehandelter Absturz bei
      Aufruf aus unlesbarem Arbeitsverzeichnis entdeckt
- [x] Curator nur beobachtet (Status, Konfiguration) — nichts
      ausgelöst oder verändert
- [x] Bewertung geschrieben (was ist gelöst / erweiterbar / besser
      unangetastet / Erweiterungspunkt / klare Upstream-Grenze) —
      mit offen benannter Einschränkung: basiert auf einer intensiven
      Sitzung, nicht auf mehrtägiger Nutzung wie ursprünglich verlangt
- [x] `docs/hermes/PRODUCTIVE_RUNTIME.md`, `docs/hermes/ASSESSMENT.md`
      neu angelegt; `docs/hermes/MODELS.md`, `OVERVIEW.md`,
      `ARCHITECTURE.md`, `INSTALL.md`, `CHANGELOG.md` aktualisiert

Ausdrücklich nicht Teil dieses Sprints: OpenWebUI, OpenRouter, Ollama,
mem0, Humalike, zusätzliche Skills, weitere Benutzer, Personas, eigene
Workspace-Struktur, eigene Datenbanken, systemd.

## Sprint 7 — Companion Foundation (abgeschlossen)

Ziel: den Companion-Stack um offiziell unterstützte Erweiterungen
ergänzen, ausschließlich self-hosted. Keine Forks, keine
Runtime-Patches an Hermes.

- [x] Humalike geprüft — bewusst abgelehnt (kostenpflichtiger externer
      Cloud-Dienst, widerspricht "Self Hosted")
- [x] PostgreSQL 17 + pgvector, Redis installiert
- [x] Lokaler Embedding-Server (llama.cpp + nomic-embed-text-v1.5,
      768-dim) als eigener systemd-Service — OpenAI und Google Gemini
      als Cloud-Alternativen geprüft und verworfen
      ([ADR 0004](ADR/0004-local-embedding-server-for-honcho.md))
- [x] Honcho selbstgehostet, als einziger externer Memory-Provider
      konfiguriert; Text-Generierung über bestehenden Grok-Key,
      Embeddings vollständig lokal
- [x] Hermes mit Honcho verbunden, Ende-zu-Ende verifiziert inkl.
      echtem Cross-Session-Recall
- [x] Erste geteilte Systemdienste des Projekts (`honcho`,
      `embeddings`) unter `/opt/companion/`
      ([ADR 0003](ADR/0003-shared-services-under-opt-companion.md))
- [x] `docs/hermes/HONCHO.md`, `docs/hermes/COMPANION_STACK.md` neu;
      Top-Level-Docs aktualisiert
- [~] Super Hermes (Drittanbieter-Skill-Paket) und Hermes Agent
      Self-Evolution (offizielles NousResearch-Repo, kostenpflichtig
      pro Lauf) geprüft — Installation/Konfiguration als separate,
      dokumentierte Schritte im Sprintverlauf

Ausdrücklich nicht Teil dieses Sprints: OpenWebUI, mem0, weitere
Cloud-Provider (OpenRouter, Ollama, OpenAI, Gemini),
Hermes-Installation für `hermes_christiane`, Personas.

## Sprint 8+ — geplant (noch nicht begonnen)

Grobe, unverbindliche Reihenfolge — Details folgen jeweils im Sprint:

- Ursache der Exa/`web_search`-Nichtnutzung klären (ggf. Upstream-Issue
  bei Nous Research melden) — vor jeder Aussage "Suche funktioniert
  produktiv"
- Mehrtägige tatsächliche Nutzung, um Sprint 6's Bewertung zu
  vervollständigen (inkl. eines echten Curator-Laufs)
- Hermes-Installation für `hermes_christiane` auf demselben Stack
  (Grok, Exa, Honcho, lokale Embeddings)
- OpenWebUI
- Produktive Skills-/MCP-Nutzung (bereits durch Hermes abgedeckt, siehe
  `docs/hermes/SKILLS.md`, `docs/hermes/MCP.md`)
- Multi-User-Ausbau, Persona-Layer

Diese Liste ist eine Absichtserklärung, kein Zeitplan.
