# SentinelReady

**Your Virtual SRO. Always Watching. Never Surprised.**

SentinelReady sits between your monitoring tools and your team. Your
monitoring tools keep doing what they do — SR makes sure the right
signals reach the right people. It triages every alert with AI,
recognizes patterns it's seen before, and only wakes you up when it
actually matters.

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
- Weekly behavior digest — Pro feature, coming soon
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

## Feedback

This is early access, invite-only for now. Your feedback shapes what
ships to general availability.

- Bugs: open an issue on this repo
- Ideas: email jeff@sentinelready.io
- What broke: same

Thank you for being here early.
