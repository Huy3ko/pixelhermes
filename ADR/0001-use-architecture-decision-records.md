# 0001. Architecture Decision Records verwenden

Status: Accepted

## Kontext

PixelHermes ist als langfristiges Infrastrukturprojekt angelegt
("Git First", "Infrastructure as Code"). Über mehrere Sprints hinweg
werden Entscheidungen getroffen — insbesondere Abweichungen vom Upstream
("Upstream First") — deren Begründung sonst nur im Gedächtnis oder in
verstreuten Commit-Messages existiert und mit der Zeit verloren geht.

## Entscheidung

Nicht-triviale Architekturentscheidungen werden als Architecture
Decision Records im Verzeichnis [`ADR/`](README.md) festgehalten, nach
dem in [ADR/README.md](README.md) beschriebenen Format und
Nummerierungsschema.

## Konsequenzen

- Entscheidungen und ihre Begründung bleiben nachvollziehbar, auch wenn
  sich der Kontext später ändert.
- Zusätzlicher, aber geringer Dokumentationsaufwand bei größeren
  Entscheidungen.
- Triviale Entscheidungen werden bewusst nicht per ADR dokumentiert, um
  das Verzeichnis nutzbar zu halten (Prinzip: Minimalistisch).
