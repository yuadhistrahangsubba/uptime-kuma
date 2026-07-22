# Contributing

Thanks for helping improve this repo. It holds the **self-hosted Uptime Kuma deployment
configuration** (Docker Compose + monitor documentation) used to monitor the EMIS
platform. It is **not** the upstream Uptime Kuma source — for the application itself, see
[louislam/uptime-kuma](https://github.com/louislam/uptime-kuma).

## Prerequisites

- Docker + Docker Compose

## Local workflow

```bash
git clone https://github.com/yuadhistrahangsubba/uptime-kuma.git
cd uptime-kuma

# validate the compose file after any change
docker compose config

# run it locally to test
docker compose up -d
docker compose logs -f
```

## Ground rules

This is a **public** repository. Keep it safe to publish:

- **Never commit secrets** — no credentials, tokens, TLS private keys, or `.env` files.
- **Never commit internal infrastructure detail** — internal service ports, private IPs,
  message-broker/queue names, or topology. The internal monitor map lives in
  `monitors-internal.md`, which is **git-ignored on purpose**. Do not un-ignore or paste
  its contents into a tracked file.
- **Only public endpoints** belong in `monitors.md` (e.g. `https://emis.systems.gov.bt/...`).
- **Pin image versions explicitly** in `docker-compose.yml` (e.g. `:2.4.0`) — no floating
  `latest`. Note the v1 → v2 data format change; back up the volume before major bumps.
- **Keep docs in sync** — update `README.md` / `monitors.md` when behavior changes.

Before opening a PR, the safety check that CI/maintainers run:

```bash
# should print "clean" — no internal ports leaked into tracked files
grep -nE '808[0-9]|801[0-3]|6081|4222|6379|5432|TRANSPORT_PORT' \
  README.md monitors.md docker-compose.yml && echo "LEAK" || echo "clean"
```

## Commit messages

Use the **gitmoji** convention (matching the project history):

| Emoji | Use for |
|---|---|
| `:sparkles:` | new capability |
| `:bug:` | bug fix |
| `:arrow_up:` | dependency / image version bump |
| `:memo:` | documentation |
| `:lock:` | security-related change |
| `:fire:` | removing code/config |

Example:

```
:memo: Document push-monitor setup for cron jobs
```

## Pull requests

1. Branch from `master`.
2. Make the change; run `docker compose config` and the leak check above.
3. Open a PR with a clear description of what and why.
4. Confirm no secrets or internal detail were added.
