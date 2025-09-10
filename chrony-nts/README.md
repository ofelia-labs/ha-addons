# Chrony Secure Time

Secure time for Home Assistant using [chrony](https://chrony.tuxfamily.org) and
Network Time Security (NTS) upstreams. Runs in two simple modes:

* **Secure Client (NTS)** – authenticates upstream time and sets the host
  clock. Does not listen on UDP/123.
* **Secure Gateway (NTS→NTP)** – authenticates upstream time with NTS and
  serves plain NTP to your LAN. The host clock is set by default but can be
  disabled.

Chronyd runs as the dedicated `chrony` user, stores its drift and NTS cookies in
`/data/chrony`, and is confined by AppArmor for least privilege.

## Configuration

```yaml
mode: secure_client | secure_gateway
nts_upstreams:
  - time.cloudflare.com
  - nts.time.nl
  - nts.netnod.se
set_system_clock: true
allow_ipv4: []         # required in secure_gateway
allow_ipv6: []         # required in secure_gateway
bootstrap_strategy: persistent_then_initstepslew
extra_config: ""
```

### Examples

Secure client (no NTP service):

```yaml
mode: secure_client
```

Secure gateway serving 192.168.1.0/24:

```yaml
mode: secure_gateway
allow_ipv4:
  - 192.168.1.0/24
```

`bootstrap_strategy` controls how cold boots recover time when no hardware
clock is available:

* `persistent_then_initstepslew` *(default)* – use previously saved time (max
  +48h jump) and then run an unauthenticated `initstepslew` with the public
  pool.
* `persistent_only` – use only the saved time.
* `initstepslew` – skip persistent time and use `initstepslew`.
* `oneshot` – run `chronyd -q` against the public pool before daemonizing
  (only if the year is < 2024).
* `none` – do nothing special; NTS may fail until the clock is close.

Additional chrony directives can be appended via `extra_config`.

## Security

* Uses explicit NTS servers; no pools for authenticated time.
* `cmdport 0` and Unix socket only; no remote control interface.
* AppArmor permits writes only to `/data/chrony/**`.
* When `set_system_clock` is disabled, chronyd runs with `-x` and will not
  adjust the host clock.

## Migration

Previous versions stored the drift file at `/data/chrony/chrony.drift`. The
init script now owns `/data/chrony` and `/etc/chrony` as `chrony:chrony`, fixing
permission issues like `chrony.drift.tmp: Permission denied`.

---

This project is provided "AS IS" without warranty of any kind. See `LICENSE`
for details.

