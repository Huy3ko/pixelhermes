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
