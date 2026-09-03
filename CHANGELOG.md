# SentinelReady — Release Notes

What changed in each release, and whether you need to do anything about it.
Upgrade with `docker compose pull && docker compose up -d`.

---

## 1.0.6 — 2026-09-03

**Recommended for everyone. Improves what SentinelReady tells you, and stops
it telling you things it could not actually know.**

- **Sitrep recommendations now come from the AI, on your measured data.**
  "Patterns Worth Fixing" used to be produced by fixed thresholds written into
  the code — they could only recognise situations we had thought of in
  advance. The AI now receives the real figures for each pattern (how often it
  fires, how long it takes to clear, how many times it was escalated, how long
  it has been known, its severity and blast radius) and decides what deserves
  engineering effort. It is also told what the data cannot tell it, so it does
  not reason past the evidence.

  Practically: a pattern that fires rarely but takes hours to clear can now be
  ranked above one that fires constantly and clears in seconds. No threshold
  could express that. And a pattern that has fired hundreds of times unchanged
  is treated as an unaddressed condition weighed against what it would affect,
  rather than as noise because it repeats.

- **Removed a claim we could not support.** Earlier sitreps said things like
  "self-resolves 100% of the time — consider suppressing permanently". That
  percentage was not measured. SentinelReady knows when an alert *cleared*,
  because your alerting system tells it — but an alert an engineer fixed and
  one that cleared by itself look identical from outside. It should never have
  reported the second as if it knew. It no longer claims a self-resolve rate,
  and no longer recommends silencing anything on that basis. The section now
  says where your volume is and suggests re-evaluating it, which is what the
  data actually supports.

  The same claim was being passed into the AI's own prompt, describing patterns
  as "known noise in this environment". That is removed too: the AI now judges
  from figures, not from a conclusion we handed it.

- **Triage no longer invents probable causes.** The AI had three slots for
  probable causes and no way to decline, so it filled them with plausible-
  sounding generalities when the alert supported none. It may now return a
  single cause, or say the alert lacks the signal to name one, and say what it
  would need instead.

- **Blast radius no longer names the monitoring endpoint.** For metric-derived
  alerts the AI sometimes reported the scrape target (e.g. a kubelet address)
  as the affected service — that is where the measurement came from, not what
  breaks.

- **Cosmetic:** numbers in AI prose round to one decimal place, and time
  estimates carry a unit.

**If you are on 1.0.5 or earlier with `outcome_learning: true` (the shipped
default in `sentinelready.yaml.example`), please upgrade.** In those versions
SentinelReady could promote a pattern to `suppress` on its own, using how
often alerts appeared to resolve themselves — a figure it was not actually
measuring. It never dropped an alert, and a suppressed pattern still appears
in your sitrep, but it could have stopped paging you for something that
warranted it. Upgrading disables that path. Any pattern already changed this
way corrects itself the next time it fires and is re-triaged; you can also
force it immediately with `POST /patterns/invalidate`.

**Note on model choice.** Two of these improvements are instructions to the AI,
and how well they are followed depends on the model. A small local model may
still produce a vaguer answer than the instruction asks for. If triage quality
matters to you, a larger local model or a hosted provider (`ai.provider:
claude` or `openai_compatible`) will follow them more closely.

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
