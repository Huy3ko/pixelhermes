# Project Structure

Kommentierte Übersicht über den Inhalt dieses Repositories.

```
pixelhermes/
├── README.md              Einstieg, Überblick, Status
├── ROADMAP.md              Sprintplan / geplante Ausbaustufen
├── ARCHITECTURE.md          Zielarchitektur & Architekturprinzipien
├── INSTALL.md                Nachvollziehbare Aufbauschritte
├── CHANGELOG.md               Änderungshistorie
├── PROJECT_STRUCTURE.md        diese Datei
├── .gitignore
│
├── ADR/                    Architecture Decision Records
│                           Jede nicht-triviale Architekturentscheidung
│                           wird hier als eigenes, nummeriertes Dokument
│                           festgehalten (Kontext, Entscheidung, Konsequenz).
│
├── docs/                   Weiterführende Dokumentation
│                           (Detaildokumente zu einzelnen Komponenten,
│                           sobald diese existieren)
│
├── configs/                Konfigurationsvorlagen (Infrastructure as Code)
│                           Werden später nach /etc/companion/ übertragen.
│                           Keine echten Secrets (siehe .gitignore).
│
├── scripts/                Hilfsskripte für Aufbau/Wartung/Betrieb
│                           Python als Tool-Runtime.
│
├── systemd/                Geplante systemd-Unit-Dateien
│                           Quelle der Wahrheit, bevor etwas auf dem
│                           System aktiviert wird.
│
├── templates/               Generische, wiederverwendbare Vorlagen
│                           (NICHT die Workspace-Struktur — siehe unten)
│
├── assets/                  Statische Begleitmaterialien fürs Repo
│                           (Diagramme, Logos o. Ä.)
│
└── backups/                 Ablagestruktur für lokale Backups
                            (Archive selbst nicht versioniert)
```

## Bewusst nicht vorhanden

- **Kein `workspace/`-Verzeichnis.** Die Workspace-Struktur wird erst
  nach Installation von Hermes Workspace analysiert und möglichst nah am
  Upstream übernommen (siehe [ARCHITECTURE.md](ARCHITECTURE.md),
  "Workspace First").
- **Keine Anwendungs-, Service- oder Datenbank-Verzeichnisse.** Diese
  entstehen erst, wenn die jeweilige Komponente tatsächlich eingeführt
  wird (siehe [ROADMAP.md](ROADMAP.md)).

## Geplante Systemstruktur (außerhalb dieses Repos)

Siehe [ARCHITECTURE.md](ARCHITECTURE.md#geplante-systemstruktur) für die
vollständige Tabelle der geplanten Linux-Pfade
(`/opt/companion/`, `/srv/companion/`, `/etc/companion/`,
`/var/log/companion/`, `/var/backups/companion/`) — in Sprint 1 nur
dokumentiert, nicht angelegt.
