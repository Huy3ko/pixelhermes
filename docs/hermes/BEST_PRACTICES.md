# Hermes Agent — Best Practices für PixelHermes

Synthese aus allen Kapiteln (siehe [OVERVIEW.md](OVERVIEW.md) für die
vollständige Dokumentliste). Dies ist die Grundlage für den späteren
Build Guide (Sprint 3+) — noch keine Installationsanleitung, sondern
die aus der offiziellen Doku abgeleiteten Leitplanken.

## 1. Upstream First konkret einhalten

- Installation ausschließlich über den offiziellen Weg (Curl-Skript /
  Desktop-Installer), niemals über pip/pipx/Homebrew/AUR — diese sind
  laut Doku explizit **nicht unterstützt**
  ([INSTALLATION.md](INSTALLATION.md)).
- Updates ausschließlich über `hermes update`, nie manuelles Patchen des
  Repos.
- Keine eigene Fork-Logik für Kontextdateien, Skills, MCP-Client,
  Gateway oder Session-Storage — alles bereits vorhanden
  ([ARCHITECTURE.md](ARCHITECTURE.md), [SKILLS.md](SKILLS.md),
  [MCP.md](MCP.md)).
- Jede geplante Abweichung vom Upstream-Verhalten braucht laut
  [ADR-Prozess](../../ADR/README.md) einen eigenen ADR-Eintrag, bevor sie
  umgesetzt wird.

## 2. Profile statt eigener Multi-Agent-Architektur

Der im Konzept vorgesehene Gedanke "mehrere benannte Agenten" bildet
sich 1:1 auf Hermes-**Profile** ab (siehe [ARCHITECTURE.md](ARCHITECTURE.md),
[PERSONAS.md](PERSONAS.md)) — kein eigenes Konzept für
Multi-Agent-Isolation nötig. Wichtig: Profile isolieren **Zustand**,
nicht das Dateisystem — für echte Sicherheitsisolation ist zusätzlich
ein Terminal-Backend wie Docker oder SSH nötig
([DEPLOYMENT.md](DEPLOYMENT.md)).

## 3. Workspace-Struktur ist bereits `~/.hermes/`

Der frühere Grundsatz "Workspace First — erst nach Installation
analysieren" ist durch diese Recherche beantwortet: die Struktur ist
vollständig dokumentiert ([WORKSPACE.md](WORKSPACE.md)). PixelHermes muss
sie nicht erfinden, nur respektieren.

## 4. Modellwahl vor Installation klären

Mindestens 64.000 Token Kontext, und bei lokalen Modellen nachweisliche
Tool-Calling-Unterstützung (siehe [MODELS.md](MODELS.md)) — sonst
funktionieren Skills/Tools/MCP nicht zuverlässig. Diese Entscheidung
sollte vor Sprint 3 als ADR festgehalten werden.

## 5. Sicherheits-Defaults nicht aufweichen

Hermes liefert bereits ein Defense-in-Depth-Modell
([SKILLS.md](SKILLS.md), Abschnitt Security). Für PixelHermes gilt:
- Produktions-Checkliste aus der Doku befolgen (Allowlists statt
  Allow-all, Docker/SSH-Backend statt local bei Fremdzugriff,
  `.env` mit `chmod 600`, DM-Pairing statt hartkodierter IDs).
- `--yolo`/`security.dangerous_command_approval: off` nicht ohne
  expliziten, dokumentierten Grund verwenden.

## 6. Systempfade sauber trennen

Hermes bringt seine eigene, in sich geschlossene Verzeichniskonvention
(`~/.hermes/`) mit. Die in [ARCHITECTURE.md](../../ARCHITECTURE.md)
geplanten `/etc/companion`, `/var/log/companion`,
`/var/backups/companion` sollten **nicht** versuchen, Hermes-interne
Pfade zu duplizieren oder zu verlegen — stattdessen ergänzend nutzen
(z. B. `/var/backups/companion/` als Ziel für **Kopien** von
`hermes backup`-Archiven, siehe [DEPLOYMENT.md](DEPLOYMENT.md)).

## 7. Reihenfolge für Sprint 3 (Empfehlung, keine Entscheidung)

1. Modell-/Provider-Entscheidung per ADR.
2. Installation nach [INSTALLATION.md](INSTALLATION.md), zunächst als
   Einzelbenutzer-Installation ohne Gateway.
3. `AGENTS.md` für das PixelHermes-Repository schreiben (sobald
   Projektkonventionen feststehen).
4. Ein erstes Profil für Test/Exploration anlegen, bevor produktive
   Profile (z. B. für benannte Agenten) entstehen.
5. Gateway/Dashboard/Reverse-Proxy erst mit konkretem Bedarf für
   Fernzugriff einrichten, nicht vorab.

Diese Reihenfolge ist eine Empfehlung für die Planung von Sprint 3, kein
Bestandteil dieses (Sprint-2-)Auftrags — es wird hier **nichts**
installiert oder konfiguriert.

## Kapitelübergreifende offene Punkte

Siehe [OPEN_QUESTIONS.md](OPEN_QUESTIONS.md) für alle Punkte, die die
offizielle Dokumentation nicht abschließend klärt.
