# Hermes Agent — Modelle / Provider

Zusätzliches Dokument über die geforderten elf hinaus (`ADR`-Prinzip
"mindestens folgende Dokumente" erlaubt Ergänzungen) — deckt die
Kategorie "Modelle" aus dem Rechercheauftrag vollständig ab.

Quellen: [Configuration](https://hermes-agent.nousresearch.com/docs/user-guide/configuration),
[Providers](https://hermes-agent.nousresearch.com/docs/integrations/providers),
[Quickstart](https://hermes-agent.nousresearch.com/docs/getting-started/quickstart),
[Nous Portal](https://hermes-agent.nousresearch.com/docs/integrations/nous-portal),
[xAI Grok OAuth](https://hermes-agent.nousresearch.com/docs/guides/xai-grok-oauth),
[Local Ollama Setup](https://hermes-agent.nousresearch.com/docs/guides/local-ollama-setup),
[Local LLM on Mac](https://hermes-agent.nousresearch.com/docs/guides/local-llm-on-mac)

## OpenAI-kompatible Provider

```yaml
model:
  default: your-model-name
  provider: custom
  base_url: http://localhost:8000/v1
  api_key: your-key-or-leave-empty-for-local
```

"If a server implements `/v1/chat/completions`, you can point Hermes at
it." Mehrere benannte Custom-Provider über `custom_providers` möglich,
jeweils mit eigenem Credential-Pool. `provider: main` verweist auf den
für normalen Chat konfigurierten Provider, egal ob benannter Custom-
Provider, eingebauter Provider oder Legacy-`OPENAI_BASE_URL`.
Genannte Beispiele: vLLM, SGLang, Together.ai, RunPod. Umgekehrt
exponiert Hermes selbst einen OpenAI-kompatiblen Endpunkt für andere
Frontends (Open WebUI, LobeChat, LibreChat, u. a.).

**Nicht dokumentiert:** eine abschließende Liste aller "unterstützten"
OpenAI-kompatiblen Provider — bewusst offenes Muster statt Fixliste.

## Grok (xAI)

Als vollwertiger Provider dokumentiert, zwei Auth-Modi:

- **`xai-oauth`:** Geräte-Code-Flow über `auth.x.ai`, Credentials in
  `~/.hermes/auth.json`, benötigt SuperGrok-Abo oder X Premium+, kein
  API-Key nötig. Deckt Chat, TTS, Bild-/Videogenerierung, Transkription
  ab. Bekanntes Problem: HTTP-403 bei bestimmten SuperGrok-Tiers,
  Workaround: API-Key-Modus.
- **`xai` (direkte API):** `XAI_API_KEY` in `.env`, Auswahl über `hermes
  model`. Läuft über die Responses API mit automatischem
  Reasoning-Support für Grok-4-Modelle.

Modelle laut OAuth-Guide: `grok-build-0.1` (Default),
`grok-4.20-0309-reasoning`, `grok-imagine-image`, `grok-imagine-video`.

### Real produktiv konfiguriert — hermes_hugo (Phase 6, 2026-07-24)

`provider: xai` (direkte API, kein OAuth) mit `XAI_API_KEY`,
`model.default: grok-build-0.1` — funktioniert für Chat, Coding und
Tool-Calling (terminal/file), siehe
[PRODUCTIVE_RUNTIME.md](PRODUCTIVE_RUNTIME.md). **Wichtiger, ungelöster
Fund:** Web-Suche über das `web_search`-Tool (Exa-Backend) wird vom
Modell in mehreren Tests nicht real aufgerufen — es generiert
stattdessen überzeugend aussehende, aber erfundene "Tool-Ergebnisse"
als Text. Vor jeder produktiven Nutzung von Websuche-Antworten dieses
Setups: Details und Belege in
[PRODUCTIVE_RUNTIME.md](PRODUCTIVE_RUNTIME.md#aufgabe-2--exa) prüfen.

## OpenRouter

Vollwertiger, benannter Provider. Setup via `OPENROUTER_API_KEY` in
`.env`. Dediziertes Provider-Routing:

```yaml
provider_routing:
  sort: "throughput"
  # only: ["anthropic"]
```

Namens-Suffixe wie `:nitro` (Durchsatz) oder `:floor` (Preis).
`extra_body` wird 1:1 an OpenRouter durchgereicht. **Nous Portal läuft
technisch über OpenRouter** — Modellverfügbarkeit/Failover entsprechen
einem OpenRouter-Key, nur über das Nous-Abo abgerechnet.

## Lokale Modelle

Dokumentierte Runtimes: **Ollama** (eigener Guide), **vLLM**
("Standard für Produktions-Serving"), **SGLang** (Prefix-Caching),
**llama.cpp**, **MLX/omlx** (Mac-spezifisch, native Metal-Optimierung).

**Ollama-Beispiel:**
```yaml
model:
  default: "gemma4:31b"
  provider: "custom"
  base_url: "http://localhost:11434/v1"
```

**Zentrale, wiederholt dokumentierte Anforderung:** mindestens **64.000
Token Kontext** — kleinere Fenster reichen laut Doku nicht für
mehrstufige Tool-Calling-Workflows.

**Tool-Calling-Einschränkung bei lokalen Modellen (wichtig):** laut
Ollama-Guide-Tabelle unterstützt nur `gemma4:31b` zuverlässig
Tool-Calling; kleinere Modelle (`gemma2:27b`, `gemma2:9b`,
`llama3.2:3b`) ignorieren häufig Funktionsaufruf-Anweisungen. Hermes hat
eine "Auto-Repair"-Funktion für fehlerhafte Tool-Calls; bei anhaltenden
Problemen: größeres Modell, Cloud-Fallback-Provider, oder `/compress`.
llama.cpp benötigt explizit das `--jinja`-Flag für Tool-Calling; vLLM/
SGLang benötigen explizite `--tool-call-parser`-Flags.

**Nicht dokumentiert:** LM Studio wird in den gecrawlten Seiten nicht
namentlich erwähnt (passt aber vermutlich generisch über den
Custom-Endpoint-Pfad — nicht als Doku-Fakt zu behandeln, nur als
Vermutung).

## Nous Portal (Zusatzintegration)

"Recommended way to run Hermes Agent" — ein Befehl (`hermes setup
--portal`), Zugriff auf 300+ Modelle (Anthropic, OpenAI, Google, u. v.
a.) über OpenRouter-Infrastruktur, inkl. "Tool Gateway" (Web-Suche,
Bildgenerierung, TTS, Browser, Cloud-Terminal-Sandboxes).

---

## PixelHermes-Mapping

**Was übernimmt Hermes bereits?** Eine vollständige, produktionsreife
Provider-Abstraktion für 30+ Anbieter inklusive lokaler Modelle, mit
dokumentierten Einschränkungen (Kontextlänge, Tool-Calling-Zuverlässigkeit)
statt Marketing-Versprechen.

**Was müssen wir NICHT selbst entwickeln?** Keine eigene
Provider-Abstraktionsschicht, keinen eigenen OpenAI-kompatiblen Proxy.

**Was passt direkt zu PixelHermes?** "Self Hosted" aus
[ARCHITECTURE.md](../../ARCHITECTURE.md) spricht für den `custom`-
Provider-Pfad mit einem selbst gehosteten OpenAI-kompatiblen Server
(vLLM/Ollama) als langfristiges Ziel — die Doku bestätigt, dass dieser
Weg vollständig unterstützt ist, inklusive der wichtigen 64k-Kontext-
und Tool-Calling-Einschränkungen, die vorab in die Modellauswahl
einfließen müssen.

**Welche Erweiterungen wären später sinnvoll?** Vor der ersten
produktiven Installation ein konkretes Modell (Cloud oder lokal)
festlegen, das die 64k-Kontext- und Tool-Calling-Anforderung
nachweislich erfüllt — als eigener [ADR](../../ADR/), sobald Sprint 3
beginnt.

**Welche Komponenten sollten wir bewusst unverändert übernehmen?** Die
gesamte Provider-Konfigurationslogik (`config.yaml`/`.env`-Muster,
`hermes model`-Wizard) unverändert übernehmen — keine eigene
Modell-Auswahl-Logik bauen.
