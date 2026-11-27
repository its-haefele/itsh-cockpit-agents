# Änderungen der Agent-Versionen

Alle Änderungen am ITSH Cockpit Agent seit Beginn der signierten Pakete, neueste oben.
Die Einträge werden beim Veröffentlichen einer Version automatisch ergänzt.
Agents vor 0.32.0 aktualisieren sich nicht selbst und werden einmal von Hand mit `install` aktualisiert.

## [0.53.0] – 2026-11-27
### Hinzugefügt
- **Schalter „Schlüssel-Anmeldung“ und „Passwort-Anmeldung“:** Direkt neben der Überschrift „SSH“ der Fernwartungs-Kachel zwei Abzeichen (grün = möglich, rot = gesperrt bzw. nicht möglich). Ein Klick schaltet um – mit Rückfrage, 2FA-Code aus dem Seitenkopf und Recht „Jobs“; der Tooltip nennt, was der Klick tut, oder den Grund, warum nicht umgeschaltet werden kann. Solange ein Schalt-Job offen ist oder auf das nächste Inventar wartet, sind beide gesperrt („wird umgeschaltet“). Agents unter 0.53.0 zeigen die Abzeichen ohne Klick („Umschalten ab Agent 0.53.0“).
- **Job `sshd_login`** mit ausdrücklichem Soll-Zustand (`pubkey`, `password`), gebildet aus dem letzten Bericht und dem geklickten Schalter. Schutz gegen Aussperren in Zentrale und Agent: nie beide aus; Passwort aus nur mit möglicher Schlüssel-Anmeldung (inkl. Schlüsseldatei bzw. Windows-Block „Match Group administrators“) und mindestens einem aktiven SSH-Zugang; Schlüssel aus nur mit Passwort an. Audit `sshd_login`, Verlauf „SSH-Anmeldung umgeschaltet“ mit Soll-Zustand. Migration 0058 (Jobtyp). Agent-Paket 0.53.0 nötig.
- **Agent:** gleiche sichere Schritte wie 0.51.0 (Prüfung, Sicherung, Schreiben, `sshd -T`, Neuladen, Rücknahme bei Fehler, höchstens 5 Sicherungen). Der neue Cockpit-Block (`# managed by ITSH Cockpit: SSH login`) oben in `sshd_config` bzw. im Linux-Drop-in `00-itsh-cockpit.conf` setzt `PubkeyAuthentication` und `PasswordAuthentication` ausdrücklich, bei gesperrtem Passwort zusätzlich `KbdInteractiveAuthentication no` und `ChallengeResponseAuthentication no`. So kann das Cockpit auch eine von Hand gesetzte Sperre aufheben. Nach dem Schreiben muss `sshd -T` den Soll-Zustand zeigen, sonst wird zurückgenommen.
### Geändert
- Ersetzt den Schalter „Nur Schlüssel-Anmeldung“ aus 0.51.0: „Nur Schlüssel-Anmeldung: an“, „Ausschalten“, „Vom Cockpit verwalten lassen“, der Erklärsatz und die Statuszeile „Schlüssel-Anmeldung: … Passwort-Anmeldung: …“ entfallen (die Abzeichen zeigen den Zustand, Gründe stehen im Tooltip und in einer kurzen Hinweiszeile). Der Endpunkt `/systems/{id}/ssh-keyonly` entfällt; die Jobs `sshd_keyonly_on/off` bleiben gültig und werden vom Agent weiter ausgeführt, aber nicht mehr angelegt. Ein alter Key-only-Block wird beim ersten Umschalten durch den neuen Block ersetzt.
- Die Warnung beim Entfernen/Widerrufen des letzten aktiven SSH-Zugangs gilt jetzt, wenn die Passwort-Anmeldung laut letztem Bericht gesperrt ist – unabhängig davon, wer sie gesperrt hat – und auch, solange ein Job zum Sperren der Passwort-Anmeldung offen ist oder auf das nächste Inventar wartet (Systemseite und Profil).
- Review vor dem Release: Rückfrage und Tooltip beim Sperren der Schlüssel-Anmeldung warnen, dass die SSH-Zugänge des Cockpits dann nicht mehr funktionieren und Konten ohne Passwort nicht mehr hineinkommen; der Agent prüft die Wirkung auch, wenn nichts zu ändern ist; das alte Drop-in wird mitgesichert (`sshd_config.itsh-<Zeit>.dropin`); „wird umgeschaltet“ berücksichtigt jeden offenen SSH-Konfigurations-Job.

