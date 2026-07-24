# Hermes Agent — MCP Integration

Quellen: [MCP Feature](https://hermes-agent.nousresearch.com/docs/user-guide/features/mcp),
[Use MCP with Hermes](https://hermes-agent.nousresearch.com/docs/guides/use-mcp-with-hermes),
[MCP Config Reference](https://hermes-agent.nousresearch.com/docs/reference/mcp-config-reference),
[optional-mcps (GitHub)](https://github.com/NousResearch/hermes-agent/tree/main/optional-mcps),
[Browser Feature](https://hermes-agent.nousresearch.com/docs/user-guide/features/browser),
[Tools Reference](https://hermes-agent.nousresearch.com/docs/reference/tools-reference)

## MCP-Architektur

Nativer MCP-Client, Transporte **stdio** (lokale Subprozesse) und
**HTTP** (Remote, inkl. Bearer-Token, OAuth 2.1/PKCE, mTLS). Config
unter `mcp_servers:` in `config.yaml`:

```yaml
mcp_servers:
  filesystem:
    command: "npx"
    args: ["-y", "@modelcontextprotocol/server-filesystem", "/tmp"]
```

Pro-Server-Optionen: `enabled`, `timeout` (Default 300s),
`connect_timeout` (Default 60s), `supports_parallel_tool_calls`,
`idle_timeout_seconds`/`max_lifetime_seconds`.

**Tool-Namenskonvention (bestätigt):** `mcp_<server_name>_<tool_name>`,
Bindestriche/Punkte werden zu Unterstrichen konvertiert. Filterung
(`include`/`exclude`) nutzt jedoch den ursprünglichen, unbereinigten
MCP-Toolnamen.

**Discovery/Auto-Reload:** Toolerkennung beim Start; Server können
Laufzeit-Updates via MCP-Standard-Notification
`notifications/tools/list_changed` pushen; `/reload-mcp` verbindet ohne
Neustart neu.

**Sampling:** MCP-Server können LLM-Inferenz über
`sampling/createMessage` anfordern (Default aktiv, pro Server
konfigurierbar mit Modell, Token-Cap, Rate-Limit).

**Hermes als MCP-Server (Rückrichtung):** `hermes mcp serve` exponiert
Hermes selbst als MCP-Server (~10 Tools für Konversationsverwaltung),
sodass andere MCP-Clients (Claude Code, Cursor, Codex) Hermes'
Messaging-Fähigkeiten nutzen können.

## Tool Integration

MCP-Tools registrieren sich in dieselbe Registry wie native Tools,
namespaced mit `mcp_<server>_`-Präfix. Steuerung über `tools.include`
(Whitelist, hat Vorrang), `tools.exclude` (Blacklist),
`tools.resources`/`tools.prompts` (MCP-Utility-Wrapper),
Server-`enabled: false`. Offizielle Empfehlung: **Allowlists statt
Blocklists** bei gefährlichen Systemen, mit einem engen Server starten,
verifizieren, dann erweitern. Native Hermes-Tools haben Vorrang, wo sie
bereits ausreichen — MCP wird nur ergänzend empfohlen.

**Kuratierter MCP-Katalog:** `hermes mcp` (interaktiv), `hermes mcp
catalog`, `hermes mcp install <name>`. Der offizielle Katalog
(`optional-mcps/` im Repo, Stand der Recherche) enthält genau vier
Einträge: **blender, linear, n8n, unreal-engine**. **Kein**
Filesystem-, Git-, Office- oder Browser-Server im offiziellen Katalog.

## Filesystem

Native, nicht-MCP-basierte Datei-Tools existieren bereits (`file`-
Toolset: `read_file`, `write_file`, `search_files`, `patch`). Für
MCP-basierten Zugriff nennt die Doku durchgehend
`@modelcontextprotocol/server-filesystem` als Beispiel — **kein**
Katalogeintrag, nur illustrative Empfehlung ("guter erster, sicherer
Server", auf ein Projektverzeichnis begrenzen). Kein
Hermes-spezifisches Sandboxing zusätzlich zur Verzeichnisbegrenzung des
Drittserver-Pakets selbst dokumentiert.

## Git

**Kein natives Git-Toolset** — Git läuft über das generische
`terminal`-Tool (Shell-Aufruf von `git`) oder MCP. Dokumentiertes
Beispiel: **`mcp-server-git`** (via `uvx`), als einer der "Recommended
First Servers". GitHub separat über
`@modelcontextprotocol/server-github`. Beides **nicht** im offiziellen
Katalog — reine Dokumentationsbeispiele.

## Office

**Kein dokumentierter, benannter Office-MCP-Server.** Einzige Erwähnung:
ein bedingter Nebensatz auf der Excel-Author-Skill-Seite ("Users in a
live Excel session with an Office MCP available..."), kein konkreter,
empfohlener Server. Was Hermes tatsächlich für Office/Tabellen bietet,
ist der **Excel-Author-Skill** (kein MCP) — generiert `.xlsx` headless
via `openpyxl`, statische Ausgabedatei, keine Live-Session-Manipulation.

## Browser

Umfangreiche **native** Browser-Automatisierung (kein MCP nötig):
mehrere Backends (Browserbase, Browser Use, Firecrawl, Camofox
selbstgehostet, lokales Chromium via CDP), Tools wie `browser_navigate`,
`browser_snapshot`, `browser_click`, `browser_cdp`. MCP wird nur als
dokumentierter Workaround für den WSL2→Windows-Grenzfall empfohlen
(`chrome-devtools-mcp`), ebenfalls nicht im offiziellen Katalog.

## Zusammenfassungstabelle

| Bereich | Nativ in Hermes? | MCP-Server in Doku genannt? | Im offiziellen Katalog? |
|---|---|---|---|
| Filesystem | Ja (`file`-Toolset) | Ja (Beispiel: `server-filesystem`) | Nein |
| Git | Nein (`terminal`) | Ja (Beispiel: `mcp-server-git`, `server-github`) | Nein |
| Office | Nein (aber Excel-Author-Skill) | Nur vage/bedingt erwähnt | Nein |
| Browser | Ja (mehrere Backends) | Ja, nur als WSL2-Workaround | Nein |

---

## PixelHermes-Mapping

**Was übernimmt Hermes bereits?** Einen vollständigen, produktionsreifen
MCP-Client mit Auth, Sampling, Auto-Reload und Sicherheitsfilterung —
sowie native (nicht-MCP) Alternativen für Filesystem und Browser, die in
den meisten Fällen MCP überhaupt unnötig machen.

**Was müssen wir NICHT selbst entwickeln?** Keinen eigenen MCP-Client,
keine eigene Filesystem- oder Browser-Automatisierung — beides bereits
nativ vorhanden und laut Doku dem MCP-Weg vorzuziehen, wenn ausreichend.

**Was passt direkt zu PixelHermes?** Der Sprint-Ausschluss "noch kein
MCP" bleibt bis zur echten Hermes-Installation korrekt — danach ist MCP
sofort nutzbar, ohne eigene Implementierung. Für Git-Operationen ist
`terminal` + `git` (nativ) der einfachste Weg — passt zu "Keep it
simple", bevor ein MCP-Git-Server eingeführt wird.

**Welche Erweiterungen wären später sinnvoll?** Falls Office-Dokumente
für PixelHermes relevant werden: zunächst den bereits vorhandenen
Excel-Author-Skill prüfen, bevor ein community-MCP-Office-Server (nicht
offiziell kuratiert, höheres Risiko) evaluiert wird.

**Welche Komponenten sollten wir bewusst unverändert übernehmen?** Den
MCP-Client und die native Filesystem-/Browser-Implementierung
vollständig unverändert; bei MCP-Servern strikt nach der offiziellen
Empfehlung vorgehen: eng scopen, Allowlists, mit einem Server beginnen.
