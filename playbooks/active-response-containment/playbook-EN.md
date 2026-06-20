# Incident Response Playbook — Active Response: Automated Containment

**Author:** Abraham Diaco
**Last updated:** 2026-06-20
**Severity:** Informational / Validation
**MITRE ATT&CK:** T1110.001 — Brute Force: Password Guessing (containment context)

---

## 1. Overview

This playbook documents the validation and operational procedure around Wazuh **Active Response**, which automatically executes a containment action (e.g. firewall block via `iptables`) when a high-severity rule fires — without waiting for human intervention. This playbook is used to confirm Active Response triggered correctly and to define manual fallback steps when it does not.

Reference implementation and live metrics: [soc-lab-wazuh](https://github.com/AbrahamDiaco/soc-lab-wazuh/tree/main/scenarios/active-response)

## 2. Detection

**Indicators that Active Response engaged:**
- `wazuh-execd` log entry showing `firewall-drop` (or configured AR script) executed
- New `iptables` DROP rule for the offending source IP
- Wazuh dashboard shows both the triggering alert (e.g. Rule 5763, Level 10) and an associated Active Response event

**Example log (sanitized):**
```
wazuh-execd: INFO: Active response 'firewall-drop' executed.
iptables -A INPUT -s 192.168.x.x -j DROP
```

**Response time target:** < 1 second from rule trigger to IP block (achieved in lab testing, zero human intervention)

## 3. Triage

Questions to answer:
- Did Active Response trigger on the correct rule and source IP?
- Was the blocked IP a legitimate internal system (false positive risk) or a confirmed attacker?
- Is the AR rule configured with an expiration/timeout, or does it require manual removal?
- Are there signs the attacker pivoted to another IP after being blocked?

**Validation checklist:**
| Check | Expected result |
|-------|------------------|
| Alert fired | Rule matches expected severity/level (e.g. Level 10) |
| AR script executed | Log confirms `firewall-drop` or equivalent ran |
| Block applied | `iptables -L` (or equivalent) shows the DROP rule |
| Response time | < 1 second between alert and block (per lab benchmark) |

## 4. Containment

Active Response *is* the containment mechanism in this scenario — the goal here is to validate it rather than perform it manually. If Active Response **failed to trigger**:

**Immediate manual fallback:**
1. Manually block the source IP: `iptables -A INPUT -s <IP> -j DROP` (or equivalent on the firewall).
2. Check Wazuh manager status and the `active-response` configuration block in `ossec.conf`.
3. Review `/var/ossec/logs/ossec.log` for Active Response errors.

## 5. Eradication

- If Active Response misfired (blocked a legitimate IP), remove the rule and document the false positive for tuning.
- If the attacker pivoted to a new IP, ensure the AR rule scope/thresholds cover repeat offenders (e.g. via a dynamic blocklist).
- Review whether the AR timeout is appropriately configured (temporary block vs. permanent until manual review).

## 6. Recovery

- Confirm the legitimate service (SSH, etc.) remains accessible to authorized sources after the block.
- Periodically review and clear expired Active Response blocks per policy.
- Re-run the test scenario periodically (e.g. quarterly) to confirm Active Response remains functional after Wazuh updates.

## 7. Post-Incident

- Log the full chain: detection → alert → Active Response trigger → block confirmed, with timestamps.
- Use this as a benchmark metric in reporting (e.g. "Mean Time to Contain: < 1 second via automation vs. X minutes manual").
- Identify additional rules/scenarios that would benefit from Active Response automation.

## 8. References

- MITRE ATT&CK T1110.001: https://attack.mitre.org/techniques/T1110/001/
- Wazuh Active Response documentation: https://documentation.wazuh.com/current/user-manual/capabilities/active-response/index.html
- Lab reference: https://github.com/AbrahamDiaco/soc-lab-wazuh
- Related playbook: SSH Brute Force Attack
