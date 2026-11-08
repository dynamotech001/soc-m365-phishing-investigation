# Incident Timeline – MSH-SOC-001

All times are from the logs, 16 August 2023 (UTC+01:00).

| Time | Event | Source | Notes |
|---|---|---|---|
| 00:02:30 | dana.prescott logs in from New York (198.51.100.25), corp-managed device | signin | Her normal pattern |
| 00:11:12 | Phishing email delivered to dana.prescott | mail | From no-reply@access-accsecurity.com, src 89.144.44.41, SPF/DMARC fail |
| 00:12:40 – 00:20:19 | Same email delivered to 7 more users | mail | marcus.hale, priya.nandakumar, tom.becker, alicia.romero, kevin.osei, nadia.farrell, sam.whitaker |
| 00:34:50 | Dana clicks link – thebandalisty.com/track/... | proxy | From her laptop 10.20.11.45. Domain categorised newly_registered, allowed |
| 00:34:56 | Redirected to fake login page account-verify.access-accsecurity.com/session/login | proxy | |
| 00:41:12 – 00:41:18 | marcus.hale clicks and reaches fake login page | proxy | No POST seen |
| 00:58:37 – 00:58:43 | priya.nandakumar clicks and reaches fake login page | proxy | No POST seen |
| 01:05:12 | **Dana submits credentials** (POST /session/login) | proxy | Key event |
| 01:05:14 | Dana redirected to real login.microsoftonline.com | proxy | Makes the phish look legit |
| 01:22:09 – 01:25:02 | 3 failed logins to Dana's account from 89.144.44.41, Amsterdam | signin | resultType 50126, unknown device |
| 01:27:41 | **Attacker logs in successfully** – Exchange Online | signin | Same IP as the phishing sender |
| 01:29:15 | Attacker accesses Microsoft Office | signin | |
| 01:33:50 | Attacker accesses Outlook Web App | signin | Mailbox access |
| 01:35:10 – 01:35:38 | **Internal phishing** – "Updated remittance – please review" sent from Dana's mailbox to tom.becker, alicia.romero, grace.lin, rachel.obi, daniel.kwon | mail | 5 emails in 28 seconds via internal relay 10.20.1.10 |
| 06:34:22 onwards | Dana logs in normally from New York, corp-managed | signin | Real user, likely unaware |
| 07:02 – 07:35 | Internal phishing recipients start their day | proxy | No visits to suspicious domains all day |

## Gaps and uncertainties

- **Inbox rule creation** – mentioned in the scenario but not present in the provided logs. Can't put a time on it.
- **Gap between credential submission (01:05) and first login attempt (01:22)** – 17 minutes. Likely the attacker collecting and testing harvested credentials.
- **3 failed logins before success** – possibly Dana mistyped her password on the fake page and the attacker tried variations, or the attacker tried other details first. Can't confirm from the logs.
- **No attacker sign-ins after 01:33 in the dataset** – but session tokens may still be valid, so containment must revoke sessions.
- **Detection time** – the dataset doesn't include SIEM alert times. In the scenario brief, the SOC was alerted 48 minutes after the attacker's login.
