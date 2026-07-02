# Operations Runbook

## Local Persistent Run

```bash
ADMIN_TOKEN=dev-secret \
WAF_IDS_STATE_PATH=./waf-ids-state.local.json \
DNSBL_ORIGIN=dnsbl.local \
EVENT_LIMIT=1000 \
cargo run
```

Open `http://127.0.0.1:8080/admin`.

## Health Check

```bash
curl -fsS http://127.0.0.1:8080/healthz
```

Expected fields:

- `status`: `ok`
- `persistence`: `memory` or `file`
- `dnsbl_origin`: configured DNSBL origin without a trailing dot
- `event_limit`: retained security event count

## Smoke Test

```bash
scripts/smoke.sh
```

The smoke test starts the service on a temporary port with a temporary JSON state file, verifies admin and management surfaces, creates a blocking route, triggers a blocked gateway request, checks KPIs and DNSBL export, restarts the process, and verifies that the route persisted.

## Safe Change Procedure

1. Start new routes in `monitor` mode.
2. Confirm recent events and KPIs show expected matches.
3. Switch only the specific route to `block` mode.
4. Keep the previous route JSON available for rollback.
5. Disable the route or switch back to `monitor` if legitimate traffic is blocked.

## Production Boundaries

This baseline is suitable for local and controlled lab deployments. Internet-facing use still requires:

- TLS termination and identity-aware admin access
- upstream allowlists and egress controls
- durable database storage with backups
- Coraza/OWASP CRS WAF adapter
- Suricata EVE ingest for IDS events
- STIX/TAXII, MISP, or OpenCTI feed import
- human approval workflow for AI SOC recommendations that change enforcement
