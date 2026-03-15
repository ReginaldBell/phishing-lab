# Sprint 6 Project Closeout

Created: `2026-03-14`

## 1. Final Project Outcome

The Phishing-Lab project is complete.

The project demonstrated a controlled phishing simulation end to end across four production-equivalent VMs, with live SIEM detection and cross-agent correlation confirmed in `alerts.json`. A full incident response deliverable was produced from the validated evidence.

**Final state**: All six sprints completed. Repo is finalized.

---

## 2. Completed Sprint Summary

| Sprint | Status | Key Output |
|---|---|---|
| Sprint 1 | Completed | Lab baseline confirmed working end to end. GoPhish, mail flow, IMAP, and Wazuh agent health validated. |
| Sprint 2 | Completed | Full phishing event timeline built and documented. Evidence consolidated across GoPhish, Dovecot, and Wazuh. |
| Sprint 3 | Completed | Custom Wazuh detection rules and decoders deployed. Primitive rules `100100`–`100103` validated live. |
| Sprint 4 | Completed | Live cross-agent correlation confirmed in `alerts.json`. Rule `100104` fired for campaign `9` on `2026-03-14`. |
| Sprint 5 | Completed | Incident response deliverable written from the validated March 14 evidence chain. |
| Sprint 6 | Completed | Repo normalized. Status documents updated. Closeout summary produced. |

---

## 3. Final Evidence Inventory

### Detection Artifacts

| File | Purpose |
|---|---|
| [wazuh/rules/phishing_lab_rules.xml](../wazuh/rules/phishing_lab_rules.xml) | Custom Wazuh rules (`100100`–`100104`) |
| [wazuh/decoders/phishing_lab_decoders.xml](../wazuh/decoders/phishing_lab_decoders.xml) | Custom Wazuh decoders |
| [wazuh/agents/gophish01-ossec.conf](../wazuh/agents/gophish01-ossec.conf) | Wazuh agent config for `gophish01` |
| [wazuh/agents/mail01-ossec.conf](../wazuh/agents/mail01-ossec.conf) | Wazuh agent config for `mail01` |

### Deployment Scripts

| File | Purpose |
|---|---|
| [scripts/linux/deploy-wazuh-phishing-rules.sh](../scripts/linux/deploy-wazuh-phishing-rules.sh) | Deploy rules and decoders to `siem01` |
| [scripts/linux/run-wazuh-logtest.sh](../scripts/linux/run-wazuh-logtest.sh) | Run `wazuh-logtest` with phishing samples |

### Sprint 4 Screenshots (Primary Evidence)

| File | Content |
|---|---|
| [wkstn01-sprint4-thunderbird-email-2026-03-14.png](assets/screenshots/wkstn01-sprint4-thunderbird-email-2026-03-14.png) | Phishing email received in Thunderbird on `wkstn01` |
| [wkstn01-sprint4-browser-rid-pl4qC1k-2026-03-14.png](assets/screenshots/wkstn01-sprint4-browser-rid-pl4qC1k-2026-03-14.png) | Browser on `wkstn01` at GoPhish landing page with RID `pl4qC1k` |
| [siem01-sprint4-click-100101-2026-03-14.png](assets/screenshots/siem01-sprint4-click-100101-2026-03-14.png) | Wazuh rule `100101` click detection |
| [siem01-sprint4-delivery-100102-2026-03-14.png](assets/screenshots/siem01-sprint4-delivery-100102-2026-03-14.png) | Wazuh rule `100102` delivery detection |
| [siem01-sprint4-correlation-100104-2026-03-14.png](assets/screenshots/siem01-sprint4-correlation-100104-2026-03-14.png) | Wazuh rule `100104` correlation alert |

### Earlier Validation Screenshots (Supporting Evidence)

| File | Content |
|---|---|
| [wkstn01-sprint1-thunderbird-2026-03-11.png](assets/screenshots/wkstn01-sprint1-thunderbird-2026-03-11.png) | Sprint 1 Thunderbird validation |
| [wkstn01-sprint1-browser-2026-03-11.png](assets/screenshots/wkstn01-sprint1-browser-2026-03-11.png) | Sprint 1 browser click validation |
| [siem01-sprint3-alerts-2026-03-11.png](assets/screenshots/siem01-sprint3-alerts-2026-03-11.png) | Sprint 3 Wazuh alert set |
| [siem01-sprint3-full-validation-2026-03-11.png](assets/screenshots/siem01-sprint3-full-validation-2026-03-11.png) | Sprint 3 full Wazuh dashboard view |

### Documentation Files

