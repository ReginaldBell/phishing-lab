# Sprint 5 And 6 Handoff

Updated: `2026-03-14`

## Purpose

This document is the historical handoff package that was used to complete Sprint 5 and Sprint 6. It is retained to show the exact grounded context that was passed forward and to reduce hallucination risk when reviewing how the final documentation was produced.

Use this file as a reference brief, not as a live task list.

## Current Project State

Project status reflected by this handoff context:

- Sprint 1: `Completed`
- Sprint 2: `Completed`
- Sprint 3: `Completed`
- Sprint 4: `Completed`
- Sprint 5: `Completed`
- Sprint 6: `Completed`

At the time this brief was created, the remaining work was documentation and closeout. That work has now been completed in:

- [docs/SPRINT5_INCIDENT_RESPONSE.md](/c:/Users/demar/Downloads/Phishing-Lab/docs/SPRINT5_INCIDENT_RESPONSE.md)
- [docs/SPRINT6_PROJECT_CLOSEOUT.md](/c:/Users/demar/Downloads/Phishing-Lab/docs/SPRINT6_PROJECT_CLOSEOUT.md)

## What Has Been Proven

The phishing lab has been validated end to end across:

- `gophish01`
- `mail01`
- `wkstn01`
- `siem01`

Validated Sprint 4 live incident path:

- Campaign: `9`
- Campaign name: `Sprint 4 Post-Restart Validation 2026-03-14`
- RID: `pl4qC1k`
- Recipient: `labuser@example.local`
- Workstation IP: `192.168.56.20`

Validated event sequence:

1. GoPhish campaign created at `2026-03-14T16:47:57.97378581Z`
2. GoPhish `Email Sent` at `2026-03-14T16:47:58.118515047Z`
3. `mail01` Dovecot mailbox delivery at `2026-03-14 16:47:54.083441+00:00`
4. `wkstn01` clicked `GET /?rid=pl4qC1k` from `192.168.56.20` at `2026-03-14 16:48:33 UTC`
5. GoPhish recorded `Clicked Link` for RID `pl4qC1k`
6. Wazuh raised:
   - rule `100100` at `2026-03-14T16:47:59.210+0000`
   - rule `100102` at `2026-03-14T16:47:59.578+0000`
   - rule `100104` at `2026-03-14T16:48:35.054+0000`

## Key Technical Conclusion

The critical live-correlation fix was updating Wazuh rule `100104` to use:

- `frequency="2"`
- `timeframe="1800"`
- `<global_frequency />`

This was required because cross-agent correlation between `mail01` and `gophish01` did not persist reliably in live analysis until `global_frequency` was added.

## Current Detection Behavior

The repo now contains working phishing-specific Wazuh content.

Current rule behavior:

- `100100`: GoPhish email send
- `100101`: GoPhish RID click
- `100102`: Dovecot mailbox delivery
- `100103`: Dovecot IMAP login from `wkstn01`
- `100104`: correlated phishing scenario when mailbox delivery is followed by a real RID click

Important design note:

- Sprint 4 correlation is intentionally based on `delivery + click`
- IMAP login is retained as supporting evidence, not a hard prerequisite

## Canonical Sources Of Truth

Use these docs first before editing anything else:

- [docs/PROJECT_STATUS.md](/c:/Users/demar/Downloads/Phishing-Lab/docs/PROJECT_STATUS.md)
- [docs/SPRINT_PROGRESS_INTERNAL_REVIEW.md](/c:/Users/demar/Downloads/Phishing-Lab/docs/SPRINT_PROGRESS_INTERNAL_REVIEW.md)
- [docs/SPRINT3_DETECTION_ENGINEERING.md](/c:/Users/demar/Downloads/Phishing-Lab/docs/SPRINT3_DETECTION_ENGINEERING.md)
- [docs/SPRINT_PLAN.md](/c:/Users/demar/Downloads/Phishing-Lab/docs/SPRINT_PLAN.md)
- [docs/PROJECT_PHASE_OVERVIEW.md](/c:/Users/demar/Downloads/Phishing-Lab/docs/PROJECT_PHASE_OVERVIEW.md)

