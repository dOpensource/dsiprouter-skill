# dSIPRouter Fraud-Detection Skill

## Purpose
Identify potentially fraudulent SIP activity in a dSIPRouter instance by analyzing CDRs (Call Detail Records), auth failures, and unusual destination patterns. Produce a concise report with flagged items and recommended mitigations.

## When to Use
- User asks to find fraudulent calls, short-duration bursts, auth failures, or unusual destinations in dSIPRouter.
- Investigations involving CDR analysis over a specific time window.

## Requirements
- dSIPRouter API base URL (typically `https://<host>:5000`)
- DSIP_TOKEN
- Time window (e.g., last 2 days, March 1–12) — use `dtfilter=YYYY-MM-DD`
- Target scope (endpoint group ID/name or endpoint ID)
- Access to auth logs (syslog or DB) if auth failure analysis is required

## Endpoints (common)
- Kamailio stats: `GET /api/v1/kamailio/stats`
- Endpoint groups: `GET /api/v1/endpointgroups`
- CDRs by endpoint group: `GET /api/v1/cdrs/endpointgroups/{id}` (use `?dtfilter=YYYY-MM-DD`)
- CDRs by endpoint: `GET /api/v1/cdrs/endpoint/{id}` (use `?dtfilter=YYYY-MM-DD`)

> Note: Use `dtfilter` to fetch records starting at the given date.

## Workflow
1. **Validate API access**
   - Call `/api/v1/kamailio/stats` with bearer token to confirm auth.
2. **Resolve target group/endpoint**
   - List endpoint groups and match by name.
3. **Fetch CDRs for the time window**
   - Use `dtfilter=YYYY-MM-DD` (records starting at that date).
4. **Analyze for fraud signals**
   - **Short-duration bursts**: multiple calls within minutes, durations < 30–60s.
   - **Repeated failed auth**: requires auth logs or DB.
   - **Unusual destinations**: sudden calls to new country codes or rare destinations.
5. **Report**
   - Provide counts, timestamps, source/destination, and reason flagged.
   - Recommend mitigations (blocklist, rate-limit, change credentials).

## Example Commands
```bash
# Validate token
curl -k -H "Authorization: Bearer $DSIP_TOKEN" https://$HOST:5000/api/v1/kamailio/stats

# List endpoint groups
curl -k -H "Authorization: Bearer $DSIP_TOKEN" https://$HOST:5000/api/v1/endpointgroups

# CDRs by endpoint group (from a start date)
curl -k -H "Authorization: Bearer $DSIP_TOKEN" \
  "https://$HOST:5000/api/v1/cdrs/endpointgroups/32?dtfilter=2026-03-01"
```

## Output Template
- **Time window:** …
- **Scope:** endpoint group / endpoint …
- **Flags:**
  - Short-duration burst (count, time range, dst)
  - Unusual destination (dst, frequency)
  - Auth failures (source, count)
- **Suggested actions:** …

## Notes
- If the API returns empty data but the UI shows calls, confirm the endpoint group ID or query pagination/filters.
- For auth failures, identify log file location (e.g., `/var/log/syslog`, `/var/log/kamailio.log`) or DB table.
