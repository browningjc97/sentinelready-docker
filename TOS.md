# SentinelReady — Terms of Service

**Version 1.0 — effective 2026-08-29.**

These Terms govern your use of SentinelReady. By installing, running, or
purchasing a license for SentinelReady, you agree to them.

---

## 1. What SentinelReady Is

SentinelReady is self-hosted software: a "Virtual SRO" that sits between
your monitoring tools and your team. Your monitoring tools keep doing
what they do — SentinelReady triages incoming alerts, recognizes
patterns, and makes escalation/suppression decisions on your behalf.
You run it on your own infrastructure. SentinelReady, LLC ("we," "us,"
"SentinelReady") provides the software; you ("you," "your," "Customer")
operate it.

Three editions exist:

- **Community** — free, self-hosted, feature-capped (currently 50
  learned behavior patterns and one customer code; caps may change
  between releases).
- **Pro** — paid, self-hosted, unlocked via a signed license key.
  Removes the pattern cap and adds the behavior digest, additional
  delivery targets, and email support.
- **Multi-Site** — paid, for operators managing multiple independent
  sites. Adds cross-site reporting and raises the customer-code limit
  according to the tier purchased.

All editions run identical AI-provider logic — Ollama (local, no license
key required, the shipped default) or a customer-supplied API key for
Claude or any OpenAI-compatible provider (OpenAI, Azure OpenAI, Groq,
Together.ai, OpenRouter, local vLLM/LM Studio). **AI provider choice is
not gated by tier.** If you configure a third-party AI provider, see the
Privacy Policy's Third-Party AI Providers section — that is a direct
relationship between you and that provider, not something SentinelReady
is a party to.

### AI provider costs are yours

The shipped default (Ollama) runs on your own hardware and has no
per-call cost. **If you configure a paid third-party provider, that
provider bills you directly for every enrichment call SentinelReady
makes with your key.** SentinelReady never sees, proxies, caps, or
reimburses that spend, and your bill is between you and that provider
regardless of alert volume — including volume driven by a monitoring
misconfiguration or an alert flood on your own systems.

SentinelReady ships three things that reduce that exposure. None of
them is a spending limit, and you should not rely on them as one:

- **Fingerprint caching** (`fingerprint.cache_enabled`, on by default)
  reuses a prior triage for a repeat of the same alert instead of
  making a fresh AI call.
- **The budget governor** (`governor.max_calls_per_minute` and
  `governor.max_calls_per_day`) is a hard ceiling on the *number* of
  enrichment calls, per minute and per day. Calls beyond either ceiling
  are not made; those alerts are delivered raw and unenriched rather
  than dropped.
- **`ai.daily_budget_usd`** sends **one warning email per day** when a
  local estimate crosses the figure you set. It is an estimate computed
  from token counts times rates *you* configure — not your provider's
  actual invoice — and it does **not** stop anything. It is a signal,
  not a cap.

**The only real spending limit is the one you set directly with your AI
provider.** Set one there before configuring a paid provider here.

## 2. License Grant

### Community
Subject to these Terms, you're granted a non-exclusive, non-transferable
license to run SentinelReady Community on infrastructure you control, at
no cost. Community is limited to one customer code and the pattern cap
stated in Section 1; you may run it on as many instances as you like
within those limits.

### Pro and Multi-Site
Require a valid, signed license key issued by SentinelReady. A key
unlocks paid features for the customer and term it was issued for.

Pro is **$199 per month**, or **$1,990 per year** (a saving of $398
against monthly billing). Multi-Site is billed monthly at **$499
(Starter, up to 10 customer codes)**, **$899 (Growth, up to 25)**, or
**$1,499 (Scale, up to 50)**. Subscriptions renew automatically until
cancelled, and you may cancel at any time through the customer portal
linked in your purchase email; cancellation takes effect at the end of
the billing period already paid for. Prices may change for future
billing periods with notice under Section 9.

An expired, missing, or invalid key **does not** stop the software from
running — it silently falls back to Community behavior. This is a
deliberate design choice (fail-open, never blocks alert delivery over a
licensing problem) and is stated as such rather than as a loophole.

## 3. The Core Liability Framing

- **SentinelReady is a Virtual SRO exercising judgment, the same way a
  human SRO would.** It makes triage, suppression, and escalation decisions
  based on learned patterns and configured rules — the same category of
  decision a human on-call engineer makes when they decide not to escalate
  a flapping alert they've seen resolve itself fifty times before.
- **Alert fatigue from un-triaged noise causes *more* missed alerts than
  intelligent suppression does.** SentinelReady's entire value
  proposition rests on this premise.
- **What's structurally guaranteed vs. what isn't.** SentinelReady's
  fail-open design (circuit breakers, budget governor, bounded flood
  queue) guarantees it never *silently* drops an alert due to internal
  failure, a flooded queue, or an exhausted AI budget; every degraded
  path falls back to a visible raw alert rather than silence. This
  eliminates the specific failure mode of human fatigue — an on-call
  engineer who's seen the same alert fifty times can genuinely stop
  looking; SentinelReady doesn't get tired. **This does not guarantee the
  correctness of any individual triage, suppression, or escalation
  decision.** That depends on the AI's judgment call on a given alert and
  on whether your own configured context rules and environment settings
  actually reflect your environment — errors in either are outside
  SentinelReady's control.
