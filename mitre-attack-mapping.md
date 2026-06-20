# MITRE ATT&CK Mapping

[#mitre-attck-mapping](#mitre-attck-mapping)

Summary of MITRE ATT&CK techniques covered by the playbooks in this repository.

| Tactic | Technique ID | Technique Name | Playbook |
|--------|--------------|-----------------|----------|
| Initial Access | T1566.001 | Phishing: Spearphishing Attachment | [Phishing Email](playbooks/phishing-email) |
| Initial Access | T1566.002 | Phishing: Spearphishing Link | [Phishing Email](playbooks/phishing-email) |
| Credential Access | T1110.001 | Brute Force: Password Guessing | [SSH Brute Force Attack](playbooks/ssh-bruteforce) |
| Credential Access | T1110.001 | Brute Force: Password Guessing (containment validation) | [Active Response — Automated Containment](playbooks/active-response-containment) |
| Defense Evasion / Impact | T1565.001 | Data Manipulation: Stored Data Manipulation | [FIM — Suspicious File Detected](playbooks/fim-suspicious-file) |
| Execution | T1059 | Command and Scripting Interpreter | [Ransomware / Malware Detection & Containment](playbooks/ransomware-malware) |
| Impact | T1486 | Data Encrypted for Impact | [Ransomware / Malware Detection & Containment](playbooks/ransomware-malware) |
| Exfiltration | T1041 | Exfiltration Over C2 Channel | [Data Exfiltration](playbooks/data-exfiltration) |
| Exfiltration | T1567 | Exfiltration Over Web Service | [Data Exfiltration](playbooks/data-exfiltration) |

---

## References

[#references](#references)

- MITRE ATT&CK Enterprise Matrix: https://attack.mitre.org/matrices/enterprise/
- MITRE ATT&CK Navigator: https://mitre-attack.github.io/attack-navigator/
