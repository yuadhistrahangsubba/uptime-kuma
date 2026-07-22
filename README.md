# education-bt Monitoring — Uptime Kuma

Availability / blackbox monitoring for the EMIS platform. Watches every service's HTTP
`/health` endpoint and TCP microservice port, plus shared infrastructure (NATS, Redis,
Postgres). Complements (does not replace) a future Prometheus + Grafana metrics stack.

> **This is an infrastructure unit, not a NestJS microservice.** It runs the upstream
> `louislam/uptime-kuma` container as-is — no `src/`, no CQRS, no NATS queue. Do **not**
> confuse it with `common-service` (the lookup-tables app).

---

## Deployment model (Flow B — production)

```
┌───────────────────────────────┐        ┌──────────────────────────────┐
│  Monitoring host (own VM/AZ)  │ private │  App failure domain          │
│                               │  VPC    │                              │
│  Uptime Kuma (this compose)   │────────▶│  service health + TCP ports  │
│  probes by private IP/DNS     │         │  + message/cache brokers     │
└──────────────┬────────────────┘         └──────────────────────────────┘
               │ heartbeat
               ▼
   External dead-man's-switch (tiny SaaS)   ← watches Kuma itself
```

**Rules this enforces (aligned with Google SRE + ISO 27001 monitoring guidance):**

1. **Separate failure domain** — run this on its own host/VM, ideally a different AZ/rack,
   so an outage of the app services does **not** take the monitor down with them.
2. **Same private network** — the host must be able to route to the services by private
   IP/DNS (that is how it reaches internal `/health` and TCP ports).
3. **Do NOT attach app Docker networks** — cross-host reach uses IP routing, not Docker
   networks. Joining app networks only works same-host and re-couples the failure domain.
4. **Watch the watcher** — add one external check on this Kuma instance (see below).

For **local dev** you may instead run this on the same host and join a shared Docker
network; it's simpler but loses the outage-survival property. Prod = Flow B.

---

## Quick start

```bash
cd monitoring
docker compose up -d
docker compose ps          # confirm healthy
```

### Access the UI (internal-only)

The UI is bound to `127.0.0.1:3001` on the monitoring host — not publicly exposed.
From your workstation:

```bash
ssh -L 3001:localhost:3001 <monitoring-host>
# then open http://localhost:3001
```

On first launch, create the admin account immediately (do not leave it unset).

### First-run setup order

1. Create admin account, enable 2FA (Settings → Security).
2. Add a notification channel (Settings → Notifications) — Telegram/Slack/email/webhook.
3. Add monitors from [`monitors.md`](./monitors.md) — TCP first (fastest coverage), then HTTP `/health`.
4. Create a Status Page grouping services by domain (grouping suggested in `monitors.md`).
5. Configure Maintenance windows for planned deployments to suppress false alerts.

---

## Watch-the-watcher (required)

Kuma has no built-in self-monitoring. Pick one:

- Point a free external monitor (UptimeRobot / Better Stack) at this Kuma's URL, **or**
- Use a dead-man's-switch (Healthchecks.io): Kuma pushes a heartbeat; you're paged if it stops.

---

## Backups

Everything (monitors, notifications, history, settings) lives in the `uptime-kuma-data`
volume. Back it up regularly:

```bash
# Snapshot the data volume to a tarball
docker run --rm -v uptime-kuma-data:/data -v "$PWD":/backup alpine \
  tar czf /backup/uptime-kuma-backup-$(date +%Y%m%d).tar.gz -C /data .
```

Also use **Settings → Backup** in the UI to export a JSON of monitors/notifications.

---

## Phase 2 — Prometheus / Grafana integration (when that stack lands)

Uptime Kuma exposes a Prometheus-format `/metrics` endpoint (uptime, response time,
cert-days-remaining), protected by an API key.

1. Kuma → **Settings → API Keys** → create a key.
2. Add a Prometheus scrape job (basic-auth: blank username, API key as password):

   ```yaml
   - job_name: uptime-kuma
     metrics_path: /metrics
     basic_auth:
       username: ""
       password: "<uptime-kuma-api-key>"
     static_configs:
       - targets: ["<monitoring-host>:3001"]
   ```

3. Import a community Uptime Kuma dashboard into Grafana.

Kuma stays the **blackbox/uptime + status-page** layer; Prometheus is the **whitebox
metrics** layer (per-service CPU/mem/latency — requires adding `prom-client` + `/metrics`
to each NestJS service, which none have today).

---

## Upgrading

```bash
docker compose pull
docker compose up -d
```

The `:1` tag stays on the v1 major line. Review release notes before jumping majors.
