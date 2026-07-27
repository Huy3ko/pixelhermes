# Hermes Agent — Google Workspace Skill

Quellen: [Google Workspace Skill (bundled)](https://hermes-agent.nousresearch.com/docs/user-guide/skills/bundled/productivity/productivity-google-workspace),
[Skills Feature](https://hermes-agent.nousresearch.com/docs/user-guide/features/skills),
[skills/productivity/google-workspace/SKILL.md (GitHub)](https://github.com/NousResearch/hermes-agent/blob/main/skills/productivity/google-workspace/SKILL.md)

Diese Recherche entstand als Audit-Anfrage ("Google Workspace für den
produktiven Hermes-Agent aktivieren"). Wie in Sprint 2 gilt: reine
Recherche und Dokumentation der **offiziellen** Integration — **keine**
Installation, Konfiguration oder OAuth-Ausführung. Grund: diese Sitzung
lief in einer isolierten, ephemeren Container-Umgebung mit ausschließlich
diesem Git-Repository — ohne Shell-/SSH-Zugriff auf den realen
`hermes_hugo`-Host (`/srv/companion/hermes_hugo/`) und ohne Zugriff auf
ein Google-Cloud-Konto des Nutzers. Beides ist für eine echte Aktivierung
zwingend erforderlich (siehe "Nächste Schritte" unten) und kann von
keiner KI-Sitzung im Namen des Nutzers ersetzt werden — insbesondere die
OAuth-Client-Erstellung in der Google Cloud Console erfordert eine
Google-Kontoanmeldung des Nutzers selbst.

## Ist-Zustand (Audit)

- **Skill vorhanden, nicht aktiviert:** `google-workspace` ist einer der
  65 builtin Skills, Kategorie `productivity` (real beobachtet bei der
  Installation, Phase 4 — siehe [SKILLS.md](SKILLS.md#real-beobachtetes-skill-inventar--hermes_hugo-phase-4-2026-07-24)).
  Builtin heißt: der Skill-Code liegt bereits auf der Instanz, **nicht**
  dass er authentifiziert oder in Benutzung ist.
- **Kein MCP-Server nötig/beteiligt:** Google Workspace ist kein Eintrag
  im offiziellen MCP-Katalog (`blender`, `linear`, `n8n`,
  `unreal-engine` — siehe [MCP.md](MCP.md)) und läuft nicht über MCP,
  sondern als eigenständiger Skill mit Python-Scripts
  (`scripts/setup.py`, `scripts/google_api.py`).
  Für dieses Ziel ist also **kein zusätzliches Plugin, kein Skill-Hub-
  Install und kein MCP-Server** nötig — der Skill ist bereits Teil der
  Standardinstallation.
- **Keine Credentials im Repository:** `configs/` ist laut
  [configs/README.md](../../configs/README.md) aktuell leer; `.gitignore`
  schließt `*.env`, `**/*_secret*`, `**/*.key`, `**/*.pem` und
  `configs/local.*` ohnehin kategorisch aus. Es existierte vor diesem
  Audit keine Google-OAuth-Konfiguration in diesem Repo — korrekt, da
  Secrets grundsätzlich nicht versioniert werden.
- **Keine Redirect-URL/Client-ID vorhanden:** Der OAuth-Client
  (Client-ID/-Secret) muss pro Google-Cloud-Projekt einmalig vom Nutzer
  selbst erzeugt werden (siehe unten) — es gibt keine von Hermes
  vorgegebene, feste Client-ID. Ein Login-Link ohne existierende
  Client-ID ist technisch nicht bildbar.

## Funktionsweise (offiziell dokumentiert)

Der Skill ist **kein OAuth-2.1/MCP-Fluss** wie bei Remote-MCP-Servern
(siehe [MCP.md](MCP.md)), sondern ein klassischer Google-OAuth-2.0-Flow
mit PKCE, komplett über zwei Python-Einstiegspunkte gesteuert:

```bash
GSETUP="python ${HERMES_HOME:-$HOME/.hermes}/skills/productivity/google-workspace/scripts/setup.py"
GAPI="python ${HERMES_HOME:-$HOME/.hermes}/skills/productivity/google-workspace/scripts/google_api.py"
```

### Setup-Ablauf (fünf Schritte, offiziell dokumentiert)

1. `$GSETUP --check` — prüft, ob bereits ein Token existiert.
2. Google-Cloud-Projekt anlegen, benötigte APIs aktivieren, OAuth-Client
   vom Typ **Desktop App** erzeugen, `google_client_secret.json`
   herunterladen (**nur der Nutzer selbst**, in der Google Cloud
   Console — kein API-Weg, den ein Agent für den Nutzer ausführen
   kann).
3. `$GSETUP --client-secret /pfad/zur/datei.json` — registriert den
   heruntergeladenen Client lokal auf der Hermes-Instanz.
4. `$GSETUP --auth-url --services email,calendar,drive,sheets,docs --format json`
   — erzeugt den echten, ausführbaren Google-OAuth-Login-Link (enthält
   die reale Client-ID aus Schritt 3 sowie einen PKCE Code Challenge —
   **ohne Schritt 3 nicht erzeugbar**).
5. Login-Link im Browser öffnen, Google-Konto autorisieren, die vom
   Browser angezeigte (ggf. fehlerhafte, weil `localhost:1` nicht
   bedient wird) Redirect-URL komplett kopieren, dann
   `$GSETUP --auth-code "VOLLSTÄNDIGE_URL_ODER_CODE" --format json`
   ausführen, abschließend mit `$GSETUP --check` verifizieren.

### Redirect URI

Fest `http://localhost:1` — es läuft **kein** lokaler Webserver, der
den Callback tatsächlich bedient. Der Browser zeigt nach der Google-
Freigabe einen Verbindungsfehler; das ist erwartet. Der Autorisierungs-
Code steckt im Query-String der (fehlgeschlagenen) Redirect-URL in der
Adressleiste, die manuell zurück in `--auth-code` eingefügt wird — kein
klassischer Server-Callback, sondern Copy/Paste-basierter Abschluss des
PKCE-Flows.

### Benötigte Google-Cloud-APIs (vom Nutzer in der Cloud Console zu aktivieren)

- Gmail API
- Google Calendar API
- Google Drive API
- Google Sheets API
- Google Docs API
- People API (nur für Kontakte, Teil von `--services all`)

### OAuth Scopes (dynamisch über `--services`, nicht pauschal `all`)

Der Skill fragt gezielt nur die Scopes an, die über `--services` gewählt
werden — offiziell empfohlen, um den Consent-Screen minimal zu halten:

| `--services`-Wert | Deckt ab |
|---|---|
| `email` | Gmail (lesen/senden/verwalten) |
| `calendar` | Google Calendar (lesen/schreiben) |
| `drive` | Google Drive (Dateizugriff) |
| `sheets` | Google Sheets |
| `docs` | Google Docs |
| `all` | alle oben + Contacts/People API |

Für PixelHermes sinnvoll (deckt exakt die in der Aufgabenstellung
genannten fünf Dienste ab, ohne Contacts): `email,calendar,drive,sheets,docs`.

### Datei-/Tokenablage

| Datei | Inhalt |
|---|---|
| `google_client_secret.json` (Pfad frei wählbar, vom Nutzer aus der Cloud Console heruntergeladen) | OAuth-Client-Credentials |
| `~/.hermes/google_token.json` | Access-/Refresh-Token, automatisch angelegt und erneuert |
| `~/.hermes/google_oauth_pending.json` | temporärer PKCE-Verifier während eines offenen Auth-Vorgangs |
| `~/.hermes/google_oauth_last_url.txt` | zuletzt erzeugter Login-Link, zur Referenz |

Keine dieser Dateien gehört ins Git-Repository (siehe `.gitignore`) —
sie entstehen ausschließlich auf der realen Hermes-Instanz.

### Alternative für reine E-Mail-Nutzung

Falls nur Gmail (kein Calendar/Drive/Sheets/Docs) benötigt wird, nennt
die offizielle Doku den bereits vorhandenen `himalaya`-Skill (Kategorie
`email`, ebenfalls builtin — siehe [SKILLS.md](SKILLS.md)) als
schnelleren Weg über ein Gmail-App-Passwort, ganz ohne Google-Cloud-
Projekt. Da die Aufgabenstellung explizit Drive/Docs/Sheets/Calendar
verlangt, ist das hier nicht ausreichend — nur der volle
`google-workspace`-Skill deckt alle fünf Dienste ab.

---

## Nächste Schritte — nur real durch den Nutzer ausführbar

Diese Schritte können **nicht** von dieser (oder einer beliebigen)
Agent-Sitzung im Namen des Nutzers durchgeführt werden, weil Schritt 1
eine interaktive Google-Kontoanmeldung erfordert und Schritt 2–4 realen
Zugriff auf den `hermes_hugo`-Host voraussetzen:

1. In der [Google Cloud Console](https://console.cloud.google.com/)
   (eigenes oder neues Projekt) die sechs oben genannten APIs aktivieren
   und einen OAuth-Client vom Typ **Desktop App** erzeugen,
   `google_client_secret.json` herunterladen.
2. Die Datei auf den `hermes_hugo`-Host übertragen (z. B. `scp`), dort
   `$GSETUP --client-secret /pfad/zur/datei.json` ausführen.
3. `$GSETUP --auth-url --services email,calendar,drive,sheets,docs --format json`
   ausführen — **das erzeugt den echten, funktionierenden Login-Link**,
   mit der realen Client-ID aus Schritt 1/2. Diesen Link öffnen,
   Google-Konto autorisieren.
4. Die vom Browser angezeigte Redirect-URL zurück in
   `$GSETUP --auth-code "URL" --format json` einfügen, danach
   `$GSETUP --check` zur Verifikation.
5. Danach real verifizieren (analog zu allen bisherigen Sprints, siehe
   [PRODUCTIVE_RUNTIME.md](PRODUCTIVE_RUNTIME.md)): je einen echten
   Lese-Aufruf über `$GAPI` gegen Gmail, Calendar, Drive, Sheets, Docs
   durchführen und im `agent.log` bestätigen — kein Aktivieren "auf
   Verdacht" ohne echten Funktionstest, passend zum Projektgrundsatz
   "real getestet, nicht vorgetäuscht".

---

## PixelHermes-Mapping

**Was übernimmt Hermes bereits?** Einen vollständigen, bereits
mitinstallierten Google-Workspace-Skill mit eigenem OAuth-/PKCE-Flow,
Token-Refresh und CLI-Wrapper für Gmail, Calendar, Drive, Sheets, Docs
und Contacts — kein separates Plugin, kein MCP-Server, kein Hub-Install
nötig.

**Was müssen wir NICHT selbst entwickeln?** Keinen eigenen OAuth-Client,
keine eigene Token-Refresh-Logik, keine eigene Google-API-Anbindung —
alles bereits Teil der Standardinstallation (65 builtin Skills, Phase 4).

**Was passt direkt zu PixelHermes?** Genau eine Client-ID pro
Google-Cloud-Projekt, ein Token pro Companion-Benutzer
(`~/.hermes/google_token.json` liegt im Home des jeweiligen
`hermes_*`-Benutzers) — passt zur bestehenden Ein-Benutzer-pro-Home-
Struktur ([ADR 0002](../../ADR/0002-companion-user-home-under-srv.md)).
Scope-Wahl `email,calendar,drive,sheets,docs` (ohne `all`/Contacts)
passt zum Minimalismus-Prinzip: nur anfragen, was gebraucht wird.

**Welche Erweiterungen wären später sinnvoll?** Nach erfolgreicher
Aktivierung für `hermes_hugo`: dieselben Schritte für
`hermes_christiane` wiederholen (eigenes Google-Konto, eigener Token —
kein Client-Secret-Sharing zwischen Companion-Benutzern).

**Welche Komponenten sollten wir bewusst unverändert übernehmen?** Den
kompletten Skill (Setup- und API-Scripts) unangetastet lassen — keine
eigene Fork-Anpassung, keine eigene Token-Ablage außerhalb von
`~/.hermes/`.
