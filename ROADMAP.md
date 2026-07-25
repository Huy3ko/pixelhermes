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
- [x] Super Hermes (Drittanbieter-Skill-Paket, `Cranot/super-hermes`)
      geprüft und installiert — Policy präzisiert statt aufgeweicht
      ([ADR 0005](ADR/0005-third-party-extension-policy.md))
- [x] Hermes Agent Self-Evolution (offizielles NousResearch-Repo,
      GitHub-API-verifiziert) installiert und konfiguriert; per
      Quellcode bestätigter kostenloser `--dry-run` ausgeführt, kein
      echter (kostenpflichtiger) Optimierungslauf

Ausdrücklich nicht Teil dieses Sprints: OpenWebUI, mem0, weitere
Cloud-Provider (OpenRouter, Ollama, OpenAI, Gemini),
Hermes-Installation für `hermes_christiane`, Personas.

## Phase 8.1 — OpenWebUI ↔ Hermes Agent API (abgeschlossen)

Ziel: unverändertes, offizielles OpenWebUI über Hermes' eigene
OpenAI-kompatible API verbinden, nur ein Benutzer (Hugo), kein Caddy/
HTTPS/DuckDNS/Reverse-Proxy. Stop nach bestätigter Verbindung — keine
Feature-Tests.

- [x] Prämisse "separate Hermes Agent API" per Quellcode geprüft und
      widerlegt (`/v1/chat/completions` = dieselbe `AIAgent`-Klasse wie
      CLI/Gateway) — Vergleich mit OpenClaw bestätigte dasselbe Muster
      dort ([ADR 0006](ADR/0006-openwebui-via-hermes-openai-endpoint.md))
- [x] Hermes-API-Server aktiviert, Gateway erstmals dauerhaft betrieben
      (offiziell `hermes gateway install`)
- [x] OpenWebUI offiziell installiert (pip, Python 3.11, kein Docker),
      als systemd-Service, `127.0.0.1`-only
- [x] Einziger Benutzer (Hugo) per offiziellem Headless-Pfad angelegt,
      Signup automatisch geschlossen
