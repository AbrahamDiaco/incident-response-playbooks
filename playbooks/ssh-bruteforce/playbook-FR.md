# Playbook de R\u00e9ponse \u00e0 Incident — Attaque par Brute Force SSH

**Auteur :** Abraham Diaco
**Derni\u00e8re mise \u00e0 jour :** 2026-06-20
**S\u00e9v\u00e9rit\u00e9 :** \u00c9lev\u00e9e
**MITRE ATT&CK :** T1110.001 — Brute Force: Password Guessing

---

## 1. Vue d'ensemble

Une attaque par brute force SSH consiste en une s\u00e9rie massive de tentatives d'authentification contre le service SSH d'une machine, g\u00e9n\u00e9ralement \u00e0 l'aide d'outils automatis\u00e9s (Hydra, Medusa, Patator) testant des listes d'identifiants. Ce playbook couvre la d\u00e9tection, le triage et la r\u00e9ponse lorsqu'une activit\u00e9 de brute force SSH est identifi\u00e9e.

Impl\u00e9mentation de r\u00e9f\u00e9rence et m\u00e9triques r\u00e9elles : [soc-lab-wazuh](https://github.com/AbrahamDiaco/soc-lab-wazuh/tree/main/scenarios/bruteforce-ssh)

## 2. D\u00e9tection

**Indicateurs de compromission (IoC) :**
- Alerte Wazuh PAM/sshd — Rule 5763 ("SSH brute force") Level 10
- Volume \u00e9lev\u00e9 de tentatives d'authentification \u00e9chou\u00e9es depuis une seule IP source sur une courte p\u00e9riode
- Entr\u00e9es r\u00e9p\u00e9t\u00e9es `Failed password for ... from <IP>` dans `/var/log/auth.log` ou le journal `sshd`
- Tentatives d'authentification testant plusieurs noms d'utilisateur sur la m\u00eame machine

**Exemple d'alerte (anonymis\u00e9) :**
```
Rule: 5763 (level 10) -> 'sshd: brute force trying to get access to the system.'
IP source : 192.168.x.x  |  Tentatives : 1 858 en < 2 minutes
```

**Objectif de temps de d\u00e9tection :** < 2 minutes (atteint en lab : 1 858 alertes d\u00e9tect\u00e9es en moins de 2 minutes)

## 3. Triage

Questions \u00e0 r\u00e9soudre avant escalade :
- L'IP source est-elle interne (trafic de lab/test, script mal configur\u00e9) ou externe (v\u00e9ritable attaquant) ?
- Une tentative d'authentification a-t-elle r\u00e9ussi ? (V\u00e9rifier la pr\u00e9sence de `Accepted password` / `Accepted publickey` depuis la m\u00eame source)
- Le compte cibl\u00e9 est-il un compte privil\u00e9gi\u00e9 ou un compte de service ?
- S'agit-il d'une seule machine attaqu\u00e9e, ou la m\u00eame IP source touche-t-elle plusieurs machines (brute force \u00e0 l'\u00e9chelle du r\u00e9seau) ?

**Crit\u00e8res de classification de la s\u00e9v\u00e9rit\u00e9 :**
| S\u00e9v\u00e9rit\u00e9 | Crit\u00e8res |
|----------|-----------|
| Faible      | Tentatives \u00e0 faible volume, source interne/connue, aucune connexion r\u00e9ussie |
| Moyenne     | Tentatives soutenues depuis une IP externe, aucune connexion r\u00e9ussie, compte cible non privil\u00e9gi\u00e9 |
| \u00c9lev\u00e9e      | Attaque soutenue \u00e0 volume \u00e9lev\u00e9, aucune connexion r\u00e9ussie, cible des comptes privil\u00e9gi\u00e9s/de service |
| Critique    | Toute authentification r\u00e9ussie suite \u00e0 des tentatives de brute force |

## 4. Confinement

**Actions imm\u00e9diates (15 premi\u00e8res minutes) :**
1. Confirmer si une tentative de connexion a r\u00e9ussi — si oui, traiter comme une compromission de compte (escalade imm\u00e9diate).
2. Identifier la/les IP source(s) et le statut actuel de l'attaque (en cours vs. arr\u00eat\u00e9e).
3. V\u00e9rifier si l'Active Response / le blocage automatique s'est d\u00e9j\u00e0 d\u00e9clench\u00e9 (voir playbook li\u00e9 : Confinement Automatique via Active Response).

**Confinement \u00e0 court terme :**
- Bloquer l'IP source au niveau de la machine (iptables/firewalld) et du pare-feu p\u00e9rim\u00e9trique.
- Limiter le d\u00e9bit ou d\u00e9sactiver temporairement l'acc\u00e8s SSH si l'attaque est en cours et \u00e9tendue.
- Forcer la r\u00e9initialisation du mot de passe / verrouiller le(s) compte(s) cibl\u00e9(s) si une connexion a r\u00e9ussi.
- Imposer ou v\u00e9rifier l'authentification par cl\u00e9 SSH et d\u00e9sactiver l'authentification par mot de passe si possible.

## 5. \u00c9radication

- V\u00e9rifier `sshd_config` pour identifier les lacunes de durcissement (`PermitRootLogin no`, `PasswordAuthentication no`, `MaxAuthTries`, Fail2Ban/Active Response Wazuh activ\u00e9).
- Si la compromission du compte est confirm\u00e9e : faire tourner les identifiants, examiner l'activit\u00e9 r\u00e9cente du compte (commandes ex\u00e9cut\u00e9es, fichiers consult\u00e9s, nouvelles cl\u00e9s SSH ajout\u00e9es \u00e0 `authorized_keys`).
- V\u00e9rifier l'absence de m\u00e9canismes de persistance laiss\u00e9s par l'attaquant en cas de compromission (nouveaux utilisateurs, t\u00e2ches cron, cl\u00e9s SSH).

## 6. R\u00e9cup\u00e9ration

- R\u00e9activer progressivement l'acc\u00e8s SSH une fois la source bloqu\u00e9e et le durcissement appliqu\u00e9.
- Surveiller \u00e9troitement les logs d'authentification pendant 24 \u00e0 48h apr\u00e8s l'incident.
- V\u00e9rifier qu'aucune autre machine n'a \u00e9t\u00e9 cibl\u00e9e par la m\u00eame IP source.

## 7. Post-Incident

- Documenter la chronologie compl\u00e8te (premi\u00e8re tentative, heure de l'alerte, heure du confinement, total des tentatives).
- Calculer les m\u00e9triques de d\u00e9tection et de r\u00e9ponse pour le reporting (ex. "1 858 tentatives d\u00e9tect\u00e9es et contenues en < 2 minutes").
- \u00c9valuer si l'automatisation Active Response devrait \u00eatre activ\u00e9e en permanence pour cette r\u00e8gle.
- Ajouter l'IP source \u00e0 une liste de blocage threat intelligence si r\u00e9currente.

## 8. R\u00e9f\u00e9rences

- MITRE ATT&CK T1110.001 : https://attack.mitre.org/techniques/T1110/001/
- Documentation Active Response Wazuh : https://documentation.wazuh.com/current/user-manual/capabilities/active-response/index.html
- R\u00e9f\u00e9rence du lab : https://github.com/AbrahamDiaco/soc-lab-wazuh
- Playbook li\u00e9 : Active Response — Confinement Automatique