## [0.51.1] – 2026-11-27
### Behoben
- **Inventar nach einem Neustart:** Der Agent merkt sich die Startzeit des Systems, die er zuletzt gemeldet hat (`bootTime` in `state.json`, Linux `/proc/stat` btime, Windows `GetTickCount64`). Beim Start des Dienstes meldet er das Inventar kurz danach, wenn sich die Startzeit geändert hat (Abweichungen bis 2 Minuten gelten als gleich). Bisher zeigte das Cockpit nach einem Neustart-Job „Neustart ausstehend“ und den alten letzten Start, bis zum nächsten täglichen Lauf, weil das Inventar nach dem Job noch vor dem Neustart gemeldet wurde. Der erste Start nach dem Update merkt sich den Wert nur. Agent-Paket 0.51.1 nötig.

## [0.51.0] – 2026-11-27
### Hinzugefügt
- **SSH-Konfiguration prüfen:** Läuft ein SSH-Dienst, ermittelt der Agent beim Inventar die wirksame Konfiguration mit `sshd -T -f <sshd_config>` (Linux `sshd` aus den Systemverzeichnissen, Windows `System32\OpenSSH\sshd.exe` bzw. `Programme\OpenSSH\sshd.exe`) und meldet `pubkey`, `password`, `kbdInteractive`, `keyFileOK` (Windows: Block `Match Group administrators` mit `administrators_authorized_keys`; Linux: `AuthorizedKeysFile` enthält `.ssh/authorized_keys`), `keyOnlyManaged` und bei einem Fehler `configError` – keine Dateiinhalte, keine Benutzer. Linux ohne `-C`: Match-Blöcke werden nicht ausgewertet (siehe `doku\SSH-Konfiguration.md`).
- **Fernwartungs-Kachel:** Unter den SSH-Zeilen „Schlüssel-Anmeldung: möglich / nicht möglich (Grund)“ und „Passwort-Anmeldung: erlaubt / gesperrt“ (gesperrt nur, wenn Passwort **und** Keyboard-Interactive aus sind). „Meinen Schlüssel eintragen“ warnt, wenn die Schlüssel-Anmeldung laut letzter Prüfung nicht möglich ist (Eintragen bleibt möglich).
- **Schalter „Nur Schlüssel-Anmeldung“:** Jobs `sshd_keyonly_on` / `sshd_keyonly_off` (Recht „Jobs“ + 2FA, Agent ab 0.51.0; Audit `sshd_keyonly_on/off`, Verlauf). Einschalten nur mit mindestens einem aktiven SSH-Zugang und möglicher Schlüssel-Anmeldung. Der Agent prüft vorher selbst, sichert `sshd_config` als `sshd_config.itsh-<Zeitstempel>` (Rechte wie das Original), setzt einen Cockpit-Block mit `PasswordAuthentication`, `KbdInteractiveAuthentication`, `ChallengeResponseAuthentication no` an den Anfang der `sshd_config` (Linux mit `Include sshd_config.d/*.conf`: stattdessen `sshd_config.d/00-itsh-cockpit.conf`), prüft mit `sshd -T` Syntax und Wirkung, lädt den Dienst neu (Linux `systemctl reload ssh|sshd`, Windows Dienst `sshd` neu starten) und stellt bei jedem Fehler die vorherigen Dateien wieder her. Ausschalten entfernt Block bzw. Datei wieder. Atomares Schreiben, keine Links/Junctions; Fehlergründe übersetzt in der Jobliste.
- Migration 0057 (Jobtypen). Agent-Paket 0.51.0 nötig.
- Review vor dem Release: `/run/sshd` wird bei Bedarf angelegt (Ubuntu mit `ssh.socket`); Windows startet `sshd` nur neu, wenn er lief; Warnung beim Entfernen/Widerrufen des letzten Zugangs eines Nur-Schlüssel-Systems; höchstens 5 Sicherungen; Ausschalten auch ohne laufenden SSH-Dienst; SELinux-Label per `restorecon`, falls vorhanden.

