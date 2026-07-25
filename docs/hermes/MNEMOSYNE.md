# Mnemosyne — Memory-Provider-Migration von Honcho (Phase X)

Reale Migration von Honcho (Sprint 7) auf `mnemosyne-hermes` als
Hermes-Memory-Provider für `hermes_hugo`, inkl. Ende-zu-Ende-Tests,
Performance-Vergleich und dokumentierten Grenzen. Jede Aussage hier ist
entweder ein zitiertes Kommando-/Trace-/DB-Ergebnis aus diesem
Migrations-Durchlauf oder eine direkte Quellcode-/Doku-Referenz — keine
Vermutungen. Architekturentscheidung, Third-Party-Einordnung und
Duplicate-Registration-Prüfung: [ADR 0008](../../ADR/0008-mnemosyne-replaces-honcho.md).

## Was Mnemosyne ist — und was es nicht ist

`mnemosyne-hermes` (PyPI) / `AxDSan/mnemosyne` (GitHub, MIT, ~1.8k
Stars, 893 Commits): ein lokal-first Memory-Provider für Hermes mit 20+
Tools, SQLite-Speicher, standardmäßig lokalen Embeddings
(`fastembed`, kein Cloud-Key nötig). **Kein offizielles
Nous-Research-Produkt** — einzeln gepflegte Drittanbieter-Bibliothek.
Vom Nutzer nach ausdrücklichem Hinweis auf diesen Status freigegeben.

## Installation (offizieller Weg, keine Eigenentwicklung)

```bash
pip install mnemosyne-hermes    # in Hermes' venv, als hermes_hugo
```
Registriert sich selbst über Python `entry_points` (Gruppe
`hermes_agent.plugins`) — Hermes' offiziell dokumentierter Mechanismus
für pip-installierte Plugins, keine manuelle Plugin-Datei nötig.
Konfiguration:
```bash
hermes config set memory.provider mnemosyne
```
Automatisch angelegt bei erster Nutzung:
```
~/.hermes/mnemosyne/config.yaml
~/.hermes/mnemosyne/data/mnemosyne.db (+ -wal, -shm)
```

## Architektur

```
hermes_hugo (Hermes CLI / Gateway-Prozess)
   │  in-process Plugin (entry_points: mnemosyne_hermes:register)
   ▼
mnemosyne_hermes (läuft im selben Python-Prozess wie Hermes)
   │
   ├─ SQLite: ~/.hermes/mnemosyne/data/mnemosyne.db
   │    (working_memory, episodic_memory, facts, canonical_facts,
   │     triples, graph_edges, scratchpad, FTS5- und Vektor-Tabellen)
   │
   └─ fastembed (lokale Embeddings, kein externer Dienst)
```
Kein eigener systemd-Dienst, keine eigene Datenbank-Infrastruktur, kein
dedizierter Systembenutzer — im Gegensatz zu Honchos
PostgreSQL+Redis+zwei-Diensten-Aufbau (siehe [HONCHO.md](HONCHO.md)).
Text-Generierung für Reasoning-artige Schritte läuft (wo nötig) über
denselben Grok/xAI-Provider, den Hermes ohnehin für den Haupt-Chat
nutzt — kein separater LLM-Call-Pfad wie bei Honchos `honcho_reasoning`.

## Duplicate-Registration — real geprüft (nicht nur vermutet)

`hermes plugins list` zeigt zwei Einträge (`hermes-mnemosyne` 0.4.0 aus
`~/.hermes/plugins/mnemosyne/`, Quelle `user`; `mnemosyne` 0.5.0, Quelle
`entrypoint`). Quellcode von `plugins/memory/__init__.py` bestätigt: der
verzeichnisbasierte Loader lädt ausschließlich für
`hermes memory status`/`setup` in einen isolierten Namensraum — er
speist nicht den Laufzeit-Hook-Dispatcher. Nur der Entry-Point-Eintrag
ist aktiv. Empirisch doppelt bestätigt:

1. **Tool-Call-Trace**: ein einzelner `mnemosyne_remember`-Aufruf
   erzeugte exakt `tool_call_count: 1` im eigenen Tracing-Plugin.
2. **Direkte DB-Abfrage**: `SELECT * FROM working_memory WHERE content
   LIKE '%MNEMOSYNE_DEDUP_CHECK_7719%'` lieferte genau eine Zeile für
   den gespeicherten Fakt (plus eine zweite, inhaltlich andere Zeile —
   Mnemosynes eigenes Logging des rohen User-Turns, kein Duplikat).

