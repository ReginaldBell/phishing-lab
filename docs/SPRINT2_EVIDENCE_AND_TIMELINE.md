# Sprint 2 Evidence And Timeline

Updated: `2026-03-11`

## Purpose

This document consolidates the March 11, 2026 phishing validation evidence into a single timeline and evidence summary for Sprint 2.

Primary validated user-driven run:

- Campaign ID: `2`
- Campaign name: `Sprint 1 Validation 2026-03-11`
- RID: `tW15oVY`
- Recipient: `labuser@example.local`
- Workstation IP: `192.168.56.20`

Secondary automation-assisted comparison run:

- Campaign ID: `3`
- Campaign name: `Sprint 1 Validation Immediate`
- RID: `8Ypz6k6`

## Consolidated Timeline

### Primary Manual Validation Path

1. `2026-03-11 05:48:38 UTC`
   GoPhish created campaign `Sprint 1 Validation 2026-03-11`.

2. `2026-03-11 05:55:21 UTC`
   GoPhish recorded `Email Sent` to `labuser@example.local` for campaign `2`.

3. `2026-03-11 05:57:55`
   `mail01` Dovecot LMTP logged delivery of a GoPhish message into the `labuser` `INBOX`.

4. `2026-03-11 05:59:54 UTC`
   GoPhish recorded `Clicked Link` for RID `tW15oVY`.

5. `2026-03-11 05:59:54 UTC`
   `gophish01` logged:
   `GET /?rid=tW15oVY` from `192.168.56.20`

6. `2026-03-11 05:59:55 UTC`
   `gophish01` logged a follow-up `GET /favicon.ico` from `192.168.56.20` with referrer `http://192.168.56.60/?rid=tW15oVY`.

7. `2026-03-11 05:59:55`
   `mail01` Dovecot LMTP logged another GoPhish message saved to `INBOX`.

8. `2026-03-11 06:03:55`
   `mail01` Dovecot IMAP logged successful `labuser` authentication from `192.168.56.20`.

### Secondary Automated Comparison Path

1. `2026-03-11 05:49:17 UTC`
   GoPhish created campaign `Sprint 1 Validation Immediate`.

2. `2026-03-11 05:53:21 UTC`
   GoPhish recorded `Email Sent` for campaign `3`.

3. `2026-03-11 05:54:39 UTC`
   GoPhish recorded `Clicked Link` for RID `8Ypz6k6` from `192.168.56.20`.

This secondary run confirms that fresh click traffic from `wkstn01` was observable by GoPhish during the same validation window, but the primary evidence path for user behavior remains campaign `2`.

## Evidence By Source

### GoPhish

Campaign `2` evidence:

- `Campaign Created`: `2026-03-11T05:48:38.754416115Z`
- `Email Sent`: `2026-03-11T05:55:21.868676249Z`
- `Clicked Link`: `2026-03-11T05:59:54.908347442Z`
- result status: `Clicked Link`
- click source IP: `192.168.56.20`
- click browser: `Mozilla/5.0 ... Edg/145.0.0.0`

Campaign `3` evidence:

- `Campaign Created`: `2026-03-11T05:49:17.366488608Z`
- `Email Sent`: `2026-03-11T05:53:21.867347337Z`
- `Clicked Link`: `2026-03-11T05:54:39.125612318Z`
- result status: `Clicked Link`
- click source IP: `192.168.56.20`

GoPhish service log excerpts confirmed:

- `GET /?rid=tW15oVY` from `192.168.56.20`
- `GET /favicon.ico` from `192.168.56.20`

### Mail Host

`mail01` Dovecot evidence confirmed:

- LMTP save to `INBOX` at `05:57:55`
- LMTP save to `INBOX` at `05:59:55`
- IMAP login by `labuser` from `192.168.56.20` at `06:03:55`

This is sufficient to show that the mailbox host processed delivery and later observed workstation mailbox access from the correct source IP.

### Wazuh

Agent health during the run:

- `WKSTN01`: `Active`
- `mail01`: `Active`
- `gophish01`: `Active`

Observed alert quality:

- Wazuh showed generic Windows logon events from `WKSTN01`
- Wazuh showed normal SIEM host SSH/PAM activity
- Wazuh did not produce a phishing-specific alert for:
  - mail delivery
  - mailbox access
  - GoPhish RID click
  - cross-host correlation of the scenario

## Stored Visual Evidence

- Thunderbird inbox and opened phishing email:
  [docs/assets/screenshots/wkstn01-sprint1-thunderbird-2026-03-11.png](/c:/Users/demar/Downloads/Phishing-Lab/docs/assets/screenshots/wkstn01-sprint1-thunderbird-2026-03-11.png)

- Landing page after manual click:
  [docs/assets/screenshots/wkstn01-sprint1-browser-2026-03-11.png](/c:/Users/demar/Downloads/Phishing-Lab/docs/assets/screenshots/wkstn01-sprint1-browser-2026-03-11.png)

## Sprint 2 Conclusion

Sprint 2 established a consolidated evidence package for the validated phishing scenario.

The evidence now supports the following defensible statement:

- GoPhish sent a controlled phishing email to `labuser@example.local`
- `mail01` delivered that message to the mailbox
- `wkstn01` accessed the mailbox from `192.168.56.20`
- the phishing link was clicked from `192.168.56.20`
- GoPhish recorded the click and the browser details
- Wazuh infrastructure was healthy, but detection content still did not elevate the scenario meaningfully

This sets up Sprint 3 directly: detection engineering for Postfix, Dovecot, GoPhish, and cross-host correlation.
