# Architecture

Dieses Dokument beschreibt die Zielarchitektur von PixelHermes und die
Prinzipien, an denen sich jede spätere Entscheidung messen lassen muss.

Seit Sprint 3 (Runtime Foundation) existiert neben dem Repository auch
die reale Linux-Systemstruktur (Verzeichnisse, Laufzeitabhängigkeiten,
zwei Benutzer) — siehe [Systemstruktur](#systemstruktur) unten und
[INSTALL.md](INSTALL.md) für die genauen Schritte. Seit Sprint 4 läuft
für den Benutzer `hermes_hugo` eine native, unveränderte Hermes-Agent-
Installation (CLI, API Server, Gateway, Default-Profil) — siehe
[Hermes-Installation](#hermes-installation-sprint-4) unten. Seit
Sprint 6 ist diese Installation mit genau zwei Providern produktiv
konfiguriert — Grok (xAI) als einziges LLM, Exa als einzige Suche —
siehe [Produktivbetrieb](#produktivbetrieb-sprint-6) unten. (Sprint 5,
ein geplantes internes Hermes-Audit, wurde vom Nutzer zugunsten des
Produktivbetriebs übersprungen — der geschriebene Plan wurde verworfen,
nicht ausgeführt.) Seit Sprint 7 läuft zusätzlich ein selbstgehosteter
Companion-Stack aus Honcho (externer Memory-Provider) und einem
lokalen Embedding-Server (llama.cpp) — die ersten echten
systemd-Services des Projekts, siehe
[Companion Stack](#companion-stack-sprint-7) unten. Seit Phase 8.1 ist
zusätzlich ein unverändertes, offizielles OpenWebUI über Hermes'
OpenAI-kompatiblen API-Server verbunden — siehe
[OpenWebUI](#openwebui-phase-81) unten. **Seit Phase X ist Honcho als
aktiver Memory-Provider durch `mnemosyne-hermes` ersetzt** (in-process
Hermes-Plugin, lokales SQLite, keine eigene Datenbank-Infrastruktur
mehr nötig) — Honcho selbst ist deaktiviert (gestoppt, nicht
deinstalliert), siehe [docs/hermes/MNEMOSYNE.md](docs/hermes/MNEMOSYNE.md)
und [ADR 0008](ADR/0008-mnemosyne-replaces-honcho.md); der
Companion-Stack-Abschnitt unten beschreibt weiterhin den historischen
Sprint-7-Aufbau. Es ist weiterhin kein mem0 oder Humalike installiert
(Humalike wurde in Sprint 7 geprüft und bewusst abgelehnt — siehe
unten).

## Architekturprinzipien

### Upstream First

Standardlösungen und Original-Projekte bevorzugen, keine unnötigen Forks
oder Custom-Patches. Abweichungen vom Upstream sind die Ausnahme und
werden per [ADR](ADR/) begründet.

### Git First

Dieses Repository ist die Single Source of Truth. Jede strukturelle oder
konfigurative Änderung wird versioniert, nicht ad-hoc auf dem Server
vorgenommen.

### Infrastructure as Code

Konfiguration, Unit-Dateien und Setup-Skripte leben als Code in diesem
Repository und werden von dort auf das System übertragen — nicht
umgekehrt. Der Systemzustand muss aus dem Repo reproduzierbar sein.

### Workspace First

Der Hermes-Workspace ist die zentrale Arbeitsumgebung des Systems. Seine
Struktur wird nicht vorab erfunden, sondern erst nach Installation von
Hermes Workspace analysiert und möglichst nah am Upstream übernommen.
Deshalb enthält dieses Repository in Sprint 1 bewusst noch keine
Workspace-Definition.

### Self Hosted

Der gesamte Stack läuft auf eigener Infrastruktur. Keine Abhängigkeit von
fremden Cloud-Diensten für den Kernbetrieb.

### Minimalistisch

So wenig bewegliche Teile wie möglich. Jede zusätzliche Komponente muss
ihren Nutzen rechtfertigen.

### Eine Runtime

Es wird auf eine einzige, klar definierte Tool-Runtime gesetzt (siehe
"Python als Tool-Runtime"), statt mehrere Runtimes parallel zu pflegen.

### Keine unnötigen Abhängigkeiten

Jede Abhängigkeit ist ein Wartungsrisiko. Nur aufnehmen, was tatsächlich
gebraucht wird.

### Python als Tool-Runtime

Werkzeuge/Tools, die Hermes orchestriert, laufen auf Python als
gemeinsamer Runtime — konsistent mit "Eine Runtime".

### Hermes orchestriert — LLM denkt — Tools arbeiten

Klare Rollentrennung:

- **Hermes** orchestriert den Ablauf und die Kommunikation zwischen den
  Komponenten.
- Das **LLM** übernimmt das Denken/Entscheiden.
- **Tools** führen konkrete, klar abgegrenzte Aktionen aus.

Keine Komponente übernimmt die Aufgabe einer anderen.

### Dokumente bleiben lokal

Dokumente und Daten bleiben auf der eigenen Infrastruktur (siehe "Self
Hosted") und werden nicht an Drittdienste ausgelagert.

## Trennung: Repository vs. System

1. **Dieses Repository (`~/companion`, GitHub: `pixelhermes`)** —
   Dokumentation, Konfigurationsvorlagen, Skripte, Versionsgeschichte.
2. **Systemstruktur (`/opt`, `/srv`, `/etc`, `/var/...`)** — der Ort, an
   dem tatsächlich Anwendungen, Daten, Konfiguration und Logs liegen.
   Seit Sprint 3 real angelegt (siehe unten), aber noch ohne installierte
   Anwendungen.

## Systemstruktur

| Pfad                          | Zweck                                              | Status |
|---------------------------------|-------------------------------------------------------|--------|
| `/opt/companion/`                 | Installierte Anwendungen/Software                     | angelegt, leer bis auf Unterstruktur |
| `/opt/companion/skills/`            | geteilte Skills über Profile/Benutzer hinweg        | angelegt, leer |
| `/opt/companion/tools/`              | geteilte Tools/Skripte                               | angelegt, leer |
| `/opt/companion/templates/`           | geteilte Vorlagen                                     | angelegt, leer |
| `/opt/companion/shared/`               | sonstige geteilte Ressourcen                           | angelegt, leer |
| `/srv/companion/`                       | Nutzdaten der Dienste — enthält die Home-Verzeichnisse der Companion-Benutzer (siehe [ADR 0002](ADR/0002-companion-user-home-under-srv.md)) | angelegt |
| `/etc/companion/`                        | Systemweite, PixelHermes-eigene Konfiguration (nicht Hermes-intern, siehe [docs/hermes/DEPLOYMENT.md](docs/hermes/DEPLOYMENT.md)) | angelegt, leer |
| `/var/log/companion/`                     | Logdateien für PixelHermes-eigene Prozesse             | angelegt, leer |
| `/var/backups/companion/`                  | Backups (u. a. Ziel für Kopien von `hermes backup`)   | angelegt, leer |

Alle Verzeichnisse gehören `root:root`, Modus `755`; es liegen noch
keine Anwendungsdaten darin. Details und ausgeführte Befehle:
[INSTALL.md](INSTALL.md).

## Benutzer

Zwei unprivilegierte Linux-Benutzer angelegt, die später je einen
Hermes-Agenten betreiben sollen — Namen entsprechen den in Sprint 1
skizzierten Agenten:

| Benutzer | UID | Home | Passwort-Login |
|---|---|---|---|
| `hermes_hugo` | 1001 | `/srv/companion/hermes_hugo` | gesperrt |
| `hermes_christiane` | 1002 | `/srv/companion/hermes_christiane` | gesperrt |

Home-Verzeichnis bewusst unter `/srv/companion/` statt `/home/`
(Begründung: [ADR 0002](ADR/0002-companion-user-home-under-srv.md)).

Stand Sprint 4: `hermes_hugo` hat eine installierte, native Hermes-
Agent-Instanz (siehe unten); `hermes_christiane` hat weiterhin
ausschließlich Konto und leeres Home-Verzeichnis — folgt laut
[ROADMAP.md](ROADMAP.md) erst nach erfolgreichem Abschluss von
`hermes_hugo`. Für keinen der beiden Benutzer existiert ein
systemd-Service.

## Laufzeitabhängigkeiten (Runtime)

Seit Sprint 3 auf dem Server installiert, als Vorbereitung für die
spätere Hermes-Installation (siehe [docs/hermes/INSTALLATION.md](docs/hermes/INSTALLATION.md)
für Hermes' eigene Anforderungen):

- **Node.js** — offizielle Active-LTS-Version (24.x) über das
  NodeSource-Repository, bewusst nicht das Debian-Paket
  ("Upstream First").
- **Python** — System-Python 3.13 plus `python3-venv`, `python3-pip`.
- **uv** — offizieller Python-Paketmanager, systemweit unter
  `/usr/local/bin` installiert (nicht nur für einen Nutzer), passend zu
  "Python als Tool-Runtime".
- **Basispakete** — `git`, `curl`, `wget`, `sqlite3`,
  `build-essential`, `ca-certificates`, `jq`, `tree`, `htop`, `zip`,
  `unzip` (bereits vorhanden bzw. ergänzt).

## Hermes-Installation (Sprint 4)

Für `hermes_hugo` wurde Hermes Agent nativ und unverändert nach
offizieller Dokumentation installiert (`curl -fsSL
https://hermes-agent.nousresearch.com/install.sh | bash`, ausgeführt als
dieser Linux-Benutzer). Umfang bewusst beschränkt auf CLI, API Server,
Gateway und Profil — keine Desktop-Version, keine experimentellen
Features, kein Workspace-Umbau, keine zusätzlichen Skills, kein MCP,
kein mem0/Humalike. Vollständige, quellenbelegte Beobachtungen:
[docs/hermes/INSTALLATION.md](docs/hermes/INSTALLATION.md).

**Verifiziert und real beobachtet** (nicht vermutet):

| Prüfung | Ergebnis |
|---|---|
| Hermes startet | `hermes --version` → `Hermes Agent v0.19.0 (2026.7.20)` |
| CLI funktioniert | `hermes doctor`, `hermes config check`, `hermes profile list` liefen fehlerfrei |
| Profil wird erkannt | `hermes profile list` zeigt `◆default`; `GET /api/status` bestätigt `"profiles": ["default"]` |
| Konfiguration wird korrekt geladen | `hermes config check` → Version 33 (nach `hermes config migrate`), `config_path`/`env_path` über API bestätigt |
| API erreichbar | `hermes serve --host 127.0.0.1 --port 9119` → `HERMES_BACKEND_READY`, `GET /api/status` → HTTP 200 |
| Gateway startet | `hermes gateway run` → läuft, `hermes gateway status` meldet explizit "Running manually, not as a system service" |

Kein Modell-Provider konfiguriert (bewusste Entscheidung für diese
Phase — siehe [ROADMAP.md](ROADMAP.md)); alle Test-Prozesse (API Server,
Gateway) wurden nach der Verifikation wieder sauber gestoppt, es läuft
aktuell nichts dauerhaft.

## Produktivbetrieb (Sprint 6)

Minimalistische Produktivarchitektur für `hermes_hugo`, ausschließlich
über `hermes config set` konfiguriert (kein manuelles Editieren von
`.env`/`config.yaml`, kein Hermes-Fork):

- **LLM:** Grok (xAI), direkte API (`provider: xai`, `XAI_API_KEY`),
  Modell `grok-build-0.1` — einziger konfigurierter LLM-Provider.
- **Suche:** Exa (`EXA_API_KEY`, `web.backend`/`web.search_backend`/
  `web.extract_backend: exa`) — einziger konfigurierte Suchanbieter.
- **Workspace/Memory/Skills/Sessions:** ausschließlich nativ, keine
  PixelHermes-Erweiterung.

Vollständige, quellenbelegte Beobachtungen (inkl. eines kritischen,
ungelösten Fundes zur Exa-Integration) in
[docs/hermes/PRODUCTIVE_RUNTIME.md](docs/hermes/PRODUCTIVE_RUNTIME.md),
Bewertung in [docs/hermes/ASSESSMENT.md](docs/hermes/ASSESSMENT.md).

**Kurzfassung der Verifikation:**

| Bereich | Ergebnis |
|---|---|
| Grok — Chat/Coding/Tool-Calls | funktioniert, real geprüft und im Log bestätigt |
| Exa — Web-Suche | **konfiguriert, aber nicht zuverlässig aufgerufen** — Modell erfindet stattdessen plausible Fake-Ergebnisse (dreifach reproduziert, per `agent.log` `tool_turns=0` widerlegt) |
| Workspace-Aufgaben (Git/Markdown/Planung) | funktionieren, real ausgeführt (u. a. echtes Git-Repo mit Commits in `~/hermes-notes/`) |
| Sessions (List/Search/Export/Archive/Optimize/Repair) | größtenteils funktionierend; `archive --older-than` zeigte einen realen, ungeklärten Nichttreffer-Befund; ein realer, unbehandelter Absturz bei Aufruf aus einem für `hermes_hugo` unlesbaren Arbeitsverzeichnis |
| Curator | nur beobachtet, nicht ausgelöst — 0 Läufe, Trigger-Bedingungen (7 Tage/2h Leerlauf) in dieser Sitzung nicht erfüllbar |

**Bewusst offen gelassen:** die Ursache der Exa-Nichtnutzung wurde
nicht im Hermes-Quellcode weiterverfolgt (würde "Upstream First"
verletzen, wenn daraus ein eigener Patch entstünde) — stattdessen
dokumentiert und als offene Frage markiert.

## Companion Stack (Sprint 7)

Erste Erweiterung über reines Hermes hinaus: ein selbstgehosteter
externer Memory-Provider (Honcho) plus ein lokaler Embedding-Server
(llama.cpp), beide als eigene, unprivilegierte Systemdienste unter
`/opt/companion/` — siehe [ADR 0003](ADR/0003-shared-services-under-opt-companion.md)
und [ADR 0004](ADR/0004-local-embedding-server-for-honcho.md). Volle
Beweisführung: [docs/hermes/HONCHO.md](docs/hermes/HONCHO.md),
Gesamtbild: [docs/hermes/COMPANION_STACK.md](docs/hermes/COMPANION_STACK.md).

**Humalike wurde geprüft und bewusst abgelehnt:** offiziell dokumentiert
(`github.com/Humalike/hermes-humalike-plugin`), aber abhängig von einem
kostenpflichtigen externen Cloud-Dienst — widerspricht dem
"Self Hosted"-Prinzip. Nicht installiert, keine Konfiguration
verändert.

**Honcho** (`plastic-labs/honcho`, offizieller Hermes-Memory-Provider-
Plugin-Partner) läuft vollständig selbstgehostet:

| Komponente | Läuft als | Zweck |
|---|---|---|
| PostgreSQL 17 + pgvector | Systembenutzer `postgres` (Debian-Paket) | Honchos persistenter Speicher |
| Redis | Systembenutzer `redis` (Debian-Paket) | von Honcho vorgesehen, aktuell ungenutzt (Cache im In-Memory-Modus) |
| Honcho API (`fastapi run`) | Systembenutzer `honcho` | REST-API, Port 127.0.0.1:8000 |
| Honcho Deriver (Hintergrund-Worker) | Systembenutzer `honcho` | asynchrone Gedächtnis-Ableitung |
| Embedding-Server (llama.cpp) | Systembenutzer `embeddings` | lokale Embeddings, Port 127.0.0.1:8081, API-Key-geschützt |

Honchos eigene Text-Generierung (Deriver/Dialectic/Summary/Dream) nutzt
den bereits vorhandenen xAI/Grok-Key (kein neuer LLM-Provider);
Embeddings laufen vollständig lokal über `nomic-embed-text-v1.5`
(768 Dimensionen) — kein Cloud-Embedding-Provider, siehe
[ADR 0004](ADR/0004-local-embedding-server-for-honcho.md) für die
Begründung (OpenAI und Gemini wurden explizit erwogen und verworfen).

**Real Ende-zu-Ende verifiziert** (nicht nur Prozess-Start):

| Prüfung | Ergebnis |
|---|---|
| Hermes → Honcho verbunden | `hermes doctor` → `Memory Provider: ✓ Honcho connected workspace=hermes mode=hybrid freq=async` |
| Schreibpfad | echter `honcho_profile`-Tool-Call, echte Zeilen in `peers`/`documents`/`queue`-Tabellen (per SQL bestätigt, nicht nur CLI-Ausgabe geglaubt) |
| Embedding-Pipeline | Deriver → lokaler llama-server → 768-dim-Vektor real in `pgvector` gespeichert |
| **Cross-Session-Recall** | neue Session erinnert sich korrekt an in einer früheren Session gespeicherte Nutzerpräferenz — der eigentliche Zweck von Honcho, real bestätigt |
| Grok/Exa/Workspace weiterhin funktionsfähig | `hermes doctor` zeigt alle zuvor funktionierenden Tool-Kategorien unverändert ✓ |

Kein Hermes-Kerndateien verändert — ausschließlich `hermes config set`,
eine `honcho.json` am dokumentierten Ort, und die neuen, komplett
separaten Systemdienste.

**Historischer Stand**: Diese Tabelle beschreibt den Sprint-7-Zustand.
Seit Phase X ist Honcho deaktiviert (`companion-honcho-api`/`-deriver`
gestoppt und `disabled`, aber nicht deinstalliert) und
`mnemosyne-hermes` der aktive Memory-Provider
(`hermes doctor` → `Memory Provider: ✓ mnemosyne provider active`).
Details, erneute Ende-zu-Ende-Verifikation und Performance-Vergleich:
[docs/hermes/MNEMOSYNE.md](docs/hermes/MNEMOSYNE.md),
[ADR 0008](ADR/0008-mnemosyne-replaces-honcho.md).

### Drittanbieter-Erweiterungen (Policy-Präzisierung)

Sprint 7 präzisierte "nur offizielle Erweiterungen" statt es
aufzuweichen — siehe [ADR 0005](ADR/0005-third-party-extension-policy.md):
Drittanbieter-Erweiterungen sind zulässig, wenn sie keine
Hermes-Kerndateien ändern, ausschließlich dokumentierte
Erweiterungsmechanismen nutzen, vollständig rückstandsfrei entfernbar
sind, und klar als optional dokumentiert werden. Erste Anwendung dieser
Regel: **Super Hermes** (`Cranot/super-hermes`, Drittanbieter-
Skill-Paket) — vollständig geprüft, manuell installiert (kein
Drittanbieter-Skript ausgeführt), Entfernbarkeit real getestet. Details:
[docs/hermes/SUPER_HERMES.md](docs/hermes/SUPER_HERMES.md).

Zusätzlich installiert (offiziell, NousResearch-verifiziert per
GitHub-API): **Hermes Agent Self-Evolution** — ein separates,
Offline-Optimierungswerkzeug für Skills/Prompts (aktuell Phase 1,
nur Skills), das Kandidaten generiert und ausschließlich als
Pull-Request zur menschlichen Prüfung vorschlägt, nie direkt committet.
Installiert in einer eigenen, von Hermes getrennten Python-Umgebung.
Per Quellcode-Prüfung bestätigt: `--dry-run` löst keinerlei API-Calls
aus (kehrt zurück, bevor der LLM-Client instanziiert wird) — genau
dieser kostenlose Modus wurde ausgeführt, kein echter, kostenpflichtiger
Optimierungslauf. Details:
[docs/hermes/SELF_EVOLUTION.md](docs/hermes/SELF_EVOLUTION.md).

## OpenWebUI (Phase 8.1)

Unverändertes, offiziell per `pip` installiertes OpenWebUI
(`companion-openwebui.service`, eigener Systembenutzer `openwebui`
unter `/opt/companion/`) verbindet sich mit `hermes_hugo`s Hermes über
dessen offiziellen OpenAI-kompatiblen API-Server
(`API_SERVER_ENABLED=true`, `/v1/chat/completions`,
`127.0.0.1:8642`). Die ursprüngliche Annahme einer separaten "Hermes
Agent API" wurde per Quellcode-Prüfung widerlegt — der OpenAI-förmige
Endpoint instanziiert dieselbe `AIAgent`-Klasse wie CLI und
Gateway-Plattformen, ist also bereits der volle Agent. Details:
[ADR 0006](ADR/0006-openwebui-via-hermes-openai-endpoint.md),
[docs/hermes/OPENWEBUI.md](docs/hermes/OPENWEBUI.md).

Damit läuft Hermes' Gateway erstmals dauerhaft (offiziell über `hermes
gateway install`, `systemd --user` + `loginctl enable-linger`) statt
nur testweise gestartet und gestoppt zu werden (bisheriges Muster aus
Sprint 4/6) — notwendig, damit die OpenWebUI-Verbindung tatsächlich
nutzbar bleibt.

**Real Ende-zu-Ende verifiziert:** direkter Hermes-Aufruf und
OpenWebUI-vermittelter Aufruf liefern nahezu identische
Prompt-Token-Zahlen (18.334 vs. 18.338) und erscheinen beide mit
`platform=api_server` in Hermes' eigenem `agent.log` — derselbe echte
Agent, kein Mock. Genau ein Benutzer (Hugo, per Env-Var-Headless-Setup
angelegt) — OpenWebUI deaktiviert Signup automatisch nach dem ersten
Nutzer. Kein Caddy, keine HTTPS, kein Reverse-Proxy, kein DuckDNS —
OpenWebUI bindet an `127.0.0.1`, Zugriff aktuell nur lokal/loopback.

**Bewusst noch nicht getestet** (nächste Phase): Datei-Uploads, Tools,
Memory (Honcho), Workspace, Sessions, Super-Hermes-Skills, Curator
über OpenWebUI — die Architektur bestätigt, dass all das über dieselbe
`AIAgent`-Instanz erreichbar ist, aber real durchgetestet wird es erst
in der nächsten Phase.

## Tracing / Observability

Ein rein additives Hermes-Plugin (`~/.hermes/plugins/companion-tracing/`,
Systembenutzer `hermes_hugo`, keine Hermes-Kerndatei verändert) misst
Request-/Session-/Tool-/LLM-Zeiten über Hermes' offizielle
Plugin-Hooks (`on_session_start`, `pre/post_api_request`,
`pre/post_tool_call`, `on_session_end`) und schreibt strukturierte
JSON-Lines-Traces nach `~/.hermes/logs/tracing/`. Hermes' eigenes
gebündeltes Langfuse-Plugin nutzt dieselben Hooks, verlangt für
Self-Hosting aber offiziell Docker — deshalb ein eigenes,
Docker-freies, vollständig lokales Plugin statt dessen
([ADR 0007](ADR/0007-local-tracing-plugin-not-langfuse.md)). Keine
Optimierung, keine Prompt-/Provider-/Memory-Änderung — ausschließlich
Messung. Volle Beweisführung inkl. ehrlicher Grenzen (kein TTFT/
Streaming/RAG messbar, da kein Hook dafür existiert) und einem realen,
dabei entdeckten Bottleneck (ein einzelner `honcho_reasoning`-Aufruf
dominierte eine gemessene Anfrage stärker als jeder LLM-Call):
[docs/hermes/TRACING.md](docs/hermes/TRACING.md).

## Workspace

Wird nicht im Repository nachgebaut. Hermes bringt seine eigene
Workspace-Konvention mit (`~/.hermes/`), dokumentiert in
[docs/hermes/WORKSPACE.md](docs/hermes/WORKSPACE.md) — inklusive der
seit Sprint 4 real beobachteten Verzeichnisstruktur unter
`/srv/companion/hermes_hugo/.hermes/`. Es wurde bewusst **nicht**
verändert, erweitert oder um eigene Ordner ergänzt — nur beobachtet.

## Ausdrücklich nicht Teil von Sprint 1–7 / Phase 8.1

- Hermes Workspace-Anpassungen, mem0, Humalike (geprüft und
  abgelehnt, siehe oben)
- Zusätzliche (Nicht-Hermes-eigene) Skills, konfigurierte MCP-Server
- Weitere LLM-/Such-/Cloud-Embedding-Provider neben Grok/Exa (kein
  OpenRouter, kein Ollama, kein OpenAI, kein Google Gemini)
- Personas, eigene Hermes-Workspace-Konfiguration
- Alles für `hermes_christiane` außer Konto und leerem Home-Verzeichnis
- Caddy, HTTPS, DuckDNS, Reverse-Proxy, Mehrbenutzerbetrieb in
  OpenWebUI
- Feature-Tests durch OpenWebUI hindurch (Uploads, Tools, Memory,
  Workspace, Sessions, Skills, Curator) — nächste Phase
  (Honcho und der Embedding-Server sind bereits für spätere
  Mitnutzung durch `hermes_christiane` ausgelegt, siehe
  [docs/hermes/COMPANION_STACK.md](docs/hermes/COMPANION_STACK.md))
