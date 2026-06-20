# Playbook de R\u00e9ponse \u00e0 Incident — Active Response : Confinement Automatique

**Auteur :** Abraham Diaco
**Derni\u00e8re mise \u00e0 jour :** 2026-06-20
**S\u00e9v\u00e9rit\u00e9 :** Informationnelle / Validation
**MITRE ATT&CK :** T1110.001 — Brute Force: Password Guessing (contexte de confinement)

---

## 1. Vue d'ensemble

Ce playbook documente la proc\u00e9dure de validation et op\u00e9rationnelle autour de l'**Active Response** de Wazuh, qui ex\u00e9cute automatiquement une action de confinement (ex. blocage pare-feu via `iptables`) lorsqu'une r\u00e8gle de s\u00e9v\u00e9rit\u00e9 \u00e9lev\u00e9e se d\u00e9clenche — sans attendre d'intervention humaine. Ce playbook sert \u00e0 confirmer que l'Active Response s'est correctement d\u00e9clench\u00e9e et \u00e0 d\u00e9finir les \u00e9tapes manuelles de secours lorsque ce n'est pas le cas.

Impl\u00e9mentation de r\u00e9f\u00e9rence et m\u00e9triques r\u00e9elles : [soc-lab-wazuh](https://github.com/AbrahamDiaco/soc-lab-wazuh/tree/main/scenarios/active-response)

## 2. D\u00e9tection

**Indicateurs que l'Active Response s'est d\u00e9clench\u00e9e :**
- Entr\u00e9e de log `wazuh-execd` montrant l'ex\u00e9cution de `firewall-drop` (ou du script AR configur\u00e9)
- Nouvelle r\u00e8gle `iptables` DROP pour l'IP source incrimin\u00e9e
- Le dashboard Wazuh affiche \u00e0 la fois l'alerte d\u00e9clenchante (ex. Rule 5763, Level 10) et un \u00e9v\u00e9nement Active Response associ\u00e9

**Exemple de log (anonymis\u00e9) :**
```
wazuh-execd: INFO: Active response 'firewall-drop' executed.
iptables -A INPUT -s 192.168.x.x -j DROP
```

**Objectif de temps de r\u00e9ponse :** < 1 seconde entre le d\u00e9clenchement de la r\u00e8gle et le blocage de l'IP (atteint en lab, sans intervention humaine)

## 3. Triage

Questions \u00e0 r\u00e9soudre :
- L'Active Response s'est-elle d\u00e9clench\u00e9e sur la bonne r\u00e8gle et la bonne IP source ?
- L'IP bloqu\u00e9e \u00e9tait-elle un syst\u00e8me interne l\u00e9gitime (risque de faux positif) ou un attaquant confirm\u00e9 ?
- La r\u00e8gle AR est-elle configur\u00e9e avec une expiration/d\u00e9lai, ou n\u00e9cessite-t-elle une suppression manuelle ?
- Y a-t-il des signes que l'attaquant a pivot\u00e9 vers une autre IP apr\u00e8s avoir \u00e9t\u00e9 bloqu\u00e9 ?

**Liste de validation :**
| V\u00e9rification | R\u00e9sultat attendu |
|-------|------------------|
| Alerte d\u00e9clench\u00e9e | La r\u00e8gle correspond \u00e0 la s\u00e9v\u00e9rit\u00e9/niveau attendu (ex. Level 10) |
| Script AR ex\u00e9cut\u00e9 | Le log confirme l'ex\u00e9cution de `firewall-drop` ou \u00e9quivalent |
| Blocage appliqu\u00e9 | `iptables -L` (ou \u00e9quivalent) montre la r\u00e8gle DROP |
| Temps de r\u00e9ponse | < 1 seconde entre l'alerte et le blocage (selon benchmark du lab) |

## 4. Confinement

L'Active Response *est* le m\u00e9canisme de confinement dans ce sc\u00e9nario — l'objectif ici est de la valider plut\u00f4t que de l'ex\u00e9cuter manuellement. Si l'Active Response **ne s'est pas d\u00e9clench\u00e9e** :

**Solution de secours manuelle imm\u00e9diate :**
1. Bloquer manuellement l'IP source : `iptables -A INPUT -s <IP> -j DROP` (ou \u00e9quivalent sur le pare-feu).
2. V\u00e9rifier l'\u00e9tat du Wazuh manager et le bloc de configuration `active-response` dans `ossec.conf`.
3. Examiner `/var/ossec/logs/ossec.log` pour d'\u00e9ventuelles erreurs d'Active Response.

## 5. \u00c9radication

- Si l'Active Response s'est d\u00e9clench\u00e9e \u00e0 tort (blocage d'une IP l\u00e9gitime), supprimer la r\u00e8gle et documenter le faux positif pour ajustement.
- Si l'attaquant a pivot\u00e9 vers une nouvelle IP, s'assurer que la port\u00e9e/les seuils de la r\u00e8gle AR couvrent les r\u00e9cidivistes (ex. via une liste de blocage dynamique).
- V\u00e9rifier que le d\u00e9lai d'expiration de l'AR est correctement configur\u00e9 (blocage temporaire vs. permanent jusqu'\u00e0 revue manuelle).

## 6. R\u00e9cup\u00e9ration

- Confirmer que le service l\u00e9gitime (SSH, etc.) reste accessible aux sources autoris\u00e9es apr\u00e8s le blocage.
- Examiner et nettoyer p\u00e9riodiquement les blocages Active Response expir\u00e9s selon la politique en vigueur.
- Relancer p\u00e9riodiquement le sc\u00e9nario de test (ex. trimestriellement) pour confirmer que l'Active Response reste fonctionnelle apr\u00e8s les mises \u00e0 jour de Wazuh.

## 7. Post-Incident

- Journaliser la cha\u00eene compl\u00e8te : d\u00e9tection \u2192 alerte \u2192 d\u00e9clenchement Active Response \u2192 blocage confirm\u00e9, avec horodatages.
- Utiliser ceci comme m\u00e9trique de r\u00e9f\u00e9rence dans le reporting (ex. "Temps moyen de confinement : < 1 seconde via automatisation contre X minutes en manuel").
- Identifier d'autres r\u00e8gles/sc\u00e9narios qui b\u00e9n\u00e9ficieraient de l'automatisation Active Response.

## 8. R\u00e9f\u00e9rences

- MITRE ATT&CK T1110.001 : https://attack.mitre.org/techniques/T1110/001/
- Documentation Active Response Wazuh : https://documentation.wazuh.com/current/user-manual/capabilities/active-response/index.html
- R\u00e9f\u00e9rence du lab : https://github.com/AbrahamDiaco/soc-lab-wazuh
- Playbook li\u00e9 : Attaque par Brute Force SSH
