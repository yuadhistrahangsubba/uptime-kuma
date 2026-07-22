# Monitor list — public endpoints

Monitors to create in Uptime Kuma for the **public-facing** endpoints of the platform,
reached through the ingress at `https://emis.systems.gov.bt`. Enter these in the UI
(Add New Monitor).

> This file intentionally lists **only public HTTPS endpoints**. Internal service ports
> and topology are kept in a separate, non-published file.

## Common settings

- **Type:** `HTTP(s)` — or `HTTP - Keyword` (keyword `ok`) to also verify the body
- **Method:** GET
- **Accepted status codes:** `200-299`
- **Interval:** 60s (use 30s for the auth services)
- **Certificate Expiry Notification:** ON — HTTPS monitors auto-track the TLS cert for
  `emis.systems.gov.bt`, so you get warned before it expires.

The health endpoints return `{"status":"ok",...}`, so a keyword monitor on `ok` confirms
the app is actually serving, not just returning a status code.

---

## A. Public health monitors (verified reachable → `200 {"status":"ok"}`)

| Service | Monitor name | URL | Keyword |
|---|---|---|---|
| admin-auth | Admin Auth | `https://emis.systems.gov.bt/svc/admin-auth/health` | `ok` |
| user-auth | User Auth | `https://emis.systems.gov.bt/svc/user-auth/health` | `ok` |
| pii | PII | `https://emis.systems.gov.bt/svc/pii/health` | `ok` |
| organization | Organization | `https://emis.systems.gov.bt/svc/organization/health` | `ok` |
| common | Common | `https://emis.systems.gov.bt/svc/common/health` | `ok` |
| academic | Academic | `https://emis.systems.gov.bt/svc/academic/health` | `ok` |
| admission-transfer | Admission Transfer | `https://emis.systems.gov.bt/svc/admission-transfer/health` | `ok` |
| result-processing | Result Processing | `https://emis.systems.gov.bt/svc/result-processing/health` | `ok` |
| tre | TRE | `https://emis.systems.gov.bt/svc/tre/health` | `ok` |
| spms | SPMS | `https://emis.systems.gov.bt/svc/spms/health` | `ok` |
| cognitive | Cognitive | `https://emis.systems.gov.bt/svc/cognitive/health` | `ok` |

## B. Not publicly monitorable via `/health` (handle internally)

Live checks showed these are **not** usable as public health monitors — they redirect to
sign-in (auth-gated) or the path is not routed publicly. Monitor these from inside the
network instead, or expose a dedicated unauthenticated health route.

| Service | Public `/health` result |
|---|---|
| staff-transfer | `302 → /signin` (auth-gated) |
| analytics | `302 → /signin` (auth-gated) |
| digital-textbook | `302 → /signin` (auth-gated) |
| nfe | `302 → /signin` (auth-gated) |
| admission-transfer-management | `404` (not routed at this public path) |

> If you add a public monitor for these anyway, set accepted codes to include `302` or it
> will report DOWN — but a redirect to sign-in is a weak liveness signal, so internal
> monitoring is preferred.

---

## C. Push monitors (jobs / cron)

For scheduled jobs: create a **Push** monitor, then have the job `curl` the generated push
URL on success. Uptime Kuma alerts if the heartbeat stops arriving.

---

## Suggested Status Page grouping

- **Auth & Identity** — Admin Auth, User Auth, PII
- **Core & Reference** — Common, Organization, Academic
- **Admissions & Transfers** — Admission Transfer
- **Results & Assessment** — Result Processing, TRE, SPMS
- **Learning & Content** — Cognitive
