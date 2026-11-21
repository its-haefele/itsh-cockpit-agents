# Änderungen der Agent-Versionen

Alle Änderungen am ITSH Cockpit Agent seit Beginn der signierten Pakete, neueste oben.
Die Einträge werden beim Veröffentlichen einer Version automatisch ergänzt.
Agents vor 0.32.0 aktualisieren sich nicht selbst und werden einmal von Hand mit `install` aktualisiert.

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

