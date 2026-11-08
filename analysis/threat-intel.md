# Threat Intelligence Enrichment

Indicators looked up on VirusTotal and AbuseIPDB. I searched the indicators only – I didn't visit any of the sites.

## Indicators of Compromise (IOCs)

| Type | Indicator | Context | VirusTotal | AbuseIPDB |
|---|---|---|---|---|
| IP | 89.144.44.41 | Sent the phishing emails and logged into Dana's account | _fill in_ | _fill in_ |
| Domain | access-accsecurity.com | Phishing sender domain | _fill in_ | n/a |
| Domain | account-verify.access-accsecurity.com | Fake Microsoft login page | _fill in_ | n/a |
| Domain | thebandalisty.com | Tracking/redirect link in the email | _fill in_ | n/a |
| Email | no-reply@access-accsecurity.com | Phishing sender | n/a | n/a |
| Email | solutionteamrecognizd03@gmail.com | Reply-to address | n/a | n/a |
| Subject | Microsoft account unusual sign in activity | External phishing lure | n/a | n/a |
| Subject | Updated remittance – please review | Internal phishing lure | n/a | n/a |

## Notes

_Example if nothing comes back:_ The indicators returned little or no reputation data, which is expected because the dataset is synthetic. This doesn't change the verdict. The contextual evidence is strong on its own: the same IP sent the phishing and logged into the account, the domains were newly registered, and the user submitted credentials. Threat intel adds weight to an investigation but doesn't replace it.

## Recommended blocks

- Block 89.144.44.41 (conditional access / firewall)
- Block access-accsecurity.com and thebandalisty.com at the proxy and email gateway
- Block sender no-reply@access-accsecurity.com
