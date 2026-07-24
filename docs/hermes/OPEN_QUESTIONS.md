# Hermes Agent — Open Questions

Alle Punkte, die in der offiziellen Dokumentation nicht (vollständig)
geklärt sind, gesammelt aus den Einzelkapiteln. Diese sollten vor
sicherheits- oder architekturrelevanten Entscheidungen in Sprint 3
gezielt nachrecherchiert oder direkt getestet werden — nicht durch
Vermutung ersetzt werden.

## Core / Architecture

- Exakte Mindestversionen für Windows/macOS jenseits von "10/11" /
  "Apple Silicon".
- Kein vollständiges, kanonisches `config.yaml`-Beispiel mit allen
  Top-Level-Keys an einer Stelle (nur "150+ Felder" im Dashboard
  referenziert).
- Ob `hermes chat` und bare `hermes` identisch sind oder sich subtil
  unterscheiden — nicht explizit gesagt.
- Exakter Verhaltensunterschied zwischen `hermes serve` und `hermes
  dashboard` jenseits von "headless".
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
- Keine einzelne, vollständige Enumeration aller gebündelten Skills und
  Kategorien an einer Stelle gefunden.
- Keine dokumentierte Regel für semantische (nicht namensbasierte)
  Skill-Konflikte.
- Uneinheitliche Toolzahl-Angaben zwischen Docpages ("30+" bis "70+") —
  als Bandbreite behandeln, nicht als Fixwert.
- Keine dokumentierte Sandboxing-Technologie (seccomp/AppArmor o. Ä.)
  für den lokalen `execute_code`-Python-Kindprozess jenseits von
  Env-Var-Stripping und Ressourcenlimits.

## MCP

- Katalogstand (`optional-mcps/`: blender, linear, n8n, unreal-engine)
  wurde am 2026-07-24 per GitHub-API geprüft — kann sich ändern, vor
  Nutzung erneut prüfen.
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

## Vorgehen bei diesen Punkten

Vor jeder Entscheidung, die von einem dieser offenen Punkte abhängt: die
verlinkte offizielle Seite erneut direkt abrufen (Stand kann sich durch
laufende Weiterentwicklung ändern) oder — wo sinnvoll — im Rahmen der
Installation in Sprint 3 direkt am System verifizieren, statt zu raten.
