# Architecture

Dieses Dokument beschreibt die Zielarchitektur von PixelHermes und die
Prinzipien, an denen sich jede spätere Entscheidung messen lassen muss.

Seit Sprint 3 (Runtime Foundation) existiert neben dem Repository auch
die reale Linux-Systemstruktur (Verzeichnisse, Laufzeitabhängigkeiten,
zwei Benutzer) — siehe [Systemstruktur](#systemstruktur) unten und
[INSTALL.md](INSTALL.md) für die genauen Schritte. Seit Sprint 4 läuft
für den Benutzer `hermes_hugo` eine native, unveränderte Hermes-Agent-
Installation (CLI, API Server, Gateway, Default-Profil) — siehe
[Hermes-Installation](#hermes-installation-sprint-4) unten. Es ist
weiterhin kein systemd-Service aktiv, es ist weiterhin kein OpenWebUI,
mem0 oder Humalike installiert.

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

## Workspace

Wird nicht im Repository nachgebaut. Hermes bringt seine eigene
Workspace-Konvention mit (`~/.hermes/`), dokumentiert in
[docs/hermes/WORKSPACE.md](docs/hermes/WORKSPACE.md) — inklusive der
seit Sprint 4 real beobachteten Verzeichnisstruktur unter
`/srv/companion/hermes_hugo/.hermes/`. Es wurde bewusst **nicht**
verändert, erweitert oder um eigene Ordner ergänzt — nur beobachtet.

## Ausdrücklich nicht Teil von Sprint 1–4

- Hermes Workspace-Anpassungen, OpenWebUI, mem0, Humalike
- Zusätzliche (Nicht-Hermes-eigene) Skills, konfigurierte MCP-Server
- systemd-Services (laufend oder aktiviert)
- Modell-/Provider-Credentials, Personas, Memory-Konfiguration
- Alles für `hermes_christiane` außer Konto und leerem Home-Verzeichnis
