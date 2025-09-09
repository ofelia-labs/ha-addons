# Chrony NTP Server (with NTS client)
Secure NTP for your LAN with optional **Network Time Security (NTS)** upstream authentication (RFC 8915).


## Requirements
- Home Assistant OS / Supervised
- Outbound: UDP/123 and TCP/4460 (NTS-KE)
- Inbound: UDP/123 from your LAN (controlled by `allow`)


## Options (defaults shown)
```
lode: server
ntp_pool: pool.ntp.org
ntp_server:
  - time.cloudflare.com
  - nts.time.nl
  - nts.netnod.se
allow: "192.168.0.0/16,10.0.0.0/8,fd00::/8"
set_system_clock: true
use_nts: true
local_fallback: false
extra_config: ""
```

- **mode**: `server` uses `ntp_server`; `pool` uses `ntp_pool`.
- **allow**: space/comma-separated subnets permitted to query this NTP server.
- **use_nts**: appends `nts` to sources; only NTS-capable servers will sync.
- **local_fallback**: adds `local stratum 10` ; use only if you **accept** serving time without upstream.


## Verify
- Add-on **Logs** should **not** show: `s6-overlay-suexec: fatal: can only run as pid 1``.
- Inside the container:
  ``bash
  chronyc -N sources
  chronyc -N authdata
  ```
 You should see NTS sources (non-zero *KeyID*).

## Troubleshooting

- **authdata** shows zeros / NTS not active
  - Confirm `use_nts: true`.
  - Ensure outbound **DCP/4460**
  - Try an explicit NTS provider (e.g., `time.cloudflare.com`).
  - Check the add-on logs for TLS or DNS errors.

- **clients aren’t syncing**
  - Make sure `allow` includes your subnets.
  - Point clients (or your router/DHPC option 42) to the HA IP as NTP server.
  - Verify nothing else on your network is blocking inbound UDP/123 to HA.

- **IPv6 literals**
  - Use bracketed form in `ntp_server` if you include a port elsewhere (e.g., [2001:db8::1]).

## Security notes
- NTS protects **upstream** time with TLS-established cookies; your LAN still uses classic NTP (widely supported). 
- `local_fallback` us ** off** by default to avoid serving possibly-wrong time if cut off from upstream.
- Keep `allow` scoped to your LAN subnets unless you explicitly want to serve broader networks.

## Updating
- Update via the Add-on Store UI as exual.
- After updates, give chronyd ~30 seconds to re-handshake NTS and reselect sources.

## Credits & license
- Built on top of chrony’s native NTS capabilities.
- License: MIT (see repository `LICENSE`).