- [x] Verbindung real Ende-zu-Ende verifiziert (Prompt-Token-Parität,
      `platform=api_server` in Hermes' `agent.log`)
- [x] `docs/hermes/OPENWEBUI.md` neu, Top-Level-Docs aktualisiert

Ausdrücklich nicht Teil dieser Phase: Caddy, HTTPS, DuckDNS,
Reverse-Proxy, Mehrbenutzerbetrieb, Feature-Tests durch OpenWebUI
(Uploads, Tools, Memory, Workspace, Sessions, Skills, Curator).

## Tracing / Observability (abgeschlossen, keine Phasennummer — Querschnittsthema)

Ziel: kompletten Anfragepfad instrumentieren, Performance-Profil
erzeugen. Ausdrücklich keine Optimierung — nur messen, dokumentieren.

- [x] Geprüft: Hermes' gebündeltes Langfuse-Plugin nutzt exakt die
      nötigen Hooks, verlangt aber Docker zum Self-Hosting — Konflikt
      mit dem projektweiten Docker-Verbot
- [x] Eigenes, rein additives Plugin gebaut (`companion-tracing`,
      offizielle Hooks: `on_session_start`, `pre/post_api_request`,
      `pre/post_tool_call`, `on_session_end`) —
      [ADR 0007](ADR/0007-local-tracing-plugin-not-langfuse.md)
- [x] Real getestet (Tool-Call, Honcho-Recall über 3 LLM-Runden,
      Super-Hermes-Skill) — dabei einen echten Bug im eigenen Code
      gefunden und behoben (verwaister State-Eintrag durch
      Key-Mismatch)
- [x] Realer Fund: `honcho_reasoning` dominierte eine gemessene
      Anfrage (23,6s von 54,6s) stärker als jeder LLM-Call
- [x] Ehrliche Grenzen dokumentiert: TTFT, Streaming-Dauer, RAG,
      Skill-Selection und Tool-Planning sind ohne Hermes-Kernpatch
      nicht separat messbar — nicht vorgetäuscht
- [x] `docs/hermes/TRACING.md` neu, Top-Level-Docs aktualisiert

## Phase X — Mnemosyne ersetzt Honcho (abgeschlossen)

Ziel: Honcho (Sprint 7) durch `mnemosyne-hermes` als Memory-Provider
ersetzen, real getestet, gemessen verglichen. Ausdrücklich keine
Eigenentwicklung für eine Honcho→Mnemosyne-Datenmigration, falls keine
offizielle existiert.

- [x] `mnemosyne-hermes` offiziell installiert (`pip install`, eigene
      `entry_points`-Registrierung — kein manuelles Plugin nötig)
- [x] `memory.provider` auf `mnemosyne` umgestellt, Gateway neu
      gestartet
- [x] Duplicate-Registration-Frage geklärt: verzeichnisbasierter
      Loader ist nur CLI-Introspektion (Quellcode-Beleg), Hooks feuern
      nachweislich nur einmal (Tool-Call-Trace + direkte
      SQLite-Abfrage)
- [x] Offiziellen Migrationsweg geprüft: keiner vorhanden (Mnemosynes
      eigene `comparison.md` erwähnt Honcho nicht) — dokumentiert,
      nicht selbst gebaut
- [x] Honcho deaktiviert (gestoppt, `disabled`, nicht deinstalliert),
      keine Honcho-Prozesse mehr aktiv
- [x] Eigenes Tracing-Plugin um `mnemosyne`-Kategorie erweitert
- [x] Alle 12 geforderten Ende-zu-Ende-Tests real durchgeführt (Speichern,
      Cross-Session-Recall, Mehrfach-Fakten, Update, Workspace,
      OpenWebUI, Grok, Exa, Skills, Datei, Session-Historie)
- [x] Realer Performance-Vergleich: `mnemosyne_recall` ⌀29,2ms (n=40)
      vs. `honcho_reasoning` 23.587,6ms; RAM/CPU-Fußabdruck verglichen
      (inkl. ehrlicher Einschränkung bei nicht direkt vergleichbaren
      CPU-Messfenstern)
- [x] Bekannte Grenze real dokumentiert: Recall-Inkonsistenz bei
      Multi-Marker-Sammelabfragen trotz bestätigt vorhandenem
      DB-Eintrag
- [x] `docs/hermes/MNEMOSYNE.md` neu, [ADR 0008](ADR/0008-mnemosyne-replaces-honcho.md),
      Top-Level-Docs aktualisiert

## Phase Y — Honcho vollständig entfernt (abgeschlossen)

Ziel: Honcho nach vollständiger Mnemosyne-Verifikation restlos
deinstallieren. Vor jeder Löschung geprüft, dass die Ressource
ausschließlich Honcho gehörte.

- [x] Vollständiges Audit vor jeder Löschung (Prozesse, Services, User,
      Datenbanken, Config, Secrets, Plugins, Docker, Cronjobs, Cache,
      Logs, PID-Dateien) — Docker war nie installiert, kein Cron-
      Subsystem vorhanden, keine lokalen TinyLlama/ctransformers-
      Pakete in Honchos venv gefunden (Honchos Text-Generierung lief
      nachweislich über Grok/xAI, nie lokal)
- [x] Backup von Honchos Konfiguration/Code (ohne venv/Cache) vor
      Löschung erstellt
- [x] PostgreSQL-Datenbank `honcho` + Rolle `honcho` gelöscht (einzige
      Nicht-System-DB/-Rolle auf der Instanz) — Server selbst auf
      Nutzerentscheidung hin installiert gelassen, keine Honcho-Daten
      mehr enthalten
- [x] Redis geprüft (leer, keine Persistenz, kein anderer Konsument) —
      nichts zu löschen, Server unverändert gelassen
- [x] systemd-Units entfernt, `daemon-reload`
- [x] Systembenutzer/-gruppe `honcho` gelöscht (inkl. Home-Verzeichnis,
      ~1,2 GB: Git-Checkout, venv, uv-Cache)
- [x] `~/.hermes/honcho.json` und die inaktive `HONCHO_API_KEY`-
      Template-Zeile in `~/.hermes/.env` entfernt — Mnemosyne-Config
      unangetastet
- [x] Hermes' eigenes gebündeltes `plugins/memory/honcho`-Plugin und
      Mnemosynes `core/importers/honcho.py` bewusst nicht angefasst
      (fremder Code, nicht unsere Installation)
- [x] Repository-weites Audit nach verbliebenen Referenzen; nur
      offensichtlich veraltete "aktuell"-Formulierungen korrigiert
      (`ARCHITECTURE.md`, `HONCHO.md`, `COMPANION_STACK.md`,
      `OVERVIEW.md`) — historische Sprint-7-Dokumentation bewusst
      erhalten
- [x] Nach-Entfernung-Verifikation: keine Honcho-Prozesse/-Services/
      -User/-DB/-Config/-Plugins/-Docker-Ressourcen/-Python-Pakete;
      `hermes memory status` → `mnemosyne`; Mnemosyne, Workspace,
      OpenWebUI, Tracing, Skills real erneut getestet — alle
      funktionsfähig
- [x] `docs/hermes/MNEMOSYNE.md` und [ADR 0008](ADR/0008-mnemosyne-replaces-honcho.md)
      um Entfernungs-Nachweis ergänzt

## Phase 8.2+ / Sprint 8+ — geplant (noch nicht begonnen)

Grobe, unverbindliche Reihenfolge — Details folgen jeweils im Sprint:

- Feature-Validierung durch OpenWebUI hindurch (Uploads, Tools, Memory/
  Mnemosyne, Workspace, Sessions, Super-Hermes-Skills, Curator) —
  direkte Fortsetzung von Phase 8.1
- Ursache der Exa/`web_search`-Nichtnutzung klären (ggf. Upstream-Issue
  bei Nous Research melden) — vor jeder Aussage "Suche funktioniert
  produktiv"
- Mehrtägige tatsächliche Nutzung, um Sprint 6's Bewertung zu
  vervollständigen (inkl. eines echten Curator-Laufs)
- Hermes-Installation für `hermes_christiane` auf demselben Stack
  (Grok, Exa, Mnemosyne, lokale Embeddings)
- Produktive Skills-/MCP-Nutzung (bereits durch Hermes abgedeckt, siehe
  `docs/hermes/SKILLS.md`, `docs/hermes/MCP.md`)
- Caddy/HTTPS/Reverse-Proxy für OpenWebUI, Multi-User-Ausbau,
  Persona-Layer

Diese Liste ist eine Absichtserklärung, kein Zeitplan.
