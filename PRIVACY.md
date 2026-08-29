# SentinelReady — Privacy Policy

**Version 1.0 — effective 2026-08-29.**

---

## The short version

SentinelReady is self-hosted software. It runs on infrastructure you
control. The software does not phone home, does not send telemetry to
SentinelReady, and does not give SentinelReady access to your alert
data, behavior patterns, or configuration. The only data SentinelReady,
LLC actually receives is what you send us directly — a purchase or
license registration, or a support request.

## What data SentinelReady (the company) collects

- **License and purchase information**: when you buy a subscription, our
  payment processor (Lemon Squeezy) collects your name, email address,
  billing details, and payment information in order to process the
  transaction. We receive your name, email address, and subscription
  status from them; **we never receive or store your full payment card
  details.** Lemon Squeezy's handling of that data is governed by their
  own privacy policy. We use what we receive to issue and renew your
  license key and to contact you about your subscription.
- **Support requests**: if you email us at
  **support@sentinelready.io**, we receive whatever you send — which may
  include diagnostic information you choose to share, such as logs or
  configuration snippets. Please redact anything sensitive before
  sending it; we do not need credentials to help you.
- **Nothing else, automatically.** SentinelReady contains no analytics,
  crash reporting, or usage telemetry that calls back to our
  infrastructure. Every outbound network call the software makes is to a
  destination *you* configured — there is no hidden default endpoint.
  The one exception is documented and narrow: instances running a paid
  subscription periodically contact our license service to renew their
  entitlement. That check-in carries the license key and an instance
  identifier only — never alert data, patterns, or configuration.

### How long we keep it

We retain license and purchase records for as long as your subscription
is active, and afterwards for as long as required to meet tax,
accounting, and legal obligations. We retain support correspondence for
up to twenty-four (24) months, after which it is deleted.

We do **not** sell your information, and we do **not** use it for
marketing beyond communication about the product you have purchased or
registered for. If we ever want to send you anything broader than that,
we will ask you to opt in first.

## What data stays entirely on your infrastructure

Everything SentinelReady processes as part of its actual function stays
local to your deployment:

- Alert data received via your configured webhook sources (Grafana,
  Datadog, CloudWatch, Slack, UDM, generic)
- Learned behavior patterns, fingerprint/dedup history, outcome-tracking
  data — stored in local SQLite databases on your infrastructure
- Configuration (`sentinelready.yaml`), including your webhook secret
  and any API keys you've supplied

None of this is transmitted to SentinelReady. We have no access to it
and no mechanism to retrieve it.

**Storage format**: the local SQLite databases and `events.jsonl` history
file above are stored **unencrypted (plaintext)** on disk. This is
standard for self-hosted software of this kind, but it means anyone with
filesystem access to your deployment (root/admin access to the host, or
access to the Docker volume) can read alert content, learned patterns,
and configuration values directly. Encrypt the underlying disk/volume if
your alert data may contain sensitive information and your environment
doesn't already provide disk-level encryption.

## Third-Party AI Providers — read this if you've configured one

SentinelReady ships with **Ollama** as the default AI provider — this
runs entirely on your own infrastructure (or infrastructure you control)
and sends no data anywhere external.

If you instead configure **Claude, OpenAI, or any OpenAI-compatible
provider** (Azure OpenAI, Groq, Together.ai, OpenRouter, etc.) — a
choice available in **every edition, not gated by license tier** — alert
data included in the enrichment prompt (service names, error messages,
alert descriptions, and other operational context) is sent to that
provider using **your own API key**. That provider bills you directly;
SentinelReady never sees or proxies that cost, and is not a party to
that data transmission. What that provider does with the data it
receives is governed by **that provider's own privacy policy and
terms**, not this one — review Anthropic's, OpenAI's, or your chosen
provider's data handling policy before configuring this, particularly if
your alert data could contain sensitive information.

This disclosure applies regardless of tier — it depends entirely on
which provider *your specific installation* is configured to use.

## Cookies and website tracking

**The SentinelReady website sets no cookies, runs no analytics, and
loads no third-party scripts or trackers.** There is nothing to opt out
of, because nothing is collected.

Purchases are handled by Lemon Squeezy on their own checkout pages;
cookies and tracking on those pages are governed by their privacy
policy, not this one.

## Your rights

Because SentinelReady collects so little, your rights here are simple to
exercise. Depending on where you live — including under the GDPR in the
EU/UK, and the CCPA/CPRA in California — you may have the right to:

- **access** the personal information we hold about you (your name,
  email address, and subscription record);
- **correct** it if it is inaccurate;
- **delete** it, subject to records we are legally required to retain
  for tax and accounting purposes;
- **obtain a copy** of it in a portable format;
- **object to or restrict** how we use it.

To exercise any of these, email **support@sentinelready.io**. We will
respond within thirty (30) days. We will not discriminate against you
for making a request.

We do not sell personal information, and we do not share it for
cross-context behavioural advertising.

Note that your alert data, patterns, and configuration never reach us at
all — those live only on your own infrastructure, so a request to us
cannot reach them, and deleting them is entirely within your control.

## Changes to this policy

We may update this policy from time to time. When we do, we will change
the version and effective date at the top of this document and publish
the updated policy at the same location as this one. For material
changes, we will additionally give notice by email to the address
associated with your license at least thirty (30) days before the change
takes effect.

## Contact

Questions about this policy, or to exercise any of the rights above:
**support@sentinelready.io**

SentinelReady, LLC — organized under the laws of the State of Georgia.
