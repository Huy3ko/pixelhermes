# 0007. Eigenes leichtgewichtiges Tracing-Plugin statt Langfuse

Status: Accepted

## Kontext

Für Observability/Performance-Tracing des Hermes-Agenten (Request-ID,
Session-ID, Schrittdauern, Tool-/LLM-Zeiten) bringt Hermes bereits ein
offizielles, gebündeltes Observability-Plugin mit (`plugins/observability/langfuse/`),
das exakt die dafür nötigen Hook-Punkte nutzt
(`pre/post_api_request`, `pre/post_tool_call`, `on_session_start`,
`on_session_end`, ...). Das spricht stark dafür, es einfach zu
aktivieren statt selbst etwas zu bauen.

Praktisches Problem: Langfuse-Self-Hosting erfordert offiziell Docker
(Postgres + ClickHouse + Redis + MinIO, kein leichtgewichtiger
nativer Pfad dokumentiert) — direkter Widerspruch zum
projektweiten "kein Docker"-Prinzip, das seit Sprint 1 durchgängig
eingehalten wurde (Honcho, Embedding-Server, alles nativ). Langfuse
Cloud wäre Docker-frei, sendet aber Trace-Daten (potenziell inkl.
Prompt-/Antwortinhalten) an einen externen Cloud-Dienst — Widerspruch
zu "Self Hosted".

## Entscheidung

Ein eigenes, minimales Hermes-Plugin (`~/.hermes/plugins/companion-tracing/`)
registriert dieselben offiziellen Hooks wie das gebündelte
Langfuse-Plugin, schreibt aber strukturierte JSON-Lines direkt lokal
nach `~/.hermes/logs/tracing/` statt an einen externen/Docker-basierten
Tracing-Server. Rein additiv — keine Hermes-Kerndatei verändert, exakt
im Rahmen der in [ADR 0005](0005-third-party-extension-policy.md)
etablierten Erweiterungspolitik (dort für Drittanbieter-Code
formuliert, hier für selbst geschriebenen, aber ebenso rein additiven
Code angewendet).

Bewusst NICHT instrumentiert, weil kein Hook dafür existiert (siehe
[docs/hermes/TRACING.md](../docs/hermes/TRACING.md) für die vollständige
Aufschlüsselung, was gemessen werden kann und was nicht):
Time-to-First-Token, Streaming-Dauer (Hermes läuft hier ohnehin mit
`streaming: false`), sowie "Prompt-Aufbereitung", "RAG", "Skill-
Selection" und "Tool-Planning" als eigenständige Phasen — diese laufen
intern innerhalb eines einzelnen LLM-Calls ab und sind nicht separat
hookbar, ohne Hermes-Kerncode zu patchen (ausdrücklich nicht erlaubt in
dieser Phase).

## Konsequenzen

- Kein neuer Dienst, kein Docker, keine externen Abhängigkeiten — die
  gesamte Instrumentierung lebt in einer einzigen Python-Datei im
  offiziell dokumentierten Plugin-Verzeichnis.
- Weniger komfortabel als Langfuses fertiges Web-Dashboard (keine
  Graphen, keine UI) — dafür exakt im Format, das angefragt wurde
  (JSON Lines + lesbare Textzusammenfassung).
- Nutzt für die LLM-Zeit Hermes' eigene, bereits berechnete
  `api_duration`/`usage`-Werte aus `post_api_request` statt eigener
  Zeitmessung — exakt, nicht approximiert.
- Sollte ein voll ausgestattetes Dashboard später gewünscht sein, bleibt
  der Wechsel zu selbstgehostetem Langfuse (mit expliziter
  Docker-Ausnahme) oder Langfuse Cloud eine spätere, bewusste
  Entscheidung — nicht ausgeschlossen, nur nicht jetzt gewählt.
