# Hermes Agent — Skills

Quellen: [Skills Feature](https://hermes-agent.nousresearch.com/docs/user-guide/features/skills),
[Work with Skills](https://github.com/NousResearch/hermes-agent/blob/main/website/docs/guides/work-with-skills.md),
[Creating Skills](https://hermes-agent.nousresearch.com/docs/developer-guide/creating-skills),
[Tools & Toolsets](https://hermes-agent.nousresearch.com/docs/user-guide/features/tools),
[Security](https://hermes-agent.nousresearch.com/docs/user-guide/security),
[Code Execution](https://hermes-agent.nousresearch.com/docs/user-guide/features/code-execution)

## Skill Discovery

Primärverzeichnis: `~/.hermes/skills/` ("primary directory and source of
truth"), organisiert in Kategorie-Unterordner. Zusätzliche Quellen über
`external_dirs` in `config.yaml`. Jedes Skill-Verzeichnis benötigt ein
`SKILL.md` mit Standard-Frontmatter.

**Progressive Disclosure** (Terminologie zwischen zwei Docpages
inkonsistent — beide beschreiben denselben Mechanismus):
`skills_list()` liefert einen günstigen Index (Name/Beschreibung/
Kategorie, ~3k Tokens) beim Sessionstart; voller Inhalt wird erst per
`skill_view(name)` geladen, wenn der Agent ihn tatsächlich braucht;
`skill_view(name, file_path)` lädt gezielt einzelne Referenzdateien
innerhalb eines Skills.

**Zugriffswege:** Slash-Commands (`/skill-name`), Chat-Anfragen, Skills
Hub (`/skills search docker`), CLI (`hermes skills list`).

**Bundled-Skill-Update-Tracking:** `.bundled_manifest` mapped jeden
gebündelten Skill auf einen Content-Hash; unveränderte Skills werden
beim Update automatisch aktualisiert, vom Nutzer modifizierte Skills
"treated as user-modified and skipped forever."

## Bundled Skills

Dokumentiert genannte Beispiele: `ascii-art`, `arxiv`,
`github-pr-workflow`, `plan` (Plan-Mode), `excalidraw`, sowie ein
selbstreferenzieller Skill `hermes-agent` (Kategorie
`autonomous-ai-agents`) als Bedienungsanleitung für Hermes selbst.
Struktur: `skills/<kategorie>/<skill>/{SKILL.md, scripts/}`, optionale
Skills separat unter `optional-skills/`. Vertrauensstufen: `builtin`,
`official`, `trusted`, `community`.

**Nicht dokumentiert:** eine einzelne, vollständige Enumeration aller
gebündelten Skills/Kategorien an einer Stelle.

## Skill Priorität

Explizit dokumentierte Regeln:

- Lokales Verzeichnis schlägt `external_dirs` bei Namenskollision.
- Skill-Bundles schlagen einzelne Skills bei Slug-Kollision.
- Unveränderte gebündelte Skills werden beim Update überschrieben,
  nutzermodifizierte dauerhaft übersprungen (siehe oben).
- Hub-installierte und selbst erstellte Skills bleiben über Updates
  hinweg erhalten.
- Plugin-Skills sind namespaced (`plugin:skill`) und erscheinen nicht in
  `skills_list()` — opt-in.

**Nicht dokumentiert:** eine Regel für semantische Konflikte (zwei
unterschiedlich benannte, aber inhaltlich überlappende Skills) — das
bleibt dem Modell an der `skills_list()`-Auswahlstelle überlassen, nicht
über eine Regel-Engine geregelt.

## Tool Runtime

Zentrale Tool-Registry (`tools/registry.py`): jede `tools/*.py`-Datei
mit einem `registry.register()`-Aufruf wird beim Import automatisch
erkannt (selbstregistrierendes Muster). Die genaue Toolanzahl variiert
je nach Docpage/aktiven Toolsets (Zahlen zwischen "30+" und "70+
registrierte Tools über ~28 Toolsets" gefunden — als Bandbreite, nicht
als Fixwert zu zitieren).

**Dokumentierte Toolsets:** web, search, terminal, file, browser,
vision, image_gen, skills, tts, todo, memory, session_search, cronjob,
code_execution, delegation, clarify, homeassistant, messaging, spotify,
discord, debugging, safe (u. a.).

**Ausführungs-Backends (Terminal):** local, Docker, SSH,
Singularity/Apptainer, Modal, Daytona. Docker-Hardening: Read-only
Root-FS, alle Linux-Capabilities gedroppt, keine Privilege-Escalation,
PID-Limit 256, volle Namespace-Isolation. Bei
Docker/Singularity/Modal/Daytona werden Dangerous-Command-Checks
übersprungen — "the container itself is the security boundary."

MCP-Server sind eine zusätzliche, dynamische Tool-Quelle (siehe
[MCP.md](MCP.md)).

## Python Integration

Python ist die dokumentierte Sprache für das **`execute_code`**-Tool
("Code Execution / Programmatic Tool Calling"): der Agent schreibt ein
Python-Skript mit `from hermes_tools import ...`; Hermes generiert
automatisch das RPC-Stub-Modul `hermes_tools.py`. Ausführung in einem
Kindprozess, Kommunikation über einen Unix-Domain-Socket zum
Hauptprozess. Nur `print()`-Output erreicht das LLM — Zwischenergebnisse
verbrauchen keine Tokens.

**Zwei Modi:** Project Mode (Default, nutzt das aktive venv im
Arbeitsverzeichnis) und Strict Mode (isoliertes Temp-Verzeichnis, Hermes-
eigener Python-Interpreter, Fokus auf Reproduzierbarkeit).

**Sicherheitsbeschränkungen:** Env-Vars mit `KEY`/`TOKEN`/`SECRET`/
`PASSWORD`/`CREDENTIAL`/`PASSWD`/`AUTH` werden vor Prozessstart entfernt;
nur `HERMES_HOME`, `HERMES_PROFILE`, `HERMES_CONFIG`, `HERMES_ENV`
passieren standardmäßig. Skills können via
`required_environment_variables` gezielt Variablen freischalten.
**Limits:** 300s Timeout, 50 KB stdout, 10 KB stderr, 50 Tool-Calls pro
Ausführung. **Nur Linux/macOS** (benötigt Unix-Domain-Sockets, unter
Windows automatisch deaktiviert).

Eine separate, eigenständige "Python-SDK" für Skill-/Tool-Autoren ist in
der offiziellen Doku **nicht** als eigenes Produkt dokumentiert — Python
taucht nur als Ausführungssprache für `execute_code` und als
Helper-Sprache innerhalb einzelner Skill-`scripts/`-Ordner auf.

## Security

Explizites "Defense-in-Depth"-Modell mit acht Schichten: (1)
Nutzerautorisierung, (2) Dangerous-Command-Approval, (3) File-Write-
Safety (Denylist + optionale Sandbox-Root via `HERMES_WRITE_SAFE_ROOT`),
(4) Container-Isolation, (5) MCP-Credential-Filtering, (6)
Kontext-Datei-Scanning gegen Prompt-Injection, (7) Cross-Session-
Isolation, (8) Input-Sanitization von Arbeitsverzeichnis-Parametern.

**Dangerous-Command-Modi:** `smart` (Default, Auxiliary-LLM bewertet
Risiko), `manual` (immer nachfragen), `off` (nur mit `--yolo`). Ein
"Hardline-Blocklist" ist immer aktiv (z. B. `rm -rf /`, Fork-Bombs).

**File-Write-Schutz:** blockiert immer Schreibzugriffe auf
Credential-Stores (`~/.ssh/`, `~/.aws/`), Hermes' eigene
`auth.json`/`.env`, sowie beliebige `.env*`-Projektdateien. Der
`terminal`-Tool kann diese Schreibsperre über eine Shell umgehen —
dokumentiert als bewusste Design-Entscheidung ("honest-but-wrong agent"
statt vollständig adversariales Bedrohungsmodell).

**Netzwerkschutz (SSRF):** blockiert private IP-Ranges (RFC 1918),
Loopback, Cloud-Metadata-Endpunkte (`169.254.169.254`), CGNAT/Tailscale.
DNS-Fehler = blockiert (fail-closed). Opt-out via
`security.allow_private_urls`.

**Prompt-Injection-Scanning:** gilt für `AGENTS.md`, `.cursorrules`,
`SOUL.md`; zusätzlich optionaler Pre-Execution-Scanner "Tirith" gegen
Homograph-URL-Spoofing und Pipe-to-Interpreter-Muster
(`curl | bash`).

**Supply-Chain-Scanning:** eingebauter Advisory-Scanner für kompromittierte
Python-Paketversionen, Warnung beim Start, Details via `hermes doctor`.

**Produktions-Checkliste** (dokumentiert): explizite Allowlists, Docker-
Backend, CPU/Memory/Disk-Limits, `.env` mit `chmod 600`, DM-Pairing statt
hartkodierter IDs, `terminal.cwd` restriktiv setzen, Gateway nicht als
Root, Logs überwachen, regelmäßig `hermes update`.

---

## PixelHermes-Mapping

**Was übernimmt Hermes bereits?** Ein vollständiges Skill-System mit
Discovery, Priorisierung, Update-Schutz für Nutzeränderungen,
Sicherheitsscanning und einer eigenen Lernschleife (siehe auch
[MEMORY.md](MEMORY.md), Abschnitt Reflection/Curator).

**Was müssen wir NICHT selbst entwickeln?** Kein eigenes Plugin-/
Skill-Format, keine eigene Sandbox-/Approval-Logik für gefährliche
Befehle, keinen eigenen Prompt-Injection-Scanner.

**Was passt direkt zu PixelHermes?** Der explizite Sprint-Ausschluss
"noch keine Skills" bleibt so lange korrekt gültig, bis Hermes
tatsächlich installiert ist — die Skill-Infrastruktur existiert dann
automatisch mit. `execute_code` als Python-Runtime deckt bereits den in
[ARCHITECTURE.md](../../ARCHITECTURE.md) formulierten Grundsatz "Python
als Tool-Runtime" exakt ab — keine eigene Runtime nötig.

**Welche Erweiterungen wären später sinnvoll?** Eigene PixelHermes-
Skills unter `~/.hermes/skills/pixelhermes/` (z. B. für
projektspezifische Automatisierung), erst nach Installation und mit
klaren, dokumentierten Anwendungsfällen — nicht spekulativ vorab.

**Welche Komponenten sollten wir bewusst unverändert übernehmen?** Die
komplette Skill-Engine, das Security-Modell und die Curator-/
Update-Schutzlogik — keine eigene Sicherheitsschicht darüber bauen, die
mit der vorhandenen kollidieren könnte.
