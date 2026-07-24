# Hermes Agent — Open Questions

Alle Punkte, die in der offiziellen Dokumentation nicht (vollständig)
geklärt sind, gesammelt aus den Einzelkapiteln. Diese sollten vor
sicherheits- oder architekturrelevanten Entscheidungen in Sprint 3
gezielt nachrecherchiert oder direkt getestet werden — nicht durch
Vermutung ersetzt werden.

## Core / Architecture

- Exakte Mindestversionen für Windows/macOS jenseits von "10/11" /
  "Apple Silicon".
- ~~Kein vollständiges, kanonisches `config.yaml`-Beispiel~~ — **teilweise
  geklärt durch reale Installation (Phase 4):** ein echtes,
  frisch-generiertes `config.yaml` (Schema v33) existiert jetzt unter
  `/srv/companion/hermes_hugo/.hermes/config.yaml`; die vollständige
  Liste "199 optionale API-Keys" wurde bei `hermes config migrate` real
  ausgegeben (siehe [INSTALLATION.md](INSTALLATION.md)). Weiterhin keine
  einzelne Doku-Seite mit allen Top-Level-Keys.
- Ob `hermes chat` und bare `hermes` identisch sind oder sich subtil
  unterscheiden — nicht explizit gesagt, in Phase 4 nicht getestet
  (nicht Teil des Verifikationsumfangs).
- Exakter Verhaltensunterschied zwischen `hermes serve` und `hermes
  dashboard` jenseits von "headless" — **Beobachtung aus Phase 4:**
  `hermes serve --stop` verweist in seiner Ausgabe auf "Restart the
  dashboard when you're ready: hermes dashboard --port <port>", was
  nahelegt, dass beide denselben Prozess-/State-Mechanismus teilen —
  löst die Frage nicht endgültig auf, ist aber ein neuer Anhaltspunkt.
- Vollständige Liste der "administrativen Routen" des API-Servers (MCP,
  Channels, Webhooks, Pairing, Credential-Pooling, Hooks).
- Details zu `/docs/user-guide/profile-distributions` und
  `/docs/user-guide/multi-profile-gateways` wurden nicht vertieft
  recherchiert (nur die Existenz der Seiten notiert).
- Exaktes DDL/Schema aller sechs `state.db`-Schemakomponenten; genaue
  FTS5-Trigger-Bedingungen (Trigram vs. Standard).

## Workspace / Context

- Verhalten bei gleichzeitiger Existenz mehrerer konkurrierender
  Kontext-Dateitypen mit unterschiedlichem Inhalt (kein Beispiel in der
  Doku).
- Kein dokumentiertes Zeichen-/Tokenlimit für SOUL.md (im Gegensatz zu
  MEMORY.md/USER.md).
- Ob AGENTS.md jemals automatisch geseedet wird (Doku deutet "nein" an,
  aber nicht explizit ausgesagt).

## Memory / Reflection

- Exakte Trigger-Grammatik für Memory-Schreibvorgänge über die
  illustrativen Beispiele hinaus.
- "Reflection" ist **kein** offizieller Begriff — das funktionale
  Äquivalent (Background-Review/Curator) ist dokumentiert, der Name
  selbst nicht. Im Build Guide korrekte Terminologie verwenden.

## Skills

- Inkonsistente Terminologie zwischen zwei Docpages: "Level 0/1/2" vs.
  "Level 1/2/3" für das Progressive-Disclosure-Modell — funktional
  identisch, aber nicht vereinheitlicht.
- ~~Keine einzelne, vollständige Enumeration aller gebündelten Skills und
  Kategorien~~ — **geklärt durch reale Installation (Phase 4):** 65
  aktivierte builtin-Skills über 13 Kategorien real ausgelesen, siehe
  [SKILLS.md](SKILLS.md). Neue, ungeklärte Diskrepanz dabei entdeckt:
  Installer-Sync-Log meldete 69 synchronisierte Skills (inkl. einer
  scheinbar ausgeblendeten `apple`-Kategorie mit 5 Einträgen im
  Dateisystem), `hermes skills list` zeigt nur 65 — Ursache (evtl.
  plattformabhängiges Ausblenden) in der offiziellen Doku nicht erklärt.
- Keine dokumentierte Regel für semantische (nicht namensbasierte)
  Skill-Konflikte.
- Uneinheitliche Toolzahl-Angaben zwischen Docpages ("30+" bis "70+") —
  als Bandbreite behandeln, nicht als Fixwert.
