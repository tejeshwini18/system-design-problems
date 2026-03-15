# Interview Gap Implementation Tracker

This document closes the gaps identified in `INTERVIEW_APPROACH_REVIEW.md` and records how they are implemented in this repo.

## Gap → Implementation

| Gap from review | What is now implemented | Where |
|---|---|---|
| Explicit capacity estimates | Mandatory back-of-envelope section (QPS split, storage/day, latency budget). | `SYSTEM_DESIGN_TEMPLATE.md` + `INTERVIEW_READY_TRANSCRIPTS.md` |
| SLO/SLA and error budget framing | Required NFR targets and SLO-driven alerting guidance. | `SYSTEM_DESIGN_TEMPLATE.md` + `INTERVIEW_READY_TRANSCRIPTS.md` |
| Read/write path clarity | Dedicated write path + read path callouts and hotspot watchpoints. | `SYSTEM_DESIGN_TEMPLATE.md` + `INTERVIEW_READY_TRANSCRIPTS.md` |
| Failure-mode walkthroughs | Standardized dependency, queue, and datastore failure handling checklist. | `SYSTEM_DESIGN_TEMPLATE.md` + `INTERVIEW_READY_TRANSCRIPTS.md` |
| Explicit trade-off analysis | Mandatory trade-off table and per-problem trade-off prompts. | `SYSTEM_DESIGN_TEMPLATE.md` + `INTERVIEW_READY_TRANSCRIPTS.md` |
| Security/compliance baseline | Required AuthN/AuthZ, encryption, secrets, PII/audit requirements. | `SYSTEM_DESIGN_TEMPLATE.md` + `INTERVIEW_READY_TRANSCRIPTS.md` |
| Operational readiness | Golden signals, SLO alerts, runbooks, canary/rollback, DR (RTO/RPO). | `SYSTEM_DESIGN_TEMPLATE.md` + `INTERVIEW_READY_TRANSCRIPTS.md` |
| Cost-awareness | Cost driver checklist (compute, storage tiering, egress, cache hit ratio). | `SYSTEM_DESIGN_TEMPLATE.md` + `INTERVIEW_READY_TRANSCRIPTS.md` |

## Practical Usage

- For any existing problem folder, use the upgraded template checklist before finalizing the HLD/LLD write-up.
- For interview practice, use the corresponding section in `INTERVIEW_READY_TRANSCRIPTS.md` and speak through each mandatory section in order.
- For consistent terminology, use `SYSTEM_DESIGN_GLOSSARY.md` while presenting.
