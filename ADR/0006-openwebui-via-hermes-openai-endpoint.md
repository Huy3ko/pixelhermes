# 0006. OpenWebUI verbindet sich über Hermes' OpenAI-kompatiblen Endpoint

Status: Accepted

## Kontext

Phase 8.1 ging ursprünglich von einer separaten, vom generischen
OpenAI-kompatiblen Zugriff getrennten "Hermes Agent API" aus. Eine
Quellcode-Prüfung von `gateway/platforms/api_server.py` widerlegte
diese Prämisse: `POST /v1/chat/completions` instanziiert exakt dieselbe
`AIAgent`-Klasse wie die CLI und jeder Gateway-Platform-Adapter
(Telegram, Discord, ...). Ein Quellcode-Kommentar bestätigt das
wörtlich: "The API server creates a server-side Hermes AIAgent." Es
gibt keine zweite, "vollständigere" API — der OpenAI-förmige Endpoint
*ist* der volle Agent, absichtlich so gebaut, damit OpenAI-Format-
Frontends (Open WebUI, LobeChat, LibreChat, ...) ohne Custom-Code den
vollen Funktionsumfang bekommen. Ein Vergleich mit OpenClaws
äquivalentem Design (`docs.openclaw.ai/gateway/openai-http-api`)
bestätigte dasselbe Muster dort: auch OpenClaws OpenAI-kompatibler
Endpoint läuft "the same codepath as `openclaw agent`".

## Entscheidung

OpenWebUI wird ausschließlich über Hermes' offiziellen, unveränderten
OpenAI-kompatiblen API-Server (`API_SERVER_ENABLED=true`,
`/v1/chat/completions`) angebunden — über OpenWebUIs eingebauten,
generischen "OpenAI"-Verbindungstyp, exakt wie im offiziellen
OpenWebUI-Integrationsguide (`hermes-agent.nousresearch.com/docs/user-guide/messaging/open-webui`)
beschrieben. Keine Kompatibilitätsschicht, kein Custom-Proxy.

Weitere damit verbundene Entscheidungen:

- **Hermes' Gateway läuft als offizieller `systemd --user`-Dienst**
  (`hermes gateway install`, nicht selbst geschrieben) — nötig, weil
  der API-Server nur läuft, während der Gateway-Prozess läuft
  (`api_server.py` ist ein Gateway-Platform-Adapter, kein eigenständiger
  Prozess). `loginctl enable-linger hermes_hugo` macht das ohne aktive
  Login-Session dauerhaft möglich.
- **OpenWebUI bekommt einen eigenen Systembenutzer** (`openwebui`,
  Home `/opt/companion/openwebui/`) nach dem in
  [ADR 0003](0003-shared-services-under-opt-companion.md) etablierten
  Muster für geteilte/Frontend-Dienste, mit eigenem
  `systemd`-System-Service (nicht `--user`, da kein eigener Login-Bedarf
  besteht).
- **OpenWebUI bindet an `127.0.0.1`, nicht den Standard `0.0.0.0`** —
  Konsistenz mit jedem anderen Dienst in diesem Stack (Honcho,
  Embedding-Server, Hermes' eigenes Dashboard-Standardverhalten), auch
  wenn `ufw` Port 8080 ohnehin bereits blockierte.
- **Einziger Benutzer über Umgebungsvariablen statt Browser-Klicks**
  (`WEBUI_ADMIN_EMAIL`/`_PASSWORD`/`_NAME`) — OpenWebUIs offiziell
  dokumentierter Headless-Setup-Pfad, kein Workaround. Deaktiviert
  Signup automatisch nach dem ersten Nutzer (OpenWebUI-eigenes
  Verhalten, keine Zusatzkonfiguration nötig).

## Konsequenzen

- Voller Funktionsumfang (Tools, Memory, Skills, Workspace) bleibt
  über OpenWebUI erreichbar — real bestätigt über Prompt-Token-Parität
  zwischen direktem Hermes-Zugriff und OpenWebUI-vermitteltem Zugriff,
  sowie über `platform=api_server`-Einträge in Hermes' eigenem
  `agent.log`.
- Erste Erweiterung des Gateway-Dauerbetriebs in diesem Projekt — bis
  Sprint 7/Phase 8.1 lief der Hermes-Gateway nur testweise manuell
  gestartet und wieder gestoppt (Sprint 4/6). Ab jetzt läuft er dauerhaft,
  da OpenWebUI eine bestehende Verbindung braucht, keine Einzeltests.
- Kein Caddy/HTTPS/Reverse-Proxy/DuckDNS in dieser Phase — Zugriff nur
  lokal/loopback, wie explizit gefordert. Öffentlicher Zugriff bleibt
  eine spätere, bewusste Entscheidung.
