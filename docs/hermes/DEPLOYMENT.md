# Hermes Agent — Deployment

Quellen: [Platform Support](https://hermes-agent.nousresearch.com/docs/getting-started/platform-support),
[CLI Commands Reference](https://hermes-agent.nousresearch.com/docs/reference/cli-commands),
[Messaging](https://hermes-agent.nousresearch.com/docs/user-guide/messaging),
[Web Dashboard](https://hermes-agent.nousresearch.com/docs/user-guide/features/web-dashboard),
[Security](https://hermes-agent.nousresearch.com/docs/user-guide/security),
[Profiles](https://hermes-agent.nousresearch.com/docs/user-guide/profiles),
[FAQ](https://hermes-agent.nousresearch.com/docs/reference/faq),
GitHub-README

**Hinweis zu Quellen:** Community-Tutorials (LumaDock, diverse
Blogs/Gists) tauchten in der Recherche auf, sind aber **nicht
offiziell** — als solche gekennzeichnet und nicht als Fakt übernommen.

## Linux

Tier-1-Plattform (x86_64, aarch64). Anforderung laut Doku: "If your
distro has glibc, systemd, and follows the Filesystem Hierarchy
Standard, it's likely to work pretty well." Installation via Curl-Skript
(siehe [INSTALLATION.md](INSTALLATION.md)). Kein dokumentierter
Mindest-Kernel, keine explizite Distro-Versionsmatrix.

## systemd

Kein statisches, mitgeliefertes `.service`-File im Repo — stattdessen
CLI-verwaltet:

- `hermes gateway install` — installiert als systemd- (Linux) bzw.
  launchd-Dienst (macOS); `sudo hermes gateway install --system` für
  systemweiten Boot-Start.
- `hermes gateway start|stop|status`.
- Boot-Start für User-Dienste: `sudo loginctl enable-linger $USER`.
- **Explizite Warnung:** kein `ExecStopPost=/bin/kill -9 $MAINPID`
  Drop-in hinzufügen — feuert bei jedem beabsichtigten Stop und erzeugt
  Neustart-Schleifen.
- WSL: "WSL's systemd support is unreliable" — Fallback `tmux new -s
  hermes 'hermes gateway run'`.

**Nicht dokumentiert:** der exakte Inhalt der generierten Unit-Datei
(Type=, ExecStart=, Restart=) wird programmatisch erzeugt, nicht
zeilenweise veröffentlicht.

## Reverse Proxy

Dokumentiert primär über das Dashboard:

- Default-Bind `127.0.0.1:9119`.
- Pfad-Präfixe (z. B. `/hermes`) werden unterstützt, wenn ein Reverse
  Proxy davor sitzt.
- Header-Weiterleitung: `X-Forwarded-Host`, `X-Forwarded-Proto`,
  `X-Forwarded-Prefix` (bei `proxy_headers=True` in uvicorn, aktiv
  sobald das Auth-Gate greift).
- `dashboard.public_url`/`HERMES_DASHBOARD_PUBLIC_URL` überschreibt die
  OAuth-Callback-URL-Rekonstruktion bei unzuverlässiger
  Header-Weiterleitung.
- **Auth-Gate ist Pflicht bei jedem Nicht-Loopback-Bind** — Dashboard
  verweigert sonst explizit den Start.
- CORS standardmäßig nur für Localhost-Origins.

**Nicht dokumentiert:** keine offizielle Nginx/Caddy/Traefik-Anleitung
mit Beispielkonfiguration gefunden; WebSocket-Timeout-Tuning-Hinweise
aus der Websuche konnten nicht gegen eine offizielle Docpage verifiziert
werden — als unbestätigt markiert.

## Mehrbenutzerbetrieb

Zwei getrennte Mechanismen:

**A. Gateway-Mehrbenutzerbetrieb (Chat-Plattformen):** mehrere Menschen
sprechen über Telegram/Discord/Slack/... mit **derselben**
Agenten-Instanz. Zugriff über Allowlists, DM-Pairing oder (nicht
empfohlen) offenen Zugang. Autorisierungsreihenfolge:
Plattform-Allow-all → DM-Pairing → Plattform-Allowlist →
globale Allowlist → Default-Deny. Admin-/Nutzer-Tiers konfigurierbar.

**B. Profile als Multi-Agent-Mechanismus:** mehrere unabhängige
Agenten-Identitäten auf einer Maschine, jeweils komplett isolierter
Zustand (siehe [ARCHITECTURE.md](ARCHITECTURE.md)). **Isolation ist
explizit nur Zustand, keine Dateisystem-/OS-Sandbox** — Tool-Subprozesse
teilen sich standardmäßig das echte OS-User-`HOME` über Profile hinweg,
sofern nicht `terminal.home_mode: profile` gesetzt ist.

**Docker-Empfehlung (falls später relevant):** ein Container für alle
Profile statt Container-pro-Profil; kritische Warnung: nie zwei
Gateway-Container gegen dasselbe Datenverzeichnis gleichzeitig
betreiben — Session-/Memory-Storage ist nicht für gleichzeitige
Schreibzugriffe ausgelegt.

**Nicht dokumentiert:** kein OS-Level-Sandboxing/starke Mandanten-
trennung zwischen Profilen auf demselben Host; kein
Ressourcen-Quota-Mechanismus pro Gateway-Nutzer.

## Logging

Alle Logs unter `~/.hermes/logs/` (bestätigt über FAQ und CLI-Referenz).
Dateien: `agent`, `errors`, `gateway`. Zugriff via `hermes logs
[log_name] [--follow] [--level] [--since] [--since]`. Level: ALL, DEBUG,
INFO, WARNING, ERROR. Rotation via `RotatingFileHandler`
(`agent.log.1`, `.2`, ...); in Docker zusätzlich s6-log-Rotation
(10 Archive × 1 MB). Sensible Daten werden redigiert (Pairing-Codes nie
geloggt, bekannte Secret-Muster durch `[REDACTED]` ersetzt).

**Nicht dokumentiert:** Standard-Loglevel in Produktion; maximale
Gesamt-Retention außerhalb der Docker-s6-Archivzahl; keine eigene
zusammenhängende "Logging"-Docpage — Informationen über mehrere Seiten
verteilt.

## Backup

- `hermes backup [-o PFAD] [--quick]` — Zip-Archiv aus Config,
  API-Keys, Skills, Sessions, Daten (Codebase selbst ausgeschlossen).
- Restore: `hermes import <zipfile>` — überschreibt bestehende Dateien
  im Hermes-Home vollständig.
- Leichtgewichtig, pro Profil: `hermes profile export/import` — schließt
  Credentials aus Sicherheitsgründen aus (anders als `hermes backup`).
- Docker: `~/.hermes` ist "the single source of truth for all state" —
  Backup-Empfehlung: dieses eine Verzeichnis auf dem Host sichern.

**Nicht dokumentiert:** keine empfohlene Backup-Frequenz/-Retention,
keine Offsite-/S3-Anleitung, kein Point-in-Time-Restore-Verfahren
über den einfachen Zip-Restore hinaus.

## "Runs anywhere"-Aussage (README) — Substanzprüfung

Sechs Terminal-Backends: local, Docker, SSH, Singularity, Modal,
Daytona. Docker ist umfassend dokumentiert (s. o.); SSH/Singularity/
Modal/Daytona wurden in dieser Recherche nur über Suchergebnis-
Zusammenfassungen erschlossen, nicht direkt aus einer Primärquelle
zitiert — vor produktivem Einsatz gegen die Live-Doku erneut prüfen.
"VPS" und "GPU-Cluster" sind Positionierungssprache, keine eigenen
dokumentierten Deployment-Modi — de facto identisch mit dem Linux-Weg.

---

## PixelHermes-Mapping

**Was übernimmt Hermes bereits?** Dienstverwaltung via
`hermes gateway install` (systemd), ein vollständiges Backup-/Restore-
Kommando, Log-Rotation, und zwei komplementäre Mehrbenutzer-Mechanismen
(Gateway-Allowlists für Menschen, Profile für Agenten-Identitäten).

**Was müssen wir NICHT selbst entwickeln?** Keine eigenen systemd-Units
von Hand schreiben (`hermes gateway install` generiert sie), kein
eigenes Backup-Tool, keine eigene Log-Rotation.

**Was passt direkt zu PixelHermes?** Die in
[ARCHITECTURE.md](../../ARCHITECTURE.md) geplanten Systempfade
(`/var/log/companion`, `/var/backups/companion`) sollten **nicht** 1:1
für Hermes-Daten wiederverwendet werden — Hermes erwartet
`~/.hermes/logs/` bzw. `hermes backup`-Zips im Home-Verzeichnis des
jeweiligen Service-/Profil-Users. Sinnvoller: `/etc/companion/` bleibt
für PixelHermes-eigene (Nicht-Hermes-)Konfiguration reserviert, während
Hermes-Instanzen ihre eigene, unveränderte `~/.hermes/`-Konvention pro
Benutzer/Profil behalten. Das ist als spätere Architekturentscheidung im
[ADR](../../ADR/)-Verzeichnis festzuhalten, sobald Sprint 3 (Installation)
beginnt.

**Welche Erweiterungen wären später sinnvoll?** Ein PixelHermes-Skript,
das `hermes backup` regelmäßig anstößt und das Ergebnis nach
`/var/backups/companion/` **kopiert** (nicht verschiebt) — Hermes bleibt
Quelle der Wahrheit, PixelHermes ergänzt nur Aufbewahrung/Rotation über
mehrere Profile hinweg. Ebenso ein Reverse-Proxy-Eintrag für
`hermes dashboard`, falls Fernzugriff gewünscht wird — erst mit
konkretem Bedarf, nicht vorab.

**Welche Komponenten sollten wir bewusst unverändert übernehmen?** Den
gesamten Gateway-Diensteinrichtungsmechanismus (`hermes gateway
install`), das Backup-Format und die Autorisierungsreihenfolge für
Mehrbenutzerzugriff — keine eigene Parallellösung.
