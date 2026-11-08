# Incident Report – MSH-SOC-001

| | |
|---|---|
| **Incident** | Microsoft 365 account compromise via credential phishing |
| **Affected user** | dana.prescott@medisurehealthnetwork.com (Finance) |
| **Date of activity** | 16 August 2023 |
| **Severity** | High |
| **Verdict** | Confirmed malicious |
| **Analyst** | Ayotunde Victor Babade, SOC Tier 1 |

---

## 1. What happened

An external attacker sent a fake "Microsoft account unusual sign in activity" email to 8 MediSure staff. Dana Prescott clicked the link, landed on a fake Microsoft login page and entered her password. The attacker used those credentials to log into her account from Amsterdam, opened her mailbox in Outlook Web App and sent a payment-themed phishing email to 5 colleagues from her account.

## 2. How I found it

1. **Baseline** – grouped all logins by user, location, IP and result. Everyone logs in from New York via 198.51.100.25. Dana was the only user with logins from another country.
2. **Dana's logins** – 3 failures then a success from 89.144.44.41 (Amsterdam), unknown device, 80 minutes after a normal New York login. Then OWA access.
3. **How they got in** – searched emails to Dana before the login. Found the phishing email, sent from the same IP 89.144.44.41.
4. **Did she click** – proxy logs show the click, the fake login page and a POST to /session/login (credentials submitted).
5. **Scope** – pivoted on the attacker IP across all logs. The phishing went to 8 users. Proxy shows 3 clicked, only Dana submitted.
6. **What the attacker did** – searched emails sent from Dana's account. 5 internal "Updated remittance" emails sent 2 minutes after OWA access.
7. **Spread check** – reviewed the 5 internal recipients' full-day proxy activity. All business sites, nothing suspicious.

## 3. Evidence

| Finding | Source | Evidence |
|---|---|---|
| Phishing email delivered | mail | From no-reply@access-accsecurity.com, src 89.144.44.41, SPF fail / DKIM none / DMARC fail, delivered |
| Click | proxy | 00:34:50 GET thebandalisty.com/track/... from 10.20.11.45 |
| Credential submission | proxy | 01:05:12 POST account-verify.access-accsecurity.com/session/login |
| Attacker login | signin | 01:27:41 success from 89.144.44.41, Amsterdam, device unknown |
| Mailbox access | signin | 01:33:50 OutlookWebApp |
| Internal phishing | mail | 01:35:10–01:35:38, 5 emails, "Updated remittance – please review" |

Timeline: [../analysis/timeline.md](../analysis/timeline.md)

## 4. Validation

**Supporting malicious activity:**
- Same IP sent the phishing email and logged into the account – direct link between the two
- Proxy POST proves credentials were submitted before the attacker's login
- Impossible travel (NY 00:02 → Amsterdam 01:22)
- Unknown device vs Dana's usual corp-managed device
- Internal emails sent during the attacker's session, 7 seconds apart, the only emails Dana's account sent all day

**Alternatives considered:**
- *Dana travelling* – ruled out. NY logins before and after, always on her corp device.
- *Corporate VPN exit in Amsterdam* – ruled out. Corporate traffic comes from 198.51.100.25 and the device wasn't managed.
- *Dana genuinely sending the remittance emails* – ruled out. Timing sits inside the attacker's session at 01:35 and the pattern is automated.

**Conclusion:** Confirmed account compromise, supported by three independent log sources.

## 5. Impact

- **Confirmed compromised:** dana.prescott
- **At risk (clicked, no submission seen):** marcus.hale, priya.nandakumar
- **Received external phish:** 8 users
- **Received internal phish:** tom.becker, alicia.romero, grace.lin, rachel.obi, daniel.kwon
- **Data exposure:** attacker had access to Dana's mailbox for an unknown period. As a finance user in a healthcare org, her mailbox may contain financial and patient-related data. Needs assessment by privacy/compliance.
- **Business risk:** the remittance lure points to an attempt at payment fraud using a trusted internal sender.

## 6. Actions

Escalated to Tier 2 – see [tier2-escalation-note.md](tier2-escalation-note.md).
Containment, eradication and recovery – see [containment-recovery-checklist.md](containment-recovery-checklist.md).

## 7. Limitations

- No mailbox audit logs – inbox rule creation (mentioned in the scenario) could not be verified
- No endpoint/EDR data – couldn't check Dana's laptop
- Proxy only sees corporate network traffic
- Threat intel lookups gave limited results due to synthetic data

## 8. What's still open

- Inbox rule / forwarding / OAuth check on Dana's mailbox
- Confirmation from Marcus and Priya that they didn't enter credentials anywhere
- Privacy assessment of what was in Dana's mailbox
