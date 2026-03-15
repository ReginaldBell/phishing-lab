# Incident Report: Phishing Simulation - Sprint 4 Post-Restart Validation

**Report ID:** `IR-2026-03-14-001`  
**Classification:** `Controlled Lab Exercise - Detection Validated`  
**Severity:** `High (rule level 10 - informational in controlled context)`  
**Status:** `Closed`  
**Authored by:** `SOC Detection Engineering`  
**Date:** `2026-03-14`

---

## Table of Contents

1. [Executive Summary](#1-executive-summary)
2. [Environment Overview](#2-environment-overview)
3. [Incident Overview](#3-incident-overview)
4. [Timeline](#4-timeline)
5. [Detection and Evidence](#5-detection-and-evidence)
6. [MITRE ATT&CK Mapping](#6-mitre-attck-mapping)
7. [Triage Process](#7-triage-process)
8. [Investigation](#8-investigation)
9. [Containment Strategy](#9-containment-strategy)
10. [Recovery](#10-recovery)
11. [Lessons Learned](#11-lessons-learned)
12. [Detection Engineering Notes](#12-detection-engineering-notes)
13. [Appendix: Evidence References](#13-appendix-evidence-references)

---

## 1. Executive Summary

On `2026-03-14`, a controlled phishing simulation was executed against the lab environment under Campaign ID `9` (`Sprint 4 Post-Restart Validation 2026-03-14`). A phishing email was delivered to `labuser@example.local` on `mail01`. The target workstation `wkstn01` clicked the embedded phishing link approximately 39 seconds after delivery.

The Wazuh SIEM on `siem01` detected the full incident chain and raised a high-confidence correlation alert (`rule 100104`, level `10`) within approximately two seconds of the link click.

No credentials were collected. No malware was executed. Incident scope is limited to phishing email delivery and a confirmed link click.

| Key Metric | Value |
|---|---|
| Detection rule fired | `100104` |
| Alert confidence | High |
| Time to detection from click | ~2 seconds |
| Time from delivery to click | ~39 seconds |
| Credential exposure | None |
| Post-click execution | None |
| Incident scope | Delivery + link click only |

---

## 2. Environment Overview

| Host | Role | IP Address |
|---|---|---|
| `mail01` | Mail server (`Postfix` / `Dovecot`) | `192.168.56.70` |
| `wkstn01` | Target workstation (`Thunderbird` + browser) | `192.168.56.20` |
| `gophish01` | Phishing infrastructure (`GoPhish`) | `192.168.56.60` |
| `siem01` | Wazuh SIEM / manager | `192.168.56.103` |

**Active Directory domain:** `example.local`  
**Target account:** `labuser@example.local`

The lab simulates an enterprise phishing detection pipeline with cross-agent SIEM correlation across the mail, workstation, and phishing infrastructure layers.

---

## 3. Incident Overview

| Field | Value |
|---|---|
| Incident date | `2026-03-14` |
| Campaign ID | `9` |
| Campaign name | `Sprint 4 Post-Restart Validation 2026-03-14` |
| Phishing RID | `pl4qC1k` |
| Target recipient | `labuser@example.local` |
| Delivering host | `mail01` (`192.168.56.70`) |
| Click source | `wkstn01` (`192.168.56.20`) |
| Phishing server | `gophish01` (`192.168.56.60`) |
| SIEM | `siem01` (`192.168.56.103`) |
| Detection rule | Wazuh rule `100104` |
| Credential harvest | None. Landing page presents no credential form |
| Post-click activity | None validated. Scope is delivery and click only |

---

## 4. Timeline

All timestamps are UTC.

| Time (UTC) | System | Event |
|---|---|---|
| `2026-03-14T16:47:54.083441` | `mail01` | Dovecot LMTP saved phishing message to `labuser` INBOX (`/var/log/syslog`) |
| `2026-03-14T16:47:57.974` | `gophish01` | GoPhish campaign `9` created and launched |
| `2026-03-14T16:47:58.119` | `gophish01` | GoPhish recorded `Email Sent` to `labuser@example.local` |
| `2026-03-14T16:47:59.210` | `siem01` | Wazuh rule `100100` raised - GoPhish email send |
| `2026-03-14T16:47:59.578` | `siem01` | Wazuh rule `100102` raised - Dovecot mailbox delivery confirmed |
| `2026-03-14T16:48:33` | `wkstn01` | Browser issued `GET /?rid=pl4qC1k` to `gophish01` |
| `2026-03-14T16:48:33.779` | `gophish01` | GoPhish recorded `Clicked Link` for RID `pl4qC1k` from `192.168.56.20` |
| `2026-03-14T16:48:35.054` | `siem01` | Wazuh rule `100104` raised - phishing correlation alert: mailbox delivery followed by RID click |

**Email delivery to link click:** ~39 seconds  
**Link click to SIEM correlation alert:** ~2 seconds

---

## 5. Detection and Evidence

### 5.1 Wazuh Rules That Fired

| Rule ID | Level | Description | Trigger |
|---|---|---|---|
| `100100` | 5 | GoPhish email sent to phishing target | GoPhish log line containing `Email sent` (decoded via `aws-eks-authenticator`) |
| `100102` | 6 | Dovecot saved GoPhish message to `labuser` INBOX | Dovecot `saved mail to INBOX` in `/var/log/syslog` on `mail01` |
| `100101` | 9 | GoPhish RID click from `wkstn01` | GoPhish log line matching `GET /?rid=` |
| `100104` | 10 | Phishing correlation: mailbox delivery followed by RID click | Correlation: rule `100102` followed by rule `100101` within 1800 seconds across any agent (`global_frequency`) |

Rule `100104` is the high-confidence incident indicator. It requires both confirmed delivery evidence and a real RID click. Favicon requests and unrelated traffic do not satisfy the correlation condition.

### 5.2 Log Sources

| System | Log Source | Content |
|---|---|---|
| `gophish01` | `/var/log/gophish/gophish.log` | Email send events, RID click events |
| `mail01` | `/var/log/syslog` | Dovecot LMTP delivery saves |
| `siem01` | `/var/ossec/logs/alerts/alerts.json` | All Wazuh alert output including rule `100104` |

### 5.3 Screenshot Evidence

Stored in `docs/assets/screenshots/`:

| Screenshot | Content |
|---|---|
| [wkstn01-sprint4-thunderbird-email-2026-03-14.png](assets/screenshots/wkstn01-sprint4-thunderbird-email-2026-03-14.png) | Thunderbird on `wkstn01` showing the received phishing email |
| [wkstn01-sprint4-browser-rid-pl4qC1k-2026-03-14.png](assets/screenshots/wkstn01-sprint4-browser-rid-pl4qC1k-2026-03-14.png) | Browser on `wkstn01` showing the GoPhish landing page with RID `pl4qC1k` |
| [siem01-sprint4-click-100101-2026-03-14.png](assets/screenshots/siem01-sprint4-click-100101-2026-03-14.png) | Wazuh dashboard showing rule `100101` click detection |
| [siem01-sprint4-delivery-100102-2026-03-14.png](assets/screenshots/siem01-sprint4-delivery-100102-2026-03-14.png) | Wazuh dashboard showing rule `100102` delivery detection |
| [siem01-sprint4-correlation-100104-2026-03-14.png](assets/screenshots/siem01-sprint4-correlation-100104-2026-03-14.png) | Wazuh dashboard showing rule `100104` correlation alert |

---

## 6. MITRE ATT&CK Mapping

| Tactic | Technique | Technique ID | Wazuh Rules |
|---|---|---|---|
| Initial Access | Phishing | `T1566` | `100101`, `100104` |

Scope note: this incident is limited to delivery and initial click. No credential submission, execution, persistence, lateral movement, or C2 activity was validated.

---

## 7. Triage Process

### 7.1 Initial Signal

The actionable alert is rule `100104` at level `10`. This rule does not fire on delivery alone or click alone. It fires only when the SIEM observes a prior mailbox delivery followed by a real GoPhish RID click within a 30-minute window across both `mail01` and `gophish01` agents.

### 7.2 Triage Steps

1. Confirm rule `100104` in the Wazuh dashboard.
2. Extract the RID from the click event in rule `100101`. In this incident: `pl4qC1k`.
3. Match the RID to a GoPhish campaign. Campaign `9` maps to this RID.
4. Identify the source IP of the click. In this incident: `192.168.56.20` (`wkstn01`).
5. Confirm mailbox delivery on `mail01` via rule `100102` or by querying `/var/log/syslog` for `saved mail to INBOX`.
6. Confirm no credential submission occurred. The GoPhish landing page in this campaign does not collect credentials.
7. Determine whether the `labuser` account requires further investigation.

### 7.3 Scope Assessment

| Category | Finding |
|---|---|
| Affected systems | `wkstn01` (click origin), `mail01` (delivery target), `gophish01` (phishing infrastructure) |
| Affected account | `labuser@example.local` |
| Credential exposure | None confirmed |
| Lateral movement | None observed |
| Persistence or execution | None observed |

---

## 8. Investigation

### 8.1 Verification Checklist

| Step | Finding |
|---|---|
| Mailbox delivery path | GoPhish campaign `9` `Email Sent` confirmed at `2026-03-14T16:47:58.119Z` |
| Mailbox receipt | Dovecot saved message to `labuser` INBOX at `16:47:54 UTC` (`/var/log/syslog` on `mail01`) |
| Email access | Rule `100103` monitors IMAP login from `wkstn01` (`192.168.56.20`) to `mail01` (`192.168.56.70`) and supports the conclusion that the email was opened in Thunderbird rather than by automated polling |
| Click confirmation | Rule `100101` matched `GET /?rid=pl4qC1k` from `192.168.56.20`; GoPhish independently recorded `Clicked Link` for the same RID at the same time |
| Landing page behavior | No credential form present; user reached a static informational page only |
| Follow-on indicators | None - no rules fired for malware execution, outbound C2, lateral movement, privilege escalation, or credential submission |

### 8.2 Investigation Workflow

```text
Rule 100104 fires
    |
    +-- Retrieve RID from rule 100101 event
    |      -> pl4qC1k
    |
    +-- Match RID to GoPhish campaign
    |      -> Campaign 9: Sprint 4 Post-Restart Validation 2026-03-14
    |
    +-- Identify click source IP
    |      -> 192.168.56.20 (wkstn01)
    |
    +-- Verify mailbox delivery via rule 100102
    |      -> mail01 /var/log/syslog: saved mail to INBOX at 16:47:54 UTC
    |
    +-- Check for IMAP login via rule 100103
    |      -> Dovecot IMAP login from 192.168.56.20 to 192.168.56.70 (supporting evidence)
    |
    +-- Inspect landing page for credential submission
    |      -> No credential form present
    |
    +-- Search for post-click indicators
           -> None detected
```

---

## 9. Containment Strategy

In this controlled simulation, containment was not operationally required. The steps below document the expected real-world response pattern for an incident of equivalent scope.

### 9.1 Immediate Containment

- Isolate `wkstn01` from the network if there is any uncertainty about post-click activity.
- Disable or lock the `labuser` Active Directory account until investigation confirms no credential exposure.
- Block the phishing domain or IP at the perimeter. Lab equivalent: `192.168.56.60` (`gophish01`).

### 9.2 SIEM Containment Actions

- Add a suppression scope or tag the `100104` alert as triaged to prevent alert fatigue on the same RID.
- If additional campaigns are suspected, search `alerts.json` for other `100101` events in the same time window.

---

## 10. Recovery

No recovery actions were operationally required. The GoPhish landing page served informational content only, and the lab environment was not impacted.

### 10.1 Recovery Actions (Real-World Equivalent)

1. If `wkstn01` was isolated, restore network connectivity after investigation confirms no post-click execution.
2. If `labuser` was disabled, re-enable the account after investigation confirms no credential compromise.
3. Clear GoPhish campaign `9` results if the lab is being reset for future exercises.
4. Document the incident outcome and close the alert in the SIEM.

---

## 11. Lessons Learned

### 11.1 What Worked

- The Wazuh detection pipeline raised a high-confidence alert within approximately two seconds of the link click.
- Cross-agent correlation between `mail01` and `gophish01` worked correctly once rule `100104` was updated to use `global_frequency`.
- Rule logic correctly filtered out favicon requests and other non-phishing traffic, eliminating false correlations.
- Screenshot evidence captured the full delivery-to-alert path across `wkstn01` and `siem01`.

### 11.2 What Required Engineering Work

| Issue | Root Cause | Resolution |
|---|---|---|
| Rule `100104` fired in `wazuh-logtest` but not in live mode | `wazuh-logtest` operates in a single local session context while live alerts cross Wazuh agent boundaries | Added `<global_frequency />` to the rule definition |
| GoPhish logs decoded by the wrong decoder | GoPhish log lines were assigned the `aws-eks-authenticator` decoder rather than a custom one | Rule `100100` was adjusted to match the actual assigned decoder |
| No `/var/log/syslog` on `mail01` | `rsyslog` was not installed | Installed `rsyslog` and reconfigured the Wazuh agent to monitor syslog as the canonical Dovecot source |

### 11.3 Known Detection Gaps

| Gap | Impact | Priority |
|---|---|---|
| Rule `100100` depends on the `aws-eks-authenticator` decoder assignment | Wazuh decoder updates may break matching | High |
| Detection scope is delivery and click only | Credential submission, malware execution, and lateral movement are not covered by current rules | High |
| `wkstn01` endpoint telemetry is limited to IMAP login visibility | Post-click process execution and network connections are not captured | High |

---

## 12. Detection Engineering Notes

### 12.1 Recommended Improvements

| Priority | Improvement |
|---|---|
| High | Write a dedicated GoPhish decoder to replace the `aws-eks-authenticator` dependency for rule `100100` |
| High | Add endpoint telemetry on `wkstn01` to capture process execution and network connections after a phishing click |
| Medium | Extend rule coverage to detect credential submission if a credential-harvesting landing page is added to future campaigns |
| Medium | Add a Wazuh active response to automatically isolate `wkstn01` when rule `100104` fires |
| Low | Add a GoPhish API integration to pull campaign metadata directly into SIEM alert enrichment |
| Low | Extend correlation time window analysis to flag repeat-click patterns across multiple campaigns |

### 12.2 Rule File Reference

- [wazuh/rules/phishing_lab_rules.xml](../wazuh/rules/phishing_lab_rules.xml)

### 12.3 Agent Configurations

- [wazuh/agents/gophish01-ossec.conf](../wazuh/agents/gophish01-ossec.conf)
- [wazuh/agents/mail01-ossec.conf](../wazuh/agents/mail01-ossec.conf)

---

## 13. Appendix: Evidence References

### Screenshots

| File | Sprint |
|---|---|
| [wkstn01-sprint4-thunderbird-email-2026-03-14.png](assets/screenshots/wkstn01-sprint4-thunderbird-email-2026-03-14.png) | Sprint 4 |
| [wkstn01-sprint4-browser-rid-pl4qC1k-2026-03-14.png](assets/screenshots/wkstn01-sprint4-browser-rid-pl4qC1k-2026-03-14.png) | Sprint 4 |
| [siem01-sprint4-click-100101-2026-03-14.png](assets/screenshots/siem01-sprint4-click-100101-2026-03-14.png) | Sprint 4 |
| [siem01-sprint4-delivery-100102-2026-03-14.png](assets/screenshots/siem01-sprint4-delivery-100102-2026-03-14.png) | Sprint 4 |
| [siem01-sprint4-correlation-100104-2026-03-14.png](assets/screenshots/siem01-sprint4-correlation-100104-2026-03-14.png) | Sprint 4 |
| [siem01-sprint3-alerts-2026-03-11.png](assets/screenshots/siem01-sprint3-alerts-2026-03-11.png) | Sprint 3 (earlier validation) |
| [siem01-sprint3-full-validation-2026-03-11.png](assets/screenshots/siem01-sprint3-full-validation-2026-03-11.png) | Sprint 3 (earlier validation) |
| [wkstn01-sprint1-thunderbird-2026-03-11.png](assets/screenshots/wkstn01-sprint1-thunderbird-2026-03-11.png) | Sprint 1 (earlier validation) |
| [wkstn01-sprint1-browser-2026-03-11.png](assets/screenshots/wkstn01-sprint1-browser-2026-03-11.png) | Sprint 1 (earlier validation) |

### Key Source Documents

| Document | Path |
|---|---|
| Project status | [docs/PROJECT_STATUS.md](PROJECT_STATUS.md) |
| Sprint internal review | [docs/SPRINT_PROGRESS_INTERNAL_REVIEW.md](SPRINT_PROGRESS_INTERNAL_REVIEW.md) |
| Detection engineering log | [docs/SPRINT3_DETECTION_ENGINEERING.md](SPRINT3_DETECTION_ENGINEERING.md) |
| Handoff notes | [docs/SPRINT5_6_HANDOFF.md](SPRINT5_6_HANDOFF.md) |

---

This report was produced as part of the Enterprise Active Directory Detection Lab portfolio. All activity was intentional, controlled, and scoped to an isolated lab environment. No production systems were affected.
