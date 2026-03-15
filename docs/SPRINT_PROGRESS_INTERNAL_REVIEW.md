# Sprint Progress Internal Review

Updated: `2026-03-14`

## Sprint 1: Baseline Validation

Status: `Completed`

Problems faced:

- `gophish01` initially appeared stalled during boot and delayed baseline checks.
- Fresh validation needed both system health confirmation and a defensible user-click proof path.

Fixes implemented:

- Waited through full guest boot and revalidated `gophish01` service reachability.
- Re-ran the phishing workflow and captured fresh GoPhish, mail, and Wazuh agent evidence.
- Added repo-stored screenshots showing Thunderbird mailbox access and the landing-page result on `wkstn01`.

Result:

- The lab baseline was confirmed working end to end.
- GoPhish recorded a manual click from `192.168.56.20`.
- `WKSTN01`, `mail01`, and `gophish01` remained active in Wazuh.

## Sprint 2: Evidence And Timeline

Status: `Completed`

Problems faced:

- Evidence existed across multiple systems with different timestamps and formats.
- The project needed one defensible timeline for internal review and later incident-response work.

Fixes implemented:

- Consolidated GoPhish events, Dovecot delivery activity, IMAP access, and Wazuh agent health into a single timeline.
- Distinguished the primary manual validation run from the secondary automation-assisted comparison run.
- Stored visual evidence in the repo to support the written timeline.

Result:

- The phishing scenario is now documented as a coherent event chain from campaign creation through user click.
- Sprint 2 established the evidence package needed for detection engineering.

## Sprint 3: Detection Engineering

Status: `Completed`

Problems faced:

- GoPhish log lines were being decoded by Wazuh as `aws-eks-authenticator`, not by the intended custom decoder.
- `mail01` was not producing usable live delivery alerts at first because `rsyslog` was not installed and `/var/log/syslog` did not exist.
- An early click rule matched `GET /favicon.ico`, which caused a false correlation hit.
- Delivery alerts duplicated because Dovecot was ingested from both `/var/log/mail.log` and `/var/log/syslog`.
- Correlation rule `100104` originally relied on a loose two-hit frequency design that did not hold up for the clean delivery-plus-click path.

Fixes implemented:

- Added custom Wazuh rules, decoders, agent configs, and deployment helpers for GoPhish and Dovecot coverage.
- Tuned the click rule to require `GET /?rid=` so favicon requests no longer count as phishing clicks.
- Installed `rsyslog` on `mail01`, restarted the Wazuh agent, and validated that Dovecot LMTP saves reached both `mail.log` and `syslog`.
- Reduced the repo-side `mail01` phishing config to one canonical Dovecot source: `/var/log/syslog`.
- Redesigned rule `100104` so the final incident alert is anchored on a real GoPhish click with a prior mailbox-delivery alert in the time window.
- Performed live validation campaigns to confirm:
  - rule `100100` for GoPhish email send
  - rule `100102` for Dovecot mailbox delivery
  - rule `100101` for GoPhish click activity

Result:

- The detection primitives are working live in `alerts.json`.
- The repo and running core lab VMs now contain the stricter correlation design needed for Sprint 4 execution.
- Static validation on `2026-03-14 15:04:48 UTC` confirmed rules `100102`, `100101`, and `100104` in `wazuh-logtest`.
- Sprint 3 no longer depended on telemetry fixes at that point; the remaining work was live proof of the new correlation path, which was later completed in Sprint 4.

## Sprint 4: Correlation And Use Case Completion

Status: `Completed`

Current focus entering the sprint:

- Deploy the redesigned `100104` rule and canonicalized `mail01` agent config.
- Validate one clean delivery-plus-click run and confirm a single high-confidence incident alert in `alerts.json`.

Planned focus:

- Validate the already-deployed correlation package with one clean campaign.
- Correlate `mail01` delivery and `gophish01` RID hits into one incident path.
- Retain `wkstn01` IMAP activity as supporting evidence, not a hard prerequisite for the incident alert.

