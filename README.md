# SentinelReady

**Your reliability superpower.**

SentinelReady sits between your monitoring tools and your team. Your
monitoring tools keep doing what they do — SR makes sure the right
signals reach the right people. It triages every alert with AI,
recognizes patterns it's seen before, and only wakes you up when it
actually matters.

---

## System Requirements

- **RAM**: 8GB minimum, 16GB comfortable. Most of this is Ollama running
  the `llama3.1` model (~5-6GB resident) — SentinelReady itself is
  lightweight (well under 1GB).
- **CPU**: 4 cores recommended. Ollama on CPU-only inference is slower
  per alert (a few seconds), but the Behavior Pattern Library means
  repeat alerts skip AI entirely after the first occurrence, so this
  matters most in the first few days.
- **Disk**: ~10GB free (model weights + Docker images + growing pattern
  data).
- **GPU**: not required. If present, Ollama uses it automatically and
  triage is significantly faster.
- **CPU architecture**: **amd64/x86_64 only** (standard Intel/AMD). Not
  yet built for arm64 — if you're on an Apple Silicon Mac (M1/M2/M3/M4)
  or an ARM-based Linux/Windows machine, this hasn't been tested and may
  not run reliably even under emulation. Use an Intel/AMD machine or VM.

**Before you start — check what you already have:**

