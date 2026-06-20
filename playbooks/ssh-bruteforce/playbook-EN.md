# Incident Response Playbook — SSH Brute Force Attack

**Author:** Abraham Diaco
**Last updated:** 2026-06-20
**Severity:** High
**MITRE ATT&CK:** T1110.001 — Brute Force: Password Guessing

---

## 1. Overview

An SSH brute force attack is a high-volume series of authentication attempts against a host's SSH service, typically using automated tools (Hydra, Medusa, Patator) cycling through credential lists. This playbook covers detection, triage, and response when SSH brute force activity is identified.

Reference implementation and live metrics: [soc-lab-wazuh](https://github.com/AbrahamDiaco/soc-lab-wazuh/tree/main/scenarios/bruteforce-ssh)

## 2. Detection

**Indicators of Compromise (IoCs):**
- Wazuh PAM/sshd alert — Rule 5763 ("SSH brute force") Level 10
- High volume of failed authentication attempts from a single source IP in a short window
- Repeated `Failed password for ... from <IP>` entries in `/var/log/auth.log` or `sshd` journal
- Authentication attempts cycling through multiple usernames against the same host

**Example alert (sanitized):**
```
Rule: 5763 (level 10) -> 'sshd: brute force trying to get access to the system.'
Source IP: 192.168.x.x  |  Attempts: 1,858 in < 2 minutes
```

**Detection time target:** < 2 minutes (achieved in lab testing: 1,858 alerts detected in under 2 minutes)

## 3. Triage

Questions to answer before escalating:
- Is the source IP internal (lab/test traffic, misconfigured script) or external (real attacker)?
- Did any authentication attempt succeed? (Check for `Accepted password` / `Accepted publickey` from the same source)
- Is the targeted account a privileged or service account?
- Is this a single host under attack, or is the same source IP hitting multiple hosts (network-wide brute force)?

**Severity classification criteria:**
| Severity | Criteria |
|----------|----------|
| Low      | Low-volume attempts, internal/known source, no successful login |
| Medium   | Sustained attempts from external IP, no successful login, non-privileged target account |
| High     | High-volume sustained attack, no successful login, targets privileged/service accounts |
| Critical | Any successful authentication following brute force attempts |

## 4. Containment

**Immediate actions (first 15 minutes):**
1. Confirm whether any login attempt succeeded — if yes, treat as account compromise (escalate immediately).
2. Identify the source IP(s) and current attack status (ongoing vs. stopped).
3. Check if Active Response / automated blocking has already engaged (see related playbook: Active Response Containment).

**Short-term containment:**
- Block the source IP at host (iptables/firewalld) and perimeter firewall level.
- Rate-limit or temporarily disable SSH access if the attack is ongoing and widespread.
- Force password reset / lock the targeted account(s) if any login succeeded.
- Enforce or verify SSH key-based authentication and disable password authentication where possible.

## 5. Eradication

- Review `sshd_config` for hardening gaps (`PermitRootLogin no`, `PasswordAuthentication no`, `MaxAuthTries`, `Fail2Ban`/Wazuh Active Response enabled).
- If account compromise is confirmed: rotate credentials, review the account's recent activity (commands run, files accessed, new SSH keys added to `authorized_keys`).
- Check for persistence mechanisms left by the attacker if compromise occurred (new users, cron jobs, SSH keys).

## 6. Recovery

- Re-enable SSH access progressively once the source is blocked and hardening is applied.
- Monitor authentication logs closely for 24–48 hours following the incident.
- Verify no other hosts were targeted by the same source IP.

## 7. Post-Incident

- Document the full timeline (first attempt, alert time, containment time, total attempts).
- Calculate detection and response metrics for reporting (e.g. "1,858 attempts detected and contained in < 2 minutes").
- Evaluate whether Active Response automation should be enabled permanently for this rule.
- Add the source IP to a threat intelligence blocklist if recurring.

## 8. References

- MITRE ATT&CK T1110.001: https://attack.mitre.org/techniques/T1110/001/
- Wazuh Active Response documentation: https://documentation.wazuh.com/current/user-manual/capabilities/active-response/index.html
- Lab reference: https://github.com/AbrahamDiaco/soc-lab-wazuh
- Related playbook: Active Response — Automated Containment
