# MediSure Health Network – Microsoft 365 Account Compromise Investigation

**Role:** SOC Tier 1 Analyst (training engagement)
**Project:** AMDARI – Introduction to SOC, "The Tuesday Morning Phish"
**Analyst:** Ayotunde Victor Babade
**Tools:** Splunk Enterprise, VirusTotal, AbuseIPDB, MITRE ATT&CK

> All data in this project is synthetic and was provided for training. MediSure Health Network is a fictional organisation.

---

## Summary

A phishing email pretending to be a Microsoft security alert was sent to 8 MediSure staff. One finance user, **dana.prescott**, clicked the link and entered her Microsoft 365 password on a fake login page. About 20 minutes later an attacker logged into her account from Amsterdam, opened her mailbox in Outlook Web App and used it to send a payment-themed phishing email ("Updated remittance – please review") to 5 colleagues.

I confirmed the compromise by correlating three log sources (sign-in, mail and web proxy) in Splunk. Two other users also clicked the link but there's no evidence they submitted credentials.

**Verdict:** Confirmed account compromise – High severity.

---

## Scenario

MediSure is a 650-person healthcare organisation running Microsoft 365, Windows endpoints and VPN access, with an internal SOC. The SOC's job is to work out whether suspicious activity is a real threat, escalate it and document the response.

The question I had to answer: is this an isolated suspicious login, or an actual account compromise that's spreading?

---

## Data

| Source | Sourcetype | Events | What it shows |
|---|---|---|---|
| Microsoft 365 sign-in log | `medisure:signin` | 136 | Logins – user, IP, location, result, device |
| Email gateway log | `medisuremail` | 249 | Emails – sender, recipient, subject, SPF/DKIM/DMARC, action |
| Web proxy log | `medisureproxy` | 368 | Web traffic – user, site visited, GET/POST, category |

All events are from 16 August 2023. Loaded into Splunk index `medisure`.

