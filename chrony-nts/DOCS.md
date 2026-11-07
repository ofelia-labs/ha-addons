# Chrony Secure Time

Chrony with Network Time Security (NTS) upstreams for Home Assistant. Two
operating modes are provided.

## Modes

| Mode | Description |
|------|-------------|
| `secure_client` | Authenticates upstream time with NTS and adjusts the host clock. No NTP is served to the LAN. |
| `secure_gateway` | Authenticates upstream time with NTS and serves plain NTP on UDP/123 to configured subnets. |

`set_system_clock` controls whether the host clock is disciplined. When set to
`false`, chronyd runs with `-x` and only provides time to clients.

## Options

```yaml
mode: secure_client | secure_gateway
nts_upstreams:       # explicit NTS-capable servers
  - time.cloudflare.com
  - nts.time.nl
  - nts.netnod.se
set_system_clock: true
allow_ipv4: []       # required in secure_gateway
allow_ipv6: []       # required in secure_gateway
bootstrap_strategy: persistent_then_initstepslew
extra_config: ""
```

### Bootstrap strategies

Used when the machine has no reliable real-time clock:

* `persistent_then_initstepslew` *(default)* – step forward to the last saved
  epoch (max +48h) then run an unauthenticated `initstepslew` against the public
  pool.
* `persistent_only` – only use the saved epoch.
* `initstepslew` – skip persistent time; rely solely on `initstepslew`.
* `oneshot` – before daemonizing, run `chronyd -q "server 0.pool.ntp.org iburst"`
  if the year is < 2025.
* `none` – no special handling.

Time is periodically written to `/data/chrony/last-epoch` and restored on the
next boot when required.

## Verification

Inside the running container:

```bash
chronyc -N authdata       # should report Type=NTS, cookies non-zero
chronyc -N -n sources -v  # show selected source
```

Client mode should not listen on UDP/123:

```bash
ss -uLpna | grep ':123'   # no output
```

Gateway mode should listen on UDP/123 and respond only to configured subnets.

## Security

* Chronyd runs as user `chrony` with `cmdport 0`.
* Writes are confined to `/data/chrony/**`; reads from `/etc/chrony/**` and
  `/etc/ssl/**` only.
* Upstreams are explicit NTS servers; NTS cookies persist across restarts.

## Troubleshooting

* **NTS handshake fails** – ensure the system clock is reasonable and that
  outbound TCP/4460 and UDP/123 are permitted.
* **Clients not syncing** – confirm the `allow_ipv4`/`allow_ipv6` lists and that
  your network permits inbound UDP/123.

---

Licensed under the MIT License.

