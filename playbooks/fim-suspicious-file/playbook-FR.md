# Playbook de R\u00e9ponse \u00e0 Incident — Fichier Suspect D\u00e9tect\u00e9 (FIM)

**Auteur :** Abraham Diaco
**Derni\u00e8re mise \u00e0 jour :** 2026-06-20
**S\u00e9v\u00e9rit\u00e9 :** Moyenne
**MITRE ATT&CK :** T1565.001 — Stored Data Manipulation

---

## 1. Vue d'ensemble

Une alerte de surveillance d'int\u00e9grit\u00e9 des fichiers (FIM) se d\u00e9clenche lorsqu'un fichier est cr\u00e9\u00e9, modifi\u00e9 ou supprim\u00e9 dans un r\u00e9pertoire surveill\u00e9 (ex. `/etc`, `/bin`, `C:\Windows\System32`). Ce playbook couvre la r\u00e9ponse lorsque le FIM d\u00e9tecte la cr\u00e9ation d'un fichier non autoris\u00e9 ou suspect, ce qui peut indiquer un d\u00e9p\u00f4t de malware, une modification de configuration non autoris\u00e9e, ou un d\u00e9but de persistance.

Impl\u00e9mentation de r\u00e9f\u00e9rence et m\u00e9triques r\u00e9elles : [soc-lab-wazuh](https://github.com/AbrahamDiaco/soc-lab-wazuh/tree/main/scenarios/FIM-malware)

## 2. D\u00e9tection

**Indicateurs de compromission (IoC) :**
- Alerte Wazuh `syscheckd` — Rule 554 ("New file added to the system") ou ID de r\u00e8gles FIM associ\u00e9es
- Fichier inattendu dans un chemin sensible (`/etc`, `/bin`, `/usr/local/bin`, dossiers de d\u00e9marrage, r\u00e9pertoires cron)
- Fichier avec permissions d'ex\u00e9cution apparaissant en dehors des fen\u00eatres de changement habituelles
- Hash de fichier inhabituel sans \u00e9diteur/signature connu(e)

**Exemple d'alerte (anonymis\u00e9) :**
```
Rule: 554 (level 5) -> 'File added to the system.'
File: '/etc/malware-test.sh'
```

**Objectif de temps de d\u00e9tection :** < 60 secondes (atteint en lab via Wazuh syscheck)

## 3. Triage

Questions \u00e0 r\u00e9soudre avant escalade :
- Ce changement de fichier \u00e9tait-il autoris\u00e9 (d\u00e9ploiement planifi\u00e9, patch, t\u00e2che admin) ?
- Le fichier correspond-il \u00e0 des hashes connus ou \u00e0 un enregistrement de gestion des changements ?
- Quel est le contenu/objectif du fichier — script, binaire, fichier de config ?
- S'agit-il d'une machine isol\u00e9e ou plusieurs machines remontent-elles des alertes similaires ?

**Crit\u00e8res de classification de la s\u00e9v\u00e9rit\u00e9 :**
| S\u00e9v\u00e9rit\u00e9 | Crit\u00e8res |
|----------|-----------|
| Faible      | Fichier cr\u00e9\u00e9 dans un chemin non sensible, correspond \u00e0 une fen\u00eatre de changement connue |
| Moyenne     | Fichier cr\u00e9\u00e9 dans un chemin syst\u00e8me sensible (`/etc`, `/bin`) sans enregistrement de changement |
| \u00c9lev\u00e9e      | Fichier ex\u00e9cutable, non sign\u00e9, avec nom/contenu suspect |
| Critique    | Fichier confirm\u00e9 malveillant (correspond \u00e0 un hash/signature de malware connu) ou d\u00e9j\u00e0 ex\u00e9cut\u00e9 |

## 4. Confinement

**Actions imm\u00e9diates (15 premi\u00e8res minutes) :**
1. Ne pas ex\u00e9cuter ni ouvrir le fichier suspect.
2. Faire une copie du fichier (vers un emplacement d'analyse isol\u00e9) et enregistrer son hash (SHA256).
3. V\u00e9rifier la liste des processus et les t\u00e2ches planifi\u00e9es/cron pour toute ex\u00e9cution du fichier.

**Confinement \u00e0 court terme :**
- Isoler la machine affect\u00e9e du r\u00e9seau si l'ex\u00e9cution est confirm\u00e9e ou suspect\u00e9e.
- D\u00e9sactiver toute t\u00e2che cron / t\u00e2che planifi\u00e9e r\u00e9f\u00e9ren\u00e7ant le fichier.
- Bloquer le fichier via EDR/antivirus si une correspondance de hash est trouv\u00e9e.

## 5. \u00c9radication

- Supprimer le fichier malveillant de tous les syst\u00e8mes affect\u00e9s.
- Rechercher le m\u00eame hash de fichier sur les autres machines de l'environnement.
- V\u00e9rifier et supprimer tout m\u00e9canisme de persistance (entr\u00e9es cron, services systemd, cl\u00e9s de registre run) cr\u00e9\u00e9 par le fichier.
- Corriger ou fermer le vecteur ayant permis la cr\u00e9ation du fichier (service expos\u00e9, identifiants faibles, application vuln\u00e9rable).

## 6. R\u00e9cup\u00e9ration

- Restaurer les fichiers de configuration affect\u00e9s \u00e0 partir d'une sauvegarde saine si modifi\u00e9s.
- Relancer une analyse de r\u00e9f\u00e9rence FIM pour confirmer l'absence d'autres changements non autoris\u00e9s.
- Surveiller \u00e9troitement la machine pendant 24 \u00e0 48h pour d\u00e9tecter une r\u00e9currence.

## 7. Post-Incident

- Documenter la chronologie compl\u00e8te : temps de d\u00e9tection, temps de triage, temps de confinement.
- D\u00e9terminer la cause racine (comment le fichier a-t-il \u00e9t\u00e9 introduit — acc\u00e8s SSH, web shell, supply chain, interne).
- Mettre \u00e0 jour les chemins surveill\u00e9s par le FIM si des lacunes sont identifi\u00e9es.
- Partager les IoC (hash de fichier, chemin, sch\u00e9ma de nommage) avec l'\u00e9quipe.

## 8. R\u00e9f\u00e9rences

- MITRE ATT&CK T1565.001 : https://attack.mitre.org/techniques/T1565/001/
- Documentation FIM Wazuh : https://documentation.wazuh.com/current/user-manual/capabilities/file-integrity/index.html
- R\u00e9f\u00e9rence du lab : https://github.com/AbrahamDiaco/soc-lab-wazuh
