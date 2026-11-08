# Tier 2 Escalation Note

**Case:** MSH-SOC-001
**Severity:** High
**Status:** Confirmed – escalating for containment
**From:** Ayotunde Victor Babade, SOC Tier 1

---

**Summary**
Confirmed compromise of dana.prescott@medisurehealthnetwork.com (finance). The user entered her M365 credentials on a phishing page. The attacker logged in from 89.144.44.41 (Amsterdam), accessed OWA and sent an internal phishing email to 5 colleagues.

**Evidence**
- 00:11 – phishing email "Microsoft account unusual sign in activity" from no-reply@access-accsecurity.com, src 89.144.44.41, SPF/DMARC fail, delivered to 8 users
- 01:05 – proxy shows Dana POSTing to account-verify.access-accsecurity.com/session/login (credential submission)
- 01:22–01:27 – 3 failed logins then success on Dana's account from 89.144.44.41, Amsterdam, unknown device (her normal logins are NY, corp-managed)
- 01:33 – attacker accessed Outlook Web App
- 01:35 – 5 emails "Updated remittance – please review" sent from Dana's mailbox to tom.becker, alicia.romero, grace.lin, rachel.obi, daniel.kwon

**Wider exposure**
- marcus.hale and priya.nandakumar clicked the link and reached the fake page – no credential submission seen in proxy logs
- 5 other users received the phishing email but didn't click (per proxy)

**Not verified**
- Inbox rule creation – no mailbox audit logs in the data. Please check Dana's inbox rules directly.

**Recommended immediate actions**
1. Disable Dana's account and revoke all sessions/tokens
2. Reset her password and re-register MFA
3. Check and remove any inbox rules, forwarding and OAuth app consents
4. Purge the external phishing email (8 mailboxes) and the internal "Updated remittance" email (5 mailboxes)
5. Block 89.144.44.41, access-accsecurity.com and thebandalisty.com
6. Precautionary password reset for marcus.hale and priya.nandakumar
7. Warn Finance to verify any payment or bank detail change requests by phone – the "remittance" lure suggests a payment fraud attempt

**Who else needs to know**
- Identity team – account disable, session revocation, MFA
- Email/Exchange admins – purge, inbox rule check, sender blocks
- Finance lead – possible payment fraud attempt from a trusted internal sender
- SOC manager – High severity incident
- Privacy/compliance – the attacker had mailbox access in a healthcare org, so possible exposure of patient or sensitive data needs assessing

**Full details:** see incident report and timeline in the case record.
