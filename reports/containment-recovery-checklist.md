# Containment, Eradication and Recovery Checklist

Case MSH-SOC-001 – dana.prescott account compromise

## Containment – stop the damage

| # | Action | Why | Owner | Done |
|---|---|---|---|---|
| 1 | Disable dana.prescott's account | Stops any further logins | Identity | ☐ |
| 2 | Revoke all sessions and refresh tokens | A password reset alone doesn't kill sessions the attacker already has | Identity | ☐ |
| 3 | Block IP 89.144.44.41 (conditional access / firewall) | Attacker's known IP | Identity / Network | ☐ |
| 4 | Block access-accsecurity.com and thebandalisty.com at the proxy | Stops anyone else reaching the phishing page | Network | ☐ |
| 5 | Block sender and domain at the email gateway | Stops more phishing coming in | Email admin | ☐ |
| 6 | Purge the external phishing email from all 8 mailboxes | Removes the lure | Email admin | ☐ |
| 7 | Purge "Updated remittance – please review" from 5 internal mailboxes | Internal phish from a trusted sender is more dangerous | Email admin | ☐ |
| 8 | Precautionary password reset for marcus.hale and priya.nandakumar | They clicked; can't rule out submission off-network | Identity | ☐ |
| 9 | Alert Finance to verify payment/bank change requests by phone | Remittance lure points to payment fraud | SOC / Finance | ☐ |

## Eradication – remove anything the attacker left

| # | Check | Done |
|---|---|---|
| 1 | Review Dana's inbox rules (Get-InboxRule) and remove anything hiding, moving or deleting mail | ☐ |
| 2 | Check for mail forwarding to external addresses | ☐ |
| 3 | Review OAuth app consents granted to Dana's account and remove anything unknown | ☐ |
| 4 | Check for new MFA methods registered by the attacker and remove them | ☐ |
| 5 | Review Dana's Sent Items and Deleted Items for anything else the attacker sent or deleted | ☐ |
| 6 | Check Dana's laptop (10.20.11.45) for anything downloaded from the phishing site – no EDR data available in this case | ☐ |

## Recovery – return to normal safely

| # | Check | Done |
|---|---|---|
| 1 | Reset Dana's password (strong, not reused) | ☐ |
| 2 | Re-register MFA with Dana on a verified device | ☐ |
| 3 | Re-enable the account once eradication checks are clear | ☐ |
| 4 | Confirm with Dana that her mailbox looks normal (no missing mail, odd rules) | ☐ |
| 5 | Phishing awareness follow-up for Dana, Marcus and Priya | ☐ |

## Monitoring for re-entry (next 30 days)

- Alert on any login to dana.prescott from outside the US or from an unmanaged device
- Alert on any activity from 89.144.44.41 or the phishing domains
- Alert on new inbox rules or forwarding on Dana's mailbox
- Watch marcus.hale and priya.nandakumar sign-ins for unusual locations
