# 0004. Lokaler llama.cpp-Embedding-Server statt Cloud-Provider für Honcho

Status: Accepted

## Kontext

Honcho (Sprint 7) benötigt für semantische Suche ein Embedding-Modell.
Honcho unterstützt dafür nur zwei Transports: `openai` und `gemini`
(Quellcode-bestätigt: `src/config.py`, `EmbeddingTransport = Literal["openai", "gemini"]`).
Der bereits produktiv genutzte Grok/xAI-Provider bietet keine
Embeddings an. Zwei Cloud-Alternativen wurden erwogen und explizit
verworfen: OpenAI (neuer externer Cloud-Provider, widerspricht "Self
Hosted") und Google Gemini (ebenfalls neuer externer Cloud-Provider —
ein bereitgestellter Gemini-Key wurde nach Abwägung bewusst nicht
verwendet).

Quellcode-Prüfung ergab: der `openai`-Transport instanziiert lediglich
den offiziellen `AsyncOpenAI`-Client mit konfigurierbarer `base_url`
(`src/embedding_client.py`) — es handelt sich um Standard-SDK-
Verhalten, nicht um einen Cloud-spezifischen Hardcode. Damit ist ein
selbstgehosteter, OpenAI-kompatibler Embedding-Server offiziell
unterstütztes Verhalten, keine Umgehung.

Ollama wurde als Optionen bewusst ausgeschlossen (bereits in Sprint 6
als Prinzip festgehalten: "kein Ollama"). Docker-basierte Lösungen
(z. B. Hugging Face TEI) scheiden wegen "kein Docker" aus.

## Entscheidung

Ein dedizierter, lokaler Embedding-Server auf Basis von **llama.cpp**
(`llama-server`, aus dem offiziellen `ggml-org/llama.cpp`-Repository
selbst gebaut) mit dem quantisierten Modell **nomic-embed-text-v1.5**
(Q8_0-GGUF, ~139 MB, 768 Dimensionen), gebunden an `127.0.0.1:8081`,
mit eigenem API-Key geschützt (`--api-key`). Läuft als eigener
systemd-Service unter einem dedizierten Systembenutzer `embeddings`
(siehe [ADR 0003](0003-shared-services-under-opt-companion.md)).

Honcho zeigt über `EMBEDDING_MODEL_CONFIG__TRANSPORT=openai` +
`EMBEDDING_MODEL_CONFIG__OVERRIDES__BASE_URL=http://127.0.0.1:8081/v1`
auf diesen Server. Honchos eigene Text-Generierung (Deriver, Dialectic,
Summary, Dream) läuft dagegen über den bereits vorhandenen xAI/Grok-Key
(ebenfalls `openai`-Transport, `base_url` auf `api.x.ai` — kein neuer
externer Provider für diesen Teil).

## Konsequenzen

- Vollständig self-hosted für Embeddings — kein neuer Cloud-Account,
  keine laufenden Kosten für diesen Teil des Stacks.
- llama.cpp ist bereits als offiziell von Hermes unterstützte lokale
  Runtime dokumentiert (`docs/hermes/MODELS.md`) — keine neue,
  ungeprüfte Technologie im Projekt.
- Läuft CPU-only, ~24 MB RSS im Leerlauf — auf dieser VPS-Hardware
  unproblematisch für den erwarteten Durchsatz eines einzelnen
  Companion-Agenten.
- Geteilte Infrastruktur (siehe ADR 0003): dieselbe Instanz kann von
  mehreren Hermes-Agenten (`hermes_hugo`, künftig `hermes_christiane`)
  gleichzeitig genutzt werden, ohne pro Agent einen eigenen
  Embedding-Server zu betreiben.
- Modellwechsel (z. B. größeres/anderes Embedding-Modell) erfordert
  einen Neustart des Dienstes und ggf. `configure_embeddings.py` in
  Honcho, falls sich die Vektordimension ändert (siehe
  `docs/hermes/HONCHO.md`).
