# Änderungen der Agent-Versionen

Alle Änderungen am ITSH Cockpit Agent seit Beginn der signierten Pakete, neueste oben.
Die Einträge werden beim Veröffentlichen einer Version automatisch ergänzt.
Agents vor 0.32.0 aktualisieren sich nicht selbst und werden einmal von Hand mit `install` aktualisiert.

## [0.36.22] – 2026-11-24
### Hinzugefügt
- **Onboarding-Freischaltung:** Die Gerätesperre `denyJobs` lässt sich aus dem Cockpit aufheben – mit Einwilligung eines auftragsberechtigten Ansprechpartners (Mail mit Link, öffentliche Seite `/onboarding/<Token>`, 3 Stunden, einmal nutzbar) und einem auf dem Entwicklungs-PC signierten Freischalt-Ticket (eigener Signatur-Kontext, 3 Stunden gültig). Kachel „Onboarding“ auf der Kundenseite (nur Globaler Admin, 2FA): Vorgang anlegen, Mail erneut senden, Ticket-Anfrage herunterladen, Ticket hochladen, abbrechen; Zustand je System, Hinweis an der Glocke, Verlauf und Audit-Log. Der Agent prüft Signatur, Agent-ID, Ablauf und Einmal-Wert, sichert die Konfiguration und meldet das Ergebnis.
- **Strenge Gerätesperre:** `install -deny-jobs-strict` (`"denyJobs": "strict"`) lässt sich nur am Gerät aufheben; Tickets werden abgelehnt, das Cockpit bietet keinen Vorgang an.
- **Werkzeug `01_build\New-UnlockTicket.ps1`** (mit `release unlock-ticket`): erzeugt aus der Ticket-Anfrage das signierte Ticket; der Signaturschlüssel (`ITSH_SIGNING_KEY`) bleibt auf dem Entwicklungs-PC. Vor dem Signieren zeigt es Vorgang, Kundennummer und Agent-IDs der Anfrage und fragt „Signieren? (ja/nein)“ (`-Yes` nur für Scripte).

### Hinweise
- **Agent-Paket 0.36.22 nötig** (Agents ab dieser Version verstehen das Ticket und die strenge Sperre).
- Migration `0040` (Onboarding-Vorgänge).
- **Sophos-WAF:** neuer öffentlicher Pfad `/onboarding/*` (Link-Seiten); ggf. Upload-Ausnahme für `/customers/*/onboarding/*/ticket`; Regelwerk in `doku\Betrieb.md`, Abschnitt 4.

## [0.35.5] – 2026-11-23
### Hinzugefügt
- **Update-Hinweis in der Abfrage** (Design: `doku/Agent-Update-Hinweis.md`): Die Antwort von `POST /api/v1/agents/poll` enthält `updatePending` (manuell angestoßenes Update steht an) und `updateCheckMinutes` (wirksamer Prüfabstand). Ein manuell angestoßenes Update holt der Agent beim nächsten Kontakt (rund 5 Minuten) statt erst im Stundenrhythmus; Auto-Update setzt keinen Hinweis.
- **Prüfabstand für Agent-Updates** einstellbar: global unter Einstellungen → Updates und je Kunde auf der Kundenseite (`POST /customers/{number}/agent-check`); 15 Min. bis 24 Std., Standard 1 Std., mit Audit-Eintrag.

### Geändert
- Agent: Die regelmäßige Update-Prüfung ist zufällig innerhalb des Prüfabstands verteilt; der Abstand kommt vom Cockpit (nicht aus der `config.json`). Die Prüfung nach einem Inventar-Bericht bleibt.
- Agent: Es bleibt kein ungenutzter Registrierungs-Token in der Konfiguration. `install -token` auf einem bereits registrierten Rechner meldet „already enrolled – token not needed“ und speichert den Token nicht; beim Start entfernt der Agent einen übrig gebliebenen `enrollToken`. Eine Neuregistrierung erfordert weiterhin zuerst `uninstall -purge`.
- Befehlsblöcke (Windows und Linux): Sie erkennen einen bereits registrierten Rechner (nicht leerer `agentKey` in der Agent-Konfiguration, ohne Werte auszugeben) und melden dann „[ITSH] Fertig. Agent aktualisiert, der Rechner bleibt im Cockpit registriert.“, sonst wie bisher „… erscheint in ca. 3 Minuten im Cockpit.“
- Doku: Prüfabstand und Hinweis in `Agent-Updates.md`, `Einstellungen.md`, `Betrieb.md`; Onboarding-Freischaltung in `Agent-Steuerung.md` und `Agent-Befehlsketten.md` auf 0.35.8 verschoben.

