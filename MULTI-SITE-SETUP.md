# Multi-Site Setup

This file only applies if you're running one SentinelReady instance
across multiple sites (Multi-Site tiers). Single-site Community/Pro
customers never need this — everything here is additional setup on top
of the main `README.md`, not a replacement for it.

For MSPs and multi-location organizations — retail chains, healthcare
networks, manufacturing plants, franchises, universities, enterprise
divisions.

Each site's alerts and sitreps need to reach only that site's own team,
never mixed together. This is built entirely on `customer_code` — a
single SentinelReady instance can serve many sites simultaneously as
long as each one tags its alerts with its own code.

## How a site gets tagged

Every webhook checks, in order:
1. A `customer_code` (or `customer`/`tenant`/`tenant_id`/`account`) field
   in the alert payload itself, if your monitoring tool includes one
2. An `x-customer-code` HTTP header on the webhook request
3. Otherwise falls back to your yaml's `customer.default_code`

## CloudWatch — no setup needed

CloudWatch alarms already include your AWS account ID in the payload
(`account` via EventBridge, `AWSAccountId` via the older SNS-forwarding
format) — SentinelReady picks this up automatically. If each site is its
own AWS account, CloudWatch multi-site just works with zero
configuration.

## Grafana / Datadog — one custom header per site

Neither includes a customer identifier automatically, so add a custom
HTTP header to each site's own webhook/contact-point configuration:

- **Grafana**: Alerting → Contact points → your webhook contact point →
  add a custom HTTP header `x-customer-code: your-site-id`. Create one
  contact point per site, all pointing at the same
  `http://your-host:8000/webhook/grafana` URL, each with its own header
  value.
- **Datadog**: Integrations → Webhooks → your webhook → add the same
  custom header under its configuration.

## Routing each site's alerts and sitreps

Once alerts are tagged, configure that site's recipients via the
delivery policy API — this part isn't yaml-configurable yet (yaml only
sets one instance-wide default), so set it per site directly:

```bash
curl -X POST http://your-host:8000/delivery/policy \
  -H "x-sentinel-secret: your-secret-from-yaml" \
  -H "Content-Type: application/json" \
  -d '{
    "customer_code": "your-site-id",
    "policy": {
      "routes": {"sre": ["site-oncall@example.com"], "all": ["site-oncall@example.com"]},
      "sitrep": {"enabled": true, "cadence": "daily", "recipients": ["site-manager@example.com"]}
    }
  }'
```

Sites are fully isolated from each other by design — a site with no
explicit policy set falls back to your yaml's instance-wide default,
never another site's configuration.