## [0.50.0] – 2026-11-27
### Hinzugefügt
- **Neustart über das Cockpit:** Auf der Systemseite (oberste Kachel) „Neu starten“ in X Minuten (vorbelegt mit der Standard-Vorwarnzeit), zu einem Zeitpunkt (höchstens 7 Tage voraus, deutsche Zeit) oder sofort (= in 1 Minute, damit der Agent sein Ergebnis noch meldet). Solange ein Neustart geplant ist, zeigt die Kachel den Zeitpunkt und „Neustart abbrechen“. Recht „Jobs“ (Globaler Admin, Kunden-Admin für eigene Kunden) und 2FA; Audit `reboot_requested`/`reboot_cancelled`, Verlauf „Neustart angefordert“/„Abbruch des Neustarts angefordert“. Auf dem Cockpit-Server zusätzlich die Warnung, dass auch das Cockpit neu startet. Ein Neustart wird weiterhin nie automatisch ausgelöst.
- **Jobs `reboot` / `reboot_cancel`:** Windows `shutdown.exe /r /t <Sekunden> /c <Nachricht> /d p:0:0` (Abbruch `/a`) aus dem Systemordner, Linux `shutdown -r +<Minuten> <Nachricht>` (Abbruch `-c`) aus den Systemverzeichnissen; Argumente einzeln, keine Shell. Erfolg = `shutdown` hat den Befehl angenommen. Angemeldete Benutzer erhalten eine feste Nachricht mit dem Namen der Eigenen Firma und der Zeit („Wartung durch …: Neustart in 5 Minuten. Bitte speichern Sie Ihre Arbeit.“); Zentrale und Agent prüfen die Zeichen (Buchstaben, Ziffern, Leerzeichen, `.:,-()`, höchstens 200). Ein Neustart „in X Minuten“ verfällt, wenn der Agent ihn nicht binnen einer Stunde abholt; einer zu einem Zeitpunkt zu diesem Zeitpunkt.
- **Einstellungen → Updates:** „Standard-Vorwarnzeit für Neustarts“ (1–1440 Minuten, Standard 5; Globaler Admin).
- **Systemübersicht:** Eigener, blinkender Status-Punkt (Türkis) für „Neustart ausstehend“ mit Legende; ohne Blinken bei „Bewegung reduzieren“. Kritische Updates (rot) haben Vorrang.
- Migration 0056 (Jobtypen). Agent-Paket 0.50.0 nötig; ältere Agents zeigen einen Hinweis statt des Knopfs (Design: `doku\Neustart.md`).
- Ein Abbruch ohne geplanten Neustart (Windows 1116) gilt als Erfolg; Linux-Systeme ohne systemd lehnen Neustart-Jobs mit Hinweis ab. Windows schließt beim geplanten Neustart offene Programme (`/t` > 0 wirkt wie `/f`); einen bereits geplanten Neustart lehnt Windows ab (1190), Linux ersetzt ihn.

### Behoben
- **Test `TestSSHKeys`:** räumt seine Daten jetzt wirklich auf (die Datenbankverbindung wurde vor dem Aufräumen geschlossen) und vergleicht ausstehende Entfernungen mit dem Startwert, damit er auf der gemeinsamen Testdatenbank stabil läuft.

