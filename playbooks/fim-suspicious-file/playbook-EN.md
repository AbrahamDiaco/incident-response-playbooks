# Incident Response Playbook — Suspicious File Detected (FIM)

**Author:** Abraham Diaco
**Last updated:** 2026-06-20
**Severity:** Medium
**MITRE ATT&CK:** T1565.001 — Stored Data Manipulation

---

## 1. Overview

A File Integrity Monitoring (FIM) alert fires when a new, modified, or deleted file is detected in a monitored directory (e.g. `/etc`, `/bin`, `C:\Windows\System32`). This playbook covers the response when FIM detects an unauthorized or suspicious file creation, which can indicate malware drop, unauthorized configuration changes, or early-stage persistence.

Reference implementation and live metrics: [soc-lab-wazuh](https://github.com/AbrahamDiaco/soc-lab-wazuh/tree/main/scenarios/FIM-malware)

## 2. Detection

**Indicators of Compromise (IoCs):**
- Wazuh `syscheckd` alert — Rule 554 ("New file added to the system") or related FIM rule IDs
- Unexpected file in a sensitive path (`/etc`, `/bin`, `/usr/local/bin`, startup folders, cron directories)
- File with executable permissions appearing outside normal change windows
- Unusual file hash with no known publisher / signature

**Example alert (sanitized):**
```
Rule: 554 (level 5) -> 'File added to the system.'
File: '/etc/malware-test.sh'
```

**Detection time target:** < 60 seconds (achieved in lab testing via Wazuh syscheck)

## 3. Triage

Questions to answer before escalating:
- Was this file change authorized (scheduled deployment, patch, admin task)?
- Does the file match known-good hashes or change management records?
- What is the file's content/purpose — script, binary, config file?
- Is this an isolated host or are multiple hosts reporting similar alerts?

**Severity classification criteria:**
| Severity | Criteria |
|----------|----------|
| Low      | File created in a non-sensitive path, matches a known change window |
| Medium   | File created in a sensitive system path (`/etc`, `/bin`) with no change record |
| High     | File is executable, unsigned, and exhibits suspicious naming/content |
| Critical | File confirmed malicious (matches known malware hash/signature) or already executed |

## 4. Containment

**Immediate actions (first 15 minutes):**
1. Do not execute or open the suspicious file.
2. Snapshot the file (copy to isolated analysis location) and record its hash (SHA256).
3. Check process list and scheduled tasks/cron for any execution of the file.

**Short-term containment:**
- Isolate the affected host from the network if execution is confirmed or suspected.
- Disable any cron job / scheduled task referencing the file.
- Block the file via EDR/AV if a hash match is found.

## 5. Eradication

- Remove the malicious file from all affected systems.
- Search the environment for the same file hash on other hosts.
- Review and remove any persistence mechanisms (cron entries, systemd services, registry run keys) created by the file.
- Patch or close the vector that allowed file creation (exposed service, weak credentials, vulnerable application).

## 6. Recovery

- Restore the affected configuration files from a known-good backup if modified.
- Re-run FIM baseline scan to confirm no further unauthorized changes.
- Monitor the host closely for 24–48 hours for recurrence.

## 7. Post-Incident

- Document full timeline: detection time, triage time, containment time.
- Determine root cause (how was the file introduced — SSH access, web shell, supply chain, insider).
- Update FIM monitored paths if gaps are identified.
- Share IoCs (file hash, path, naming pattern) with the team.

## 8. References

- MITRE ATT&CK T1565.001: https://attack.mitre.org/techniques/T1565/001/
- Wazuh FIM documentation: https://documentation.wazuh.com/current/user-manual/capabilities/file-integrity/index.html
- Lab reference: https://github.com/AbrahamDiaco/soc-lab-wazuh
