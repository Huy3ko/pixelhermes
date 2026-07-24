# Install — Sprint 1: Foundation

Dieses Dokument beschreibt, wie das aktuelle Infrastruktur-Fundament
entstanden ist, damit es jederzeit reproduzierbar ist. Es wurde in
diesem Sprint **keine Software installiert** und **keine
Systemkonfiguration verändert** — ausschließlich dieses Git-Repository
wurde erweitert.

## Voraussetzungen

- Debian 13
- Git

## Ausgangspunkt

Bestehendes GitHub-Repository `Huy3ko/pixelhermes`, geklont nach
`~/companion`, mit einem initialen `README.md`.

## Schritte

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

## Ergebnis

Nach diesen Schritten existiert ausschließlich:

- ein versioniertes Git-Repository (`pixelhermes`) mit vollständiger
  Kerndokumentation
- die dokumentierte Verzeichnisstruktur
- keine laufenden Services, keine Benutzer, keine Datenbanken, kein
  Workspace, keine Agenten

## Nächste Schritte

Weitere Installationsschritte (Hermes Workspace, Systemverzeichnisse,
Benutzer, Pakete, Dienste) folgen in späteren Sprints und werden hier
ergänzt, sobald sie tatsächlich durchgeführt werden. Siehe
[ROADMAP.md](ROADMAP.md).