## [0.49.0] – 2026-11-27
### Hinzugefügt
- **SSH-Schlüssel je Benutzer:** Im Profil lässt sich ein Schlüsselpaar (ed25519) erzeugen oder ein eigener öffentlicher Schlüssel hinterlegen, ersetzen und widerrufen (Globale Admins, Kunden-Admins, 2FA). Der private Schlüssel liegt nur verschlüsselt im Cockpit und wird nie angezeigt.
- **Jobs `ssh_key_add` / `ssh_key_remove`:** Der Agent trägt den Schlüssel in `authorized_keys` ein bzw. entfernt ihn (Linux je Benutzer, `root` abgelehnt; Windows nur Administratoren über `administrators_authorized_keys`). Schlüsselwechsel trägt den neuen Schlüssel überall ein, wo der alte war.
- **SSH-Zugänge am System:** Liste „SSH-Zugänge“ in der Fernwartungs-Kachel mit Eintragen/Entfernen und Job-Ergebnis.
- **Glocke und Automatik:** Ausstehende Entfernungen erscheinen in der Glocke (auch nach fehlgeschlagenem oder verfallenem Job, bei entfernten Systemen mit „von Hand entfernen“ und Bestätigung). Fehlgeschlagene Entfernungen werden bei jeder Meldung erneut angelegt (1 Stunde Abstand). Die tägliche Aufräumung widerruft Schlüssel deaktivierter und ausgeschiedener Benutzer sowie von Rollen ohne Schlüsselrecht; Rollen-, Kunden- und Systemzuordnungswechsel entfernen die Zugänge; die Agent-Entfernung wartet auf die Schlüssel-Entfernung (die Entfernen-Jobs werden vor der Entfernungsanforderung angelegt). Die tägliche Aufräumung holt außerdem das Entfernen offener Zugänge widerrufener Schlüssel nach; hängende Eintragungen (Job verfallen, abgebrochen oder gelöscht) gelten als fehlgeschlagen und lassen sich neu eintragen.
- **Kunden und SSH-Zugänge:** Wird ein Kunde inaktiv, werden die SSH-Zugänge auf seinen Systemen entfernt. Solange dort noch SSH-Zugänge offen sind, wird der Kunde weder von Hand noch automatisch gelöscht.
- Migrationen 0054 (Jobtypen) und 0055 (`ssh_keys`, `ssh_key_grants`). Agent-Paket 0.49.0 nötig (Design: `doku\Web-Terminal.md`).

### Behoben
- **Windows-Update-Jobs:** Der `param()`-Block stand nicht am Anfang des Skripts, daher wurden `-Mode`, `-UpdateId` und `-Exclude` seit 0.9.0 nie übergeben. Jetzt steht `param()` zuerst.

## [0.48.0] – 2026-11-27
### Hinzugefügt
- **SSH in der Fernwartung:** Der Agent meldet einen laufenden SSH-Server (Linux `ssh`/`sshd`, Windows-Dienst „OpenSSH SSH Server“) samt Port aus der `sshd_config`. Die Fernwartungs-Kachel zeigt je privater IPv4 eine Zeile „SSH“ mit Befehl, „Kopieren“ und „Verbinden“ (`ssh://`). Neues Feld „SSH-Benutzer“ je System (Recht Asset-Daten bearbeiten, Audit `system_ssh_user`, Verlauf). Migration 0053.
- **Linux ohne Desktop:** Der Agent meldet, ob ein X-Server/Xwayland installiert ist. Fehlt er und ist RustDesk installiert, zeigt die Kachel „kein Bildschirm – bitte SSH nutzen“ (Anlass: DMZ253, RustDesk-Server stürzt mit „cannot open display“ ab).
- **Einstellungen → RustDesk:** Schalter „Linux ohne Desktop zulassen“ (`allow-linux-headless`, Standard an), wird bei RustDesk-Installationen unter Linux gesetzt.
- Agent-Paket 0.48.0 nötig (Design: `doku\SSH-Fernwartung.md`).

## [0.47.0] – 2026-11-27
### Hinzugefügt
- **Updates als ok vermerken:** Jede Update-Zeile der Systemseite hat „als ok vermerken“ (Recht „Jobs“ + 2FA-Code, Audit `update_marked`/`update_unmarked`, Verlaufseintrag). Vermerkte Updates gelten je System und Paket für jede Version, zählen nicht mehr als offen (Update-Zahl, Systemliste, Kundenübersicht, kritisch) und stehen im zugeklappten Abschnitt „Vermerkte Updates (n)“ mit „Vermerk aufheben“. „Alle installieren“ überspringt sie bei Chocolatey (`choco upgrade all --except=…`) und Windows Update (`-Exclude`); unter Linux gibt es dafür einen Hinweis. Agents unter 0.47.0 bekommen kein „Alle installieren“, solange Vermerke bestehen. Migration 0051.
- **Benutzerkontext erkennen:** Der Agent meldet je Programm, ob es für den Rechner oder nur in einem Benutzerprofil installiert ist (ohne Benutzernamen), und zusätzlich die Store-Pakete (MSIX/Appx) aller Benutzer. Chocolatey-Updates solcher Programme tragen das Abzeichen „Benutzerkontext (vermutet)“; scheitert ein Job mit `0x80073CF9`, merkt sich das Cockpit das („Benutzerkontext (bestätigt)“) und die Jobzeile empfiehlt, als Benutzer zu aktualisieren oder das Update als ok zu vermerken. Anlass: oh-my-posh auf NBMH.
- **Paketlisten aktualisieren (Linux):** Neuer Job `pkg_refresh` (`apt-get update`, `dnf`/`yum makecache`, `zypper refresh`), Knopf rechts oben in der Update-Kachel – auch ohne offene Updates – und in der Zeile „Paketlisten veraltet“. Danach schickt der Agent ein neues Inventar.
- Agent-Paket 0.47.0 nötig (Design: `doku\Update-Vermerk-Benutzerkontext.md`).

