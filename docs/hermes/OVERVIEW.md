# Hermes Agent — Overview

Phase 2 (Hermes Architecture Research). Dieses und die weiteren Dokumente
unter `docs/hermes/` fassen die **offizielle Dokumentation** von Hermes
Agent (Nous Research) zusammen, um PixelHermes anschließend möglichst
nah am Upstream aufzubauen. Es wurde **nichts installiert, nichts
konfiguriert, nichts am System verändert** — ausschließlich Recherche
und Dokumentation.

Jede Aussage in diesen Dokumenten ist entweder mit einer Quell-URL aus
der offiziellen Dokumentation belegt, oder explizit als "in offizieller
Doku nicht gefunden" markiert. Es wurden bewusst keine Vermutungen als
Fakten dargestellt (siehe [OPEN_QUESTIONS.md](OPEN_QUESTIONS.md) für alle
offen gebliebenen Punkte).

## Was ist Hermes Agent?

Hermes Agent ist ein Open-Source-KI-Agent-Framework von **Nous
Research** (MIT-Lizenz). Aus dem README: ein Terminal-Agent-Loop mit
persistentem Gedächtnis, wiederverwendbaren Skills, 60+ eingebauten
Tools, MCP-Unterstützung, Messaging-Integrationen und lokalen oder
isolierten Ausführungs-Backends. Es beschreibt sich selbst als "the only
agent with a built-in learning loop."

Quelle: [github.com/NousResearch/hermes-agent](https://github.com/NousResearch/hermes-agent),
[hermes-agent.nousresearch.com/docs/](https://hermes-agent.nousresearch.com/docs/)

## Recherchierte Kapitel

| Kapitel | Dokument | Deckt ab |
|---|---|---|
| Core | [INSTALLATION.md](INSTALLATION.md), [ARCHITECTURE.md](ARCHITECTURE.md) | Installation, Configuration, CLI, API Server, Gateway, Profiles, Sessions |
| Runtime | [WORKSPACE.md](WORKSPACE.md), [PERSONAS.md](PERSONAS.md), [MEMORY.md](MEMORY.md) | Workspace, Context, SOUL.md, AGENTS.md, Memory, Reflection, Persona |
| Skills | [SKILLS.md](SKILLS.md) | Skill Discovery, Bundled Skills, Priorität, Tool Runtime, Python, Security |
| MCP | [MCP.md](MCP.md) | MCP-Architektur, Tool Integration, Filesystem, Git, Office, Browser |
| Deployment | [DEPLOYMENT.md](DEPLOYMENT.md) | Linux, systemd, Reverse Proxy, Mehrbenutzerbetrieb, Logging, Backup |
| Modelle | [MODELS.md](MODELS.md) | OpenAI-kompatible Provider, Grok, OpenRouter, lokale Modelle |
| Synthese | [BEST_PRACTICES.md](BEST_PRACTICES.md), [OPEN_QUESTIONS.md](OPEN_QUESTIONS.md) | Kapitelübergreifende Empfehlungen und offene Fragen |
| Produktivbetrieb (Phase 6) | [PRODUCTIVE_RUNTIME.md](PRODUCTIVE_RUNTIME.md), [ASSESSMENT.md](ASSESSMENT.md) | Grok+Exa produktiv konfiguriert und real verifiziert, inkl. eines kritischen, ungelösten Fundes (Exa/`web_search` wird nicht zuverlässig aufgerufen) |
| Companion Stack (Sprint 7) | [HONCHO.md](HONCHO.md), [COMPANION_STACK.md](COMPANION_STACK.md) | Selbstgehosteter Honcho-Memory-Server + lokaler Embedding-Server (llama.cpp), real installiert, konfiguriert und Ende-zu-Ende verifiziert (Cross-Session-Recall bestätigt) |
| Drittanbieter-Erweiterungen (Sprint 7) | [SUPER_HERMES.md](SUPER_HERMES.md) | Erste geprüfte, bewusst zugelassene Drittanbieter-Erweiterung (Skill-Paket), inkl. Entfernbarkeits-Nachweis |

## Architektur in einem Satz

Ein einzelner Python-Prozess pro Profil (`HERMES_HOME` = i. d. R.
`~/.hermes/`) treibt einen Konversations-Loop: ein zentrales
Tool-Registry (nativ + MCP + Skills) wird dem LLM angeboten, Kontext
kommt aus SOUL.md (Identität), AGENTS.md/HERMES.md (Projektwissen) und
MEMORY.md/USER.md (kuratiertes Gedächtnis), Sitzungen werden in einer
SQLite-Datenbank (`state.db`, WAL-Modus, FTS5-Volltextsuche) persistiert,
und ein optionaler Gateway-Prozess verbindet dieselbe Agent-Instanz mit
25+ Messaging-Plattformen. Siehe [ARCHITECTURE.md](ARCHITECTURE.md) für
Details.

## PixelHermes-Gesamteinschätzung

**Was Hermes bereits übernimmt (kurz):** Orchestrierung des
Agenten-Loops, Tool-Registry inkl. MCP-Client, Skill-System mit
Lernschleife, mehrstufiges Kontext-/Gedächtnismodell, Multi-Plattform-
Gateway, Multi-Profil-Isolation, ein umfangreiches Security-Modell
(Defense-in-Depth), Provider-Abstraktion für 30+ LLM-Anbieter inklusive
lokaler Modelle. Das deckt praktisch die gesamte Definition von "Hermes
orchestriert — LLM denkt — Tools arbeiten" aus
[ARCHITECTURE.md](../../ARCHITECTURE.md) bereits ab.

**Was das für PixelHermes bedeutet:** PixelHermes muss keinen eigenen
Agenten-Loop, keine eigene Tool-Registry, kein eigenes Skill-System und
keinen eigenen Gateway bauen. Die Aufgabe von PixelHermes ist,
Hermes **möglichst unverändert** zu betreiben und PixelHermes-spezifische
Anpassungen ausschließlich über die dafür vorgesehenen Erweiterungspunkte
vorzunehmen: `config.yaml`, `.env`, `SOUL.md`/`AGENTS.md`, eigene Skills
in `skills/`, eigene MCP-Server-Einträge, eigene Profile. Details je
Kapitel in den einzelnen Dokumenten.

Details, konkrete Empfehlungen und Abweichungsentscheidungen: siehe
[BEST_PRACTICES.md](BEST_PRACTICES.md).
