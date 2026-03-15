# Sprint 3 Detection Engineering

Updated: `2026-03-14`

## Current Status

Sprint 3 detection engineering is complete at the repo level. The primitive detection package is in place, the Sprint 4 correlation redesign was successfully deployed on the core lab VMs, and the project now has a fully demonstrated incident path documented through Sprint 6.

What is complete:

- custom Wazuh rule and decoder artifacts were added to the repo
- `siem01` accepted the deployed package and `wazuh-manager` is running
- `gophish01` Wazuh agent collection was corrected to stop treating `gophish.log` as JSON
- `mail01` Wazuh agent collection was reduced to one canonical Dovecot source: `/var/log/syslog`
- `wazuh-logtest` now raises custom phishing alerts for:
  - GoPhish email send events
  - GoPhish link click events
  - Dovecot mailbox delivery events
  - Dovecot IMAP login events from `wkstn01`
- correlation rule `100104` was redesigned to fire on a real click event only when a prior mailbox-delivery alert exists in the time window
- helper scripts now include correlation-focused `wazuh-logtest` and `alerts.json` inspection modes

Known limitations that remain:

- rule `100100` for GoPhish email send still relies on the decoder Wazuh is actually assigning to those lines: `aws-eks-authenticator`

## Files Added For Sprint 3

Repo artifacts:

- [wazuh/decoders/phishing_lab_decoders.xml](/c:/Users/demar/Downloads/Phishing-Lab/wazuh/decoders/phishing_lab_decoders.xml)
- [wazuh/rules/phishing_lab_rules.xml](/c:/Users/demar/Downloads/Phishing-Lab/wazuh/rules/phishing_lab_rules.xml)
- [wazuh/agents/gophish01-ossec.conf](/c:/Users/demar/Downloads/Phishing-Lab/wazuh/agents/gophish01-ossec.conf)
- [wazuh/agents/mail01-ossec.conf](/c:/Users/demar/Downloads/Phishing-Lab/wazuh/agents/mail01-ossec.conf)
- [scripts/linux/deploy-wazuh-phishing-rules.sh](/c:/Users/demar/Downloads/Phishing-Lab/scripts/linux/deploy-wazuh-phishing-rules.sh)
- [scripts/linux/tmp_sprint3_collect.sh](/c:/Users/demar/Downloads/Phishing-Lab/scripts/linux/tmp_sprint3_collect.sh)
- [scripts/linux/run-wazuh-logtest.sh](/c:/Users/demar/Downloads/Phishing-Lab/scripts/linux/run-wazuh-logtest.sh)

## Deployed Changes

### gophish01

Updated Wazuh log collection so:

- `/var/log/gophish/gophish.log` is collected as `syslog`-style text instead of `json`

This was necessary because the real GoPhish log format looks like:

```text
time="2026-03-11T05:59:54Z" level=info msg="192.168.56.20 - - [11/Mar/2026:05:59:54 +0000] "GET /?rid=tW15oVY HTTP/1.1" 200 ..."
```

That format is not JSON.

### mail01

Updated Wazuh log collection so:

- `/var/log/syslog` is monitored as the canonical Dovecot source
- `/var/log/mail.log` is no longer part of the repo-side phishing configuration
- `/var/log/email-signals.log` remains configured

This was necessary because the live March 11 evidence came from Dovecot activity visible in syslog/journal, while `/var/log/mail.log` was both unreliable during collection and a source of duplicate delivery alerts once `rsyslog` was installed.

### siem01

Deployed custom Wazuh artifacts and restarted `wazuh-manager` successfully after syntax fixes.

Manager status after deployment:

- `wazuh-manager`: `active`

## Validation Results So Far

### Working

- `wazuh-manager` is healthy after deployment
- `wazuh-logtest` raises rule `100100` for GoPhish `Email sent`
- `wazuh-logtest` raises rule `100101` for GoPhish click requests
- `wazuh-logtest` raises rule `100102` for Dovecot `saved mail to INBOX`
- `wazuh-logtest` raises rule `100103` for Dovecot IMAP login from `192.168.56.20` to `192.168.56.70`
- the repo now contains a reproducible deployment path for phishing-specific Wazuh content

### Accepted Limitations

- GoPhish email-send lines are still being decoded as `aws-eks-authenticator`, so rule `100100` still depends on that decoder name

## Sprint 4 Static Validation

Static deployment and rule validation completed on `2026-03-14 15:04:48 UTC`.

Validated on running `siem01`, `mail01`, and `gophish01`:

- the updated `phishing_lab_rules.xml` deployed cleanly and `wazuh-manager` returned to `active`
- the updated `mail01` agent config deployed with `/var/log/syslog` as the canonical Dovecot source
- `wazuh-logtest` raised rule `100102` for the Dovecot mailbox-delivery sample
- `wazuh-logtest` raised rule `100101` for the GoPhish RID click sample
- `wazuh-logtest` raised rule `100104` when the delivery sample was followed by the click sample in the same session
- the click-only negative sample raised rule `100101` and did not raise rule `100104`

## Live Validation Notes

