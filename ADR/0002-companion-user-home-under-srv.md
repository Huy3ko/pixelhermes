# 0002. Home-Verzeichnis der Companion-Benutzer unter /srv/companion

Status: Accepted

## Kontext

Phase 3 (Runtime Foundation) legt die ersten zwei realen Linux-Benutzer
an (`hermes_hugo`, `hermes_christiane`), die später je einen Hermes
Agent betreiben sollen. Laut
[docs/hermes/WORKSPACE.md](../docs/hermes/WORKSPACE.md) legt Hermes
seinen gesamten Zustand (Config, Skills, Memory, Sessions, Logs) unter
`$HERMES_HOME` ab, das standardmäßig `~` des ausführenden Linux-Users
ist (`~/.hermes/`). Der Standard-Debian-Pfad für Nutzer-Homes ist
`/home/<user>`.

Die in [ARCHITECTURE.md](../ARCHITECTURE.md) bereits in Sprint 1
geplante Systemstruktur sieht `/srv/companion/` explizit als
"Nutzdaten der Dienste (inkl. späterem Workspace)" vor — das war zu dem
Zeitpunkt eine Vorab-Planung ohne Kenntnis der tatsächlichen
Hermes-Konvention.

## Entscheidung

Die Home-Verzeichnisse der Companion-Benutzer werden auf
`/srv/companion/<benutzername>` gesetzt (`useradd -m -d
/srv/companion/<name>`), **nicht** auf den Debian-Standardpfad
`/home/<name>`.

Damit landet der spätere Hermes-Workspace jedes Benutzers automatisch
unter `/srv/companion/<name>/.hermes/` — exakt an der Stelle, die
bereits in Sprint 1 für Dienst-Nutzdaten vorgesehen war, ohne dass
Hermes' eigene Konvention verändert oder umgangen werden muss
("Upstream First").

## Konsequenzen

- Kein Zielkonflikt zwischen der in Sprint 1 geplanten Systemstruktur
  und Hermes' tatsächlicher `~/.hermes/`-Konvention.
- Abweichung vom Debian-Standardpfad `/home/`, muss bei künftigen
  Administrations-/Backup-Skripten berücksichtigt werden (z. B.
  `hermes backup` läuft pro Benutzer korrekt, solange es als der
  jeweilige Benutzer ausgeführt wird — der Pfad selbst ist für Hermes
  irrelevant, da es stets von `$HOME` ausgeht).
- `/etc/companion/` bleibt dadurch klar für PixelHermes-eigene
  (Nicht-Hermes-)Konfiguration reserviert, wie in
  [docs/hermes/DEPLOYMENT.md](../docs/hermes/DEPLOYMENT.md) empfohlen.
