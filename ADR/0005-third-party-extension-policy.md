# 0005. Bedingte Zulassung von Drittanbieter-Erweiterungen

Status: Accepted

## Kontext

Sprint 7 wollte ursprünglich ausschließlich offiziell unterstützte
Erweiterungen verwenden ("Es werden ausschließlich offiziell
unterstützte Erweiterungen verwendet. Keine Forks."). Im Sprintverlauf
wurde mit **Super Hermes** (`Cranot/super-hermes`) eine reale,
funktionierende, aber nicht-offizielle (Einzelentwickler,
Drittanbieter, MIT-lizenziert) Skill-Sammlung zur Installation
vorgeschlagen. Eine strikte Auslegung von "nur offiziell" hätte das
kategorisch ausgeschlossen — das erschien nach Prüfung des konkreten
Falls zu starr: das Repository fügt ausschließlich Skills über Hermes'
eigenen, offiziell dokumentierten Skill-Mechanismus hinzu und ändert
laut eigener Aussage ("does not modify core files") und eigener
Quellcode-Prüfung keine Hermes-Kerndateien.

## Entscheidung

Die Regel "nur offizielle Erweiterungen" wird präzisiert statt
aufgeweicht: **offizielle Nous-Research-Projekte bleiben bevorzugt.**
Drittanbieter-Erweiterungen dürfen zusätzlich verwendet werden, wenn
**alle** folgenden Bedingungen erfüllt sind:

1. Sie verändern keine Hermes-Kerndateien.
2. Sie nutzen ausschließlich Hermes' eigene, dokumentierte
   Erweiterungsmechanismen (Skills, Plugins, MCP — nicht z. B.
   Patches am `hermes-agent`-Checkout selbst).
3. Sie sind vollständig rückstandsfrei entfernbar (reines
   Dateisystem-Feature, keine Datenbank-/Config-Seiteneffekte, die
   ein Rollback erschweren).
4. Sie werden im Repo klar als optionale Drittanbieter-Komponente
   dokumentiert, nicht als offizieller Hermes-Bestandteil dargestellt.

Vor jeder Installation: Repository-Inhalt vollständig lesen (nicht nur
README), keine automatisierten Install-Skripte Dritter blind
ausführen — stattdessen die tatsächlich benötigten Dateien manuell an
die dokumentierten Zielorte kopieren.

## Konsequenzen

- Super Hermes qualifiziert sich unter dieser Regel (siehe
  [docs/hermes/SUPER_HERMES.md](../docs/hermes/SUPER_HERMES.md) für die
  Prüfung im Detail) und wurde entsprechend installiert.
- Zukünftige Drittanbieter-Vorschläge werden an denselben vier
  Kriterien gemessen, nicht Fall für Fall neu diskutiert.
- Erhöhtes, aber begrenztes Vertrauensrisiko: Drittanbieter-Skills sind
  reiner Prompt-Text (keine ausführbare Logik) — das Risiko bleibt auf
  "das Modell befolgt möglicherweise unerwünschte Anweisungen"
  begrenzt, nicht auf "beliebiger Code läuft mit Systemrechten".
- Erweiterungen, die Kerndateien patchen oder nicht sauber entfernbar
  sind (z. B. Humalike, das Hermes-interne Turn-Taking-Mechanismen
  hookt), bleiben unter dieser Regel weiterhin ausgeschlossen.
