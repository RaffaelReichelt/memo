---
created: '2026-09-28T06:19:05.405+02:00'
extra:
  entities:
  - Problem CasaOrfeo
  - Mac Mini
  - Tailscales MagicDNS
  - Domain UND
  - Allowed IPs
  - Erneuter Live
  - Die Speedport
  - Internet
  - Casa
  - Wire
  - Wannsee
  - Internetverbindung
  - Mini
  - Nutzers
  - Magic
  - Tunnel
  - Resolver
  - Domain
  - Ursache
  - Default
  owner_principal: 4c3e0ce04a3b
  trust_tier: agent_inferred
  visibility: owner
  write_policy:
    actor_id: memo
    allowed: true
    conflicts: []
    override: false
    policy_version: memo.write_policy.v1
    reason: allowed
    trust_tier: agent_inferred
    visibility: owner
id: de2c04a0995041cd8f710ed2f025b8da
normalized_hash: edbff5f8863f66cd
tags:
- casaorfeo
- homeassistant
- wireguard
- tailscale
- vpn
- networking
- project:casaorfeo
title: 'CasaOrfeo: WireGuard "MacMini" Full-Tunnel-Regression brach Internet, nicht
  DNS-Problem'
type: bug
updated: '2026-09-28T06:19:05.405+02:00'
valid_at: '2026-09-28T06:19:05.405+02:00'
verification_state: unverified
---

CasaOrfeo, 2026-09-28. Nutzer meldete: sobald der WireGuard-Tunnel "MacMini" (nativer macOS WireGuard.app, verbindet zum Wannsee-Speedport-Router) geöffnet ist, funktioniert die Internetverbindung auf dem Mac Mini nicht mehr. Vermutung des Nutzers: Tailscales MagicDNS (100.100.100.100) würde nicht mehr greifen.

**Live-Test widerlegte die DNS-Theorie eindeutig:** Bei geöffnetem Tunnel zeigte `scutil --dns` weiterhin 100.100.100.100 als primären unscoped Resolver, `dscacheutil`-Lookups für eine öffentliche Domain UND für *.tail5a2ccd.ts.net liefen beide sauber durch.

**Tatsächliche Ursache: Full-Tunnel-Regression.** `route -n get default` zeigte `utun17` (WireGuard) statt `en0` als Default-Interface, dazu massenhaft einzelne /32-Host-Routen über utun17 in `netstat -rn -f inet` - klares Full-Tunnel-Muster (AllowedIPs faktisch wieder 0.0.0.0/0), obwohl CLAUDE.md dokumentierte, dass der Tunnel bereits auf Split-Tunnel (`192.168.1.0/24, 10.200.200.0/24`) umgestellt war. Live-Beweis: `ping 1.1.1.1` bei offenem Tunnel -> 66% Paketverlust (Speedport-Router ist kein Internet-Gateway für einen entfernten Full-Tunnel-Client), nach Schließen sofort wieder 0%.

**Fix:** WireGuard-App -> Tunnel "MacMini" -> Bearbeiten -> Allowed IPs zurück auf `192.168.1.0/24, 10.200.200.0/24`. Erneuter Live-Test danach bestätigt: Default-Route bleibt en0, ping 1.1.1.1 0% Verlust, ping 192.168.1.1 (Wannsee-LAN) funktioniert weiterhin über den Tunnel, HA über Tailscale weiterhin erreichbar.

**Merke:** Bei "Internet bricht ab, sobald VPN X offen ist" zuerst `route -n get default`/`netstat -rn -f inet` prüfen (Full-Tunnel-Regression?), bevor an DNS-Resolvern gesucht wird. Die Speedport-generierte Config kann sich offenbar bei einem Reimport/Update in der WireGuard-App unbemerkt auf die Full-Tunnel-Standardeinstellung zurücksetzen - das ist mindestens das zweite Mal (siehe ursprüngliche Split-Tunnel-Doku vom 2026-08-05), dass das passiert.

Dokumentiert in CLAUDE.md unter "VPN-Zugang zum Wannsee-Router".