# Hermes Agent — Personas

Quellen: [Personality Feature](https://hermes-agent.nousresearch.com/docs/user-guide/features/personality),
[Use SOUL.md with Hermes](https://hermes-agent.nousresearch.com/docs/guides/use-soul-with-hermes),
[Profiles](https://hermes-agent.nousresearch.com/docs/user-guide/profiles)

## Schichtenmodell (drei Ebenen)

1. **SOUL.md** — dauerhafte Basis-Stimme, Fundament des Prompts (Slot
   #1). Siehe [WORKSPACE.md](WORKSPACE.md) für Details zu Ort/Format.
2. **`/personality [name]`** — Session-lokaler, temporärer Overlay/
   Moduswechsel, ändert SOUL.md nicht dauerhaft.
3. **Custom Personalities** — definiert in `config.yaml`.

**Gesamt-Prompt-Hierarchie:** SOUL.md (Fundament) → Tool-Guidance →
Memory-Kontext → Skills → Kontextdateien.

## Eingebaute Presets

12 Preset-Persönlichkeiten via `/personality [name]`, von technisch bis
verspielt. In der Doku namentlich genannt: kawaii, pirate, catgirl,
noir (vollständige 12er-Liste nicht an einer Stelle dokumentiert
gefunden).

## Beispiel-Personas aus der SOUL.md-Anleitung

Pragmatic Engineer, Research Partner, Teacher/Explainer, Tough Reviewer
— jeweils mit kurzer Ton-/Verhaltensbeschreibung. Ein einzeiliges
Beispiel aus den Tips: "You are a senior backend engineer. Be terse and
direct."

## Beziehung zu Profilen

SOUL.md ist Teil des pro-Profil isolierten Zustands — jedes Profil
(`~/.hermes/profiles/<name>/`) erhält seine eigene SOUL.md, sodass
verschiedene Profile parallel völlig unterschiedliche Personas fahren
können (siehe [ARCHITECTURE.md](ARCHITECTURE.md), Abschnitt Profiles).

## Best-Practice-Hinweise aus der Doku

Effektive SOUL.md-Dateien sollen "stable, broadly applicable, and
specific in voice" sein, generische Floskeln wie "be helpful" vermeiden;
iteratives Verfeinern statt Perfektion beim ersten Versuch wird
empfohlen.

**Nicht dokumentiert:** vollständige Liste aller 12 Presets; ob
`/personality`-Auswahl persistent gespeichert wird oder rein
session-lokal im Speicher bleibt.

---

## PixelHermes-Mapping

**Was übernimmt Hermes bereits?** Ein dreistufiges, bereits fertiges
Persona-System inklusive 12 Presets und klarer Trennregeln zu
projektbezogenem Kontext (AGENTS.md).

**Was müssen wir NICHT selbst entwickeln?** Kein eigenes
Persönlichkeits-/Prompt-Layering-System, keine eigenen Preset-Personas.

**Was passt direkt zu PixelHermes?** Die im ursprünglichen
Konzeptentwurf angedachten benannten "Agenten" lassen sich sauber als
je ein Hermes-**Profil** mit eigener SOUL.md abbilden — ein Profil pro
gewünschter Persona/Rolle, nicht als eigene Systembenutzer.

**Welche Erweiterungen wären später sinnvoll?** Eigene, PixelHermes-
spezifische SOUL.md-Inhalte je Profil, formuliert nach den
dokumentierten Best Practices (spezifisch, stabil, keine Floskeln) —
erst nach echter Installation und mit konkretem Bedarf, nicht
spekulativ vorab entworfen.

**Welche Komponenten sollten wir bewusst unverändert übernehmen?** Das
Schichtenmodell (SOUL.md → Personality-Overlay → Custom Personalities)
und die Preset-Persönlichkeiten vollständig unverändert übernehmen.
