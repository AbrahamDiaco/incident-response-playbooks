# Playbook de Réponse à Incident — Attaque par Force Brute SSH

**Auteur :** Abraham Diaco
**Dernière mise à jour :** 2026-06-20
**Sévérité :** Élevée
**MITRE ATT&CK :** T1110.001 — Brute Force: Password Guessing

---

## 1. Vue d'ensemble

Une attaque par force brute SSH consiste en une série massive de tentatives d'authentification contre le service SSH d'un hôte, généralement à l'aide d'outils automatisés (Hydra, Medusa, Patator) qui parcourent des listes d'identifiants. Ce playbook couvre la détection, le triage et la réponse lorsqu'une activité de force brute SSH est identifiée.

Implémentation de référence et métriques en direct : [soc-lab-wazuh](https://github.com/AbrahamDiaco/soc-lab-wazuh/tree/main/scenarios/bruteforce-ssh)

## 2. Détection

**Indicateurs de compromission (IoC) :**
- Alerte Wazuh PAM/sshd — Règle 5763 (« SSH brute force ») Niveau 10
- Volume élevé de tentatives d'authentification échouées depuis une seule IP source sur une courte fenêtre de temps
- Entrées répétées `Failed password for ... from <IP>` dans `/var/log/auth.log` ou le journal `sshd`
- Tentatives d'authentification parcourant plusieurs noms d'utilisateur contre le même hôte

**Exemple d'alerte (anonymisé) :**
```
Rule: 5763 (level 10) -> 'sshd: brute force trying to get access to the system.'
Source IP: 192.168.x.x  |  Attempts: 1,858 in < 2 minutes
```

**Objectif de délai de détection :** moins de 2 minutes (atteint en laboratoire : 1 858 alertes détectées en moins de 2 minutes)

## 3. Triage

Questions à résoudre avant l'escalade :
- L'IP source est-elle interne (trafic de lab/test, script mal configuré) ou externe (attaquant réel) ?
- Une tentative d'authentification a-t-elle réussi ? (Vérifier la présence de `Accepted password` / `Accepted publickey` depuis la même source)
- Le compte visé est-il un compte privilégié ou un compte de service ?
- S'agit-il d'un seul hôte attaqué, ou la même IP source frappe-t-elle plusieurs hôtes (force brute à l'échelle du réseau) ?

**Critères de classification de la sévérité :**

| Sévérité | Critères |
|----------|----------|
| Faible | Tentatives à faible volume, source interne/connue, aucune connexion réussie |
| Moyenne | Tentatives soutenues depuis une IP externe, aucune connexion réussie, compte cible non privilégié |
| Élevée | Attaque soutenue à fort volume, aucune connexion réussie, cible des comptes privilégiés/de service |
| Critique | Toute authentification réussie suite à des tentatives de force brute |

## 4. Confinement

**Actions immédiates (15 premières minutes) :**
1. Confirmer si une tentative de connexion a réussi — si oui, traiter comme une compromission de compte (escalader immédiatement).
2. Identifier la/les IP source(s) et le statut actuel de l'attaque (en cours vs. arrêtée).
3. Vérifier si la Réponse Active/le blocage automatisé s'est déjà engagé (voir le playbook lié : Confinement par Réponse Active).

**Confinement à court terme :**
- Bloquer l'IP source au niveau de l'hôte (iptables/firewalld) et du pare-feu périmétrique.
- Limiter le débit ou désactiver temporairement l'accès SSH si l'attaque est en cours et étendue.
- Forcer la réinitialisation du mot de passe/verrouiller le(s) compte(s) ciblé(s) si une connexion a réussi.
- Imposer ou vérifier l'authentification par clé SSH et désactiver l'authentification par mot de passe lorsque possible.

## 5. Éradication

- Examiner `sshd_config` pour identifier les lacunes de durcissement (`PermitRootLogin no`, `PasswordAuthentication no`, `MaxAuthTries`, Fail2Ban/Réponse Active Wazuh activée).
- Si une compromission de compte est confirmée : faire une rotation des identifiants, examiner l'activité récente du compte (commandes exécutées, fichiers consultés, nouvelles clés SSH ajoutées à `authorized_keys`).
- Vérifier la présence de mécanismes de persistance laissés par l'attaquant en cas de compromission (nouveaux utilisateurs, tâches cron, clés SSH).

## 6. Récupération

- Réactiver progressivement l'accès SSH une fois la source bloquée et le durcissement appliqué.
- Surveiller étroitement les logs d'authentification pendant 24 à 48 heures suivant l'incident.
- Vérifier qu'aucun autre hôte n'a été ciblé par la même IP source.

## 7. Post-Incident

- Documenter la chronologie complète (première tentative, heure d'alerte, heure de confinement, nombre total de tentatives).
- Calculer les métriques de détection et de réponse pour le rapport (par ex. « 1 858 tentatives détectées et confinées en moins de 2 minutes »).
- Évaluer si l'automatisation par Réponse Active devrait être activée de façon permanente pour cette règle.
- Ajouter l'IP source à une liste de blocage de threat intelligence si elle est récurrente.

## 8. Références

- MITRE ATT&CK T1110.001 : https://attack.mitre.org/techniques/T1110/001/
- Documentation Wazuh Active Response : https://documentation.wazuh.com/current/user-manual/capabilities/active-response/index.html
- Référence du lab : https://github.com/AbrahamDiaco/soc-lab-wazuh
- Playbook lié : Réponse Active — Confinement Automatisé