- A live click on `2026-03-11 07:01:01 UTC` generated rule `100101` in `alerts.json`
- An earlier version of rule `100101` was too broad and also matched `GET /favicon.ico`, which incorrectly allowed correlation rule `100104` to fire
- The click rule was tightened after that validation to require `GET /?rid=`
- A fresh post-fix validation on `2026-03-11 08:30:58 UTC` generated rule `100101` for the real click and did not generate a new false-positive `100104`
- `mail01` originally failed to generate live delivery alerts because `rsyslog` was not installed and no `/var/log/syslog` existed for the Wazuh agent to read
- `rsyslog` was installed on `mail01` on `2026-03-11`, after which Dovecot LMTP saves began appearing in `/var/log/syslog`
- Fresh campaign `Sprint6Validation` on `2026-03-11` produced:
  - GoPhish `Email Sent` at `2026-03-11 09:11:28 UTC`
  - Dovecot `saved mail to INBOX` at `2026-03-11 09:15:51 UTC`
  - GoPhish `Clicked Link` for RID `AYYJntJ` at `2026-03-11 09:13:16 UTC`
- Live Wazuh alerts for campaign `6` confirmed:
  - rule `100100` on `gophish01`
  - rule `100102` on `mail01`
  - rule `100101` on `gophish01`
- Correlation rule `100104` did not fire for the clean campaign `6` sequence, which is why the redesign and later March 14 validation were required
- Fresh screenshot evidence from `siem01` is stored at:
  - [docs/assets/screenshots/siem01-sprint3-alerts-2026-03-11.png](/c:/Users/demar/Downloads/Phishing-Lab/docs/assets/screenshots/siem01-sprint3-alerts-2026-03-11.png)
  - [docs/assets/screenshots/siem01-sprint3-full-validation-2026-03-11.png](/c:/Users/demar/Downloads/Phishing-Lab/docs/assets/screenshots/siem01-sprint3-full-validation-2026-03-11.png)
- Fresh screenshot evidence from `wkstn01` is stored at:
  - [docs/assets/screenshots/wkstn01-sprint3-postfix-click-2026-03-11.png](/c:/Users/demar/Downloads/Phishing-Lab/docs/assets/screenshots/wkstn01-sprint3-postfix-click-2026-03-11.png)

## Sprint 4 Live Validation

Live correlation validation completed on `2026-03-14`.

Validated sequence:

- campaign `9` (`Sprint 4 Post-Restart Validation 2026-03-14`) was created at `2026-03-14T16:47:57.97378581Z`
- GoPhish recorded `Email Sent` at `2026-03-14T16:47:58.118515047Z`
- `mail01` Dovecot logged mailbox delivery at `2026-03-14 16:47:54.083441+00:00`
- `wkstn01` requested `GET /?rid=pl4qC1k` from `192.168.56.20` at `2026-03-14 16:48:33 UTC`
- GoPhish recorded `Clicked Link` for RID `pl4qC1k` from `192.168.56.20`
- Wazuh raised:
  - rule `100100` for GoPhish email send at `2026-03-14T16:47:59.210+0000`
  - rule `100102` for Dovecot mailbox delivery at `2026-03-14T16:47:59.578+0000`
  - rule `100104` for the final correlation at `2026-03-14T16:48:35.054+0000`

Critical rule change that enabled live correlation:

- rule `100104` now uses `frequency="2"` with `timeframe="1800"` and `<global_frequency />`
- this was required because live cross-agent correlation between `mail01` and `gophish01` did not persist the way `wazuh-logtest` did in a single local session

Fresh screenshot evidence from `siem01` is stored at:

- [docs/assets/screenshots/siem01-sprint4-click-100101-2026-03-14.png](/c:/Users/demar/Downloads/Phishing-Lab/docs/assets/screenshots/siem01-sprint4-click-100101-2026-03-14.png)
- [docs/assets/screenshots/siem01-sprint4-delivery-100102-2026-03-14.png](/c:/Users/demar/Downloads/Phishing-Lab/docs/assets/screenshots/siem01-sprint4-delivery-100102-2026-03-14.png)
- [docs/assets/screenshots/siem01-sprint4-correlation-100104-2026-03-14.png](/c:/Users/demar/Downloads/Phishing-Lab/docs/assets/screenshots/siem01-sprint4-correlation-100104-2026-03-14.png)

Fresh screenshot evidence from `wkstn01` is stored at:

- [docs/assets/screenshots/wkstn01-sprint4-thunderbird-email-2026-03-14.png](/c:/Users/demar/Downloads/Phishing-Lab/docs/assets/screenshots/wkstn01-sprint4-thunderbird-email-2026-03-14.png)
- [docs/assets/screenshots/wkstn01-sprint4-browser-rid-pl4qC1k-2026-03-14.png](/c:/Users/demar/Downloads/Phishing-Lab/docs/assets/screenshots/wkstn01-sprint4-browser-rid-pl4qC1k-2026-03-14.png)

## Practical Conclusion

Sprint 3 has reached the point where the repo and the running core lab VMs contain a coherent detection package and a live-validated Sprint 4 correlation design.

The remaining project work described earlier has now been completed in Sprint 5 and Sprint 6. This Sprint 3 document should now be read as the technical build and validation record for the detection package.

## Follow-On Deliverables Completed

1. Sprint 5 incident response deliverable written at [docs/SPRINT5_INCIDENT_RESPONSE.md](/c:/Users/demar/Downloads/Phishing-Lab/docs/SPRINT5_INCIDENT_RESPONSE.md)
2. Status and summary docs updated to reflect project completion
3. Final Sprint 4 screenshots consolidated in [docs/assets/screenshots](/c:/Users/demar/Downloads/Phishing-Lab/docs/assets/screenshots)
4. Sprint 6 closeout completed at [docs/SPRINT6_PROJECT_CLOSEOUT.md](/c:/Users/demar/Downloads/Phishing-Lab/docs/SPRINT6_PROJECT_CLOSEOUT.md)