- Keine dokumentierte Sandboxing-Technologie (seccomp/AppArmor o. Ä.)
  für den lokalen `execute_code`-Python-Kindprozess jenseits von
  Env-Var-Stripping und Ressourcenlimits.

## MCP

- Katalogstand (`optional-mcps/`: blender, linear, n8n, unreal-engine)
  wurde am 2026-07-24 per GitHub-API geprüft — **bestätigt durch reale
  Installation am selben Tag** (`hermes mcp catalog` auf hermes_hugo
  zeigt identische vier Einträge) — kann sich künftig trotzdem ändern,
  vor Nutzung erneut prüfen.
- Kein offiziell benannter/empfohlener Office-MCP-Server.

## Deployment

- Kein offizielles, vollständiges Beispiel einer generierten
  systemd-Unit-Datei.
- Keine offizielle Nginx/Caddy/Traefik-Anleitung mit Beispielkonfig;
  WebSocket-Timeout-Hinweise aus Websuche nicht gegen Primärquelle
  verifiziert.
- Kein dokumentiertes OS-Level-Sandboxing zwischen Profilen auf
  demselben Host; kein Ressourcen-Quota-Mechanismus pro Gateway-Nutzer.
- Kein Standard-Loglevel für Produktion dokumentiert; keine
  Gesamt-Retention-Policy für Logs außerhalb der Docker-s6-Archivzahl.
- Keine empfohlene Backup-Frequenz/-Retention, keine Offsite-Anleitung,
  kein Point-in-Time-Restore-Verfahren.
- SSH-/Singularity-/Modal-/Daytona-Backend-Details wurden nur über
  Suchergebnis-Zusammenfassungen erschlossen, nicht direkt aus einer
  Primärquelle zitiert — vor produktivem Einsatz erneut verifizieren.

## Modelle

- Keine abschließende Liste aller "unterstützten" OpenAI-kompatiblen
  Provider (bewusst offenes Muster).
- LM Studio nicht namentlich erwähnt — Kompatibilität über den
  Custom-Endpoint-Pfad ist plausibel, aber nicht dokumentiert bestätigt.
- Keine Preisangaben für direkte xAI-API-Nutzung; keine explizite
  Aussage zur Tool-Calling-Zuverlässigkeit bei Grok-Modellen (im
  Gegensatz zu den expliziten Angaben bei lokalen Modellen).

## Neu aus der realen Installation (Phase 4, hermes_hugo, 2026-07-24)

- **`web`/`pty`-Extras:** die Doku (Feature-Seite API/Web-Dashboard)
  behauptet, die Standardinstallation enthalte keine HTTP/PTY-
  Abhängigkeiten. Real beobachtet: sie waren bereits vorhanden (Teil des
  hash-verifizierten `[all]`-Installs). Unklar, ob sich das Verhalten
  zwischen Installer-Versionen geändert hat oder die Doku-Aussage sich
  auf einen anderen Installationsweg bezieht (z. B. Docker-Image) — nicht
  aufgelöst.
- **Config-Template-Inkonsistenz:** `hermes config migrate` meldet
  `platform 'teams' references unknown toolset 'hermes-teams'` und
  `platform 'google_chat' references unknown toolset 'hermes-google_chat'`
  auf einer komplett unveränderten Neuinstallation — wirkt wie ein Fehler
  im mitgelieferten Default-Config-Template selbst, nicht wie eine
  Fehlkonfiguration unsererseits. In der offiziellen Doku nicht erwähnt.
- **Playwright/Chromium-Installation:** `npx playwright install-deps
  chromium` (als root, im Repo-Verzeichnis) schlug mit `playwright: not
  found` fehl, obwohl `node_modules/` in mehreren Unterordnern existiert
  — die tatsächliche Node-Package-Struktur für die Browser-Engine wurde
  nicht weiter untersucht (außerhalb des Phase-4-Scopes). Browser-Tools
  sind auf dieser Installation entsprechend nicht nutzbar
  (`hermes doctor`: "Playwright Chromium not installed").
- **Skill-Zähldiskrepanz** 69 (Sync-Log) vs. 65 (`hermes skills list`) —
  siehe Abschnitt "Skills" oben.

## Vorgehen bei diesen Punkten

Vor jeder Entscheidung, die von einem dieser offenen Punkte abhängt: die
verlinkte offizielle Seite erneut direkt abrufen (Stand kann sich durch
laufende Weiterentwicklung ändern) oder — wo sinnvoll — im Rahmen der
Installation in Sprint 3 direkt am System verifizieren, statt zu raten.