## [0.46.0] – 2026-11-26
### Entfernt
- **winget:** Der Agent fragt winget nicht mehr ab, das Cockpit zeigt und zählt keine winget-Updates mehr (auch nicht von älteren Agents). winget läuft nur im Benutzerkontext, seine Updates ließen sich über das Cockpit nie installieren und waren irreführend. Software-Updates laufen nur über Chocolatey bzw. den Linux-Paketmanager und Windows Update. Agent-Paket 0.46.0.

## [0.45.0] – 2026-11-26
### Hinzugefügt
- **Inventar jetzt anfordern:** In der obersten Kachel der Systemseite (Knopf „Inventar jetzt anfordern“, danach „Inventar angefordert (seit HH:MM)“) und auf der Systemübersicht („Inventar für alle anfordern“) lässt sich das Inventar sofort abholen, statt bis zum nächsten Tageslauf zu warten (Recht „Jobs“, ohne 2FA, Audit `inventory_requested`, kein Verlaufseintrag). Der Agent (ab 0.45.0) startet auf den Hinweis `reportInventory` der nächsten Abfrage sofort einen Inventarlauf; der Agent schickt den Zeitstempel der Anforderung mit dem neuen Inventar zurück, und nur dieses Inventar erledigt die Anforderung (unabhängig von der Uhrzeit des Rechners). Entfernte Systeme und Agents unter 0.45.0 werden übersprungen. Migration 0050, Agent-Paket 0.45.0 nötig (Design: `doku\Inventar-Anfordern.md`).

## [0.44.0] – 2026-11-26
### Hinzugefügt
- **Jobs abbrechen:** Auf der Systemseite (Kachel „Jobs“) lässt sich jeder wartende oder laufende Job mit „Abbrechen“ beenden, dazu „Alle offenen Jobs abbrechen“ je System (Recht „Jobs“, 2FA, Bestätigungsdialog mit Hinweis auf eine mögliche halbe Installation, Audit und Verlauf). Wartende Jobs werden sofort abgebrochen und nie ausgeliefert; bei laufenden zeigt der Job „Abbruch angefordert“, der Agent (ab 0.44.0) fragt während der Abarbeitung alle 30 Sekunden nach, beendet den Prozessbaum und überspringt noch nicht gestartete Jobs der Lieferung. Neuer Status „abgebrochen“; ältere Agents zeigen bei laufenden Jobs den Hinweis „Abbrechen erst ab Agent 0.44.0“. Migration 0049, Agent-Paket 0.44.0 nötig (Design: `doku\Job-Abbrechen.md`).

### Behoben
- **Glocke „Speicher des Cockpit-Servers knapp“:** nennt jetzt die geltende Schwelle (eigene des Systems, sonst die zentrale Einstellung) statt fest „weniger als 20 %“.
- **Job-Ergebnis wurde nicht gespeichert:** Meldete ein Windows-Installer den Rückgabewert −1 als 4294967295 (z. B. oh-my-posh), lehnte die Datenbank das Ergebnis ab und der Job blieb auf „läuft“. Solche Werte werden jetzt in ihre vorzeichenbehaftete Form umgerechnet.
- **Sicherheit:** Das Zugriffs-Log des Reverse-Proxys schreibt Agent-Schlüssel, Cookies und Authorization-Kopfzeilen nicht mehr mit.

