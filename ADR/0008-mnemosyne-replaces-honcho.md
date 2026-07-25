# 0008. Mnemosyne ersetzt Honcho als Memory-Provider

Status: Accepted

## Kontext

Honcho (Sprint 7, [ADR 0004](0004-local-embedding-server-for-honcho.md))
lief produktiv als self-hosted Memory-Provider: eigener PostgreSQL 17 +
pgvector + Redis-Stack, zwei systemd-Dienste
(`companion-honcho-api`, `companion-honcho-deriver`), Reasoning-Schritt
über Grok. Das eigene Tracing-Plugin ([ADR 0007](0007-local-tracing-plugin-not-langfuse.md))
deckte dabei real einen erheblichen Flaschenhals auf: `honcho_reasoning`
allein brauchte in einem gemessenen Request 23,6 s von 54,6 s Gesamtzeit
(siehe [docs/hermes/TRACING.md](../docs/hermes/TRACING.md)) — Honchos
Reasoning-Schritt ruft selbst einen LLM-Call auf, was Speicher-Recall
strukturell teuer macht.

`mnemosyne-hermes` (PyPI, Repo `AxDSan/mnemosyne`) wurde als
Alternative recherchiert und getestet. Wichtige Einordnung, die nicht
verschwiegen wird: dies ist **kein offizielles Nous-Research-Produkt**,
sondern eine einzeln gepflegte Drittanbieter-Bibliothek (MIT, ~1.8k
Stars, 893 Commits, Maintainer "Abdias J" / AxDSan). Der Nutzer wurde
darauf hingewiesen und hat die Installation ausdrücklich freigegeben
("wir haben uns unterhalten und ist in ordnung zu installieren").
Bewertung nach der in [ADR 0005](0005-third-party-extension-policy.md)
etablierten Vier-Punkte-Prüfung für Drittanbieter-Erweiterungen:
offizieller Installationsweg (`pip install mnemosyne-hermes`, offizielle
Hermes-Plugin-Mechanik via `entry_points`), keine Kernänderung an
Hermes, reversibel (deaktivierbar über `hermes plugins disable`,
Provider zurück auf Honcho umstellbar), aktiv gepflegt.

## Migrationsweg (real geprüft, nicht angenommen)

Honchos Daten wurden **nicht** automatisch übernommen. Direkte Prüfung
von Mnemosynes eigener Dokumentation (`comparison.md` im
`AxDSan/mnemosyne`-Repo) ergab: Honcho wird darin nicht einmal erwähnt,
es existiert kein `hermes mnemosyne import --from honcho` oder
Vergleichbares. Es wurde **keine Eigenentwicklung** für eine Migration
gebaut (ausdrücklicher Teil des Auftrags) — Mnemosynes Gedächtnis
startet leer und baut sich ab Umstellungszeitpunkt neu auf.

## Duplicate-Registration-Frage (real verifiziert)

Bei `hermes plugins list` erscheinen zwei Einträge
(`hermes-mnemosyne` 0.4.0, Quelle `user`, aus
`~/.hermes/plugins/mnemosyne/`, sowie `mnemosyne` 0.5.0, Quelle
`entrypoint`). Quellcode-Analyse von `plugins/memory/__init__.py`
zeigte: der verzeichnisbasierte Loader lädt Provider ausschließlich in
einen isolierten Namensraum für `hermes memory status`/`hermes memory
setup` (CLI-Introspektion) — er speist **nicht** den Hook-Dispatcher zur
Laufzeit. Nur der `entrypoint`-Eintrag ist tatsächlich "enabled" und
aktiv. Empirisch bestätigt über zwei unabhängige Wege: (1) das eigene
Tracing-Plugin zeigte für einen einzelnen `mnemosyne_remember`-Aufruf
exakt `tool_call_count: 1`, keine Duplikate; (2) direkte SQLite-Abfrage
der `working_memory`-Tabelle zeigte für den gespeicherten Testwert genau
eine Zeile. Hooks feuern nachweislich nur einmal.

## Entscheidung

`memory.provider` wird auf `mnemosyne` gesetzt, Honcho bleibt zunächst
deaktiviert (gestoppt, `disabled`, aber nicht deinstalliert) bis zur
vollständigen Verifikation, danach vollständig entfernt (separate
Phase). Mnemosyne läuft **in-process als Hermes-Plugin** (kein eigener
systemd-Dienst, keine eigene Datenbank-Infrastruktur) mit lokalem
SQLite unter `~/.hermes/mnemosyne/data/mnemosyne.db` und lokalen
Embeddings via `fastembed` (kein Cloud-Key nötig, kein externer
Dienst — erfüllt "Self Hosted").

## Konsequenzen

- **Massiv geringere Recall-Latenz**: gemessen `mnemosyne_recall`
  ⌀ 29,2 ms (n=40) vs. Honchos `honcho_reasoning` 23.588 ms (Einzelmessung,
  Sprint 7) — kein separater LLM-Reasoning-Call mehr nötig für Recall.
  Details in [docs/hermes/MNEMOSYNE.md](../docs/hermes/MNEMOSYNE.md).
- **Weniger Infrastruktur**: kein dedizierter PostgreSQL/Redis-Stack,
  keine zwei zusätzlichen systemd-Dienste, kein dedizierter
  `honcho`-Systembenutzer nötig — Speicherbedarf sinkt entsprechend
  (Honcho-Prozesse zusammen ~617 MB systemd-`MemoryCurrent`, Mnemosyne
  läuft im ohnehin laufenden Hermes-Gateway-Prozess, ~116 MB).
- **Kein offizieller Support-Status**: anders als Hermes selbst oder
  Hermes Agent Self-Evolution ist Mnemosyne einzeln gepflegt — Risiko
  für PixelHermes, transparent dokumentiert, vom Nutzer akzeptiert.
- **Bekannte Recall-Inkonsistenz** real beobachtet (nicht vermutet):
  ein direkt danach gefragtes Fact wurde zuverlässig gefunden, dieselbe
  Information verschwand aber teils aus einer Multi-Marker-Listenabfrage
  trotz bestätigt vorhandenem DB-Eintrag — dokumentiert als bekannte
  Grenze, siehe [docs/hermes/MNEMOSYNE.md](../docs/hermes/MNEMOSYNE.md#bekannte-einschränkungen).
- Honcho-Entfernung (Pakete, Datenbanken, Dienste, Systembenutzer)
  erfolgt als eigene, spätere Phase erst nach dieser vollständigen
  Verifikation.
