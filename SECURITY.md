# Security Policy

This repository contains the **deployment configuration** for a self-hosted Uptime Kuma
instance that monitors the EMIS platform. It is public, so it is maintained to contain
**no secrets and no internal infrastructure detail**.

> For vulnerabilities in the Uptime Kuma application itself, report upstream at
> [louislam/uptime-kuma](https://github.com/louislam/uptime-kuma/security).

## Supported versions

| Component | Version | Status |
|---|---|---|
| Uptime Kuma image | `2.4.0` (pinned in `docker-compose.yml`) | Supported |
| Uptime Kuma v1.x | any | Not supported — upgrade to v2 |

Security fixes are delivered by bumping the pinned image tag. Always run a supported,
explicitly pinned version — never a floating `latest`.

## Reporting a vulnerability

**Do not open a public issue for security problems.**

Please report privately via GitHub:

1. Go to the **Security** tab → **Report a vulnerability** (private advisory).
2. Describe the issue, affected files/config, and reproduction steps.

You can expect an acknowledgement within a few working days and a coordinated fix before
any public disclosure.

## Scope

In scope for this repo:

- Secrets, credentials, TLS keys, or `.env` values accidentally committed
- Internal infrastructure detail leaked into tracked files (internal ports, private IPs,
  topology, message-broker/queue names)
- Insecure defaults in `docker-compose.yml` (e.g. exposing the UI publicly, missing
  volume persistence)

Out of scope (report upstream instead):

- Vulnerabilities in the Uptime Kuma application code or image

## Hardening baseline enforced here

- **UI is bound to `127.0.0.1`** — never exposed publicly; access via SSH tunnel.
- **Only public HTTPS endpoints** appear in tracked docs; the internal monitor map lives
  in `monitors-internal.md`, which is **git-ignored**.
- **Pinned image version** with documented, backed-up upgrade path.
- **Separate failure domain** — the monitor runs on its own host, isolated from the
  services it watches (see `README.md`, Flow B).

If you find any committed secret or internal detail, treat it as a vulnerability and
report it privately as above — and assume the exposed value must be rotated.
