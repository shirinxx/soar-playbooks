# soar-playbooks

SOAR incident-response playbooks written and maintained as a working SOC
automation practice repo. Each playbook documents not just the task flow
but the reasoning behind it — why it branches where it branches, what it
catches, and what it deliberately leaves to a human.

## Why this repo exists

A playbook diagram on its own doesn't tell you much. What matters is the
judgment behind it: which steps are safe to automate, which need a human
in the loop, and where the failure modes are. This repo is the working
notebook for that judgment, built from real SOC automation practice and
sanitized for public use — no org-specific integrations, instance names,
or internal identifiers.

## Structure

```
soar-playbooks/
├── README.md
└── playbooks/
    └── phishing-triage/
        ├── playbook.yml   # task flow, branching logic, inputs/outputs
        └── README.md      # trigger → logic → output, with rationale
```

Each playbook is represented as a portable flow spec (`playbook.yml`) —
trigger, tasks, branching conditions, inputs/outputs — rather than a raw
platform export, since a raw export is mostly platform-specific noise
(canvas coordinates, integration instance IDs) that would be stripped for
public use anyway. The logic maps directly onto Cortex XSOAR-style
playbook concepts (tasks, `nexttasks`, conditional branches).

## Current playbooks (1)

| Playbook | Trigger | Handles |
|---|---|---|
| [Phishing Triage](playbooks/phishing-triage/) | User report / abuse mailbox | Indicator extraction, enrichment, sandbox detonation, verdict scoring, auto-close or analyst escalation |

## Roadmap

- **IOC enrichment** — standalone enrichment sub-playbook (threat intel +
  internal telemetry lookup) that other playbooks call, instead of each
  playbook re-implementing enrichment logic
- **Alert escalation** — severity-based routing/paging playbook for
  SIEM-generated alerts that don't originate from a user report
- `tests/` with sample trigger payloads per playbook

## About

Shirin Shukurov — Senior Cybersecurity Engineer (SIEM/XDR/SOAR, SOC operations,
threat intelligence). Built while relocating for cyber security roles in the EU.