## [0.35.4] – 2026-11-22
### Hinzugefügt
- Installations-Befehlsketten (0.35.4): Nach dem Erstellen eines Registrierungs-Tokens zeigt die Kundenseite fertige Befehlsblöcke je Plattform (Windows PowerShell, Linux) in den Varianten „mit Jobs“ und „nur Inventar“, mit Token-Abfrage oder Token aus Datei. Die Blöcke laden den Agent vom Cockpit und vergleichen den SHA-256-Hash mit dem Wert, den das Cockpit aus seinem signaturgeprüften Manifest einsetzt (die Signatur prüft der Block selbst nicht; Vertrauensanker ist die Cockpit-Seite per HTTPS).
- Öffentlicher Download der freigegebenen Agent-Version unter `GET /downloads/public/agent/*` (nur freigegebene, signaturgeprüfte Version, strenge Prüfung von Manifest und Host).
- Agent: Option `install -token-file <Datei>` (erste nicht leere Zeile, Datei bleibt erhalten); die Blöcke nutzen sie nur für Agents ab 0.35.4.
- Server-Adresse in den Blöcken aus `COCKPIT_PUBLIC_HOST`.

### Sicherheit
- Windows-Block: Setup-Ordner `C:ProgramDataITSH Cockpitsetup` mit Besitzer und geschützter DACL, nach dem Schützen erneut geprüft, Datei-Handle während der Prüfung offen gehalten.
- Linux-Block läuft in einer Subshell-Funktion; die Token-Abfrage liest von `/dev/tty` ohne Echo, der Token landet nicht in der History.

## [0.35.0] – 2026-11-22
### Hinzugefügt
- **Agent-Steuerung:** Schalter „Jobs“ je System im Cockpit (Systemseite, mit 2FA); Abzeichen „Jobs: an“, „Jobs: aus“, „Jobs: am Gerät gesperrt“, „Wird entfernt“, „Entfernt“ und Hinweis bei Agents < 0.35.0. Neues Recht `permAgentControl` (Globaler Admin, Kunden-Admin für eigene Kunden).
- **Agent entfernen:** je System und gesammelt je Kunde (Kundenseite, hinter „Ändern“); der Agent entfernt sich nach Bestätigung selbst (Dienst, Konfiguration, Schlüssel, Logs) und stoppt ohne Neustart; Zurücknehmen bis zur Bestätigung möglich. Bestätigt wird erst, wenn der Helfer kopiert und geprüft ist; der Helfer schreibt `uninstall.log` neben seine Kopie. Hinweis bei inaktiven Kunden mit Link; Systemfilter „Entfernt“.
- Agent: `install -deny-jobs` und Konfiguration `denyJobs`, `allowRemoteCommands`; gemeinsame Abfrage `POST /api/v1/agents/poll` für Jobs und Befehle alle `jobPollMinutes` (Standard 5), auch bei `denyJobs`. Ausgelieferte Jobs auf gesperrten Geräten werden als fehlgeschlagen („denied on device“) gemeldet.
- Migration `0031_agent_control.sql`; Verlauf und Audit für Schalter, Entfernen und Gerätesperre.

### Geändert
- `allowRemoteJobs` entfällt (wird ignoriert, einmaliger Log-Hinweis, Datei wird nicht umgeschrieben). **Hinweis:** Agents nehmen nach dem Update auf 0.35.0 Jobs an, sofern nicht `denyJobs` gesetzt ist (bewusst; bisher sind nur interne Agents im Einsatz).
- Entfernte Systeme zählen nicht mehr für Kacheln, Agent-Updates und die Managed-Pauschale. Die alte Job-Abfrage `PathJobs` bleibt für Agents < 0.35.0 bestehen.

### Behoben
- Agent entfernen (Linux): Helfer liegt unter `/var/lib/itsh-cockpit-remove` statt `/run` (auf Debian 13 `noexec`); vor der Bestätigung wird geprüft, dass der Ordner nicht `noexec` ist. `uninstall.log` übersteht einen Neustart.

### Sicherheit
- Der Server prüft Jobs-Schalter, Entfernen und Gerätesperre atomar beim Ausliefern von Jobs (`ClaimJobs`); Jobs entfernter Agents werden mit „agent removed“ abgebrochen. Der Agent prüft `denyJobs` zusätzlich selbst. Das Entfernen-Hilfsprogramm liegt außerhalb des Programmordners (Windows: geschützter Ordner unter `%ProgramData%`, Linux: `systemd-run`).

### Hinweis
- „Agent entfernen“ ist bisher nur mit Tests (Attrappen) geprüft; der Echttest unter Windows (DEUMTITSHNB00) und Linux (DEUMTITSHDMZ253) folgt nach dem Release.

