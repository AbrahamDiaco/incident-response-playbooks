# Incident Response Playbook — Data Exfiltration

**Author:** Abraham Diaco
**Last updated:** 2026-06-20
**Severity:** Critical
**MITRE ATT&CK:** T1041 — Exfiltration Over C2 Channel / T1567 — Exfiltration Over Web Service

---

## 1. Overview

Data exfiltration is the unauthorized transfer of data out of an organization's environment, whether by an external attacker, a compromised account, or a malicious/negligent insider. This playbook covers detection and response when exfiltration is suspected or confirmed.

## 2. Detection

**Indicators of Compromise (IoCs):**
- DLP (Data Loss Prevention) alert for sensitive data leaving the network (PII, credentials, intellectual property)
- Unusual outbound data volume from a host or account (large uploads to cloud storage, file-sharing sites, or unknown external IPs)
- Use of unsanctioned cloud storage / file-sharing services (Mega, unsanctioned Dropbox/Drive instances) detected via proxy/firewall logs
- Compression of large data sets shortly before an outbound transfer (`.zip`, `.rar`, `.7z` archives staged in temp directories)
- Abnormal access to file shares/databases outside of a user's normal job function or working hours

**Example indicators (sanitized):**
```
Outbound transfer: 2.3 GB to unrecognized-cloud-storage[.]example via HTTPS
User: jdupont — accessed 400+ files in Finance share within 10 minutes
Archive staged: C:\Users\jdupont\AppData\Local\Temp\export_backup.zip
```

**Detection time target:** < 10 minutes from DLP/anomaly alert to triage start

## 3. Triage

Questions to answer before escalating:
- Is the activity consistent with the user's normal role and behavior, or clearly anomalous?
- What data was accessed/transferred — does it include regulated data (PII, financial, health, IP)?
- Is the account confirmed compromised, or is this potential insider activity?
- Has the transfer completed, or is it still in progress (containment window)?

**Severity classification criteria:**
| Severity | Criteria |
|----------|----------|
| Low      | Anomaly flagged but data is non-sensitive, low volume, explainable by legitimate business need |
| Medium   | Unusual transfer pattern involving internal (non-regulated) data, account behavior unclear |
| High     | Confirmed unauthorized transfer of sensitive/internal data, account likely compromised |
| Critical | Confirmed exfiltration of regulated data (PII, financial, health) or trade secrets/IP |

## 4. Containment

**Immediate actions (first 15 minutes):**
1. If transfer is in progress: block the destination IP/domain at the firewall/proxy immediately to stop ongoing data loss.
2. Disable or suspend the account involved (if compromise or insider threat is suspected).
3. Preserve evidence — do not delete logs, staged archive files, or alter the affected system before forensic capture.

**Short-term containment:**
- Isolate the affected host from the network if malware-driven exfiltration is suspected.
- Revoke active sessions/API tokens for the affected account.
- Identify and block any C2 infrastructure or exfiltration channel used (cloud storage API, DNS tunneling, email).

## 5. Eradication

- Determine root cause: compromised credentials, malware, misconfigured permissions, or insider action.
- Remove any malware or persistence mechanism involved in automating the exfiltration.
- Review and tighten access controls / least-privilege permissions on the data that was accessed.
- Close the technical vector (exposed share, weak credentials, missing DLP coverage, OAuth app over-permissioned).

## 6. Recovery

- Restore normal account access only after credentials are rotated and the compromise vector is closed.
- Re-enable systems progressively with enhanced monitoring on the previously affected account/host.
- Validate DLP rules are correctly tuned to catch similar patterns going forward.

## 7. Post-Incident

- Document the full scope: what data, how much, where it went, confirmed or suspected exfiltration.
- Determine legal/regulatory notification obligations (GDPR, sector-specific regulations) based on the data type and jurisdiction — involve legal/compliance teams.
- Conduct root cause analysis and update detection rules (DLP policies, anomaly thresholds).
- If insider threat is confirmed, follow organizational HR/legal escalation procedures.

## 8. References

- MITRE ATT&CK T1041: https://attack.mitre.org/techniques/T1041/
- MITRE ATT&CK T1567: https://attack.mitre.org/techniques/T1567/
- Related playbook: Ransomware / Malware Detection & Containment
