# SentinelReady — Release Notes

What changed in each release, and whether you need to do anything about it.
Upgrade with `docker compose pull && docker compose up -d`.

---

## 1.0.5 — 2026-09-02

**Upgrade if you are on a paid plan.**

- **Fixed: a false "license renewal has not landed" warning.** Every sitrep
  and the dashboard warned that check-ins were not reaching the licensing
  service, on instances where they were working perfectly. Subscription
  entitlements are short-lived and refresh automatically, so the old check —
  "expires within 7 days" — was true from the moment a licence was issued.
  It now warns only when a refresh has genuinely not succeeded for 24 hours.
  If you saw this warning, nothing was ever wrong with your licence.

- **Fixed: repeat alerts were re-analysed instead of served from the pattern
  library.** The AI was being asked whether an alert "requires review" without
  being told what the question meant, and reasonably answered "yes, a human
  should look at this incident" — which the library treated as "never reuse
  this verdict". Recognised patterns now serve from the library as intended.
  Existing patterns correct themselves the next time they fire; nothing to do.

- **Fixed:** the sitrep preview endpoint built a report for the wrong customer
  code when called without one, showing zero alerts on a busy instance.

## 1.0.4 — 2026-09-02

**Upgrade immediately if you are on a paid plan. 1.0.3 cannot activate a
licence at all.**

- **Fixed: a paid licence could not activate.** On startup with a licence key
  present, the container failed and restarted in a loop, never reaching Pro.
  Community installs were unaffected — the fault was on the licence path only.

## 1.0.3 — 2026-09-01

- Burst handling reworked. A pattern firing far above its usual rate no longer
  discards what has been learned about it and re-analyses from scratch. The
  protection is unchanged: an alert that would normally be suppressed is
  surfaced in the digest while it is bursting, so a known-noisy pattern firing
  abnormally never goes silent.
- Run against a hosted AI provider without a local model: see
  `docker-compose.no-ollama.yml`.
- Pointing at an existing Ollama on your network now actually works — the
  override set a variable the application did not read.

## 1.0.2 — 2026-08-31

- Reliability fixes to log collection and AI provider handling.

---

Full source and issues: https://github.com/sentinelready/sentinelready-docker
