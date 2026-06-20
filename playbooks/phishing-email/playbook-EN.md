# Incident Response Playbook — Phishing Email

**Author:** Abraham Diaco
**Last updated:** 2026-06-20
**Severity:** Medium to High (depends on user interaction)
**MITRE ATT&CK:** T1566.001 / T1566.002 — Phishing: Spearphishing Attachment / Link

---

## 1. Overview

Phishing remains one of the most common initial access vectors. This playbook covers the response when a phishing email is reported by a user or detected by email security tooling, including cases where the user may have clicked a link, opened an attachment, or submitted credentials.

## 2. Detection

**Indicators of Compromise (IoCs):**
- User-reported suspicious email (via "Report Phishing" button or helpdesk ticket)
- Email gateway / secure email gateway (SEG) alert — spoofed sender, suspicious attachment, known malicious URL
- Anomalous mail flow patterns (e.g. mass send from a single mailbox, lookalike domain)
- Sign-in anomaly shortly after the email was opened (new location, new device, impossible travel)

**Example indicators (sanitized):**
```
Sender: support@micros0ft-secure.com  (lookalike domain)
Subject: "Action Required: Verify Your Account"
Attachment: invoice_2026.html.zip
```

**Detection time target:** < 15 minutes from user report to triage start

## 3. Triage

Questions to answer before escalating:
- Did the user click the link, open the attachment, or enter credentials?
- Is this a single targeted user or a mass campaign across the organization?
- Does the email match a known phishing campaign (check threat intel feeds)?
- Is the sender domain/IP already flagged elsewhere?

**Severity classification criteria:**
| Severity | Criteria |
|----------|----------|
| Low      | Reported but not opened/clicked; clearly identifiable as phishing |
| Medium   | Link clicked but no credentials entered, no attachment executed |
| High     | Credentials entered on a phishing page, or attachment executed |
| Critical | Confirmed account compromise or malware execution following the click |

## 4. Containment

**Immediate actions (first 15 minutes):**
1. Identify all recipients of the email across the organization (search mail logs/SIEM).
2. Remove the email from all inboxes (quarantine via email security tooling).
3. Block the sender domain/IP and any malicious URLs at the email gateway and web proxy/firewall.

**Short-term containment:**
- If credentials were submitted: force immediate password reset and invalidate active sessions/tokens for the affected account.
- If an attachment was executed: isolate the affected endpoint from the network pending investigation (see Ransomware/Malware playbook if applicable).
- Notify affected users and provide guidance (do not click further links, report any unusual activity).

## 5. Eradication

- Confirm no persistence mechanism was installed (check scheduled tasks, browser extensions, mailbox rules — attackers often create forwarding rules to exfiltrate future emails).
- Remove any malicious mailbox rules or OAuth app grants created by the attacker.
- Block the phishing infrastructure (domain, IP, URL) at all layers (email, web, DNS).

## 6. Recovery

- Restore the affected endpoint from a clean image if malware was executed.
- Re-enable the user account with new credentials and, where possible, enforce MFA.
- Monitor the affected account/endpoint closely for 7–14 days for signs of follow-on activity.

## 7. Post-Incident

- Document the full timeline: email delivery time, report time, containment time, number of affected users.
- Share IoCs (sender domain, URLs, attachment hash) with the security team and update email filtering rules.
- Consider targeted awareness training for affected users or department.
- Track campaign-level metrics if multiple phishing incidents are linked (recurring sender infrastructure, themes).

## 8. References

- MITRE ATT&CK T1566.001: https://attack.mitre.org/techniques/T1566/001/
- MITRE ATT&CK T1566.002: https://attack.mitre.org/techniques/T1566/002/
- Related playbook: Ransomware / Malware Detection & Containment
