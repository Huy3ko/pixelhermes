# Install

Dieses Dokument beschreibt, wie das Infrastruktur-Fundament und die
Runtime von PixelHermes entstanden sind, damit alles jederzeit
reproduzierbar ist.

## Sprint 1 — Foundation

**Keine Software installiert, keine Systemkonfiguration verändert** —
ausschließlich das Git-Repository wurde aufgebaut.

### Voraussetzungen

- Debian 13
- Git

### Ausgangspunkt

Bestehendes GitHub-Repository `Huy3ko/pixelhermes`, geklont nach
`~/companion`, mit einem initialen `README.md`.

### Schritte

```bash
cd ~/companion

# Verzeichnisstruktur
mkdir -p docs configs scripts systemd templates assets backups ADR

# .gitignore sowie Kerndokumentation anlegen:
# README.md, ROADMAP.md, ARCHITECTURE.md, INSTALL.md, CHANGELOG.md,
# PROJECT_STRUCTURE.md, ADR/ (siehe Repo-Inhalt)

git add .
git commit -m "chore(structure): scaffold infrastructure directories"
git commit -m "docs: complete core documentation for Sprint 1"
git commit -m "docs(adr): establish architecture decision record process"

git push origin main
```

### Ergebnis

Ausschließlich ein versioniertes Git-Repository mit vollständiger
Kerndokumentation und der dokumentierten Verzeichnisstruktur — keine
laufenden Services, keine Benutzer, keine Datenbanken, kein Workspace,
keine Agenten.

## Sprint 2 — Hermes Architecture Research

Reine Recherche, keine System- oder Repository-Struktur-Änderung
außerhalb von `docs/hermes/`. Siehe
[docs/hermes/OVERVIEW.md](docs/hermes/OVERVIEW.md).

## Sprint 3 — Runtime Foundation

Erste **echten** Systemänderungen: Laufzeit-Abhängigkeiten und
Grundstruktur, auf der Hermes später installiert wird. Weiterhin
**keine** Installation von Hermes, Hermes Workspace, OpenWebUI, mem0
oder Humalike; **keine** systemd-Services, **keine** Workspaces, **keine**
SQLite-Datenbanken.

### Node.js (offizielle LTS, nicht die Debian-Paketversion)

Installiert über das offizielle NodeSource-Repository (Active LTS zum
Zeitpunkt der Installation: Node.js 24):

```bash
curl -fsSL https://deb.nodesource.com/setup_24.x | sudo -E bash -
sudo apt-get install -y nodejs
```

Ergebnis: `node v24.18.0`, `npm 11.16.0`, installiert unter
`/usr/bin/node` (NodeSource-Repository, nicht das Debian-Bookworm/Trixie-
`nodejs`-Paket).

### Python

System-Python bereits vorhanden (`python3 --version` → `Python
3.13.5`). Ergänzt über `apt`:

```bash
sudo apt-get install -y python3-venv python3-pip
```

Zusätzlich der offizielle Python-Paketmanager **uv**, systemweit
installiert (nicht nur für den aktuellen Nutzer), damit er später auch
für die Companion-Benutzer verfügbar ist:

```bash
curl -LsSf https://astral.sh/uv/install.sh | sudo env UV_INSTALL_DIR="/usr/local/bin" sh
```

Ergebnis: `uv 0.11.32`, `uvx 0.11.32` unter `/usr/local/bin/`.

### Basispakete

Geprüft, bereits vorhanden: `git`, `curl`, `wget`, `sqlite3`,
`build-essential`, `ca-certificates`, `jq`, `tree`, `htop`, `zip`,
`unzip`. Keine weitere Installation nötig.

### Companion-Systemstruktur (real angelegt)

```bash
sudo mkdir -p /opt/companion/{skills,tools,templates,shared} \
  /etc/companion \
  /srv/companion \
  /var/log/companion \
  /var/backups/companion
sudo chown -R root:root /opt/companion /etc/companion /srv/companion \
  /var/log/companion /var/backups/companion
sudo chmod 755 /opt/companion /etc/companion /srv/companion \
  /var/log/companion /var/backups/companion
```

### Companion-Benutzer

Zwei unprivilegierte Linux-Benutzer, Home-Verzeichnis bewusst unter
`/srv/companion/<name>` statt `/home/<name>` (Begründung:
[ADR 0002](ADR/0002-companion-user-home-under-srv.md)), Passwort-Login
gesperrt (Zugriff vorerst nur über `sudo -u <name>`):

```bash
sudo useradd -m -d /srv/companion/hermes_hugo -s /bin/bash \
  -c "Companion Agent hermes_hugo" hermes_hugo
sudo useradd -m -d /srv/companion/hermes_christiane -s /bin/bash \
  -c "Companion Agent hermes_christiane" hermes_christiane
sudo passwd -l hermes_hugo
sudo passwd -l hermes_christiane
```

Ergebnis: `hermes_hugo` (UID 1001), `hermes_christiane` (UID 1002),
Home-Verzeichnisse `700`, kein Hermes, kein systemd-Service, kein
Workspace, keine Datenbank — ausschließlich Benutzer und leere
Home-Verzeichnisse.

## Nächste Schritte

Modell-/Provider-Entscheidung (ADR), danach die eigentliche
Hermes-Installation je Benutzer nach
[docs/hermes/INSTALLATION.md](docs/hermes/INSTALLATION.md). Siehe
[ROADMAP.md](ROADMAP.md).