Repo files relevant to the validated detection logic:

- [wazuh/rules/phishing_lab_rules.xml](/c:/Users/demar/Downloads/Phishing-Lab/wazuh/rules/phishing_lab_rules.xml)
- [wazuh/agents/mail01-ossec.conf](/c:/Users/demar/Downloads/Phishing-Lab/wazuh/agents/mail01-ossec.conf)
- [wazuh/agents/gophish01-ossec.conf](/c:/Users/demar/Downloads/Phishing-Lab/wazuh/agents/gophish01-ossec.conf)

## Evidence Files Already Stored In The Repo

Sprint 4 screenshots:

- [docs/assets/screenshots/siem01-sprint4-click-100101-2026-03-14.png](/c:/Users/demar/Downloads/Phishing-Lab/docs/assets/screenshots/siem01-sprint4-click-100101-2026-03-14.png)
- [docs/assets/screenshots/siem01-sprint4-delivery-100102-2026-03-14.png](/c:/Users/demar/Downloads/Phishing-Lab/docs/assets/screenshots/siem01-sprint4-delivery-100102-2026-03-14.png)
- [docs/assets/screenshots/siem01-sprint4-correlation-100104-2026-03-14.png](/c:/Users/demar/Downloads/Phishing-Lab/docs/assets/screenshots/siem01-sprint4-correlation-100104-2026-03-14.png)
- [docs/assets/screenshots/wkstn01-sprint4-thunderbird-email-2026-03-14.png](/c:/Users/demar/Downloads/Phishing-Lab/docs/assets/screenshots/wkstn01-sprint4-thunderbird-email-2026-03-14.png)
- [docs/assets/screenshots/wkstn01-sprint4-browser-rid-pl4qC1k-2026-03-14.png](/c:/Users/demar/Downloads/Phishing-Lab/docs/assets/screenshots/wkstn01-sprint4-browser-rid-pl4qC1k-2026-03-14.png)

Earlier validation screenshots remain available for comparison:

- [docs/assets/screenshots/wkstn01-sprint1-thunderbird-2026-03-11.png](/c:/Users/demar/Downloads/Phishing-Lab/docs/assets/screenshots/wkstn01-sprint1-thunderbird-2026-03-11.png)
- [docs/assets/screenshots/wkstn01-sprint1-browser-2026-03-11.png](/c:/Users/demar/Downloads/Phishing-Lab/docs/assets/screenshots/wkstn01-sprint1-browser-2026-03-11.png)
- [docs/assets/screenshots/siem01-sprint3-alerts-2026-03-11.png](/c:/Users/demar/Downloads/Phishing-Lab/docs/assets/screenshots/siem01-sprint3-alerts-2026-03-11.png)
- [docs/assets/screenshots/siem01-sprint3-full-validation-2026-03-11.png](/c:/Users/demar/Downloads/Phishing-Lab/docs/assets/screenshots/siem01-sprint3-full-validation-2026-03-11.png)

## Environment Reference

Core hosts:

- `gophish01`: `192.168.56.60`
- `mail01`: `192.168.56.70`
- `siem01`: `192.168.56.103`
- `wkstn01`: `192.168.56.20`

Important service notes:

- GoPhish admin UI: `https://192.168.56.60:3333`
- GoPhish phishing web server: `http://192.168.56.60`
- Wazuh dashboard: `https://192.168.56.103/`

## Snapshot Context

Baseline snapshot information:

- Snapshot name: `baseline-2026-03-11-internal-review`
- Snapshot created for:
  - `dc01`
  - `dc02`
  - `siem01`
  - `wkstn01`
  - `gophish01`
  - `mail01`

This matters only as environment recovery context. Sprint 5 and Sprint 6 were documentation-driven and did not require restoring or redesigning the lab.

## Sprint 5 Instructions

Status: `Completed`

Original objective:

