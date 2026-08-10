# Alert Escalation

Routes SIEM correlation-rule alerts to the right responder based on
severity and asset criticality, and pages up the chain if nobody
acknowledges in time.

## Trigger

A SIEM correlation rule fires and creates an incident. Unlike
[Phishing Triage](../phishing-triage/), there's no reporting user — the
alert comes from detection logic, so the playbook has to derive priority
from context (what fired, on what asset) rather than a person's judgment
call.

## Logic

```
Alert created
     │
     ▼
Look up asset criticality (CMDB/asset inventory)
     │
     ▼
Enrich related indicators (calls IOC Enrichment, if any attached)
     │
     ▼
Compute effective priority
 (severity + asset criticality + indicator reputation)
     │
   ┌─┼──────────┐
   ▼ ▼          ▼
critical  high   medium/below
   │       │          │
   ▼       ▼          ▼
Page      SOC queue  SOC queue
on-call   (1h SLA)   (4h SLA)
(15m SLA)
   │       │          │
   └───────┴──────────┘
           ▼
   Monitor for ack ──breached──► Escalate to next tier
           │                            │
      acknowledged                (restarts ack timer,
           │                       loops back to monitor)
           ▼
    Track to resolution
```

## Why it's shaped this way

- **Asset criticality can override raw severity**: a "medium" alert on a
  domain controller is escalated to high. A correlation rule's default
  severity is set at write-time and can't account for which specific
  asset it fires on later — the playbook corrects for that at runtime.
- **Unknown assets default to tier 2, not tier 3**: an asset that isn't
  in the CMDB is itself worth noticing (shadow IT, missed onboarding,
  or a gap in asset discovery), so it shouldn't get the lowest-priority
  treatment by default.
- **Indicator reputation can de-escalate but has a floor**: if attached
  indicators come back benign, priority can drop — but never below
  medium. The correlation rule firing is itself a signal; a clean
  indicator lookup doesn't mean the underlying behavior wasn't
  suspicious, just that this one angle didn't confirm it.
- **SLA timers scale with priority, and breaches escalate rather than
  silently expire**: a critical alert getting paged and then sitting
  un-acknowledged is worse than the original alert in some cases. The
  escalate-and-restart-timer loop (task 9 back to task 7) means a
  non-response keeps climbing the chain instead of stalling.
- **Reuses IOC Enrichment rather than re-implementing lookups**: this
  playbook and Phishing Triage need the same enrichment logic for
  different trigger types — see [IOC Enrichment](../ioc-enrichment/) for
  why that's factored out.

## What this doesn't do

No containment action — this playbook is purely about getting the alert
to the right person at the right urgency. Containment logic (blocking,
isolating, quarantining) lives in the playbooks that handle a specific
alert type once a human or a more specialized playbook takes it from
here.