Result:

- Fresh live validation on `2026-03-14` confirmed rule `100104` in `alerts.json`.
- Campaign `9` (`Sprint 4 Post-Restart Validation 2026-03-14`) produced:
  - GoPhish `Email Sent` at `2026-03-14T16:47:58.118515047Z`
  - Dovecot mailbox delivery at `2026-03-14 16:47:54.083441+00:00`
  - GoPhish `Clicked Link` for RID `pl4qC1k` from `192.168.56.20` at `2026-03-14T16:48:33.779247483Z`
  - Wazuh rule `100104` at `2026-03-14T16:48:35.054+0000`
- The final live fix was adding `frequency="2"` plus `<global_frequency />` to rule `100104` so cross-agent correlation persisted in live analysis, not just in `wazuh-logtest`.
- Repo-stored visual evidence for this validation is now available:
  - [docs/assets/screenshots/siem01-sprint4-click-100101-2026-03-14.png](/c:/Users/demar/Downloads/Phishing-Lab/docs/assets/screenshots/siem01-sprint4-click-100101-2026-03-14.png)
  - [docs/assets/screenshots/siem01-sprint4-delivery-100102-2026-03-14.png](/c:/Users/demar/Downloads/Phishing-Lab/docs/assets/screenshots/siem01-sprint4-delivery-100102-2026-03-14.png)
  - [docs/assets/screenshots/siem01-sprint4-correlation-100104-2026-03-14.png](/c:/Users/demar/Downloads/Phishing-Lab/docs/assets/screenshots/siem01-sprint4-correlation-100104-2026-03-14.png)
  - [docs/assets/screenshots/wkstn01-sprint4-thunderbird-email-2026-03-14.png](/c:/Users/demar/Downloads/Phishing-Lab/docs/assets/screenshots/wkstn01-sprint4-thunderbird-email-2026-03-14.png)
  - [docs/assets/screenshots/wkstn01-sprint4-browser-rid-pl4qC1k-2026-03-14.png](/c:/Users/demar/Downloads/Phishing-Lab/docs/assets/screenshots/wkstn01-sprint4-browser-rid-pl4qC1k-2026-03-14.png)

## Sprint 5: Incident Response Deliverables

Status: `Completed`

Result:

- Incident response deliverable written from the validated March 14 evidence chain.
- Covers triage, containment, investigation, recovery, lessons learned, and recommended improvements.
- Grounded in exact timestamps, rule IDs, RID `pl4qC1k`, and repo-stored screenshots.
- No credential harvest, malware execution, or lateral movement claimed. Scope is phishing delivery and link click detection.
- Deliverable: [docs/SPRINT5_INCIDENT_RESPONSE.md](/c:/Users/demar/Downloads/Phishing-Lab/docs/SPRINT5_INCIDENT_RESPONSE.md)

## Sprint 6: Final Repo And Closeout

Status: `Completed`

Result:

- Status documents updated to reflect all six sprints completed.
- File path and screenshot references verified across all docs.
- Repo narrative is internally consistent across status, overview, sprint, and handoff documents.
- Deliverable: [docs/SPRINT6_PROJECT_CLOSEOUT.md](/c:/Users/demar/Downloads/Phishing-Lab/docs/SPRINT6_PROJECT_CLOSEOUT.md)

## Overall Assessment

The project is complete. All six sprints have been executed and documented. The phishing lab demonstrates a repeatable end-to-end incident path from email delivery through SIEM correlation, with a full incident response deliverable and a finalized repo.

## Baseline Snapshot Plan

Baseline VirtualBox snapshots were requested after this review for:

- `dc01`
- `dc02`
- `siem01`
- `wkstn01`
- `gophish01`
- `mail01`

Completed baseline action:

- Snapshot name: `baseline-2026-03-11-internal-review`
- Snapshot created for all six VMs
- Final VM state after snapshotting: all six VMs powered off

The intended outcome is a recoverable baseline for future validation and documentation work.