- Produce the incident response deliverable using the validated March 14 phishing incident path.

Required content for Sprint 5:

- Incident summary
- Scope of affected systems
- Detection summary
- Triage actions
- Containment actions
- Investigation workflow
- Eradication and recovery actions
- Lessons learned
- Recommended improvements
- References to actual evidence, rule IDs, hosts, timestamps, and screenshots

Required grounding rules:

- Do not invent a credential-harvest step. The landing page explicitly states that no credentials were collected.
- Do not claim malware execution, lateral movement, privilege escalation, or persistence. None of that has been validated.
- Keep the incident scoped to phishing delivery and link click detection.
- Use exact dates and timestamps from the validated March 14 run.
- Reference the Wazuh rule IDs exactly as implemented.

Completed output:

- [docs/SPRINT5_INCIDENT_RESPONSE.md](/c:/Users/demar/Downloads/Phishing-Lab/docs/SPRINT5_INCIDENT_RESPONSE.md)

Suggested Sprint 5 structure:

1. Executive Summary
2. Incident Overview
3. Timeline
4. Detection And Evidence
5. Triage
6. Containment
7. Investigation
8. Recovery
9. Lessons Learned
10. Recommended Improvements
11. Appendix: Evidence References

## Sprint 6 Instructions

Status: `Completed`

Original objective:

- Normalize the repository so it clearly reflects the completed project state.

Required Sprint 6 tasks:

- Update status documents so they consistently reflect Sprint 4 completion and Sprint 5 completion once written
- Add links to the Sprint 5 incident response deliverable
- Verify that screenshot links and file paths resolve correctly
- Confirm the repo narrative is internally consistent across status, overview, and sprint docs
- Produce a final closeout summary

Completed output:

- [docs/SPRINT6_PROJECT_CLOSEOUT.md](/c:/Users/demar/Downloads/Phishing-Lab/docs/SPRINT6_PROJECT_CLOSEOUT.md)
- or update [docs/SESSION_REPORT.md](/c:/Users/demar/Downloads/Phishing-Lab/docs/SESSION_REPORT.md) if you choose to use it as the closeout narrative

Suggested Sprint 6 structure:

1. Final Project Outcome
2. Completed Sprint Summary
3. Final Evidence Inventory
4. Remaining Limitations
5. Recommended Future Improvements
6. Repo Finalization Notes

## Do Not Re-Open These Questions Unless Evidence Changes

- Sprint 4 live correlation is complete
- Rule `100104` fired successfully in live `alerts.json`
- The current phishing use case is `delivery + click`
- `/var/log/syslog` is the canonical Dovecot source on `mail01`
- Duplicate `mail.log` ingestion was intentionally removed from the repo-side phishing config

## Known Limitations That Are Real

- Rule `100100` still depends on the decoder Wazuh actually assigns to GoPhish email-send lines: `aws-eks-authenticator`
- The project proves phishing delivery and click detection, not a full credential theft scenario
- The current work is lab validation and documentation, not production hardening

## Recommended Working Approach Used

1. Read [docs/PROJECT_STATUS.md](/c:/Users/demar/Downloads/Phishing-Lab/docs/PROJECT_STATUS.md), [docs/SPRINT_PROGRESS_INTERNAL_REVIEW.md](/c:/Users/demar/Downloads/Phishing-Lab/docs/SPRINT_PROGRESS_INTERNAL_REVIEW.md), and [docs/SPRINT3_DETECTION_ENGINEERING.md](/c:/Users/demar/Downloads/Phishing-Lab/docs/SPRINT3_DETECTION_ENGINEERING.md) first.
2. Write Sprint 5 from the validated March 14 evidence chain, not from generic phishing templates.
3. Update the existing docs only after the Sprint 5 deliverable exists.
4. Finish Sprint 6 by making the repo tell one consistent final story.

## Bottom Line

This handoff has been fulfilled. The project is now complete, and the validated March 14 incident path has been carried through the Sprint 5 incident response deliverable and Sprint 6 closeout documentation.
