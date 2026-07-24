# Architecture

Dieses Dokument beschreibt die Zielarchitektur von PixelHermes und die
Prinzipien, an denen sich jede spätere Entscheidung messen lassen muss.
Stand Sprint 1 ist ausschließlich dieses Repository real vorhanden — alle
hier beschriebenen Systemkomponenten sind **geplant, aber noch nicht
angelegt**.

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
   dem später tatsächlich Anwendungen, Daten, Konfiguration und Logs
   liegen. In Sprint 1 existiert davon noch nichts.

## Geplante Systemstruktur

| Pfad                          | Zweck                                                      | Status        |
|---------------------------------|--------------------------------------------------------------|---------------|
| `/opt/companion/`                 | Installierte Anwendungen/Software                            | geplant, leer |
| `/srv/companion/`                  | Nutzdaten der Dienste (inkl. späterem Workspace)             | geplant, leer |
| `/etc/companion/`                   | Systemweite Konfiguration                                    | geplant, leer |
| `/var/log/companion/`                | Logdateien                                                    | geplant, leer |
| `/var/backups/companion/`             | Backups                                                       | geplant, leer |

Diese Verzeichnisse werden erst angelegt, wenn ein konkreter Dienst sie
tatsächlich benötigt (Infrastructure as Code: erst im Repo definieren,
dann anwenden) — nicht vorab, um keine unnötige Systemänderung
vorzunehmen.

## Workspace

Bewusst nicht definiert. Siehe "Workspace First" oben sowie
[README.md](README.md), Abschnitt "Workspace".

## Ausdrücklich nicht Teil von Sprint 1

- Hermes, Hermes Workspace, OpenWebUI, mem0, Humalike, MCP, Skills
- Workspaces, Agenten
- Systembenutzer
- systemd-Services (laufend oder aktiviert)
- Datenbanken
- Änderungen an bestehender Systemkonfiguration
