# Case Record – MSH-SOC-001

| Field | Detail |
|---|---|
| Case ID | MSH-SOC-001 |
| Analyst | Ayotunde Victor Babade (Tier 1) |
| Affected user | dana.prescott@medisurehealthnetwork.com (Finance) |
| Other users involved | marcus.hale, priya.nandakumar (clicked); tom.becker, alicia.romero, grace.lin, rachel.obi, daniel.kwon (internal phish) |
| Source | Splunk – index medisure (signin, mail, proxy) |
| Severity | High |
| Status | Confirmed – escalated to Tier 2 |

## Hypothesis (Day 1)
A finance user's M365 account may have been compromised through phishing, with the attacker using the mailbox to hide activity and send phishing internally. Treated as a working theory, not a conclusion.

## Log

**Visibility**
- Ingested 3 sources: signin (136), mail (249), proxy (368). All events 16 Aug 2023.
- Gaps: no endpoint/EDR, no mailbox audit logs.

**Alert generation / triage**
- Baseline: users log in from New York via 198.51.100.25.
- Outlier: dana.prescott – 3 failed + 3 successful logins from 89.144.44.41, Amsterdam, unknown device. Impossible travel from NY.
- Triage decision: escalate for investigation – failed→success pattern, new country, unmanaged device, mailbox access.

**Investigation**
- Phishing email to Dana at 00:11 from no-reply@access-accsecurity.com, src 89.144.44.41 – same IP as the logins.
- Dana clicked at 00:34, submitted credentials at 01:05 (proxy POST).
- Campaign: 8 recipients. marcus.hale and priya.nandakumar clicked, no POST.
- Attacker sent "Updated remittance – please review" to 5 internal users at 01:35.
- Internal recipients' proxy activity: business only, nothing suspicious.
- Inbox rule: not in telemetry – documented as a gap.

**Validation**
- Confirmed account compromise. Alternatives (travel, VPN, legitimate sending) ruled out.

**Escalation**
- Tier 2 escalation note written. Identity, Email admins, Finance, SOC manager and Privacy/compliance to be notified.

## Actions recommended
See reports/containment-recovery-checklist.md

## Outstanding
- Inbox rule / forwarding / OAuth check
- Threat intel lookups to complete
- Confirm Marcus and Priya didn't submit credentials off-network
- Privacy assessment of mailbox contents
