# Architecture Decision Records

Dieses Verzeichnis enthält die Architecture Decision Records (ADRs) von
PixelHermes. Ein ADR hält eine einzelne, nicht-triviale
Architekturentscheidung fest: Kontext, Entscheidung, Konsequenzen.

## Wann ein ADR anlegen

- Bei jeder Abweichung vom Upstream ("Upstream First" verlangt eine
  Begründung).
- Bei jeder Entscheidung, die schwer rückgängig zu machen ist oder
  mehrere spätere Sprints beeinflusst.
- Bei jeder Entscheidung zwischen mehreren plausiblen Alternativen.

Triviale, leicht reversible Entscheidungen brauchen kein ADR.

## Format

Ein ADR pro Datei, fortlaufend nummeriert:
`NNNN-kurzer-titel-in-kebab-case.md`

```markdown
# NNNN. Titel

Status: Proposed | Accepted | Superseded by NNNN

## Kontext

Welches Problem oder welche Frage liegt vor?

## Entscheidung

Was wurde entschieden?

## Konsequenzen

Was folgt daraus — positiv wie negativ?
```

## Index

- [0001. Architecture Decision Records verwenden](0001-use-architecture-decision-records.md)
- [0002. Home-Verzeichnis der Companion-Benutzer unter /srv/companion](0002-companion-user-home-under-srv.md)
- [0003. Geteilte Dienste als eigene Systembenutzer unter /opt/companion](0003-shared-services-under-opt-companion.md)
- [0004. Lokaler llama.cpp-Embedding-Server statt Cloud-Provider für Honcho](0004-local-embedding-server-for-honcho.md)
- [0005. Bedingte Zulassung von Drittanbieter-Erweiterungen](0005-third-party-extension-policy.md)
- [0006. OpenWebUI verbindet sich über Hermes' OpenAI-kompatiblen Endpoint](0006-openwebui-via-hermes-openai-endpoint.md)
- [0007. Eigenes leichtgewichtiges Tracing-Plugin statt Langfuse](0007-local-tracing-plugin-not-langfuse.md)
- [0008. Mnemosyne ersetzt Honcho als Memory-Provider](0008-mnemosyne-replaces-honcho.md)
