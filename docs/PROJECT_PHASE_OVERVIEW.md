# Project Phase Overview

Updated: `2026-03-14`

This document provides a structured overview of the `Phishing-Lab` project by phase. All six phases are now complete. Phases 1 through 4 contain the core execution history, the major technical challenges, and the validated detection path. Phases 5 and 6 cover the incident response deliverable and final repo closeout.

## Phase 1: Baseline Validation

### Problems Faced

- The baseline lab could not be trusted without a fresh validation run, even though earlier project notes suggested the environment had worked previously.
- `gophish01` initially appeared slow or stalled during boot, which created uncertainty about whether the issue was a real service failure or only delayed startup.
- A defensible proof path was required. It was not enough to show that a campaign existed; the project needed evidence that mail was delivered, opened, and clicked from the expected workstation.
- The validation had to account for multiple participating systems at once: `gophish01`, `mail01`, `wkstn01`, and `siem01`.

### Solutions Implemented

- The environment was revalidated with a fresh March 11, 2026 campaign rather than relying on historical assumptions.
- `gophish01` was allowed to complete boot and was rechecked for service reachability before declaring a fault.
- Evidence was collected from GoPhish, Dovecot, Thunderbird, and Wazuh agent status.
- Screenshots were added to the repository to support the written validation record with user-visible proof.

### Plan Executed

- Start the core infrastructure, confirm GoPhish admin reachability, verify mail flow, and ensure Wazuh agents remained active.
- Deliver a controlled phishing email to `labuser@example.local`.
- Open the message on `wkstn01` and perform a manual click to validate a realistic user path.
- Record the event chain and preserve artifacts in the repo.

### Intricacies Pertaining to the Phase

- Two campaign styles were used: a manual UI-driven validation and a secondary automation-assisted comparison run. The manual run remained the stronger evidence path.
- The validation had to distinguish "system works" from "system works with human-observable proof."
- A key lesson from this phase was that baseline confirmation must include both infrastructure health and user-action evidence; otherwise later detection work rests on weak assumptions.

## Phase 2: Evidence And Timeline

### Problems Faced

- Evidence existed across different systems, formats, and timestamps, which made the project difficult to defend as a single coherent incident story.
- GoPhish, Dovecot, and Wazuh each represented the same workflow differently, so the team could not rely on one source alone.
- The presence of both a manual validation run and an automation-assisted run introduced a risk of mixing evidence from separate campaign paths.
- Without a normalized timeline, later incident-response and detection work would have lacked a clear reference chain.

### Solutions Implemented

- The project consolidated campaign creation, email delivery, click activity, IMAP access, and Wazuh agent health into one documented timeline.
- The primary user-driven campaign was clearly separated from the secondary comparison run to preserve evidentiary quality.
- Source-specific details were preserved instead of over-normalizing them away, which kept the timeline defensible.
- Screenshots and repo-stored artifacts were linked to the narrative so the written account could be corroborated visually.

### Plan Executed

- Collect technical evidence from GoPhish events and service logs.
- Collect Dovecot delivery and IMAP access logs from `mail01`.
- Verify agent health in Wazuh to show telemetry was present during the same time window.
- Align timestamps and order the workflow from campaign creation through link click.

### Intricacies Pertaining to the Phase

- The timeline showed that delivery, click, and mailbox access did not occur in one perfectly linear system view; each host saw a different slice of the incident.
- `GET /favicon.ico` appeared after the real click, which later became important during detection tuning.
- The major lesson from this phase was that correlation work must respect the reality of multi-host evidence rather than assume one log source tells the full story.

## Phase 3: Detection Engineering

### Problems Faced

- Wazuh was operational, but it initially produced only generic host activity and did not elevate the phishing scenario meaningfully.
- GoPhish log lines were not cleanly claimed by the intended custom decoder and were instead being interpreted through `aws-eks-authenticator`.
- `mail01` did not initially provide a reliable live delivery signal because `rsyslog` was missing and `/var/log/syslog` did not exist.
- The first click rule was too broad and incorrectly matched `GET /favicon.ico`, creating a false correlation path.
- Dovecot delivery alerts later duplicated because both `/var/log/mail.log` and `/var/log/syslog` carried overlapping evidence.

### Solutions Implemented