## [0.43.0] – 2026-11-26
### Hinzugefügt
- **Chocolatey über das Cockpit installieren:** Neuer Job „Chocolatey installieren“ für Windows-Systeme ohne Chocolatey, einzeln auf der Systemseite oder für alle betroffenen Windows-Systeme eines Kunden (Recht „Jobs“, 2FA, Audit und Verlauf; Systeme mit ausgeschalteten Jobs, deaktivierte und entfernte Systeme werden übersprungen). Der Agent lädt das offizielle Paket von chocolatey.org (nur HTTPS, Weiterleitungen nur auf chocolatey.org, höchstens 50 MB), entpackt es geschützt gegen Pfade außerhalb des Ziels, prüft danach jede entpackte Datei („alles verboten außer erlaubt“: Authenticode mit Herausgeber „Chocolatey Software, Inc.“, reine Datendateien, Lader-Dateien und das unsignierte 7-Zip/RefreshEnv.cmd nur an festen Pfaden) und führt das Install-Skript erst danach aus (Frist 15 Minuten); danach meldet er sofort ein neues Inventar. Ist Chocolatey schon da, endet der Job mit „already installed“. Migration 0048, Agent-Paket 0.43.0 nötig (Design: `doku\Chocolatey-Installation.md`).

## [0.42.1] – 2026-11-26
### Behoben
- **Agent hing bei Software-Jobs:** Ließ ein von Chocolatey gestartetes Setup (z. B. AutoHotkey) die Ausgabe offen, wartete der Agent endlos, meldete keine Ergebnisse mehr und holte keine weiteren Jobs. Jetzt wartet er nach Programmende höchstens 30 Sekunden auf die Ausgabe. Bei Zeitüberschreitung beendet er alle vom Job gestarteten Programme (Windows: Job-Objekt, Linux: Prozessgruppe). Höchstdauer je Job 30 statt 60 Minuten.

## [0.41.0] – 2026-11-25
### Hinzugefügt
- **RustDesk über das Cockpit installieren:** Grundkonfiguration unter Einstellungen → RustDesk (Version, Passwortverfahren, Einmalpasswort, Sitzung bestätigen, Fernkonfiguration, Update-Prüfung, eigener Server/Key/Config-String; nur Globaler Admin, 2FA). Jobs „RustDesk installieren/aktualisieren“ und „Neues RustDesk-Passwort“ auf der Systemseite und beim Kunden (2FA). Der Agent lädt RustDesk direkt aus dem offiziellen GitHub-Repo, prüft unter Windows die Signatur (Herausgeber PURSLANE) und installiert unter Windows bzw. Debian/Ubuntu; danach wird die installierte Version geprüft. Ein Zufallspasswort je Rechner geht versiegelt ans Cockpit, wird verschlüsselt gespeichert und nur mit 2FA angezeigt (Audit). Migrationen 0044 und 0045. Agent-Paket 0.41.0. Design: `doku/Fernwartung.md`.
### Sicherheit
- Restrisiko (bewusst): Beim Setzen steht das Passwort kurz in der Prozessliste (RustDesk-CLI bietet keinen anderen Weg); für Debian-Pakete gibt es keine Signatur von RustDesk.

## [0.40.0] – 2026-11-25
### Hinzugefügt
- **Fernwartungs-IDs:** Der Agent meldet die Rechner-ID der installierten Fernwartungs-Tools – RustDesk, TeamViewer und AnyDesk (Windows und Linux), Supremo (Windows). Nur die ID wird gelesen, keine Passwörter oder Sitzungs-IDs; ohne Installation (QuickSupport) wird nichts gemeldet. Anzeige auf der Systemseite und beim Asset (Abschnitt „Fernwartung“ mit Kopieren und Verbinden) sowie als Spalte in Systeme, Asset-Center und Assets beim Kunden; Änderungen stehen im Verlauf des Systems. Migration 0043. Design: `doku/Fernwartung.md`.
- **Agent-Paket 0.40.0** – enthält auch das Icon für `ITSH-Freischaltung.exe` (aus 0.38.0).

