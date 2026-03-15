# Sprint 5 Incident Response

Created: `2026-03-14`

## 1. Executive Summary

On `2026-03-14`, a controlled phishing simulation was executed against the lab environment. The campaign delivered a phishing email to `labuser@example.local` on `mail01`, and the target workstation `wkstn01` clicked the embedded phishing link 35 seconds after delivery. The Wazuh SIEM on `siem01` detected the full incident chain and raised a high-confidence correlation alert within two seconds of the link click.

No credentials were collected. No malware was executed. The incident scope is limited to phishing email delivery and a successful link click.

Detection status: **Confirmed**. Wazuh raised rule `100104` in live `alerts.json` at `2026-03-14T16:48:35.054+0000`.

---

## 2. Incident Overview

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
| Credential harvest | None. Landing page explicitly presents no credential form. |
| Post-click activity | None validated. Scope is delivery and click only. |

---

## 3. Timeline

All times are UTC.

| Time (UTC) | System | Event |
|---|---|---|
| `2026-03-14T16:47:54.083441` | `mail01` | Dovecot LMTP saved phishing message to `labuser` INBOX (`/var/log/syslog`) |
| `2026-03-14T16:47:57.974` | `gophish01` | GoPhish campaign `9` created and launched |
| `2026-03-14T16:47:58.119` | `gophish01` | GoPhish recorded `Email Sent` to `labuser@example.local` |
| `2026-03-14T16:47:59.210` | `siem01` | Wazuh rule `100100` raised: GoPhish email send |
| `2026-03-14T16:47:59.578` | `siem01` | Wazuh rule `100102` raised: Dovecot mailbox delivery confirmed |
| `2026-03-14T16:48:33` | `wkstn01` | Browser on `wkstn01` issued `GET /?rid=pl4qC1k` to `gophish01` |
| `2026-03-14T16:48:33.779` | `gophish01` | GoPhish recorded `Clicked Link` for RID `pl4qC1k` from `192.168.56.20` |
| `2026-03-14T16:48:35.054` | `siem01` | Wazuh rule `100104` raised: **phishing correlation alert** — mailbox delivery followed by RID click |

Elapsed time from email delivery to link click: approximately **39 seconds**.
Elapsed time from link click to SIEM correlation alert: approximately **2 seconds**.

---

## 4. Detection And Evidence

### 4.1 Wazuh Rules That Fired

| Rule ID | Level | Description | Trigger |
|---|---|---|---|
| `100100` | 5 | GoPhish email sent to phishing target | GoPhish log line containing `Email sent` (decoded via `aws-eks-authenticator`) |
| `100102` | 6 | Dovecot saved GoPhish message to labuser INBOX | Dovecot `saved mail to INBOX` in `/var/log/syslog` on `mail01` |
| `100101` | 9 | GoPhish RID click from wkstn01 | GoPhish log line matching `GET /?rid=` |
| `100104` | 10 | **Phishing scenario: mailbox delivery followed by RID click** | Correlation: rule `100102` followed by rule `100101` within 1800 seconds across any agent (`global_frequency`) |

Rule `100104` is the high-confidence incident indicator. It requires both delivery evidence and a real RID click, not a favicon request or unrelated traffic.

### 4.2 MITRE ATT&CK Mapping

| Technique | ID | Rules |
|---|---|---|
| Phishing | T1566 | `100101`, `100104` |

### 4.3 Screenshot Evidence

Stored in `docs/assets/screenshots/`:

| Screenshot | Content |
|---|---|
| [wkstn01-sprint4-thunderbird-email-2026-03-14.png](assets/screenshots/wkstn01-sprint4-thunderbird-email-2026-03-14.png) | Thunderbird on `wkstn01` showing the received phishing email |
| [wkstn01-sprint4-browser-rid-pl4qC1k-2026-03-14.png](assets/screenshots/wkstn01-sprint4-browser-rid-pl4qC1k-2026-03-14.png) | Browser on `wkstn01` showing the GoPhish landing page with RID `pl4qC1k` |
| [siem01-sprint4-click-100101-2026-03-14.png](assets/screenshots/siem01-sprint4-click-100101-2026-03-14.png) | Wazuh dashboard on `siem01` showing rule `100101` click detection |
| [siem01-sprint4-delivery-100102-2026-03-14.png](assets/screenshots/siem01-sprint4-delivery-100102-2026-03-14.png) | Wazuh dashboard on `siem01` showing rule `100102` delivery detection |
| [siem01-sprint4-correlation-100104-2026-03-14.png](assets/screenshots/siem01-sprint4-correlation-100104-2026-03-14.png) | Wazuh dashboard on `siem01` showing rule `100104` correlation alert |

### 4.4 Log Sources