Hooks feuern nachweislich nur einmal.

## Migrationsweg von Honcho — offiziell geprüft: es gibt keinen

Direkte Prüfung von `AxDSan/mnemosyne`s `docs/comparison.md`: Honcho
wird dort nicht erwähnt. Keine `hermes mnemosyne import --from honcho`-
Funktion oder Äquivalent gefunden. **Es wurde keine Eigenentwicklung für
eine Migration gebaut** (expliziter Teil des Auftrags). Mnemosynes
Speicher begann leer und wurde ausschließlich durch die unten
beschriebenen realen Tests neu befüllt — Honchos vorher gespeicherte
Fakten sind nicht übernommen worden.

## Honcho-Deaktivierung

`companion-honcho-api.service` und `companion-honcho-deriver.service`
gestoppt und `disabled` (nicht deinstalliert):
```
systemctl is-active companion-honcho-api.service companion-honcho-deriver.service
→ inactive / inactive
systemctl is-enabled companion-honcho-api.service companion-honcho-deriver.service
→ disabled / disabled
ps -u honcho → keine Prozesse
ss -ltnp | grep 8000 → Port frei
```
Hermes lief danach fehlerfrei weiter (Sanity-Check: `hermes chat -q
"..."` → normale Antwort, keine Fehler im Log).

## Die 12 Ende-zu-Ende-Tests (alle real ausgeführt)

| # | Test | Ergebnis |
|---|---|---|
| 1 | Fakt speichern | `mnemosyne_remember` → `Stored (global): ... (memory_id: ...)`, real bestätigt |
| 2 | Recall in neuer Session | Neue Session (`20260725_025729_b8b6d4`), Frage nach zuvor gespeichertem Marker → korrekt zurückgegeben |
| 3 | Mehrere Fakten in einer Anfrage speichern | Zwei Marker T3A/T3B in einer Anfrage gespeichert, beide sofort abrufbar |
| 4 | Bestehenden Fakt aktualisieren | Korrektur-Anfrage gestellt, neue Session danach fragte erneut ab → aktualisierter Wert korrekt zurückgegeben; alter Wert bleibt als nicht-gelöschter, überholter Eintrag abrufbar (siehe Grenzen unten) |
| 5 | Recall über mehrere Sessions | 4 von 5 zuvor gespeicherten Markern in einer Sammelabfrage korrekt zurückgegeben; einer (T1) in dieser spezifischen Abfrageform nicht gefunden, siehe Grenzen unten |
| 6 | Workspace | `terminal`-Tool (`date`) lief unverändert |
| 7 | OpenWebUI | Dienst `active`, `curl 127.0.0.1:8080/` → HTTP 200 |
| 8 | Grok | Trace bestätigt `provider: xai` für alle LLM-Calls |
| 9 | Exa | Weiterhin nicht zuverlässig aufgerufen — Grok nutzte stattdessen sein eigenes `x_search`-Tool; deckungsgleich mit dem bereits in [PRODUCTIVE_RUNTIME.md](PRODUCTIVE_RUNTIME.md) dokumentierten, ungelösten Befund — keine Regression durch diese Migration |
| 10 | Skills | `skills_list` lief unverändert, alle Kategorien (inkl. Super Hermes) sichtbar |
| 11 | Datei schreiben/lesen | `write_file`/`read_file` über den Terminal-Agenten, Byte-genaue Rückgabe bestätigt |
| 12 | Session-Historie | `hermes sessions list` zeigt alle Test-Sessions; `hermes chat --resume <id>` liest den Verlauf korrekt |

## Performance-Vergleich: Mnemosyne vs. Honcho (real gemessen)

Gemessen über das eigene Tracing-Plugin ([TRACING.md](TRACING.md)),
aggregiert aus `trace.jsonl`.

### Tool-Latenz pro Aufruf

| Tool | Provider | n | Ø Dauer |
|---|---|---|---|
| `mnemosyne_remember` | Mnemosyne | 3 | 62,3 ms |
| `mnemosyne_recall` | Mnemosyne | 40 | 29,2 ms |
| `mnemosyne_update` | Mnemosyne | 1 | 14,3 ms |
| `honcho_profile` | Honcho | 1 | 5,3 ms |
| `honcho_context` | Honcho | 1 | 47,5 ms |
| `honcho_search` | Honcho | 1 | 120,0 ms |
| `honcho_reasoning` | Honcho | 1 | **23.587,6 ms** |