| File | Purpose |
|---|---|
| [docs/SPRINT5_INCIDENT_RESPONSE.md](SPRINT5_INCIDENT_RESPONSE.md) | Sprint 5 incident response deliverable |
| [docs/PROJECT_STATUS.md](PROJECT_STATUS.md) | Current lab and detection status |
| [docs/SPRINT_PROGRESS_INTERNAL_REVIEW.md](SPRINT_PROGRESS_INTERNAL_REVIEW.md) | Sprint-by-sprint progress record |
| [docs/SPRINT3_DETECTION_ENGINEERING.md](SPRINT3_DETECTION_ENGINEERING.md) | Detection engineering decisions and validation notes |
| [docs/SPRINT_PLAN.md](SPRINT_PLAN.md) | Original sprint planning document |
| [docs/PROJECT_PHASE_OVERVIEW.md](PROJECT_PHASE_OVERVIEW.md) | Phase-by-phase project narrative |
| [docs/SPRINT5_6_HANDOFF.md](SPRINT5_6_HANDOFF.md) | Handoff brief for Sprint 5 and 6 execution |

---

## 4. Remaining Limitations

These limitations are real and accepted as part of the lab scope. They are not open work items.

| Limitation | Notes |
|---|---|
| Rule `100100` depends on `aws-eks-authenticator` decoder | GoPhish email-send log lines are decoded via this name rather than a custom decoder. The rule works, but it carries an implicit dependency on Wazuh's current decoder assignment behavior. |
| No credential harvest coverage | The validated incident path does not include a credential submission step. The GoPhish landing page in this project is informational only. |
| No host-based telemetry on `wkstn01` | Post-click process and network activity on `wkstn01` is not visible to Wazuh. Only IMAP login activity is captured via rule `100103`. |
| Lab scale only | This is a four-VM closed network. The detection approach has not been validated against real-world traffic volumes or attacker evasion techniques. |

---

## 5. Recommended Future Improvements

These are improvements worth pursuing if the lab is extended or adapted for production use.

| Priority | Improvement |
|---|---|
| High | Write a dedicated GoPhish log decoder to remove the `aws-eks-authenticator` dependency |
| High | Install a Wazuh agent on `wkstn01` with process and network monitoring to capture post-click host activity |
| Medium | Extend the GoPhish landing page to simulate credential harvesting and add a rule for `100105: credential submission detected` |
| Medium | Add a Wazuh active response rule to automatically isolate `wkstn01` when rule `100104` fires |
| Medium | Develop a second detection use case, such as spear-phishing with attachment execution, to extend the lab scope |
| Low | Integrate GoPhish API-based campaign metadata into Wazuh alert enrichment |
| Low | Add a SOAR playbook that ingests rule `100104` and automates the triage workflow described in Sprint 5 |

---

## 6. Repo Finalization Notes

### Document Consistency

All status documents have been updated to reflect the completed project state:

- `SPRINT_PROGRESS_INTERNAL_REVIEW.md`: Sprint 5 and Sprint 6 marked as `Completed`
- `PROJECT_PHASE_OVERVIEW.md`: Phase 5 and Phase 6 sections updated from forward-looking to completed
- `PROJECT_STATUS.md`: Recommended next work section updated to reflect final state

### File Path Verification

Screenshot references across all docs point to files in `docs/assets/screenshots/`. The following Sprint 4 screenshots were confirmed present at the time of writing:

- `siem01-sprint4-click-100101-2026-03-14.png`
- `siem01-sprint4-delivery-100102-2026-03-14.png`
- `siem01-sprint4-correlation-100104-2026-03-14.png`
- `wkstn01-sprint4-thunderbird-email-2026-03-14.png`
- `wkstn01-sprint4-browser-rid-pl4qC1k-2026-03-14.png`

Earlier validation screenshots from Sprint 1 and Sprint 3 remain in the same directory as supporting evidence.

### Environment Baseline

The lab environment was snapshotted at `baseline-2026-03-11-internal-review` across all six VMs:

- `dc01`, `dc02`, `siem01`, `wkstn01`, `gophish01`, `mail01`

This snapshot provides a recovery point if the lab is restored for future work.

### Validated Incident Path (Final Reference)

The canonical validated incident:

- **Date**: `2026-03-14`
- **Campaign**: `9` — `Sprint 4 Post-Restart Validation 2026-03-14`
- **RID**: `pl4qC1k`
- **Recipient**: `labuser@example.local`
- **Delivery**: `mail01` at `2026-03-14T16:47:54Z`
- **Click**: `wkstn01` at `2026-03-14T16:48:33Z`
- **Correlation alert**: Wazuh rule `100104` at `2026-03-14T16:48:35.054+0000`

This is the primary evidence chain for the Sprint 5 incident response deliverable and the final claim of project completion.
