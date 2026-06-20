# 🛡️ Incident Response Playbooks

[#-incident-response-playbooks](#-incident-response-playbooks)

**Author:** Abraham Diaco | Cybersecurity Specialist | CEH · ISO/IEC 27001 Lead Auditor
**Status:** 🟢 Active — bilingual (FR/EN) playbooks mapped to MITRE ATT&CK

---

## About

[#about](#about)

This repository is a collection of bilingual (French/English) Incident Response playbooks, written to standardize detection, triage, containment, eradication, and recovery procedures for common SOC scenarios. Each playbook follows the same structure and is mapped to the relevant [MITRE ATT&CK](https://attack.mitre.org) technique(s).

Three of the playbooks are directly grounded in real detection metrics from my hands-on SOC lab — see [soc-lab-wazuh](https://github.com/AbrahamDiaco/soc-lab-wazuh) — where I deployed Wazuh and simulated each scenario end-to-end (FIM, SSH brute force, and automated Active Response containment).

---

## Playbooks

[#playbooks](#playbooks)

| # | Scenario | MITRE ATT&CK | Severity | Source |
|---|----------|---------------|----------|--------|
| 1 | [FIM — Suspicious File Detected](playbooks/fim-suspicious-file) | T1565.001 | Medium | SOC lab (Wazuh) |
| 2 | [SSH Brute Force Attack](playbooks/ssh-bruteforce) | T1110.001 | High | SOC lab (Wazuh) |
| 3 | [Active Response — Automated Containment](playbooks/active-response-containment) | T1110.001 | Validation | SOC lab (Wazuh) |
| 4 | [Phishing Email](playbooks/phishing-email) | T1566.001 / T1566.002 | Medium–High | General SOC scenario |
| 5 | [Ransomware / Malware Detection & Containment](playbooks/ransomware-malware) | T1486 / T1059 | Critical | General SOC scenario |
| 6 | [Data Exfiltration](playbooks/data-exfiltration) | T1041 / T1567 | Critical | General SOC scenario |

Each playbook folder contains:
- `playbook-EN.md` — English version
- `playbook-FR.md` — French version

---

## Playbook Structure

[#playbook-structure](#playbook-structure)

Every playbook follows the same 8-section format, based on the standard IR lifecycle:

1. **Overview** — what the incident is and why it matters
2. **Detection** — IoCs, alert sources, detection time targets
3. **Triage** — questions to answer, severity classification criteria
4. **Containment** — immediate and short-term actions
5. **Eradication** — removing the root cause and persistence mechanisms
6. **Recovery** — restoring systems safely
7. **Post-Incident** — documentation, root cause analysis, lessons learned
8. **References** — MITRE ATT&CK links, related tooling, related playbooks

A blank, reusable version is available in [`templates/`](templates) for creating new playbooks.

---

## Repository Structure

[#repository-structure](#repository-structure)

```
incident-response-playbooks/
├── README.md
├── mitre-attack-mapping.md
├── templates/
│   ├── playbook-template-EN.md
│   └── playbook-template-FR.md
└── playbooks/
    ├── fim-suspicious-file/
    │   ├── playbook-EN.md
    │   └── playbook-FR.md
    ├── ssh-bruteforce/
    │   ├── playbook-EN.md
    │   └── playbook-FR.md
    ├── active-response-containment/
    │   ├── playbook-EN.md
    │   └── playbook-FR.md
    ├── phishing-email/
    │   ├── playbook-EN.md
    │   └── playbook-FR.md
    ├── ransomware-malware/
    │   ├── playbook-EN.md
    │   └── playbook-FR.md
    └── data-exfiltration/
        ├── playbook-EN.md
        └── playbook-FR.md
```

---

## MITRE ATT&CK Coverage

[#mitre-attck-coverage](#mitre-attck-coverage)

See [`mitre-attack-mapping.md`](mitre-attack-mapping.md) for the full technique-to-playbook mapping.

---

## Related Projects

[#related-projects](#related-projects)

- [soc-lab-wazuh](https://github.com/AbrahamDiaco/soc-lab-wazuh) — SIEM/XDR home lab where the Wazuh-based scenarios in this repo were tested and measured
- [network-security-labs](https://github.com/AbrahamDiaco/network-security-labs) — network recon & traffic analysis exercises
- [tryhackme-writeups](https://github.com/AbrahamDiaco/tryhackme-writeups) — SOC Level 1 path write-ups

---

## Contact

[#contact](#contact)

📧 tanguy.diaco@gmail.com
🔗 [linkedin.com/in/abraham-diaco](http://www.linkedin.com/in/abraham-diaco)

---

*Educational / portfolio project — Abraham Diaco · Cybersecurity Specialist | CEH · ISO 27001 LA*
