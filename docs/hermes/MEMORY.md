# Hermes Agent — Memory & Reflection

Quellen: [Memory Feature](https://hermes-agent.nousresearch.com/docs/user-guide/features/memory),
[Curator](https://hermes-agent.nousresearch.com/docs/user-guide/features/curator),
[Skills Feature](https://hermes-agent.nousresearch.com/docs/user-guide/features/skills),
[Architecture](https://hermes-agent.nousresearch.com/docs/developer-guide/architecture),
[Tips](https://hermes-agent.nousresearch.com/docs/guides/tips)

## Memory

Zwei getrennte Systeme:

**1. Kuratiertes, begrenztes Gedächtnis (`MEMORY.md`/`USER.md`)** —
reine Markdown-Dateien unter `~/.hermes/memories/`:
- `MEMORY.md`: Umgebungsfakten und gelernte Lektionen, Limit **2.200
  Zeichen** (~800 Tokens).
- `USER.md`: Nutzerpräferenzen/Kommunikationsstil, Limit **1.375
  Zeichen** (~500 Tokens).

**2. Sitzungssuche:** SQLite `~/.hermes/state.db` mit **FTS5-
Volltextsuche** über alle CLI- und Messaging-Sessions, inkl.
Lineage-Tracking (Eltern/Kind über Kompressionen hinweg) — siehe
[ARCHITECTURE.md](ARCHITECTURE.md), Abschnitt Sessions.

**Injektion:** MEMORY.md/USER.md werden als **eingefrorener Snapshot
zum Sessionstart** in den System-Prompt injiziert (schützt den
LLM-Prefix-Cache). Schreibvorgänge persistieren sofort, wirken sich aber
erst in der **nächsten** Session auf den Prompt aus.

**Schreiben/Lifecycle:** Tool `memory` mit Aktionen `add`, `replace`
(Substring-Match), `remove`. **Kein Auto-Kompaktieren** — bei
Überschreitung des Zeichenlimits gibt das Tool einen Fehler zurück statt
Daten stillschweigend zu verwerfen; der Agent muss konsolidieren.

**Externe Memory-Provider (mem0-artige Integrationen, bestätigt):**
acht Plugins — Honcho, OpenViking, Mem0, Hindsight, Holographic,
RetainDB, ByteRover, Supermemory — laufen **neben** dem eingebauten
Gedächtnis, nie als Ersatz; fügen semantische Suche, Knowledge Graphs
und automatische Fakten-Extraktion hinzu.

**Memory vs. Skills:** Memory = Fakten, Skills = Prozeduren
(mehrstufige Workflows).

## Reflection

**Es gibt keinen offiziell benannten Begriff "Reflection"** — die
Architekturseite stellt explizit fest, es werde kein expliziter
Reflexions-/Selbstverbesserungsmechanismus auf dieser Seite beschrieben;
der Agent arbeite über eine Standard-Konversationsschleife.

Das funktionale Äquivalent existiert unter anderen Namen:

- **Hintergrund-Selbstverbesserungs-Review:** läuft periodisch (~alle 10
  Turns), eine kurze Auxiliary-Model-Prüfung, die Konsolidierungen oder
  Patches gegen Drift vorschlägt — "runs in its own prompt cache and
  never touches the active conversation."
- **Autonome Skill-Erstellung/-Verbesserung:** über das Tool
  `skill_manage`, beschrieben als "the agent's procedural memory".
  Trigger: erfolgreiche komplexe Aufgaben (5+ Tool-Calls), erfolgreiche
  Fehlerbehebung nach Sackgassen, Nutzerkorrektur mit neuem Ansatz,
  identifizierter wiederverwendbarer Workflow.
- **Curator (Skill-Lifecycle-Wartung):** separater, geplanter
  Wartungslauf (Default `interval_hours: 168`, benötigt 2h
  Agenten-Leerlauf), überführt Skills deterministisch aktiv → veraltet
  (30 Tage ungenutzt) → archiviert (90 Tage, nach `.archive/`, nie
  automatisch gelöscht). Optionale LLM-Konsolidierung
  (`curator.consolidate`, Default aus). Skills können `pinned` werden,
  um sie von Mutation/Löschung auszunehmen. Backups vor jedem echten Lauf
  in `.curator_backups/<iso-ts>/`.
- **Write-Approval-Gate:** `skills.write_approval: true` staged alle
  Skill-Schreibvorgänge in `~/.hermes/pending/`; Freigabe über
  `/skills pending`, `/skills diff`, `/skills approve`.

**Fazit für "Reflection":** kein offizieller Hermes-Begriff — im
Build Guide stattdessen "Selbstverbesserungs-Review", "Curator" und
`skill_manage`-gesteuertes prozedurales Lernen verwenden.

---

## PixelHermes-Mapping

**Was übernimmt Hermes bereits?** Ein vollständiges,
zweistufiges Gedächtnismodell (kuratiert + durchsuchbarer Verlauf) plus
eine dokumentierte Selbstverbesserungsschleife für Skills (Curator +
Background-Review) — funktional das, was im ursprünglichen
Konzeptentwurf lose als "mem0" und "Reflection" bezeichnet wurde.

**Was müssen wir NICHT selbst entwickeln?** Kein eigenes mem0-artiges
System, keinen eigenen Reflexionsmechanismus, keine eigene
Skill-Alterungs-/Archivierungslogik — Curator deckt das bereits ab.

**Was passt direkt zu PixelHermes?** Der Sprint-Ausschluss "noch kein
mem0" war korrekt vorsichtig formuliert: Hermes bringt mit Curator +
externen Provider-Plugins (inkl. `Mem0` selbst als Plugin) bereits eine
vollständige, offiziell unterstützte Antwort mit. Ein separates,
eigenständiges mem0-Setup wäre eine unnötige Abhängigkeit
("Upstream First", "Keine unnötigen Abhängigkeiten").

**Welche Erweiterungen wären später sinnvoll?** Falls PixelHermes echten
Bedarf an semantischer Suche/Knowledge-Graphs über das eingebaute
Gedächtnis hinaus entwickelt: eines der acht offiziell dokumentierten
Provider-Plugins aktivieren (z. B. Mem0), statt selbst zu bauen — erst
nach konkretem, belegtem Bedarf.

**Welche Komponenten sollten wir bewusst unverändert übernehmen?** Das
komplette Memory-System (MEMORY.md/USER.md-Limits, state.db/FTS5) und
den Curator-Mechanismus vollständig unverändert.
