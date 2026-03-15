# Project Status

Updated: `2026-03-14`

## Environment

- `gophish01` at `192.168.56.60`
- `mail01` at `192.168.56.70`
- `siem01` at `192.168.56.103`
- `wkstn01` at `192.168.56.20`

## What Is Working

- GoPhish admin UI on `https://192.168.56.60:3333`
- GoPhish phishing web server on `http://192.168.56.60`
- Postfix on `mail01` accepts mail from `192.168.56.0/24`
- `mail01` acts as the final mailbox host for `example.local`
- Dovecot IMAP/LMTP is working on `mail01`
- Thunderbird on `wkstn01` can read `labuser@example.local`
- Wazuh agents for `gophish01`, `mail01`, and `WKSTN01` are active in `siem01`
- Controlled phishing simulation completed end to end

## Validated End-to-End Test

Campaign: `Lab Click Test`

Observed path:

1. GoPhish created the campaign at `2026-03-10 07:26:38 UTC`.
2. GoPhish sent the email to `labuser@example.local`.
3. `mail01` delivered the message to the `labuser` mailbox.
4. Thunderbird on `wkstn01` received the message.
5. `wkstn01` clicked the phishing link.
6. GoPhish recorded `Clicked Link` from `192.168.56.20`.

GoPhish evidence:

- Campaign result status: `Clicked Link`
- Recipient: `labuser@example.local`
- Click source IP: `192.168.56.20`
- RID observed in URL: `yKY3Ezw`

Mail evidence:

- Dovecot saved the GoPhish message into `INBOX`
- Thunderbird authenticated to IMAP from `192.168.56.20`

Wazuh evidence:

- Agents are active and forwarding logs
- No phishing-specific Wazuh alert was generated for the delivery/click workflow

## Detection Status

Wazuh now has phishing-specific repo content for this scenario, and live correlation has been validated on the core lab VMs.

What Wazuh now raises with the current repo package:

- GoPhish email send alerts
- GoPhish RID click alerts
- Dovecot mailbox delivery alerts
- Dovecot IMAP login alerts from `wkstn01`
- a correlation alert in `wazuh-logtest` when mailbox delivery is followed by a GoPhish RID click
- a live rule `100104` correlation alert in `alerts.json` when mailbox delivery is followed by a GoPhish RID click

Latest live proof:

- campaign `9` generated GoPhish `Email Sent`, Dovecot mailbox delivery, GoPhish `Clicked Link`, and Wazuh rule `100104`
- validated click RID: `pl4qC1k`
- validated click source IP: `192.168.56.20`
- Wazuh correlation alert time: `2026-03-14T16:48:35.054+0000`

## Current Credentials

GoPhish:

- URL: `https://192.168.56.60:3333`
- Username: `admin`
- Password: `<redacted>`

SSH:

- `gophish01`: `labadmin / <redacted>`
- `mail01`: `labadmin / <redacted>`
- `siem01`: `labadmin / <redacted>`

Mailbox:

- Address: `labuser@example.local`
- IMAP host: `192.168.56.70`
- Ports: `143`, `993`
- Username: `labuser`
- Password: `<redacted>`

## Screenshots

Sprint 4 live validation evidence:

- `wkstn01` Thunderbird phishing email view:
  - [docs/assets/screenshots/wkstn01-sprint4-thunderbird-email-2026-03-14.png](/c:/Users/demar/Downloads/Phishing-Lab/docs/assets/screenshots/wkstn01-sprint4-thunderbird-email-2026-03-14.png)
- `wkstn01` browser landing page with RID `pl4qC1k`:
  - [docs/assets/screenshots/wkstn01-sprint4-browser-rid-pl4qC1k-2026-03-14.png](/c:/Users/demar/Downloads/Phishing-Lab/docs/assets/screenshots/wkstn01-sprint4-browser-rid-pl4qC1k-2026-03-14.png)
- `siem01` Wazuh click detection, rule `100101`:
  - [docs/assets/screenshots/siem01-sprint4-click-100101-2026-03-14.png](/c:/Users/demar/Downloads/Phishing-Lab/docs/assets/screenshots/siem01-sprint4-click-100101-2026-03-14.png)
- `siem01` Wazuh delivery detection, rule `100102`:
  - [docs/assets/screenshots/siem01-sprint4-delivery-100102-2026-03-14.png](/c:/Users/demar/Downloads/Phishing-Lab/docs/assets/screenshots/siem01-sprint4-delivery-100102-2026-03-14.png)
- `siem01` Wazuh final correlation detection, rule `100104`:
  - [docs/assets/screenshots/siem01-sprint4-correlation-100104-2026-03-14.png](/c:/Users/demar/Downloads/Phishing-Lab/docs/assets/screenshots/siem01-sprint4-correlation-100104-2026-03-14.png)

Earlier validation screenshots retained for comparison:

- [docs/assets/screenshots/wkstn01-click-test.png](/c:/Users/demar/Downloads/Phishing-Lab/docs/assets/screenshots/wkstn01-click-test.png)
- [docs/assets/screenshots/siem01-current.png](/c:/Users/demar/Downloads/Phishing-Lab/docs/assets/screenshots/siem01-current.png)
- [docs/assets/screenshots/siem01-ip-set.png](/c:/Users/demar/Downloads/Phishing-Lab/docs/assets/screenshots/siem01-ip-set.png)

## Project Completion

All six sprints are complete. The project deliverables are:

- [docs/SPRINT5_INCIDENT_RESPONSE.md](/c:/Users/demar/Downloads/Phishing-Lab/docs/SPRINT5_INCIDENT_RESPONSE.md): Incident response deliverable for the validated March 14 phishing incident
- [docs/SPRINT6_PROJECT_CLOSEOUT.md](/c:/Users/demar/Downloads/Phishing-Lab/docs/SPRINT6_PROJECT_CLOSEOUT.md): Final project closeout summary

No further work is required unless the lab is extended for new use cases.
