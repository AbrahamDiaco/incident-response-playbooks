# Playbook de Réponse à Incident — Fichier Suspect Détecté (FIM)

**Auteur :** Abraham Diaco
**Dernière mise à jour :** 2026-06-20
**Sévérité :** Moyenne
**MITRE ATT&CK :** T1565.001 — Stored Data Manipulation

---

## 1. Vue d'ensemble

Une alerte de surveillance d'intégrité des fichiers (FIM) se déclenche lorsqu'un fichier nouveau, modifié ou supprimé est détecté dans un répertoire surveillé (par ex. `/etc`, `/bin`, `C:\Windows\System32`). Ce playbook couvre la réponse lorsque le FIM détecte la création d'un fichier non autorisé ou suspect, ce qui peut indiquer le dépôt d'un malware, une modification de configuration non autorisée, ou un mécanisme de persistance en phase précoce.

Implémentation de référence et métriques en direct : [soc-lab-wazuh](https://github.com/AbrahamDiaco/soc-lab-wazuh/tree/main/scenarios/FIM-malware)

## 2. Détection

**Indicateurs de compromission (IoC) :**
- Alerte Wazuh `syscheckd` — Règle 554 (« New file added to the system ») ou autres IDs de règles FIM associées
- Fichier inattendu dans un chemin sensible (`/etc`, `/bin`, `/usr/local/bin`, dossiers de démarrage, répertoires cron)
- Fichier avec permissions d'exécution apparaissant hors des fenêtres de changement habituelles
- Hash de fichier inhabituel sans éditeur/signature connu

**Exemple d'alerte (anonymisé) :**
```
Rule: 554 (level 5) -> 'File added to the system.'
File: '/etc/malware-test.sh'
```

**Objectif de délai de détection :** moins de 60 secondes (atteint en laboratoire via Wazuh syscheck)

## 3. Triage

Questions à résoudre avant l'escalade :
- Cette modification de fichier était-elle autorisée (déploiement planifié, correctif, tâche d'administration) ?
- Le fichier correspond-il à des hashs connus comme légitimes ou à un enregistrement de gestion des changements ?
- Quel est le contenu/l'objectif du fichier — script, binaire, fichier de configuration ?
- S'agit-il d'un hôte isolé ou plusieurs hôtes signalent-ils des alertes similaires ?

**Critères de classification de la sévérité :**

| Sévérité | Critères |
|----------|----------|
| Faible | Fichier créé dans un chemin non sensible, correspond à une fenêtre de changement connue |
| Moyenne | Fichier créé dans un chemin système sensible (`/etc`, `/bin`) sans enregistrement de changement |
| Élevée | Fichier exécutable, non signé, présentant un nom/contenu suspect |
| Critique | Fichier confirmé malveillant (correspond à un hash/signature de malware connu) ou déjà exécuté |

## 4. Confinement

**Actions immédiates (15 premières minutes) :**
1. Ne pas exécuter ni ouvrir le fichier suspect.
2. Faire une copie du fichier (vers un emplacement d'analyse isolé) et enregistrer son hash (SHA256).
3. Vérifier la liste des processus ainsi que les tâches planifiées/cron pour toute exécution du fichier.

**Confinement à court terme :**
- Isoler l'hôte affecté du réseau si l'exécution est confirmée ou suspectée.
- Désactiver toute tâche cron/tâche planifiée référençant le fichier.
- Bloquer le fichier via EDR/antivirus si une correspondance de hash est trouvée.

## 5. Éradication

- Supprimer le fichier malveillant de tous les systèmes affectés.
- Rechercher le même hash de fichier sur les autres hôtes de l'environnement.
- Examiner et supprimer tout mécanisme de persistance (entrées cron, services systemd, clés de registre run) créé par le fichier.
- Corriger ou fermer le vecteur ayant permis la création du fichier (service exposé, identifiants faibles, application vulnérable).

## 6. Récupération

- Restaurer les fichiers de configuration affectés à partir d'une sauvegarde saine connue si modifiés.
- Relancer une analyse de référence FIM pour confirmer l'absence de modifications non autorisées supplémentaires.
- Surveiller étroitement l'hôte pendant 24 à 48 heures pour détecter une récidive.

## 7. Post-Incident

- Documenter la chronologie complète : heure de détection, heure de triage, heure de confinement.
- Déterminer la cause racine (comment le fichier a-t-il été introduit — accès SSH, web shell, chaîne d'approvisionnement, menace interne).
- Mettre à jour les chemins surveillés par le FIM si des lacunes sont identifiées.
- Partager les IoC (hash du fichier, chemin, schéma de nommage) avec l'équipe.

## 8. Références

- MITRE ATT&CK T1565.001 : https://attack.mitre.org/techniques/T1565/001/
- Documentation Wazuh FIM : https://documentation.wazuh.com/current/user-manual/capabilities/file-integrity/index.html
- Référence du lab : https://github.com/AbrahamDiaco/soc-lab-wazuh
