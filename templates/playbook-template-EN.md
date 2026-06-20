# Incident Response Playbook — [Incident Name]

**Author:** [Name]
**Last updated:** [YYYY-MM-DD]
**Severity:** Low / Medium / High / Critical
**MITRE ATT&CK:** [Txxxx — Technique Name]

---

## 1. Overview

Brief description of the incident type, why it matters, and the systems typically affected.

## 2. Detection

**Indicators of Compromise (IoCs):**
- Log sources / alert sources to monitor
- Key signatures, rule IDs, or alert names
- Example alert payload (sanitized)

**Detection time target:** [e.g. < 2 minutes]

## 3. Triage

Questions to answer before escalating:
- Is this a true positive or false positive?
- What is the scope (single host, multiple hosts, network-wide)?
- Is data or a critical system at risk?

**Severity classification criteria:**
| Severity | Criteria |
|----------|----------|
| Low      | ... |
| Medium   | ... |
| High     | ... |
| Critical | ... |

## 4. Containment

**Immediate actions (first 15 minutes):**
1. Step
2. Step

**Short-term containment:**
- Isolate affected host(s)
- Disable compromised accounts
- Block malicious IP/domain

## 5. Eradication

- Remove malicious artifacts (files, scheduled tasks, persistence mechanisms)
- Patch the exploited vulnerability
- Reset credentials if compromised

## 6. Recovery

- Restore systems from clean backups / images
- Monitor for recurrence
- Validate system integrity before returning to production

## 7. Post-Incident

- Document timeline of events
- Root cause analysis
- Lessons learned / process improvements
- Update detection rules if applicable

## 8. References

- MITRE ATT&CK: [link]
- Internal documentation / runbooks
- Related tooling
