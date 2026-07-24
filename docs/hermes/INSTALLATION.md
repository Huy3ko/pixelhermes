# Hermes Agent — Installation

Quellen: [Installation](https://hermes-agent.nousresearch.com/docs/getting-started/installation),
[Quickstart](https://hermes-agent.nousresearch.com/docs/getting-started/quickstart),
[Platform Support](https://hermes-agent.nousresearch.com/docs/getting-started/platform-support)

## Installationswege

1. **Curl-Skript** (Linux/macOS/WSL2/Termux):
   `curl -fsSL https://hermes-agent.nousresearch.com/install.sh | bash`
2. **PowerShell-Skript** (natives Windows):
   `iex (irm https://hermes-agent.nousresearch.com/install.ps1)`
3. **Hermes Desktop Installer** (macOS/Windows, empfohlen für Desktop)
4. **Manuelle/Entwickler-Installation** aus dem Quellcode
5. **Docker** (`docker pull`) — **unterstützt kein `hermes update`**
6. **Nix** — ausdrücklich "best effort" ("bricht oft wegen Node.js-Packaging")

**Ausdrücklich NICHT unterstützt:** pip, pipx, Homebrew, AUR.

## Plattform-Matrix

| Tier | Plattformen | Hinweise |
|---|---|---|
| Tier 1 (priorisiert) | macOS (nur Apple Silicon, Intel nicht unterstützt), Windows 10/11 (x86_64, aarch64), Linux/WSL2 (x86_64, aarch64), Docker (x86_64, aarch64) | Linux: glibc + systemd + FHS-konforme Distro vorausgesetzt |
| Tier 2 (best effort) | Android/Termux (aarch64), Nix | eingeschränkter Funktionsumfang |

## Voraussetzungen

- Alle Plattformen: Git
- Linux: `curl`, `xz-utils`
- Desktop-App: `g++`/`build-essential`
- Automatisch vom Installer eingerichtet (kein manueller Schritt nötig):
  `uv`, Python 3.11, Node.js v22, ripgrep, ffmpeg

## Installationsorte (Linux)

- Pro Nutzer: Code unter `~/.hermes/hermes-agent/`, Binary unter
  `~/.local/bin/hermes`, Daten unter `~/.hermes/`
- System-/Root-Installation: FHS-Layout unter
  `/usr/local/lib/hermes-agent/` und `/usr/local/bin/hermes`

## Nach der Installation

Shell neu laden, dann `hermes setup` oder `hermes model`, um einen
LLM-Provider zu konfigurieren.

## Updates (`hermes update`)

- Erkennt automatisch den Installationsweg (Git-Installer, Docker, Nix)
  und wählt den passenden Update-Befehl.
- Standardablauf: Pre-Update-Snapshot (Config/Auth/Cron, steuerbar über
  `updates.pre_update_backup`) → `git pull` (inkl. Submodule) →
  Syntaxprüfung mit automatischem Rollback (`git reset --hard`) bei
  Fehlern → Dependency-Refresh (`uv pip install -e ".[all]"`) →
  Config-Migrationsprompts → Gateway-Neustart.
- Varianten: `hermes update --branch <name>`, `--check` (Trockenlauf),
  `--backup` (vollständiges `HERMES_HOME`-Backup vorher).
- Remote-Trigger: `/update` per Telegram/Discord/Slack/WhatsApp/Teams.
- Manuelles Update ohne Installer:
  `git pull origin main && uv pip install -e ".[all]" && hermes config check && hermes config migrate`
- Rollback: `git log --oneline`, `git checkout <hash>`, Dependencies neu
  installieren, `hermes gateway restart`.
- **Docker-Installationen unterstützen `hermes update` nicht** — Image
  muss neu gepullt werden.

## Deinstallation

`hermes uninstall` (fragt, ob `~/.hermes/` erhalten bleiben soll) oder
manuell `rm -f ~/.local/bin/hermes`, `rm -rf <repo-pfad>`, optional
`rm -rf ~/.hermes`. Gateway-Dienste vorher stoppen/deaktivieren
(`hermes gateway stop`, dann `systemctl --user disable hermes-gateway`
unter Linux bzw. `launchctl remove ai.hermes.gateway` unter macOS).

## Nicht dokumentiert

Exakte Mindestversionen (z. B. welche Windows-/macOS-Builds) über
"10/11" bzw. "Apple Silicon" hinaus sind in der offiziellen Doku nicht
angegeben.

## Reale Installation — hermes_hugo (Phase 4, 2026-07-24)

Tatsächlich durchgeführte, native Installation für den Linux-Benutzer
`hermes_hugo` (Home: `/srv/companion/hermes_hugo`), via
`curl -fsSL https://hermes-agent.nousresearch.com/install.sh | bash`,
non-interaktiv über `sudo -u hermes_hugo -H bash -lc '...'`. Reale
Beobachtungen, keine Vermutungen:

- **Managed uv:** Der Installer installiert zusätzlich eine eigene,
  "gemanagte" `uv`-Kopie nach `~/.hermes/bin/uv` (unabhängig von der in
  Phase 3 systemweit unter `/usr/local/bin/uv` installierten Version) und
  darüber eine eigene **Python 3.11.15** (via `uv python install`),
  getrennt vom System-Python 3.13.5. Hermes bringt seine Python-Runtime
  also faktisch selbst mit, unabhängig vom System-Python.
- **Nicht-interaktive Lücken:** Zwei Schritte scheiterten erwartungsgemäß
  ohne TTY und wurden übersprungen: (1) `ripgrep`/`ffmpeg` — vom
  Installer selbst mit `sudo apt install -y ripgrep ffmpeg` nachinstalliert,
  danach verifiziert (`ripgrep 14.1.1`, `ffmpeg 7.1.5`); (2) Playwright-
  Chromium-Browser-Engine — Versuch, die System-Bibliotheken via
  `sudo npx playwright install-deps chromium` nachzuinstallieren, schlug
  fehl (`playwright: not found`, keine lokale `playwright`-node-
  Installation im Repo auffindbar trotz vorhandener `node_modules/` unter
  `hermes-agent/`, `hermes-agent/web/`, `hermes-agent/ui-tui/`) — nicht
  weiter verfolgt, da Browser-Tools außerhalb des Phase-4-Scopes
  (CLI/API Server/Gateway/Profil) liegen. `hermes doctor` bestätigt:
  "Playwright Chromium not installed (browser_\* tools will be hidden
  from the agent)" — der Rest der Installation ist davon unberührt.
- **`web`/`pty`-Extras bereits im Standardinstall enthalten (Abweichung
  von der bisherigen Doku-Vermutung):** Ein separates
  `uv pip install -e ".[web,pty]"` (siehe Doku-Hinweis in
  [ARCHITECTURE.md](ARCHITECTURE.md#api-server)) installierte **keine**
  zusätzlichen Pakete — `fastapi`, `uvicorn`, `ptyprocess`, `starlette`
  waren bereits Teil des Standard-`[all]`-Installs über
  `install.sh` (hash-verifizierter `uv.lock`-Tier, 95 Pakete). Der API
  Server war damit ohne weiteres Zutun sofort startbar.
- **Config-Migration:** frisch generierte `config.yaml` hatte
  Schemaversion 0; `hermes config migrate` hob sie ohne Rückfrage nach
  API-Keys auf Version 33 (u. a. Timezone-Erkennung, Curator-
  Konfigurationsblock, `agent.verify_on_stop: false` als neuer Default).
  Dabei zwei unabhängig vom Update bestehende Warnungen beobachtet:
  `platform 'teams' references unknown toolset 'hermes-teams'` und
  `platform 'google_chat' references unknown toolset 'hermes-google_chat'`
  — ungeklärte Inkonsistenz im mitgelieferten Default-Config-Template,
  siehe [OPEN_QUESTIONS.md](OPEN_QUESTIONS.md).
- **`sudo` wird nur für Build-Tools angefragt, nicht für den Agenten
  selbst:** Installer-Text bestätigt wörtlich: "Hermes Agent itself does
  not require or retain root access." In der non-interaktiven Umgebung
  scheiterte der `sudo`-Aufruf für Build-Tools mangels TTY, die
  Installation lief trotzdem durch (`build-essential`/`python3-dev` aus
  Phase 3 waren bereits vorhanden).
- **Bundled Skills:** Sync-Log meldete "Done: 69 new, 0 updated, 0
  unchanged. 69 total bundled", `hermes skills list` zeigt danach 65
  aktivierte `builtin`-Skills — Diskrepanz (69 vs. 65) real beobachtet,
  nicht aufgelöst. Details: [SKILLS.md](SKILLS.md).

---

## PixelHermes-Mapping

**Was übernimmt Hermes bereits?** Den kompletten Installations- und
Update-Mechanismus inklusive Plattformerkennung, Rollback-Sicherung und
Dependency-Management. Ein eigener Installer wäre eine Verletzung von
"Upstream First".

**Was müssen wir NICHT selbst entwickeln?** Ein Installations-, Update-
oder Uninstall-Skript. `hermes update` deckt genau das ab, was
PixelHermes unter "Infrastructure as Code" ohnehin fordert — solange die
Installation über den offiziellen Weg erfolgt und nicht manuell gepatcht
wird.

**Was passt direkt zu PixelHermes?** Die Pro-Nutzer-Installation
(`~/.hermes/hermes-agent`, `~/.local/bin/hermes`, `~/.hermes/`) passt
exakt zur geplanten Systemstruktur in [ARCHITECTURE.md](../../ARCHITECTURE.md)
(Home-Verzeichnis eines künftigen Service-Users statt Root-Installation)
— siehe auch [DEPLOYMENT.md](DEPLOYMENT.md) zur Multi-User-Frage.
Docker wird laut Prinzip "Keine unnötigen Abhängigkeiten" und explizitem
Sprint-Verbot ("Kein Docker") in Sprint 1/2 nicht verwendet — das
schließt auch aus, dass `hermes update` nicht funktioniert.

**Welche Erweiterungen wären später sinnvoll?** Ein PixelHermes-eigenes
`scripts/`-Wrapperskript, das `hermes update --check` vor echten Updates
aufruft und das Ergebnis loggt — kein Ersatz, nur eine dünne
Orchestrierungsschicht.

**Welche Komponenten sollten wir bewusst unverändert übernehmen?** Den
gesamten Installer, den Update-Mechanismus und die Verzeichniskonvention
`~/.hermes/`. Kein Fork, kein eigener Installationspfad.
