# Architecture

Dieses Dokument beschreibt die Zielarchitektur von PixelHermes und die
Prinzipien, an denen sich jede spätere Entscheidung messen lassen muss.

Seit Sprint 3 (Runtime Foundation) existiert neben dem Repository auch
die reale Linux-Systemstruktur (Verzeichnisse, Laufzeitabhängigkeiten,
zwei Benutzer) — siehe [Systemstruktur](#systemstruktur) unten und
[INSTALL.md](INSTALL.md) für die genauen Schritte. Es läuft weiterhin
kein Dienst, es ist weiterhin keine Anwendung (Hermes, OpenWebUI, mem0,
Humalike) installiert.

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
Noch **kein** Hermes, **kein** systemd-Service, **kein** Workspace,
**keine** Datenbank für diese Benutzer — ausschließlich Konten und leere
Home-Verzeichnisse.

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

## Workspace

Wird nicht im Repository nachgebaut. Hermes bringt seine eigene
Workspace-Konvention mit (`~/.hermes/`), dokumentiert in
[docs/hermes/WORKSPACE.md](docs/hermes/WORKSPACE.md). Die realen
Home-Verzeichnisse dafür existieren seit Sprint 3 (siehe "Benutzer"
oben); der Workspace-Inhalt selbst entsteht erst mit der tatsächlichen
Hermes-Installation.

## Ausdrücklich nicht Teil von Sprint 1–3

- Hermes, Hermes Workspace, OpenWebUI, mem0, Humalike, MCP, Skills
  (Hermes-eigene)
- systemd-Services (laufend oder aktiviert)
- Datenbanken
- Persona-/Memory-Konfiguration
