# Lessons Learned – MSH-SOC-001

## Why the phishing worked

- **Convincing lure** – a fake Microsoft security alert scares people into acting fast
- **Email filtering didn't stop it** – all 8 emails failed SPF and DMARC but were delivered. The same filter quarantined 6 other emails, so DMARC failures aren't being enforced consistently.
- **Proxy didn't stop it** – the domains were flagged as `newly_registered` but still allowed
- **Redirect to the real Microsoft page** after the POST meant Dana had no reason to think anything went wrong

## Was MFA enough?

There's no sign of MFA blocking the attacker – the login went straight from failure to success with no challenge recorded. Either MFA wasn't enforced on this account, or it was bypassed. Either way, it didn't protect the account here.

Recommendations:
- Enforce MFA for all users, finance first
- Move to phishing-resistant MFA (FIDO2 / Windows Hello / number matching)
- Conditional access to block logins from unmanaged devices or outside expected countries

## Detection and response speed

- Credentials submitted at 01:05, attacker in at 01:27, internal phish at 01:35 – **30 minutes from click to spread**
- In the scenario brief the SOC was alerted 48 minutes after the attacker logged in. By then the internal phishing had already gone out.
- Faster detection on the login itself would have stopped the internal phishing

## Detection rules I'd add

These are in [../queries/investigation-queries.spl](../queries/investigation-queries.spl) and saved as Splunk alerts.

| Detection | Logic | MITRE |
|---|---|---|
| Failed → successful login from a new country | ≥3 failures then a success from a non-US location | T1078.004 |
| DMARC fail but delivered | dmarc=fail AND action=delivered | T1566.002 |
| Credential POST to newly registered domain | proxy method=POST AND category=newly_registered | T1566.002 |
| Internal email burst | one internal sender, ≥5 recipients in 5 minutes | T1534 |
| New inbox rule (needs audit logs) | New-InboxRule / Set-InboxRule events | T1564.008 |

## SOAR / automation ideas

A playbook that triggers on the "failed → successful login from new country" alert:
1. Enrich the IP (AbuseIPDB, VirusTotal, geolocation)
2. Check if the user had a recent POST to a newly registered domain
3. If both are true → automatically revoke sessions and disable the account, then page Tier 2
4. Search for the same sender/subject in other mailboxes and auto-purge

This would cut response time from ~48 minutes to a couple of minutes.

## Visibility gaps to fix

- Collect M365 **Unified Audit Log** (mailbox rules, forwarding, OAuth consents)
- Collect **endpoint/EDR** telemetry
- Consider logging off-network browsing (e.g. via a cloud proxy / SWG) so mobile and home activity is visible

## What I'd do differently as an analyst

- Pivot on the attacker IP earlier – it would have shown the full campaign straight away
- Always sanity-check a search that returns 0 results – once my time window was wrong, not the data
- Check user working hours before setting time windows
