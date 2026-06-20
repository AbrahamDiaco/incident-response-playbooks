# Playbook de Réponse à Incident — [Nom de l'Incident]

**Auteur :** [Nom]
**Dernière mise à jour :** [AAAA-MM-JJ]
**Sévérité :** Faible / Moyenne / Élevée / Critique
**MITRE ATT&CK :** [Txxxx — Nom de la Technique]

---

## 1. Vue d'ensemble

Brève description du type d'incident, de son importance, et des systèmes généralement affectés.

## 2. Détection

**Indicateurs de compromission (IoC) :**
- Sources de logs / sources d'alertes à surveiller
- Signatures clés, IDs de règles, ou noms d'alertes
- Exemple de contenu d'alerte (anonymisé)

**Objectif de délai de détection :** [par ex. moins de 2 minutes]

## 3. Triage

Questions à résoudre avant l'escalade :
- S'agit-il d'un vrai positif ou d'un faux positif ?
- Quel est le périmètre (hôte unique, plusieurs hôtes, à l'échelle du réseau) ?
- Des données ou un système critique sont-ils en danger ?

**Critères de classification de la sévérité :**

| Sévérité | Critères |
|----------|----------|
| Faible | ... |
| Moyenne | ... |
| Élevée | ... |
| Critique | ... |

## 4. Confinement

**Actions immédiates (15 premières minutes) :**
1. Étape
2. Étape

**Confinement à court terme :**
- Isoler le(s) hôte(s) affecté(s)
- Désactiver les comptes compromis
- Bloquer l'IP/le domaine malveillant

## 5. Éradication

- Supprimer les artefacts malveillants (fichiers, tâches planifiées, mécanismes de persistance)
- Corriger la vulnérabilité exploitée
- Réinitialiser les identifiants en cas de compromission

## 6. Récupération

- Restaurer les systèmes à partir de sauvegardes/images saines
- Surveiller toute récidive
- Valider l'intégrité du système avant le retour en production

## 7. Post-Incident

- Documenter la chronologie des événements
- Analyse de cause racine
- Enseignements tirés / améliorations de processus
- Mettre à jour les règles de détection si applicable

## 8. Références

- MITRE ATT&CK : [lien]
- Documentation interne / runbooks
- Outils associés
