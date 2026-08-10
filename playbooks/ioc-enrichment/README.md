# IOC Enrichment

Reusable sub-playbook: takes a list of indicators, returns an aggregated
reputation score. Called by other playbooks instead of each one
re-implementing threat-intel lookups.

## Trigger

Not incident-driven — invoked by a parent playbook (currently
[Phishing Triage](../phishing-triage/), and referenced by the planned
Alert Escalation playbook) with a list of indicators as input.

## Logic

```
Indicators in
     │
     ▼
Cache hit? ──yes──► Return cached result
     │ no
     ▼
Query VirusTotal (hashes, URLs)
     │
     ▼
Query URLhaus / AbuseIPDB (domains, IPs)
     │
     ▼
Query internal telemetry (SIEM/EDR, last 30 days)
     │
     ▼
Aggregate weighted score, write to cache
     │
     ▼
Return score + source detail
```

## Why it's shaped this way

- **Cache-first**: threat-intel APIs are rate-limited, and the same
  indicators show up repeatedly across incidents (mass-phish campaigns
  reuse infrastructure). A 24h cache avoids burning API quota re-querying
  an indicator that was already scored an hour ago.
- **Multiple sources, weighted**: no single source is reliable enough
  alone — VirusTotal can lag on brand-new infrastructure, URLhaus/AbuseIPDB
  cover different indicator types, and vendor false-positive rates vary.
  Weighting multi-source agreement higher than a single low-confidence
  hit reduces both false positives and false negatives versus trusting
  any one source.
- **Internal telemetry is a first-class input, not an afterthought**: an
  indicator's external reputation doesn't tell you blast radius. Checking
  internal prevalence (how many hosts/mailboxes actually saw it) is what
  lets a calling playbook decide between "quiet single-target block" and
  "this is spreading, escalate now."
- **Built as a sub-playbook, not copy-pasted logic**: enrichment is the
  one piece of logic almost every SOAR playbook needs. Centralizing it
  means a change to scoring weights or a new intel source propagates to
  every playbook that calls it, instead of drifting out of sync across
  copies.

## What this doesn't do

No verdict, no containment action — this playbook only scores and
returns data. The calling playbook (e.g. Phishing Triage) owns the
decision of what to do with the score. Keeping enrichment and
decision-making separate is what makes this reusable across playbook
types with very different response logic.
