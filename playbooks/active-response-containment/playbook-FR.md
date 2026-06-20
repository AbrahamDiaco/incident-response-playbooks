# Playbook de Réponse à Incident — Réponse Active : Confinement Automatisé

**Auteur :** Abraham Diaco
**Dernière mise à jour :** 2026-06-20
**Sévérité :** Informationnel / Validation
**MITRE ATT&CK :** T1110.001 — Brute Force: Password Guessing (contexte de confinement)

---

## 1. Vue d'ensemble

Ce playbook documente la procédure de validation et opérationnelle autour de la **Réponse Active** (Active Response) de Wazuh, qui exécute automatiquement une action de confinement (par ex. blocage pare-feu via `iptables`) lorsqu'une règle de haute sévérité se déclenche — sans attendre d'intervention humaine. Ce playbook permet de confirmer que la Réponse Active s'est déclenchée correctement et de définir les étapes manuelles de secours lorsque ce n'est pas le cas.

Implémentation de référence et métriques en direct : [soc-lab-wazuh](https://github.com/AbrahamDiaco/soc-lab-wazuh/tree/main/scenarios/active-response)

## 2. Détection

**Indicateurs que la Réponse Active s'est engagée :**
- Entrée de log `wazuh-execd` montrant l'exécution de `firewall-drop` (ou du script AR configuré)
- Nouvelle règle `iptables` DROP pour l'IP source fautive
- Le tableau de bord Wazuh affiche à la fois l'alerte déclenchante (par ex. Règle 5763, Niveau 10) et un événement de Réponse Active associé

**Exemple de log (anonymisé) :**
```
wazuh-execd: INFO: Active response 'firewall-drop' executed.
iptables -A INPUT -s 192.168.x.x -j DROP
```

**Objectif de temps de réponse :** moins d'1 seconde entre le déclenchement de la règle et le blocage de l'IP (atteint en laboratoire, sans intervention humaine)

## 3. Triage

Questions à résoudre :
- La Réponse Active s'est-elle déclenchée sur la bonne règle et la bonne IP source ?
- L'IP bloquée était-elle un système interne légitime (risque de faux positif) ou un attaquant confirmé ?
- La règle AR est-elle configurée avec une expiration/délai, ou nécessite-t-elle une suppression manuelle ?
- Y a-t-il des signes que l'attaquant a basculé vers une autre IP après le blocage ?

**Liste de validation :**

| Vérification | Résultat attendu |
|-------|------------------|
| Alerte déclenchée | La règle correspond à la sévérité/au niveau attendu (par ex. Niveau 10) |
| Script AR exécuté | Le log confirme que `firewall-drop` ou équivalent s'est exécuté |
| Blocage appliqué | `iptables -L` (ou équivalent) affiche la règle DROP |
| Temps de réponse | Moins d'1 seconde entre l'alerte et le blocage (selon le benchmark du lab) |

## 4. Confinement

Dans ce scénario, la Réponse Active *est* le mécanisme de confinement — l'objectif ici est de la valider plutôt que de l'exécuter manuellement. Si la Réponse Active **ne s'est pas déclenchée** :

**Solution de secours manuelle immédiate :**
1. Bloquer manuellement l'IP source : `iptables -A INPUT -s <IP> -j DROP` (ou équivalent sur le pare-feu).
2. Vérifier l'état du Wazuh Manager et le bloc de configuration `active-response` dans `ossec.conf`.
3. Examiner `/var/ossec/logs/ossec.log` pour identifier d'éventuelles erreurs de Réponse Active.

## 5. Éradication

- Si la Réponse Active s'est déclenchée par erreur (blocage d'une IP légitime), supprimer la règle et documenter le faux positif pour ajustement.
- Si l'attaquant a basculé vers une nouvelle IP, s'assurer que la portée/les seuils de la règle AR couvrent les récidivistes (par ex. via une liste de blocage dynamique).
- Vérifier que le délai d'expiration de la Réponse Active est configuré de manière appropriée (blocage temporaire vs. permanent jusqu'à revue manuelle).

## 6. Récupération

- Confirmer que le service légitime (SSH, etc.) reste accessible aux sources autorisées après le blocage.
- Examiner et lever périodiquement les blocages de Réponse Active expirés, conformément à la politique en vigueur.
- Relancer périodiquement le scénario de test (par ex. trimestriellement) pour confirmer que la Réponse Active reste fonctionnelle après les mises à jour de Wazuh.

## 7. Post-Incident

- Consigner la chaîne complète : détection → alerte → déclenchement de la Réponse Active → blocage confirmé, avec horodatages.
- Utiliser ceci comme métrique de référence dans les rapports (par ex. « Temps moyen de confinement : moins d'1 seconde via l'automatisation contre X minutes en manuel »).
- Identifier d'autres règles/scénarios qui bénéficieraient de l'automatisation par Réponse Active.

## 8. Références

- MITRE ATT&CK T1110.001 : https://attack.mitre.org/techniques/T1110/001/
- Documentation Wazuh Active Response : https://documentation.wazuh.com/current/user-manual/capabilities/active-response/index.html
- Référence du lab : https://github.com/AbrahamDiaco/soc-lab-wazuh
- Playbook lié : Attaque par Force Brute SSH
