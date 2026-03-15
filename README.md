# phishing-lab
A phishing detection engineering lab using GoPhish, Postfix/Dovecot, Wazuh, and a Windows workstation to validate delivery, click, correlation, and incident response workflows.

## Managing Sensitive Data

This repository intentionally **does not** store secrets, passwords, IP addresses, or any local-only lab details.  
The following measures are in place to prevent accidental exposure:

| File / Pattern | Purpose |
|---|---|
| `.gitignore` | Excludes `.env`, TLS keys/certs, GoPhish DB & config, Postfix `sasl_passwd`, Wazuh keys, and other sensitive files from being tracked |
| `.env.example` | Documents every required environment variable with placeholder values — copy it to `.env` and fill in real values locally |

### Quick-start

```bash
cp .env.example .env
# Edit .env with your real IP addresses, credentials, and keys
$EDITOR .env
```

### What must never be committed

- `.env` (real credentials and IP addresses)
- `gophish/config.json` (admin credentials and API key)
- `postfix/sasl_passwd` / `postfix/sasl_passwd.db` (SMTP relay credentials)
- `dovecot/passwd` (mailbox passwords)
- `wazuh/client.keys` / `wazuh/authd.pass` (agent registration secrets)
- Any `*.pem`, `*.key`, `*.crt`, `*.p12`, or `*.pfx` file (TLS material)
- Personal lab notes containing IP addresses or hostnames (use the `notes/local-*` naming convention and they will be ignored automatically)

If you believe a secret was committed by mistake, rotate it immediately and consider the old value compromised.
