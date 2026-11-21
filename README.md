# ITSH Cockpit – Agent-Pakete

Signierte Release-Pakete des **ITSH Cockpit Agents** (Windows und Linux, amd64) von IT SERVICES HAEFELE.
Dieses Repository enthält **nur** die fertigen, signierten Pakete und den Änderungsverlauf – keinen Quellcode.

## Wofür
Jedes ITSH Cockpit holt neue Agent-Versionen von hier ab (`releases/latest/download/itsh-cockpit-agents.zip`)
oder ein Administrator lädt das Paket im Cockpit unter **System-Center → Agent-Versionen** hoch.
Verteilt wird eine Version erst, nachdem ein Globaler Admin die Änderungen geprüft und sie freigegeben hat.

## Inhalt eines Pakets `itsh-cockpit-agents-X.Y.Z.zip`
| Datei | Inhalt |
|---|---|
| `manifest.json` | Version, Mindestversion des Cockpits, Änderungen, SHA-256 je Datei |
| `manifest.sig` | Ed25519-Signatur des Manifests (Schlüssel-ID und Signatur) |
| `ITSH_Cockpit-agent_windows_amd64.exe` | Agent für Windows |
| `ITSH_Cockpit-agent_linux_amd64` | Agent für Linux |

## Sicherheit
- Das Manifest ist mit einem Ed25519-Schlüssel signiert, der nur in der Entwicklung liegt.
  Cockpit und Agent prüfen Signatur und Prüfsummen mit dem eingebauten öffentlichen Schlüssel
  (Schlüssel-ID `2dc0d3f27c347ae2`); ungültige Pakete werden abgelehnt.
- Ein Agent installiert nur **neuere** Versionen und fällt auf die vorherige zurück, wenn die neue den Server nicht erreicht.

## Änderungen
Alle Änderungen seit der ersten Version stehen in [CHANGELOG.md](CHANGELOG.md).