- Custom Wazuh rules, decoders, agent configurations, and deployment helpers were added to the repo.
- GoPhish collection on `gophish01` was corrected so `gophish.log` was treated as text instead of JSON.
- `rsyslog` was installed on `mail01`, after which Dovecot delivery activity became visible through `/var/log/syslog`.
- The click rule was narrowed to `GET /?rid=` to exclude favicon traffic.
- The repo-side `mail01` phishing configuration was simplified to one canonical Dovecot source: `/var/log/syslog`.

### Plan Executed

- Identify stable log patterns for email send, mailbox delivery, IMAP access, and phishing-link clicks.
- Write custom rules for those primitives and test them with both `wazuh-logtest` and live traffic where possible.
- Tune the rules iteratively as real data exposed false positives and ingestion gaps.
- Deploy the updated package to the manager and agents, then recheck manager health.

### Intricacies Pertaining to the Phase

- This phase proved that "telemetry exists" is not the same thing as "telemetry is usefully detectable."
- The decoder mismatch did not block all progress, but it forced pragmatic rule design rather than idealized decoder purity.
- A major lesson was that live testing is indispensable. Several issues, including favicon false positives and duplicate delivery alerts, only became obvious when the rules were exercised against real traffic.

## Phase 4: Correlation And Use Case Completion

### Problems Faced

- The original correlation rule `100104` was too loose. It relied on broad frequency logic and did not consistently fire for the clean delivery-plus-click path.
- Cross-host correlation remained the project's central unresolved problem even after primitive detections were working.
- The project needed a design that was strict enough to avoid false positives but practical enough to survive the actual data produced by the lab.
- The final live-proof requirement had not yet been satisfied when the phase began.

### Solutions Implemented

- Rule `100104` was redesigned so the final incident alert is anchored on a real click event and requires a prior delivery alert within the allowed time window.
- The supporting scripts were extended to test delivery-then-click correlation and inspect correlation alerts and archive paths more directly.
- The updated package was deployed to the running core VMs and the manager returned to a healthy `active` state.
- Static validation on March 14, 2026 confirmed that rules `100101`, `100102`, and `100104` behaved correctly in `wazuh-logtest`, including a click-only negative case.
- Live validation on March 14, 2026 then confirmed the full path in `alerts.json`, including the final correlated incident alert.

### Plan Executed

- Tighten the correlation design around a reliable `delivery + click` path rather than requiring IMAP login as a hard prerequisite.
- Keep IMAP activity as supporting evidence for investigation context.
- Deploy the revised rules and canonicalized `mail01` configuration.
- Validate primitive alerts first, then validate the multi-event correlation chain.

### Intricacies Pertaining to the Phase

- The design had to balance ideal fidelity with Wazuh's practical rule behavior and the real shape of the available fields.
- The click rule was ultimately made more reliable by reducing dependency on fragile decoder assumptions for the correlation path.
- The final live fix required `frequency="2"` plus `<global_frequency />` so the cross-host delivery-plus-click chain persisted in live analysis, not only in `wazuh-logtest`.
- The repo now includes a complete visual evidence set for this phase: `wkstn01` email and browser screenshots plus `siem01` screenshots for rules `100101`, `100102`, and `100104`.
- The key lesson from this phase is that a strong phishing use case depends on disciplined rule sequencing: stable primitives first, correlation second, and live campaign proof last.

## Phase 5: Incident Response Deliverables

Status: `Completed`

The Sprint 5 incident response deliverable was written from the validated March 14 evidence chain. The document covers executive summary, incident timeline, detection and evidence, triage, containment, investigation, recovery, lessons learned, and recommended improvements. All claims are anchored to exact timestamps, Wazuh rule IDs, and repo-stored screenshots. The scope is limited to phishing delivery and link click detection. No credential harvest, malware execution, or lateral movement is claimed.

Deliverable: [docs/SPRINT5_INCIDENT_RESPONSE.md](/c:/Users/demar/Downloads/Phishing-Lab/docs/SPRINT5_INCIDENT_RESPONSE.md)

## Phase 6: Final Repo And Closeout

Status: `Completed`

The repo was normalized and all status documents were updated to reflect the final completed project state. The Sprint 6 closeout document provides a final evidence inventory, a completed sprint summary, remaining limitations, and recommended future improvements.

Deliverable: [docs/SPRINT6_PROJECT_CLOSEOUT.md](/c:/Users/demar/Downloads/Phishing-Lab/docs/SPRINT6_PROJECT_CLOSEOUT.md)
