# Hermes Agent — Workspace, Context, SOUL.md, AGENTS.md

## Workspace

Quelle: [Configuration](https://hermes-agent.nousresearch.com/docs/user-guide/configuration),
[Profiles](https://hermes-agent.nousresearch.com/docs/user-guide/profiles),
[Curator](https://hermes-agent.nousresearch.com/docs/user-guide/features/curator)

Standardort: `~/.hermes` (= `HERMES_HOME`) — zugleich das Default-Profil
selbst. Dokumentierte Verzeichnisstruktur:

```
~/.hermes/
├── config.yaml            # Haupteinstellungen
├── .env                    # API-Keys/Secrets
├── auth.json                # OAuth-Credentials
├── SOUL.md                   # Agenten-Identität (Slot #1 im System-Prompt)
├── memories/
│   ├── MEMORY.md               # Umgebungsfakten/gelernte Lektionen
│   └── USER.md                  # Nutzerpräferenzen
├── skills/                        # Agent-erstellte Skills
│   ├── .usage.json                  # Curator-Telemetrie
│   ├── .archive/                     # 90+ Tage ungenutzte Skills
│   └── .curator_backups/<iso-ts>/     # Snapshots vor Konsolidierung
├── cron/                                # geplante Jobs
├── sessions/                             # Gateway-Session-Zuordnung
├── logs/                                  # Fehler-/Gateway-Logs (Secrets redigiert)
├── state-snapshots/                        # Pre-Update-Backups
├── backups/                                  # vollständige HERMES_HOME-Zips
├── cache/                                     # Remote-Syncs/Caches
├── pending/                                     # gestagte Skill-/Memory-Writes
├── sandboxes/                                    # Container-/Sandbox-Storage
├── state.db                                       # SQLite, Sessions/FTS5
└── profiles/<name>/                                # weitere Profile (Parallelstruktur)
```

Weitere Profile (`hermes -p <name>`) erhalten ein vollständig paralleles
Verzeichnis unter `~/.hermes/profiles/<name>/` mit eigenem
`config.yaml`, `SOUL.md`, `.env`, Memory, Sessions, Skills und
Gateway-State. **Isolation ist Zustandsisolation, keine Dateisystem-
Sandbox** (siehe [ARCHITECTURE.md](ARCHITECTURE.md)).

## Context Files

Quelle: [Context Files](https://hermes-agent.nousresearch.com/docs/user-guide/features/context-files)

Ein Projekt-Kontext-Dateityp pro Session, **erster Treffer gewinnt**,
Priorität von oben nach unten:

| Datei | Zweck | Suchbereich |
|---|---|---|
| `.hermes.md` / `HERMES.md` | Projektanweisungen (höchste Priorität) | bis zum Git-Root |
| `AGENTS.md` | Projektstruktur & Konventionen | CWD + Unterverzeichnisse |
| `CLAUDE.md` | Claude-Code-Kontext | CWD + Unterverzeichnisse |
| `.cursorrules` / `.cursor/rules/*.mdc` | Cursor-IDE-Konventionen | nur CWD |

`SOUL.md` läuft **unabhängig** davon — immer aus `HERMES_HOME`, nie aus
dem Arbeitsverzeichnis.

**Injektionsmechanik:** Beim Start werden Kandidaten gescannt, auf
Prompt-Injection geprüft, bei über 20.000 Zeichen gekürzt (70 % Kopf /
20 % Ende) und unter einer `# Project Context`-Überschrift in den
System-Prompt injiziert. Während der Session verfolgt ein
`SubdirectoryHintTracker` Tool-Aufrufe; berührte Unterverzeichnisse
liefern zusätzliche Kontextdateien (bis 5 Elternverzeichnisse), gekürzt
auf 8.000 Zeichen, angehängt an das jeweilige Tool-Ergebnis.

**Sicherheitsscan:** alle Kontextdateien (inkl. SOUL.md) werden auf
Prompt-Injection geprüft (Instruction-Override, verstecktes
HTML/CSS, Credential-Exfiltration, unsichtbarer Unicode). Blockierter
Inhalt wird durch `[BLOCKED: ...]` ersetzt.

**Nicht dokumentiert:** Verhalten bei gleichzeitiger Existenz mehrerer
konkurrierender Dateitypen mit unterschiedlichem Inhalt (kein
Beispiel in der Doku).

## SOUL.md

Quelle: [Personality](https://hermes-agent.nousresearch.com/docs/user-guide/features/personality),
[Use SOUL.md with Hermes](https://hermes-agent.nousresearch.com/docs/guides/use-soul-with-hermes)

Primäre, dauerhafte Identitätsdatei — "defines who the agent is, how it
speaks, and what it avoids", Slot #1 im System-Prompt, ersetzt die
eingebaute Default-Identität vollständig. Ort: `$HERMES_HOME/SOUL.md`,
nie projektbezogen. Hermes seedet automatisch eine Starter-SOUL.md, wenn
keine existiert; eine vom Nutzer bearbeitete Datei wird nie
überschrieben.

**Soll enthalten:** Tonfall, Kommunikationsstil, Umgang mit
Unsicherheit/Widerspruch, stilistische Vermeidungen. Faustregel: "if it
should apply everywhere, put it in SOUL.md."

**Soll NICHT enthalten:** projektspezifische Anweisungen, Pfade,
Repo-Konventionen — das gehört in AGENTS.md.

## AGENTS.md

Quelle: [Context Files](https://hermes-agent.nousresearch.com/docs/user-guide/features/context-files),
[Tips](https://hermes-agent.nousresearch.com/docs/guides/tips)

Projektspezifische Anweisungen — Architekturentscheidungen, Coding-
Konventionen, Projektstruktur. Wird im Projekt-Root (oder
Unterverzeichnissen) abgelegt; Root-Level lädt beim Start, Unterordner
lazy über die progressive Discovery. Anders als SOUL.md wird AGENTS.md
laut Doku **nicht** automatisch geseedet — rein nutzerverfasst.

**Abgrenzung:** SOUL.md = dauerhafte, instanzweite Persönlichkeit;
AGENTS.md = projektspezifische Fakten/Konventionen, konkurriert mit
`.hermes.md`/`CLAUDE.md`/`.cursorrules` um denselben Slot (nur ein
Dateityp lädt pro Session).

---

## PixelHermes-Mapping

**Was übernimmt Hermes bereits?** Das komplette Kontextmodell
(Workspace-Layout, Priorisierung, Sicherheitsscan, progressive
Discovery) sowie die Trennung Identität (SOUL.md) vs. Projektwissen
(AGENTS.md).

**Was müssen wir NICHT selbst entwickeln?** Kein eigenes
Kontext-Injection-System, keinen eigenen Prompt-Injection-Scanner, keine
eigene Workspace-Verzeichnisstruktur — `~/.hermes/` ist bereits die
vollständige Antwort auf das, was in
[ARCHITECTURE.md](../../ARCHITECTURE.md) als "Workspace" angedeutet wird.

**Was passt direkt zu PixelHermes?** Genau das ist der Grund, warum der
frühere Grundsatz "Workspace First — die Struktur wird erst nach
Installation von Hermes Workspace analysiert" jetzt beantwortet ist: die
Struktur *ist* `~/.hermes/`, keine eigene Erfindung nötig. PixelHermes-
spezifisches Projektwissen gehört in ein projektweites `AGENTS.md`
(z. B. im Repo-Root von `~/companion`), PixelHermes-spezifische
Persönlichkeit in `SOUL.md` je Profil.

**Welche Erweiterungen wären später sinnvoll?** Ein PixelHermes-eigenes
`AGENTS.md`-Template unter `templates/`, sobald konkrete Projektregeln
feststehen — aber erst nach Installation, nicht vorab spekulativ.

**Welche Komponenten sollten wir bewusst unverändert übernehmen?** Die
komplette Verzeichnisstruktur unter `~/.hermes/`, das
Priorisierungsschema der Kontextdateien und den Sicherheitsscan — kein
eigener Fork der Context-Loading-Logik.
