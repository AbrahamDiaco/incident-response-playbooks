# Playbook de R\u00e9ponse \u00e0 Incident — [Nom de l'Incident]

**Auteur :** [Nom]
**Derni\u00e8re mise \u00e0 jour :** [AAAA-MM-JJ]
**S\u00e9v\u00e9rit\u00e9 :** Faible / Moyenne / \u00c9lev\u00e9e / Critique
**MITRE ATT&CK :** [Txxxx — Nom de la technique]

---

## 1. Vue d'ensemble

Description br\u00e8ve du type d'incident, de son importance, et des syst\u00e8mes g\u00e9n\u00e9ralement affect\u00e9s.

## 2. D\u00e9tection

**Indicateurs de compromission (IoC) :**
- Sources de logs / sources d'alertes \u00e0 surveiller
- Signatures cl\u00e9s, ID de r\u00e8gles ou noms d'alertes
- Exemple de payload d'alerte (anonymis\u00e9)

**Objectif de temps de d\u00e9tection :** [ex. < 2 minutes]

## 3. Triage

Questions \u00e0 r\u00e9soudre avant escalade :
- S'agit-il d'un vrai positif ou d'un faux positif ?
- Quel est le p\u00e9rim\u00e8tre (une seule machine, plusieurs, tout le r\u00e9seau) ?
- Des donn\u00e9es ou un syst\u00e8me critique sont-ils en danger ?

**Crit\u00e8res de classification de la s\u00e9v\u00e9rit\u00e9 :**
| S\u00e9v\u00e9rit\u00e9 | Crit\u00e8res |
|----------|-----------|
| Faible      | ... |
| Moyenne     | ... |
| \u00c9lev\u00e9e      | ... |
| Critique    | ... |

## 4. Confinement (Containment)

**Actions imm\u00e9diates (15 premi\u00e8res minutes) :**
1. \u00c9tape
2. \u00c9tape

**Confinement \u00e0 court terme :**
- Isoler la/les machine(s) affect\u00e9e(s)
- D\u00e9sactiver les comptes compromis
- Bloquer l'IP/domaine malveillant

## 5. \u00c9radication

- Supprimer les artefacts malveillants (fichiers, t\u00e2ches planifi\u00e9es, m\u00e9canismes de persistance)
- Corriger la vuln\u00e9rabilit\u00e9 exploit\u00e9e
- R\u00e9initialiser les identifiants compromis

## 6. R\u00e9cup\u00e9ration (Recovery)

- Restaurer les syst\u00e8mes \u00e0 partir de sauvegardes/images saines
- Surveiller toute r\u00e9currence
- Valider l'int\u00e9grit\u00e9 du syst\u00e8me avant remise en production

## 7. Post-Incident

- Documenter la chronologie des \u00e9v\u00e9nements
- Analyse des causes racines
- Le\u00e7ons apprises / am\u00e9liorations de processus
- Mettre \u00e0 jour les r\u00e8gles de d\u00e9tection si applicable

## 8. R\u00e9f\u00e9rences

- MITRE ATT&CK : [lien]
- Documentation interne / runbooks
- Outils li\u00e9s
