# CasaOrfeo: hass --script auth change_password braucht Container-Neustart, sonst "Ungültiges Passwort" trotz korrektem neuem Passwort

CasaOrfeo, 2026-09-20. Ergänzung zu [[fa70310d]] (hass --script auth braucht -c /config).

**Zweiter Fallstrick beim selben Passwort-Reset:** Nach `hass --script auth -c /config change_password raffael '<neues-passwort>'` (Datei-mtime verifiziert geändert) blieb der Login im WebUI trotzdem bei "Ungültiger Benutzername oder Passwort" hängen.

**Ursache:** HA Core lädt die lokalen Auth-Credentials beim Start einmal in den Arbeitsspeicher. Der `hass --script auth`-Aufruf manipuliert nur die Datei auf der Platte über eine eigene, separate Prozessinstanz — der laufende HA-Prozess merkt das nicht automatisch und prüft Logins weiterhin gegen den alten Hash im RAM. Bestätigt über den Zeitvergleich: Container-Start 10:33 Uhr, Datei-Änderung 13:04 Uhr — fast 2,5h Differenz.

**Fix:** `docker restart homeassistant` nach dem `change_password`-Aufruf. Danach lädt HA den neuen Hash aus `.storage/auth_provider.homeassistant` und der Login funktioniert.

**Gesamter Ablauf für künftige Passwort-Resets:**
1. `hass --script auth -c /config change_password <user> '<neues-pw>'` im Container ausführen.
2. `docker restart homeassistant` (zwingend, sonst wirkt der neue Hash nicht).
3. Login im WebUI testen.