## [0.33.15] – 2026-11-21
### Geändert
- Systemübersicht, Spalte Kunde: kompakt „Nummer · Kürzel“ bzw. nur die Kundennummer, wenn kein Kürzel eingetragen ist; der volle Name als Tooltip. Die Liste scrollt dadurch nicht mehr seitlich.
### Behoben
- Agent (Windows): Nach einem Selbst-Update startet ein kurzer Hilfsprozess den Dienst sofort neu. Bisher übernahm das die Dienst-Wiederherstellung, die nach mehreren Updates am selben Tag 5 bzw. 15 Minuten wartete. Wirkt ab dem nächsten Update nach 0.33.15 (der Wechsel auf 0.33.15 selbst läuft noch über die Wiederherstellung).

## [0.33.10] – 2026-11-21
### Behoben
- CSV-Export: Schutz gegen Formel-Einschleusung prüft das erste sichtbare Zeichen (vorher ließ ein führendes Leerzeichen z. B. „ =BEFEHL“ durch).
- Speicher knapp: Berechnung ohne möglichen Zahlenüberlauf.
- `grow-root.sh`: Der Probelauf ändert wirklich nichts mehr (Neueinlesen der Platte nur mit `--apply`).
- Agent (Linux): Austausch beim Selbst-Update in einem Schritt (atomar) – der Programmpfad zeigt nie auf eine fehlende Datei; die alte Version bleibt als Kopie für den Rückfall. Windows unverändert (laufende Programme lassen sich dort nur umbenennen).

## [0.32.2] – 2026-11-21
### Behoben
- Agent: Der gewollte Neustart nach einem Selbst-Update wird als Info („restarting for the agent update“) statt als Fehler „agent failed“ protokolliert.
- Hinweis: Der Tag `ITSH_Cockpit/v0.32.2` zeigt auf den Code-Stand; dieser CHANGELOG-Eintrag wurde unmittelbar danach nachgetragen.

## [0.32.1] – 2026-11-21
### Behoben
- Versionsnummern von Agent-Paketen werden streng geprüft (nur `X.Y.Z` aus reinen Zahlen, keine führenden Nullen oder Vorzeichen).
### Hinweis
- Erste Version, die über das Auto-Update an Agents ab 0.32.0 verteilt wird (Test des Selbst-Updates unter Windows und Linux).

## [0.32.0] – 2026-11-21
### Hinzugefügt
- **Agent-Releases und Auto-Update** (Design: `doku/Agent-Updates.md`): signierte Agent-Pakete (Ed25519, Schlüssel nur in der Entwicklung) mit Änderungstext; Veröffentlichung im öffentlichen Repo `its-haefele/itsh-cockpit-agents` (`Build.ps1 -Release`, `Publish-AgentRelease.ps1`, Hilfsprogramm `cmd/release`).
- **Agent-Versionen im System-Center:** täglicher automatischer Abruf (abschaltbar) oder Hochladen durch den Globalen Admin; nur gültig signierte Pakete werden übernommen (höchstens 3 Versionen plus die genutzten). Neue Versionen müssen mit 2FA freigegeben werden; Hinweis oben für Globale Admins.
- **Verteilung:** Update für ein System (vor der Freigabe nur Globaler Admin zum Testen), für alle Systeme eines Kunden oder alle Systeme (erst wenn ein System die Version meldet), jeweils mit 2FA; Auto-Update global, je Kunde und je System (System „aus“ hält die Version fest). Kunden-Admins für ihre Kunden; Hinweis „Agent-Update verfügbar“ für Kunden-Admins.
- **Agent aktualisiert sich selbst:** stündliche Prüfung und nach jedem Bericht, unabhängig von `allowRemoteJobs`; nur neuere, signierte Versionen; Austausch mit Neustart über die Dienst-Wiederherstellung, Rückfall auf die alte Version nach 3 Starts ohne Serverkontakt; abschaltbar mit `disableUpdates`.
- Migration `0029_agent_updates.sql` (Agent-Versionen, Dateien, Update-Ziel je Agent).
### Geändert
- Download für Neuinstallationen nimmt die neueste freigegebene Version (sonst die Dateien aus dem Image).
- Windows-Dienst: Wiederherstellung greift auch bei gewolltem Beenden mit Fehlercode (nötig für den Neustart nach dem Update) – wirkt nach `install` mit Agent ab 0.32.0.
### Sicherheit
- Jedes Agent-Update (System, Kunde, alle) und jede Freigabe nur mit 2FA; der Agent installiert nur neuere, signierte Versionen (kein Zurückstufen).
- Breite Verteilung und Auto-Update erst, wenn ein vom Cockpit angestoßenes Update erfolgreich war (nicht anhand einer vom Agent selbst gemeldeten Version).
- Release-Pakete: Grenzen für entpackte Größe und Dateianzahl, eindeutige Dateien und Plattformen im signierten Manifest.