1. **Docker.** Check: `docker --version`
   - Already installed? Skip to step 2.
   - Missing? Install [Docker Desktop](https://www.docker.com/products/docker-desktop/) (Mac/Windows), or Docker Engine + the Compose plugin on Linux.
2. **Git.** Check: `git --version`
   - Already installed? Skip to "Get Running" below.
   - Missing? Either install it ([git-scm.com](https://git-scm.com/downloads)), or skip `git clone` entirely — download `docker-compose.yml` and `sentinelready.yaml.example` directly from the repo page instead.

The included `docker-compose.yml` does not set memory/CPU limits on the
containers, so on a small or shared machine, consider adding your own
`deploy.resources.limits` if you want to cap what Ollama can consume.

---

## Get Running in 3 Commands

The image is currently private (early access) — you'll need to authenticate
once before the first pull. You should have received a token separately;
if not, ask for one.

```bash
docker login ghcr.io -u YOUR_GITHUB_USERNAME
# paste the token you were given when prompted for a password
```

Then:

```bash
git clone https://github.com/browningjc97/sentinelready-docker
cd sentinelready-docker
cp sentinelready.yaml.example sentinelready.yaml
# Edit sentinelready.yaml — set your webhook_secret and team name
docker compose up -d
```

SentinelReady will pull Ollama automatically and download the llama3.1 model
on first start. First startup takes 3-5 minutes depending on your connection.

---

## Upgrading

New version released? Two commands:

```bash
docker compose pull
docker compose up -d
```

That's it. Your behavior patterns, learned history, environment context
rules, and configuration are all preserved. SentinelReady stores everything
in Docker volumes that are never touched by an upgrade.

Downtime is the ~10 seconds it takes the container to restart.

---

## Verify It's Running

```bash
curl http://localhost:8000/health
```

You should see something like:
```json
{
  "status": "ok",
  "version": "0.5.0",
  "ai_enabled": true,
  "ai_provider": "ollama",
  "behavior_patterns_learned": 0,
  "license_tier": "community"
}
```

---

## Point Your Alerting Tool at SentinelReady

| Source | Webhook URL |
|--------|-------------|
| Grafana | `http://your-host:8000/webhook/grafana` |
| DataDog | `http://your-host:8000/webhook/datadog` |
| CloudWatch | `http://your-host:8000/webhook/cloudwatch` |
| Slack | `http://your-host:8000/webhook/slack` |
| UDM | `http://your-host:8000/webhook/udm` |
| UniFi Protect | `http://your-host:8000/webhook/unifi-protect` |
| Generic | `http://your-host:8000/webhook/generic` |

Set the webhook secret header: `x-sentinel-secret: your-secret-from-yaml`

**No configuration needed for source/instance tracking.** When a flood of
similar alerts gets collapsed into one summary, SentinelReady reports
whether it was really one host repeating vs. many different hosts
failing independently — automatically, using whatever instance/host/pod
field your monitoring tool already includes natively (Grafana's
`instance`/`pod`/`node` labels, Datadog's `host` field, etc.). Nothing to
set up — if your alert source includes it, it's used; if it doesn't, the
summary just says "source unknown" and everything else works exactly the
same. For the generic webhook specifically, you can optionally include
an `instance` or `host` field in your payload if you want this detail
for custom integrations — still entirely optional.

---

Running SentinelReady across multiple sites on one instance
(Multi-Site tiers)? See [`MULTI-SITE-SETUP.md`](MULTI-SITE-SETUP.md) —
single-site Community/Pro customers can skip this entirely.

---

## Send a Test Alert

```bash
curl -X POST http://localhost:8000/webhook/generic \
  -H "Content-Type: application/json" \
  -H "x-sentinel-secret: change-me-please" \
  -d '{
    "alert_name": "HighCPU",
    "service": "api-server",
    "severity": "warning",
    "environment": "production",
    "description": "CPU above 85% for 10 minutes",
    "value": "87%"
  }'
```

---

## How It Works

SentinelReady builds a **Behavior Pattern Library** from every alert it sees.
The first time an alert fires, AI triages it and stores the result.
The second time the same pattern fires, it recognizes it instantly —
no AI call needed, no delay, no cost.

Over time it learns which alerts self-resolve, which ones need you,
and adjusts its confidence automatically.

Community edition stores up to 50 behavior patterns.

SentinelReady also fails open — a slow AI dependency, an alert flood, or
an exhausted daily budget all degrade to a visible raw alert rather than
silence. You'll never lose an alert to an internal hiccup.

---

## Sitreps — a periodic digest of everything SentinelReady saw

Instead of (or alongside) real-time escalations, SentinelReady can send
you a digest — how many alerts came in, what got escalated, what
patterns it's learned, what's worth a permanent fix rather than repeated
manual triage.

Off by default. Turn it on in `sentinelready.yaml`:

```yaml
sitrep:
  enabled: true
  cadence: daily          # daily | weekly | monthly
  recipients:
    - "you@yourcompany.com"
```

Once enabled, SentinelReady schedules and sends these itself — no
external cron needed, nothing else to set up. It checks every 15 minutes
whether the configured cadence is due and sends when it is.

Want a second digest at a different cadence — e.g. a high-level weekly
summary for a manager alongside your own daily one — the `summary:`
section works the same way, independently:

```yaml
summary:
  enabled: true
  cadence: weekly
  recipients:
    - "manager@yourcompany.com"
```

Community edition includes 5 free sitrep sends (lifetime, per team);
after that, `/sitrep/send` returns a preview only until you upgrade to
Pro for unlimited sitreps.

---

## Email Delivery

SentinelReady sends email over plain SMTP — no proprietary API, no
lock-in to a specific provider. Point it at whatever mail relay you
already use:

```yaml
delivery:
  mode: smtp
  smtp:
    host: smtp.yourprovider.com
    port: 587
    username: your-username
    password: your-password
    use_tls: true
    from_addr: sentinelready@yourcompany.com
```

Works with Gmail, Office 365, a self-hosted mail server, or any
transactional email provider that hands out SMTP credentials (Resend,
SendGrid, Mailgun, Amazon SES, etc.) — SentinelReady doesn't know or
care which one you use.

A couple of things worth knowing before you rely on it:

- **Deliverability is on your mail provider, not SentinelReady.** A
  "sent" status only means SMTP accepted the handoff — actual delivery
  to an inbox (or a carrier's email-to-SMS gateway) depends on your
  provider's own sender reputation, SPF/DKIM, and (for self-hosted
  relays) a valid PTR record. If alerts aren't arriving, check your
  mail server's own logs, not just SentinelReady's.
- **`mode: local`** (the default) logs email content instead of
  sending it — useful for testing without touching a real mail
  provider, but nothing actually reaches anyone until you switch to
  `mode: smtp`.

---

## SMS Delivery

**Community/Pro — carrier email-to-text (at your own risk).** Critical
alerts can reach a phone as a text message with zero extra
infrastructure: put your carrier's email-to-SMS gateway address in
`delivery.routes` instead of (or alongside) a real email address —
SentinelReady just sends it as a plain-text email, same `smtp` config
as everything else above.

| Carrier | Gateway |
|---|---|
| AT&T | `number@txt.att.net` |
| T-Mobile | `number@tmomail.net` |
| Verizon | `number@vtext.com` |
| Sprint | `number@messaging.sprintpcs.com` |
| US Cellular | `number@email.uscc.net` |
| Boost | `number@sms.myboostmobile.com` |
| Cricket | `number@sms.cricketwireless.net` |
| Metro PCS | `number@mymetropcs.com` |
| Google Fi | `number@msg.fi.google.com` |

"At your own risk" because this depends entirely on the carrier's own
gateway — no delivery guarantee, no read receipt, and carriers have
been known to rate-limit or block gateway traffic that looks
automated. It works, but it's not a substitute for a real SMS provider
if reliability matters to you.

**For guaranteed delivery**, point an [outbound webhook](#outbound-webhooks)
at your own Twilio-connected integration (Zapier, a PagerDuty/Opsgenie
routing rule, etc.) — SentinelReady doesn't have a native Twilio
integration today, and never hosts or pays for SMS delivery itself.
The carrier gateway above is the only SMS path SentinelReady sends
directly.

---

## Outbound Webhooks

SR works with your existing incident management tools — it makes what
they receive smarter, not replace them.

SentinelReady never integrates with individual tool APIs — one
standard JSON payload, infinite integrations. Configure a list of URLs
per tier (`critical`, `high`, `sitrep`) in `sentinelready.yaml` and
SentinelReady fires its own triage payload at each one:

```yaml
delivery:
  webhooks:
    critical:
      - type: pagerduty
        url: https://events.pagerduty.com/v2/enqueue
        routing_key: your-pagerduty-integration-key
      - type: slack
        url: https://hooks.slack.com/services/T000/B000/XXXXXXXX
    high:
      - type: slack
        url: https://hooks.slack.com/services/T000/B000/XXXXXXXX
    sitrep:
      - type: generic
        url: https://your-endpoint.example.com/ingest
```

`critical`/`high` fire per alert, in real time — the same moment as
any SMS/push escalation. `sitrep` fires once per digest, alongside the
sitrep email.

- **`type: pagerduty`** — native PagerDuty Events API v2 format.
- **`type: slack`** — native Slack incoming-webhook format.
- **`type: generic`** (or omit `type`) — SentinelReady's own plain
  JSON (the alert + AI triage brief, or the full sitrep report) —
  point this at anything that accepts a JSON POST.

This also works per-site if you're running Multi-Site tiers — see
[`MULTI-SITE-SETUP.md`](MULTI-SITE-SETUP.md).

---

## What's Included

- Webhook receiver: Grafana, DataDog, CloudWatch, Slack, UDM, Generic
- AI triage via Ollama (local, free, your hardware)
- Behavior Pattern Library (50 patterns)
- Environment context rules — tell SentinelReady what's normal
- Causal correlation — detects node failure cascades and memory patterns
- Outcome learning — free in both Community and Pro
- Escalation decision engine — escalate, sitrep, or suppress
- Read-only status page: `GET /status`
- Pattern stats: `GET /patterns/stats`

---

## Known Limitations (Current Release)

- Mobile push notifications — Pro feature, coming soon
- SSL certificate monitoring — coming soon
- Web dashboard — coming after Pro launch
- SMTP delivery defaults to local logging — configure smtp section in yaml to enable email
- Single-instance only — no HA/failover. See the reliability note above; pair
  with an external dead-man's-switch if that matters for your setup.

---

## Known Gotchas

### Ollama on a separate host

If running Ollama on a different host from SentinelReady (not via the
docker-compose sidecar), Ollama binds to localhost by default and
SentinelReady cannot reach it.

Fix — add to Ollama's systemd service:

```
Environment="OLLAMA_HOST=0.0.0.0"
```

Then: `systemctl daemon-reload && systemctl restart ollama`

This is not needed when using the standard `docker-compose.yml` — both
containers share an internal Docker network automatically.

### Firewall / network exposure

SentinelReady is designed to sit behind your own firewall, reachable only
by the monitoring tools you point at it — not exposed directly to the
public internet.

- **Port 8000 (SentinelReady itself)** — needs to be reachable by whatever
  is sending it webhooks (Grafana, DataDog, CloudWatch, your alerting
  tool). It does **not** need to be reachable from the public internet.
  The webhook secret (`x-sentinel-secret` header) and per-client rate
  limiting both help if it's ever exposed anyway, but neither is a
  substitute for keeping it on a private network.
- **Port 11434 (Ollama)** — should **never** be exposed beyond your
  internal network. It has no authentication of its own; anyone who can
  reach it can run inference on your GPU/CPU for free. Only needs to be
  reachable by the SentinelReady container itself (automatic on the
  standard `docker-compose.yml`'s internal network).
- **Outbound only**: SMTP (if `delivery.mode: smtp`), and the AI provider's
  API (Claude/OpenAI, if configured) — no inbound firewall rule needed for
  either.

If you do need SentinelReady reachable from outside your network (e.g. a
cloud-hosted alerting tool that can't reach your internal IPs), put it
behind a reverse proxy or VPN rather than exposing port 8000 directly.

---

## Activating Pro

If you've purchased Pro, you'll receive a license key. Set it in your
`sentinelready.yaml`:

```yaml
license:
  tier: pro
```

And provide the key itself as an environment variable — add this to the
`sentinelready` service in your `docker-compose.yml`:

```yaml
environment:
  - SENTINEL_CONFIG=/app/sentinelready.yaml
  - SENTINEL_LICENSE_KEY=<the key you were given>
```

Then `docker compose up -d` to restart with Pro unlocked. Check
`docker compose logs sentinelready` — you should see a `License:` line
confirming your customer name and expiry (if any). If the key is missing,
invalid, or expired, SentinelReady automatically and silently falls back
to Community — it never crashes or blocks alert delivery over a licensing
problem.

This unlocks the Pro tier itself (unlimited patterns, team members, weekly
digest, mobile push, email support) regardless of AI provider — Pro
doesn't require switching off Ollama. Outcome learning ships free in both
editions.

Swapping the AI provider is a separate, unrelated step available in
**either** edition — Community or Pro, you bring your own API key and
that provider bills you directly; SentinelReady never sees or proxies the
cost.

**Claude:**

```yaml
ai:
  provider: claude
  model: claude-haiku-4-5-20251001   # or another Claude model
```

```yaml
environment:
  - ANTHROPIC_API_KEY=<your own Anthropic API key>
```

**Any OpenAI-compatible API** (OpenAI, Azure OpenAI, Groq, Together.ai,
OpenRouter, local vLLM/LM Studio):

```yaml
ai:
  provider: openai_compatible
  model: gpt-4o-mini                       # whatever model your provider offers
  base_url: https://api.openai.com/v1      # your provider's API base URL
  auth_header_style: bearer                # bearer (most) | api-key (Azure OpenAI)
  api_version: ""                          # Azure only, e.g. 2024-10-21
```

```yaml
environment:
  - SENTINEL_AI_API_KEY=<your own API key>
```

## Cost Management

If using a cloud AI provider (Claude, OpenAI) set a monthly spending
limit before connecting SR to production alert traffic. SR processes
every alert — a noisy environment can generate significant API calls in
the first few weeks before pattern recognition reduces them.

Recommended limits for getting started:
- Claude API: $20/mo at console.anthropic.com → Settings → Billing →
  Monthly spend limit
- OpenAI API: $20/mo at platform.openai.com → Settings → Billing →
  Usage limits

SR gets cheaper over time as pattern recognition eliminates repeat AI
calls — most customers see 80% cost reduction within 90 days.

---

## Grafana Dashboards

If you're already running Prometheus + Grafana (the included
`docker-compose.yml` does not bundle these — bring your own or point at
an existing stack), SentinelReady exposes Prometheus-format metrics at
`GET /metrics`: alerts processed, escalations, sitreps delivered, AI
calls avoided (pattern-cache hits), and patterns learned per customer.

Import the dashboard:

1. In Grafana, go to **Dashboards → New → Import**.
2. Upload `dashboard/sentinelready-dashboard.json` (in this repo) or
   paste its contents.
3. Select your Prometheus datasource when prompted.
4. Point that Prometheus instance at `your-host:8000/metrics` (add a
   scrape job or ServiceMonitor, depending on your setup).

---

## Feedback

This is early access, invite-only for now. Your feedback shapes what
ships to general availability.

- Bugs: open an issue on this repo
- Ideas: email jeff@sentinelready.io
- What broke: same

Thank you for being here early.
