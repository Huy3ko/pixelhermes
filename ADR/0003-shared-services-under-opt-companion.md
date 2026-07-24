# 0003. Geteilte Dienste als eigene Systembenutzer unter /opt/companion

Status: Accepted

## Kontext

Sprint 7 führt die ersten Dienste ein, die nicht — wie Hermes selbst —
an einen einzelnen Companion-Benutzer gebunden sind, sondern von
mehreren Hermes-Agenten (aktuell `hermes_hugo`, künftig
`hermes_christiane`) gemeinsam genutzt werden sollen: ein
selbstgehosteter Honcho-Memory-Server und ein lokaler
Embedding-Server (llama.cpp). Beide benötigen persistente,
langlaufende Prozesse (kein CLI-first-Modell wie Hermes) und damit
erstmals echte systemd-Services in diesem Projekt.

Die bisherige Konvention (ADR 0002) galt für Companion-*Agenten*
(Home-Verzeichnis unter `/srv/companion/<agent>`), passt aber nicht für
geteilte *Infrastruktur*-Dienste, die keinem einzelnen Agenten gehören.

## Entscheidung

Jeder geteilte Dienst bekommt:

1. Einen eigenen, unprivilegierten System-Benutzer (`useradd -r`, kein
   Passwort-Login, `-s /usr/sbin/nologin`) — z. B. `honcho`,
   `embeddings`.
2. Ein eigenes Home-/Installationsverzeichnis unter
   `/opt/companion/<dienst>/`, passend zur bereits in Sprint 1/3
   geplanten Bedeutung dieses Pfads ("Installierte Anwendungen/
   Software").
3. Eine eigene systemd-Unit (`companion-<dienst>.service`), die den
   Dienst als diesen Benutzer ausführt, mit `NoNewPrivileges=true` und
   — wo sinnvoll — `ProtectSystem=strict`/`ReadWritePaths` eingegrenzt.

Damit ist jeder Dienst vom OS her vollständig von den
Companion-Agent-Benutzern isoliert — kein gemeinsames Home, kein
gemeinsamer Prozessbesitzer.

## Konsequenzen

- Erste echte systemd-Services im Projekt — bisher galt für Hermes
  selbst durchgängig "kein systemd" (Sprint 4/6), das betrifft aber
  Hermes' eigenen Gateway/API-Server, nicht geteilte
  Infrastrukturdienste, die von Natur aus dauerhaft laufen müssen, um
  nutzbar zu sein.
- Erweiterbar: ein dritter, vierter geteilter Dienst folgt demselben
  Muster ohne neue Entscheidung.
- PostgreSQL und Redis (Debian-Pakete) laufen unter ihren eigenen,
  vom Paket vorgegebenen Systembenutzern (`postgres`, `redis`) — hier
  wird bewusst nicht von der Debian-Konvention abgewichen.