| System | Log Source | Content |
|---|---|---|
| `gophish01` | `/var/log/gophish/gophish.log` | Email send events, RID click events |
| `mail01` | `/var/log/syslog` | Dovecot LMTP delivery saves |
| `siem01` | `alerts.json` | All Wazuh alert output including rule `100104` |

---

## 5. Triage

### Initial Signal

The actionable alert is rule `100104` at level 10. This rule does not fire on delivery alone or click alone. It fires only when the SIEM observes a prior mailbox delivery followed by a real GoPhish RID click within a 30-minute window, across both `mail01` and `gophish01` agents.

### Triage Steps

1. Confirm rule `100104` in Wazuh dashboard on `https://192.168.56.103/`.
2. Extract the RID from the click event in rule `100101`. In this incident: `pl4qC1k`.
3. Match the RID to a GoPhish campaign on `https://192.168.56.60:3333`. Campaign `9` maps to this RID.
4. Identify the source IP of the click. In this incident: `192.168.56.20` (`wkstn01`).
5. Confirm mailbox delivery on `mail01` by reviewing rule `100102` or querying `/var/log/syslog` for `saved mail to INBOX`.
6. Confirm no credential submission occurred. In this lab, the GoPhish landing page does not collect credentials.
7. Determine whether the user account `labuser` requires further investigation.

### Scope Assessment

- Affected systems: `wkstn01` (click origin), `mail01` (delivery target), `gophish01` (phishing infrastructure)
- Affected account: `labuser@example.local`
- Credential exposure: None confirmed
- Lateral movement: None observed
- Persistence or execution: None observed

---

## 6. Containment

Because the lab confirmed no credential collection and no post-click execution, the containment scope is limited. In a real-world incident of equivalent scope, the following actions apply.

### Immediate Containment

- Isolate `wkstn01` from the network if there is any uncertainty about post-click activity.
- Disable or lock the `labuser` Active Directory account until the investigation confirms no credential exposure.
- Block the phishing domain or IP at the perimeter. In this lab: `192.168.56.60` (`gophish01`).

### SIEM Containment Actions

- Add a suppression scope or tag the `100104` alert as triaged to prevent alert fatigue on the same RID.
- If additional campaigns are suspected, search `alerts.json` for other `100101` events in the same time window.

### Lab-Specific Note

In this controlled environment, containment was not operationally required. The simulation was intentional. These steps are documented to demonstrate the expected response pattern.

---

## 7. Investigation

### What To Verify

1. **Mailbox delivery path**: Confirm GoPhish campaign `9` sent the email. Confirmed via GoPhish `Email Sent` at `2026-03-14T16:47:58.119Z`.
2. **Mailbox receipt**: Confirm Dovecot saved the message to `labuser` INBOX. Confirmed via `/var/log/syslog` on `mail01` at `2026-03-14 16:47:54 UTC`.
3. **Email access**: Rule `100103` monitors IMAP login from `wkstn01` (`192.168.56.20`) to `mail01` (`192.168.56.70`). This provides supporting evidence that the email was opened via Thunderbird, not automated polling.
4. **Click confirmation**: Rule `100101` matched `GET /?rid=pl4qC1k` from `192.168.56.20`. GoPhish independently recorded `Clicked Link` for the same RID at the same time.
5. **Landing page behavior**: The GoPhish landing page in this campaign does not present a credential form. The user reached a static informational page only.
6. **No follow-on indicators**: No rule fired for malware execution, outbound C2, lateral movement, privilege escalation, or credential submission. The incident path terminates at the click.

### Investigation Workflow

```
Rule 100104 fires
    |
    ├── Retrieve RID from rule 100101 event
    |       → pl4qC1k
    |
    ├── Match RID to GoPhish campaign
    |       → Campaign 9: Sprint 4 Post-Restart Validation 2026-03-14
    |
    ├── Identify click source IP
    |       → 192.168.56.20 (wkstn01)
    |
    ├── Verify mailbox delivery via rule 100102
    |       → mail01 /var/log/syslog: saved mail to INBOX at 16:47:54 UTC
    |
    ├── Check for IMAP login via rule 100103
    |       → Dovecot IMAP login from 192.168.56.20 to 192.168.56.70 (supporting evidence)
    |
    ├── Inspect landing page for credential submission
    |       → No credential form present
    |
    └── Search for post-click indicators
            → None detected
```

---

## 8. Recovery

Because no credential exposure, malware execution, or persistence was confirmed, the recovery steps are minimal.

### Recovery Actions

- If `wkstn01` was isolated: restore network connectivity after investigation confirms no post-click execution.
- If `labuser` was disabled: re-enable the account after investigation confirms no credential compromise.
- Clear the GoPhish campaign `9` results if the lab is being reset for future use.
- Document the incident outcome and close the alert in the SIEM.

### Lab-Specific Note