**Not available:** endpoint/EDR telemetry and mailbox audit logs. See [Gaps and limitations](#gaps-and-limitations).

---

## How I approached it

I followed the 12-phase SOC workflow from the project brief:

| Phase | What I did |
|---|---|
| 1. Visibility | Ingested the 3 logs into Splunk, checked event counts per source, noted missing sources |
| 2. Alert generation | Looked for unusual locations, failed→successful logins, email anomalies |
| 3. Triage | Built a baseline of normal logins, identified dana.prescott as the outlier |
| 4. Investigation | Traced the attack backwards (how did they get in?) and forwards (what did they do?) |
| 5. Timeline | Combined all 3 sources into one time-ordered view |
| 6. Validation | Tested alternative explanations before confirming |
| 7. Escalation | Wrote a Tier 2 escalation note |
| 8–10. Containment, eradication, recovery | Documented response actions and checks |
| 11. Documentation | This repo |
| 12. Lessons learned | Control gaps and detection ideas |

### Event vs alert vs incident (using this case)

- **Event:** any single log line, e.g. a login from Amsterdam.
- **Alert:** a pattern a tool or analyst flags as suspicious, e.g. 3 failed logins then a success from a new country on an unmanaged device.
- **Incident:** an alert confirmed as malicious with evidence, e.g. the login IP matches the phishing email's sender, the user submitted credentials to a fake page, and the account then sent phishing internally.

---

## Key findings

### 1. Normal login behaviour (baseline)
Almost every user logs in from New York via `198.51.100.25` (the office's public IP) on corp-managed devices. Occasional single failures are normal typos.

![Sign-in baseline](screenshots/02-signin-baseline.png)

### 2. Dana's account – suspicious logins
- 00:02 – normal login from New York, corp-managed device
- 01:22–01:25 – 3 failed logins (`50126` = wrong username/password) from **89.144.44.41, Amsterdam**, unknown device
- 01:27 – **successful** login from the same IP
- 01:29 – Microsoft Office, 01:33 – **Outlook Web App**

New York to Amsterdam in 80 minutes = impossible travel.

![Dana sign-in timeline](screenshots/03-dana-signin-timeline.png)

### 3. The phishing email
- From: `no-reply@access-accsecurity.com` (lookalike, not Microsoft)
- Reply-to: `solutionteamrecognizd03@gmail.com` (from/reply-to mismatch)
- Subject: "Microsoft account unusual sign in activity"
- Sending IP: **89.144.44.41** – the same IP that later logged into Dana's account
- SPF fail, DKIM none, DMARC fail – **still delivered**

![Phishing email](screenshots/04-phishing-email.png)

### 4. It was a campaign, not a one-off
The same email went to 8 users between 00:11 and 00:20, all delivered.

![Campaign recipients](screenshots/06-phishing-campaign-recipients.png)

### 5. Click and credential submission
Proxy logs show Dana (`10.20.11.45`) clicking a tracking link on `thebandalisty.com`, getting redirected to the fake login page on `account-verify.access-accsecurity.com`, then a **POST** to `/session/login` at 01:05 – that's her submitting her password. She was then bounced to the real Microsoft login page so nothing looked wrong.

Marcus Hale and Priya Nandakumar also clicked and reached the fake page, but there's **no POST** for either of them.

![Proxy – who clicked](screenshots/07-proxy-who-clicked.png)

### 6. Internal phishing from Dana's mailbox
Two minutes after the attacker opened OWA, Dana's account sent "Updated remittance – please review" to 5 colleagues in 28 seconds. These are the only emails her account sent all day. They went through MediSure's own mail server (`10.20.1.10`) so they passed SPF/DKIM/DMARC.

None of the 5 recipients visited any suspicious sites afterwards (checked their full-day proxy activity).

![Internal phishing](screenshots/08-internal-phishing.png)

### 7. Correlated view
One search pulling all three sources into a single timeline:

![Correlated timeline](screenshots/11-correlated-timeline.png)

Full timeline: [analysis/timeline.md](analysis/timeline.md)
All Splunk queries: [queries/investigation-queries.spl](queries/investigation-queries.spl)

---

## Verdict

**Confirmed account compromise of dana.prescott@medisurehealthnetwork.com – Severity: High**

Why I'm confident:
1. The phishing email and the malicious logins came from the **same IP** (89.144.44.41)
2. The proxy shows Dana **submitting credentials** to the fake page before the attacker logged in
3. The attacker's session was followed by **internal phishing** from her mailbox

Alternatives I ruled out:
- **Dana travelling** – she logged in from New York 80 minutes earlier and again at 06:34, always on her corp device
- **VPN** – the Amsterdam logins came from an unknown device, not her managed laptop, and the IP is the phishing sender
- **Dana sending the remittance emails herself** – sent at 01:35 during the attacker's session, 7 seconds apart, and her account sent nothing else all day

Full reasoning: [reports/incident-report.md](reports/incident-report.md)

---

## Response

- Escalation note to Tier 2: [reports/tier2-escalation-note.md](reports/tier2-escalation-note.md)
- Containment, eradication and recovery: [reports/containment-recovery-checklist.md](reports/containment-recovery-checklist.md)

---

## MITRE ATT&CK mapping

| Technique | ID | Evidence |
|---|---|---|
| Phishing: Spearphishing Link | T1566.002 | Fake Microsoft alert sent to 8 users |
| User Execution: Malicious Link | T1204.001 | Dana, Marcus and Priya clicked |
| Valid Accounts: Cloud Accounts | T1078.004 | Attacker logged in with Dana's real credentials |
| Internal Spearphishing | T1534 | "Updated remittance" sent from Dana's mailbox |
| Hide Artifacts: Email Hiding Rules | T1564.008 | Described in scenario – **not verified** (no audit logs) |

---

## Gaps and limitations

- **No mailbox audit logs** – the scenario mentions an inbox rule being created, but the mail log only records delivery (`delivered` / `quarantined`). I couldn't verify the rule. Needs the M365 Unified Audit Log.
- **No endpoint/EDR data** – couldn't check Dana's laptop for anything left behind.
- **Proxy only covers the corporate network** – if anyone opened the email on a personal phone, it wouldn't show here. So for Marcus and Priya I can only say "no evidence of credential submission", not "definitely didn't".
- **Threat intel** – see [analysis/threat-intel.md](analysis/threat-intel.md).

---

## Lessons learned

Short version:
- Email filtering let 8 DMARC-failing emails through
- Proxy allowed newly registered domains
- Nothing stopped a login from a new country on an unmanaged device
- Mailbox audit logs weren't being collected

Full write-up and detection ideas: [reports/lessons-learned.md](reports/lessons-learned.md)

---

## Repo structure

```
├── README.md
├── case/case-record.md
├── analysis/
│   ├── timeline.md
│   └── threat-intel.md
├── queries/investigation-queries.spl
├── reports/
│   ├── incident-report.md
│   ├── tier2-escalation-note.md
│   ├── containment-recovery-checklist.md
│   └── lessons-learned.md
└── screenshots/
```

---

## What I took from this

The biggest thing for me was not trusting a single log. The sign-in log on its own just shows a weird login. It only became a confirmed incident once I tied it to the phishing email's IP and the proxy POST. I also learned to sanity-check searches that return nothing – at one point I got 0 results because my time window was wrong, not because nothing happened.