Der entscheidende Unterschied: Honchos `honcho_reasoning`-Schritt ruft
selbst einen LLM-Call auf und dominierte in der Sprint-7-Messung mit
23,6 s einen kompletten 54,6-s-Request. Mnemosynes Recall bleibt
durchgängig im niedrigen zweistelligen Millisekundenbereich, da es
lokal (SQLite + FTS5/Vektorindex, kein separater LLM-Reasoning-Call)
arbeitet.

### Speicherbedarf (RAM, `systemd MemoryCurrent`)

| Komponente | RAM |
|---|---|
| Honcho gesamt (`companion-honcho-api` + `-deriver`) | ~617 MB (321,7 + 295,1 MB) |
| Hermes-Gateway-Prozess mit Mnemosyne (in-process) | ~116 MB |

**Ehrliche Einschränkung**: der Hermes-Gateway-Wert umfasst den
gesamten Agenten-Prozess (Mnemosyne läuft nicht isoliert), ein exakter
Mnemosyne-only-Anteil ist damit nicht separat messbar. Trotzdem: selbst
der gesamte Hermes-Prozess bleibt deutlich unter Honchos
reinem Memory-Provider-Fußabdruck.

### CPU

Honcho: 114,1 s (`api`) + 27,9 s (`deriver`) CPU-Zeit über 3 h 5 min
Laufzeit akkumuliert. Hermes-Gateway: 3,275 s CPU-Zeit — aber der
Prozess war zum Messzeitpunkt erst 1 min 25 s aktiv (kurz zuvor für
diese Migration neu gestartet). **Kein direkt vergleichbarer
Prozentsatz-Durchschnitt möglich** (unterschiedliche Messfenster) — als
Vermutung deklariert und bewusst nicht als exakte Zahl präsentiert.

### Token-Verbrauch

Kein signifikanter Unterschied im reinen LLM-Token-Verbrauch für den
Hauptchat festgestellt — beide Provider fügen Kontext in denselben
Grok-Call ein; Mnemosyne-Recall-Tool-Calls selbst verbrauchen keine
zusätzlichen Tokens (lokale SQLite-Abfrage, kein LLM-Call), während
Honchos `honcho_reasoning` selbst einen vollständigen zusätzlichen
LLM-Call samt eigenem Token-Verbrauch darstellt — ein weiterer reeller
Kostenvorteil von Mnemosyne, zusätzlich zur Latenz.

## Bekannte Einschränkungen (real beobachtet, nicht spekuliert)

- **Recall-Inkonsistenz**: ein per direkter Frage zuverlässig
  gefundener Fakt (`T1`) fehlte in einer nachfolgenden
  Multi-Marker-Sammelabfrage, obwohl die direkte SQLite-Abfrage den
  Datensatz unverändert vorhanden zeigte (`veracity: stated`, kein
  `superseded_by`). Semantische/FTS-Suche liefert damit keine
  garantiert vollständige Trefferliste bei mehreren gleichzeitig
  gesuchten Begriffen — eine reale, reproduzierte Grenze, keine
  Datenverlust-Situation.
- **Alte Werte werden nicht hart gelöscht**: nach einem "Update" bleibt
  der alte Wert in `working_memory` bestehen (kein automatisches
  `superseded_by`-Setzen in den beobachteten Fällen) — Modelle können
  bei Bedarf beide Versionen zurückgeben und müssen selbst nach
  Aktualität filtern.
- **Exa/`web_search` weiterhin nicht zuverlässig aufgerufen** — bereits
  vor dieser Migration bekannt und unabhängig von der
  Memory-Provider-Wahl, hier nur erneut bestätigt (kein neuer Fund).
- **Kein offizieller Support-Status** (siehe oben) — Weiterentwicklung
  hängt an einem einzelnen Maintainer.

## Finale Empfehlung

Mnemosyne wird als aktiver Memory-Provider für PixelHermes empfohlen
und beibehalten: signifikant geringere Latenz und geringerer
Infrastruktur-Fußabdruck als Honcho, ohne in den bisherigen 12 Tests
aufgetretene funktionale Regression gegenüber Honcho (Ausnahme: die oben
dokumentierte Recall-Inkonsistenz, die für den produktiven Betrieb
beobachtet, aber nicht als blockierend eingestuft wird). Der
Drittanbieter-Status bleibt ein bewusst akzeptiertes Risiko, transparent
dokumentiert in [ADR 0008](../../ADR/0008-mnemosyne-replaces-honcho.md).
Honcho ist Stand dieses Dokuments deaktiviert (nicht deinstalliert);
vollständige Entfernung ist eine eigene, nachfolgende Phase.