The lab environment was not operationally impacted. The GoPhish landing page served only informational content. No recovery actions were needed in practice.

---

## 9. Lessons Learned

### What Worked

- The Wazuh detection pipeline raised a high-confidence alert within two seconds of the link click.
- Cross-agent correlation between `mail01` and `gophish01` worked correctly once rule `100104` was updated to use `global_frequency`.
- The rule logic correctly filtered out favicon requests and other non-phishing traffic from triggering false correlations.
- Screenshot evidence captured the full delivery-to-alert path across `wkstn01` and `siem01`.

### What Required Engineering Work

- Rule `100104` initially failed to fire in live mode even though it worked in `wazuh-logtest`. The root cause was that `wazuh-logtest` operates in a single local session context while live alerts cross Wazuh agent boundaries. Adding `<global_frequency />` resolved this.
- GoPhish log lines were being decoded by the `aws-eks-authenticator` decoder rather than a custom decoder. Rule `100100` was adjusted to work with the actual assigned decoder rather than requiring a new decoder to be written.
- `mail01` had no `/var/log/syslog` at first because `rsyslog` was not installed. Installing `rsyslog` and reconfiguring the Wazuh agent to monitor syslog as the canonical Dovecot source resolved mailbox delivery detection.

### Known Detection Gaps

- Rule `100100` depends on the `aws-eks-authenticator` decoder assignment. If Wazuh updates its default decoders, this may stop matching. A dedicated GoPhish decoder would eliminate this dependency.
- The current use case is scoped to delivery and click. A credential submission step, malware execution, or lateral movement scenario would require additional rules and data sources.
- `wkstn01` endpoint telemetry is currently limited to IMAP login visibility. A host-based agent or process monitoring would improve post-click investigation capability.

---

## 10. Recommended Improvements

| Priority | Improvement |
|---|---|
| High | Write a dedicated GoPhish decoder to replace the `aws-eks-authenticator` dependency for rule `100100` |
| High | Add endpoint telemetry on `wkstn01` to capture process execution and network connections after a phishing click |
| Medium | Extend rule coverage to detect credential submission if a credential-harvesting landing page is added to future campaigns |
| Medium | Add a Wazuh active response to isolate `wkstn01` automatically when rule `100104` fires |
| Low | Add a GoPhish API integration to pull campaign metadata directly into the SIEM alert enrichment |
| Low | Extend the correlation time window analysis to flag repeat-click patterns across multiple campaigns |

---

## 11. Appendix: Evidence References

### Rule File

- [wazuh/rules/phishing_lab_rules.xml](../wazuh/rules/phishing_lab_rules.xml)

### Agent Configs

- [wazuh/agents/gophish01-ossec.conf](../wazuh/agents/gophish01-ossec.conf)
- [wazuh/agents/mail01-ossec.conf](../wazuh/agents/mail01-ossec.conf)

### Screenshots

- [wkstn01-sprint4-thunderbird-email-2026-03-14.png](assets/screenshots/wkstn01-sprint4-thunderbird-email-2026-03-14.png)
- [wkstn01-sprint4-browser-rid-pl4qC1k-2026-03-14.png](assets/screenshots/wkstn01-sprint4-browser-rid-pl4qC1k-2026-03-14.png)
- [siem01-sprint4-click-100101-2026-03-14.png](assets/screenshots/siem01-sprint4-click-100101-2026-03-14.png)
- [siem01-sprint4-delivery-100102-2026-03-14.png](assets/screenshots/siem01-sprint4-delivery-100102-2026-03-14.png)
- [siem01-sprint4-correlation-100104-2026-03-14.png](assets/screenshots/siem01-sprint4-correlation-100104-2026-03-14.png)

### Earlier Validation References

- [siem01-sprint3-alerts-2026-03-11.png](assets/screenshots/siem01-sprint3-alerts-2026-03-11.png)
- [siem01-sprint3-full-validation-2026-03-11.png](assets/screenshots/siem01-sprint3-full-validation-2026-03-11.png)
- [wkstn01-sprint1-thunderbird-2026-03-11.png](assets/screenshots/wkstn01-sprint1-thunderbird-2026-03-11.png)
- [wkstn01-sprint1-browser-2026-03-11.png](assets/screenshots/wkstn01-sprint1-browser-2026-03-11.png)

### Key Source Documents

- [docs/PROJECT_STATUS.md](PROJECT_STATUS.md)
- [docs/SPRINT_PROGRESS_INTERNAL_REVIEW.md](SPRINT_PROGRESS_INTERNAL_REVIEW.md)
- [docs/SPRINT3_DETECTION_ENGINEERING.md](SPRINT3_DETECTION_ENGINEERING.md)
- [docs/SPRINT5_6_HANDOFF.md](SPRINT5_6_HANDOFF.md)
