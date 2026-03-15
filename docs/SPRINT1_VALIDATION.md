# Sprint 1 Validation

Updated: `2026-03-11`

## Outcome

Sprint 1 baseline validation passed.

The phishing lab was revalidated with:

- healthy VM startup and service reachability
- active Wazuh agents on `WKSTN01`, `mail01`, and `gophish01`
- fresh GoPhish email delivery to `labuser@example.local`
- a fresh manual user click recorded from `wkstn01`

## Environment

- `wkstn01`: `192.168.56.20`
- `gophish01`: `192.168.56.60`
- `mail01`: `192.168.56.70`
- `siem01`: `192.168.56.103`

## Fresh Validation Campaigns

Two fresh campaigns were created during Sprint 1 validation on `2026-03-11`.

### Manual UI Validation

- Campaign ID: `2`
- Name: `Sprint 1 Validation 2026-03-11`
- RID: `tW15oVY`

Observed GoPhish timeline:

- `Campaign Created`: `2026-03-11 05:48:38 UTC`
- `Email Sent`: `2026-03-11 05:55:21 UTC`
- `Clicked Link`: `2026-03-11 05:59:54 UTC`

Recorded result:

- status: `Clicked Link`
- recipient: `labuser@example.local`
- source IP: `192.168.56.20`
- browser: `Edg/145.0.0.0`

### Automated Workstation Validation

- Campaign ID: `3`
- Name: `Sprint 1 Validation Immediate`
- RID: `8Ypz6k6`

Observed GoPhish timeline:

- `Campaign Created`: `2026-03-11 05:49:17 UTC`
- `Email Sent`: `2026-03-11 05:53:21 UTC`
- `Clicked Link`: `2026-03-11 05:54:39 UTC`

Recorded result:

- status: `Clicked Link`
- recipient: `labuser@example.local`
- source IP: `192.168.56.20`
- client: `WindowsPowerShell/5.1.19041.3803`

This second run was used only to confirm that `wkstn01` could generate a fresh request to GoPhish from the expected workstation IP. The stronger user-validation evidence for Sprint 1 is campaign `2`, where the email was opened in Thunderbird and the link was clicked manually.

## Supporting Evidence

### GoPhish

Fresh event extraction showed:

- campaign `2` click event at `2026-03-11 05:59:54 UTC`
- campaign `3` click event at `2026-03-11 05:54:39 UTC`

### Mail

`mail01` Dovecot logs showed fresh mailbox saves:

- `2026-03-11 05:57:55` saved mail to `INBOX`
- `2026-03-11 05:59:55` saved mail to `INBOX`

`mail01` also showed fresh IMAP login activity from the workstation:

- user: `labuser`
- remote IP: `192.168.56.20`
- time: `2026-03-11 06:03:55`

### Wazuh

Agent status remained healthy during validation:

- `WKSTN01`: `Active`
- `mail01`: `Active`
- `gophish01`: `Active`

## Stored Screenshots

Stored in the repo:

- Thunderbird inbox and opened phishing email on `wkstn01`: [docs/assets/screenshots/wkstn01-sprint1-thunderbird-2026-03-11.png](/c:/Users/demar/Downloads/Phishing-Lab/docs/assets/screenshots/wkstn01-sprint1-thunderbird-2026-03-11.png)
- Manual landing-page result on `wkstn01`: [docs/assets/screenshots/wkstn01-sprint1-browser-2026-03-11.png](/c:/Users/demar/Downloads/Phishing-Lab/docs/assets/screenshots/wkstn01-sprint1-browser-2026-03-11.png)
- Legacy click-test screenshot retained for comparison: [docs/assets/screenshots/wkstn01-click-test.png](/c:/Users/demar/Downloads/Phishing-Lab/docs/assets/screenshots/wkstn01-click-test.png)

## Sprint 1 Conclusion

Sprint 1 is complete because the lab now has:

- a fresh March 11, 2026 campaign creation event
- a fresh March 11, 2026 email delivery event
- a fresh March 11, 2026 manual click event from `192.168.56.20`
- active SIEM agents across the participating systems

The next step is Sprint 2: consolidate evidence and produce the final timeline package for the validated phishing scenario.
