# Phishing Triage

Automates first-pass triage of phishing reports so an analyst only sees the
ones that actually need a human decision.

## Trigger

An incident is created from either:
- A user clicking "Report Phishing" in their mail client
- The abuse mailbox ingesting a forwarded email

## Logic

```
Extract indicators (sender domain, URLs, attachment hashes)
        │
        ▼
Known bad/good already? ──yes──► fast path (skip enrichment)
        │ no
        ▼
Enrich via threat intel (VirusTotal, URLhaus, AbuseIPDB)
        │
        ▼
Detonate attachment/URL in sandbox (if applicable)
        │
        ▼
Score verdict: malicious / suspicious / benign
        │
   ┌────┼────┐
   ▼    ▼    ▼
malicious  suspicious  benign
   │         │           │
   ▼         ▼           ▼
Block +   Escalate    Auto-close
contain   to analyst
   │         │           │
   └─────────┴───────────┘
             ▼
    Notify reporting user
```

## Why it's shaped this way

- **Known bad/good short-circuit before enrichment** — no point burning
  API calls on threat-intel lookups for an indicator already in the
  blocklist; also cuts triage time for the highest-confidence cases.
- **Detonation only runs when needed** — most reported phishing is a
  credential-harvesting link or a malicious attachment, but a chunk are
  benign marketing/spam that threat intel alone can already classify.
  Sandboxing is comparatively slow and expensive, so it's gated behind
  the "still unknown after intel enrichment" branch.
- **Suspicious always goes to a human** — the scoring thresholds are
  deliberately conservative. A false auto-close on a real phish is much
  worse than an analyst spending two minutes on a false positive, so
  ambiguous cases never auto-resolve.
- **Blocklisting happens before containment, not after** — pushing the
  indicator to the email gateway/EDR first means the org-wide mailbox
  search for "who else got this" and the quarantine step are working
  against a system that's already blocking re-delivery.
- **Every branch ends in a reply to the reporting user** — closing the
  loop is what keeps a user-report phishing program alive; if reporting
  something never gets acknowledged, people stop reporting.

## What this catches vs. misses

Catches: known-bad infrastructure reuse, mass-phish campaigns already
fingerprinted by threat intel, malicious attachments a sandbox detonates
cleanly.

Misses: novel infrastructure with no threat-intel footprint and a sandbox
that doesn't trigger (e.g., a phishing page that only renders for a
specific target, or waits out automated analysis). Those rely on the
"suspicious → analyst" branch rather than automated detection.