- **Recommended practice, not a requirement: periodic review of
  SentinelReady's decisions.** The same way a healthy on-call program
  periodically reviews its escalation policies and near-misses, we
  recommend you periodically review SentinelReady's Behavior Pattern
  Library and suppressed-alert history (`GET /patterns/stats`, outcome
  tracking, and the behavior digest exist specifically to support this)
  to confirm its judgment continues to match your environment as it
  evolves. This is offered as guidance toward a healthier alerting
  practice, not a condition of using the product or a shift of
  responsibility onto you for decisions SentinelReady makes.
- **You remain responsible for your own monitoring strategy.**
  SentinelReady is a tool that assists triage and escalation decisions; it
  does not replace your judgment about what should be monitored, what
  your escalation policy should be, or whether SentinelReady's
  configuration correctly reflects your environment.
- **No SLA is implied on any edition unless explicitly stated in a
  separate written agreement.** Uptime, alert-delivery latency, and AI
  enrichment availability are not contractually guaranteed by these
  Terms alone.
- **Sacred severities.** You may configure specific severities
  (`alerting.sacred_severities`) that always escalate — never
  suppressed, never downgraded, never flood-aggregated, regardless of
  learned-pattern confidence or AI judgment. This is a
  customer-controlled override that wins over SentinelReady's own
  automated decisions, including a previously cached suppress decision
  on a recurring alert. It is not license-gated, is free in every
  edition, and is opt-in (empty by default — you choose what is sacred
  in your environment). This is the mechanism that makes the earlier
  bullet's "structurally guaranteed" language concrete for whatever
  severities you designate, not just a general fail-open promise.

## 4. Disclaimer of Warranties

SentinelReady is provided "AS IS" and "AS AVAILABLE," without warranty
of any kind, express or implied, including without limitation warranties
of merchantability, fitness for a particular purpose, and
non-infringement. We do not warrant that the software will be
uninterrupted, error-free, or that it will correctly triage, suppress,
or escalate any particular alert.

## 5. Limitation of Liability

To the maximum extent permitted by applicable law, and without limiting
Section 3:

- SentinelReady, LLC will not be liable for any indirect, incidental,
  special, consequential, exemplary, or punitive damages, or for any
  loss of profits, revenue, data, goodwill, or business interruption,
  arising out of or relating to your use of the software, whether based
  in contract, tort, negligence, strict liability, or any other theory,
  and whether or not we have been advised of the possibility of such
  damages.
- In particular, and consistent with Section 3, SentinelReady is not
  liable for missed alerts, delayed alerts, downtime, or incidents
  occurring on your infrastructure. Your monitoring strategy, your
  escalation policy, and the accuracy of your SentinelReady
  configuration remain your responsibility.
- Our total aggregate liability for all claims arising out of or
  relating to the software or these Terms will not exceed the greater of
  (a) the amounts you actually paid us for the software in the twelve
  months immediately preceding the event giving rise to the claim, or
  (b) one hundred U.S. dollars ($100). Because Community is provided at
  no charge, our aggregate liability to a Community user will not exceed
  one hundred U.S. dollars ($100).

Some jurisdictions do not allow the exclusion or limitation of certain
damages, so some of the above may not apply to you. In that case our
liability is limited to the greatest extent permitted by law.

## 6. Data Ownership

All alert data, behavior patterns, and configuration processed by your
SentinelReady instance belongs to you. SentinelReady (the company) does
not have access to this data — it lives entirely on your own
infrastructure (SQLite databases, local filesystem) and is never
transmitted to SentinelReady's servers. This local storage is
**unencrypted** — you are responsible for disk/volume-level encryption
and filesystem access controls if your alert data may contain sensitive
information. See the Privacy Policy for what limited data *is* collected
(license/registration contact info) and the Third-Party AI Providers
section for data that leaves your infrastructure by your own
configuration choice.

## 7. Termination

You may stop using SentinelReady at any time, and may cancel a paid
subscription as described in Section 2.

We may suspend or revoke a paid license key if a subscription payment
fails and is not resolved, if a subscription is cancelled or refunded,
or if the license is used in a way that materially breaches these Terms.
Where a revocation is for non-payment or breach rather than a completed
refund, we will make a reasonable effort to contact you at the address
associated with the license before revoking.

One fact holds regardless: revoking a paid license does **not** disable
your self-hosted instance. It falls back to Community automatically
(the same fail-open behavior as an expired key, Section 2). There is no
remote kill-switch, and no ability on our part to stop your instance
from processing alerts.

## 8. Governing Law

These Terms are governed by the laws of the State of Georgia, without
regard to conflict-of-law principles. SentinelReady, LLC is organized
under the laws of the State of Georgia.

## 9. Changes to These Terms

We may update these Terms from time to time. When we do, we will change
the version and effective date at the top of this document and publish
the updated Terms at the same location as this one.

For material changes affecting paid subscribers, we will additionally
give notice by email to the address associated with your license at
least thirty (30) days before the change takes effect for your next
billing period. Continuing to use SentinelReady after the effective date
means you accept the updated Terms; if you do not accept them, you may
cancel your subscription as described in Section 2.

## 10. Contact

Questions about these Terms: **jeff@sentinelready.io**