## [0.37.0] – 2026-11-25
### Hinzugefügt
- **Freischalt-Schlüssel je Firma:** Jedes Cockpit hat einen eigenen Ed25519-Schlüssel (Hauptschlüssel plus Ersatzschlüssel) für die Onboarding-Freischaltung, getrennt vom Release-Schlüssel von ITSH. Jeder Globale Admin kann damit Tickets signieren, auch in anderen Firmen. Konzept: `doku\Freischalt-Schluessel.md`.
- **Werkzeug „ITSH Freischaltung“** (`ITSH-Freischaltung.exe` für Windows, `ITSH-Freischaltung_linux_amd64` für Linux; Konsolen-Assistent und Kommandozeile): Schlüssel erzeugen (passwortgeschützte Dateien, scrypt + AES-256-GCM), Onboarding-Tickets signieren, Schlüsselwechsel signieren. Zeigt vor jedem Signieren den Inhalt und fragt nach Bestätigung. Die Dateien stehen im signierten Manifest (Feld `tools`, mit SHA-256) und als eigene GitHub-Release-Assets.
- **Seite Einstellungen → Freischalt-Schlüssel** (Globaler Admin, 2FA): öffentliche Schlüssel hinterlegen (korrigierbar, bis ein Agent einen davon gemeldet hat – diese Sperre bleibt dauerhaft bis zum Notfall-Zurücksetzen), Vorlage und Upload für den Schlüsselwechsel, Gerätetabelle (aktuell, Wechsel ausstehend, andere Schlüssel – Block mit `-replace-unlock-keys` einfügen, kein Schlüssel, Agent zu alt; Filter nach Kunde), Hinweise in der Glocke (nicht eingerichtet, Geräte mit ausstehendem Wechsel, „Schlüssel wechseln, bevor <Name> ausscheidet“).
- **Schlüsselwechsel:** Wechsel-Nachricht (Kontext `itsh-cockpit-unlockkeys-v1`, steigende Nummer, ohne Ablauf), mit einem aktuellen Schlüssel signiert; das Cockpit verteilt sie der Reihe nach an alle Agents (Rückmeldung `POST /api/v1/agents/unlock-keys/ack`, Verlaufseinträge). Geräte, die Wechsel verpasst haben, holen sie nacheinander ab.
- **Hilfe-Seite „Freischaltung signieren“** (Avatar-Menü und Link in der Onboarding-Kachel): Schritt-für-Schritt-Anleitung, Download-Links und SHA-256 der Werkzeug-Dateien aus dem signaturgeprüften Manifest.
- **Notfall-Befehl** auf dem Server-Host für den Fall, dass beide Schlüssel verloren sind: `server unlock-keys reset` (mit Abfrage, Audit-Eintrag zuerst, nie über die Weboberfläche).
- Agent: Parameter `install -unlock-key` (max. 2), `-unlock-keys-seq` und `-replace-unlock-keys`; `config.json` kennt `unlockKeys` und `unlockKeysSeq`; die Abfrage meldet Stand und Schlüssel-IDs.

### Geändert
- **Tickets gelten nur noch mit dem Firmenschlüssel**, nicht mehr mit dem Release-Schlüssel – Agent und Cockpit prüfen ausschließlich gegen die Firmenschlüssel (kein Übergang). Ohne gespeicherten Schlüssel lehnt der Agent Tickets ab („no unlock key - paste the install block again“); das Cockpit bietet solche Geräte im Onboarding nicht an. `release unlock-ticket` entfällt; `01_build\New-UnlockTicket.ps1` ruft das Werkzeug auf und braucht `-Key` (kein `ITSH_SIGNING_KEY` mehr).
- **Installationsblöcke enthalten die Schlüssel** (`-unlock-key … -unlock-keys-seq …`), sobald im Cockpit welche hinterlegt sind und die freigegebene Agent-Version mindestens 0.37.0 ist. Ein erneut eingefügter Block ersetzt vorhandene Schlüssel am Gerät nicht (nur mit `-replace-unlock-keys`).
- Onboarding setzt Agent ab 0.37.0 mit aktuellem Schlüsselstand voraus.
- `Build.ps1` baut das Werkzeug mit, `Build.ps1 -Release` und `Publish-AgentRelease.ps1` nehmen es ins Manifest bzw. als Release-Assets auf.

### Hinweise
- **Agent-Paket 0.37.0 nötig.** Bestehende Agents haben keinen Firmenschlüssel: Installationsblock auf jedem Gerät, das freigeschaltet werden soll, **erneut einfügen** (Registrierung bleibt).
- Einrichtung nach dem Update: Werkzeug holen, Schlüssel erzeugen, öffentliche Texte unter Einstellungen → Freischalt-Schlüssel eintragen (`doku\Betrieb.md`, Abschnitt 7c).
- Migration `0041` (Freischalt-Schlüssel, Wechsel-Nachrichten, Stand je Agent).

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

