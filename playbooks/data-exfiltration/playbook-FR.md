# Playbook de R\u00e9ponse \u00e0 Incident — Exfiltration de Donn\u00e9es

**Auteur :** Abraham Diaco
**Derni\u00e8re mise \u00e0 jour :** 2026-06-20
**S\u00e9v\u00e9rit\u00e9 :** Critique
**MITRE ATT&CK :** T1041 — Exfiltration Over C2 Channel / T1567 — Exfiltration Over Web Service

---

## 1. Vue d'ensemble

L'exfiltration de donn\u00e9es est le transfert non autoris\u00e9 de donn\u00e9es hors de l'environnement d'une organisation, que ce soit par un attaquant externe, un compte compromis, ou un acteur interne malveillant/n\u00e9gligent. Ce playbook couvre la d\u00e9tection et la r\u00e9ponse lorsqu'une exfiltration est suspect\u00e9e ou confirm\u00e9e.

## 2. D\u00e9tection

**Indicateurs de compromission (IoC) :**
- Alerte DLP (Data Loss Prevention) pour des donn\u00e9es sensibles quittant le r\u00e9seau (PII, identifiants, propri\u00e9t\u00e9 intellectuelle)
- Volume de donn\u00e9es sortantes inhabituel depuis une machine ou un compte (gros t\u00e9l\u00e9versements vers du stockage cloud, sites de partage de fichiers, ou IP externes inconnues)
- Utilisation de services de stockage cloud / partage de fichiers non autoris\u00e9s (Mega, instances Dropbox/Drive non sanctionn\u00e9es) d\u00e9tect\u00e9e via les logs proxy/pare-feu
- Compression de gros volumes de donn\u00e9es peu avant un transfert sortant (archives `.zip`, `.rar`, `.7z` pr\u00e9par\u00e9es dans des r\u00e9pertoires temporaires)
- Acc\u00e8s anormal \u00e0 des partages de fichiers/bases de donn\u00e9es en dehors du p\u00e9rim\u00e8tre habituel ou des horaires de travail d'un utilisateur

**Exemples d'indicateurs (anonymis\u00e9s) :**
```
Transfert sortant : 2,3 Go vers unrecognized-cloud-storage[.]example via HTTPS
Utilisateur : jdupont — a acc\u00e9d\u00e9 \u00e0 400+ fichiers du partage Finance en 10 minutes
Archive pr\u00e9par\u00e9e : C:\Users\jdupont\AppData\Local\Temp\export_backup.zip
```

**Objectif de temps de d\u00e9tection :** < 10 minutes entre l'alerte DLP/anomalie et le d\u00e9but du triage

## 3. Triage

Questions \u00e0 r\u00e9soudre avant escalade :
- L'activit\u00e9 est-elle coh\u00e9rente avec le r\u00f4le et le comportement habituel de l'utilisateur, ou clairement anormale ?
- Quelles donn\u00e9es ont \u00e9t\u00e9 consult\u00e9es/transf\u00e9r\u00e9es — incluent-elles des donn\u00e9es r\u00e9glement\u00e9es (PII, financi\u00e8res, sant\u00e9, PI) ?
- Le compte est-il confirm\u00e9 compromis, ou s'agit-il potentiellement d'une activit\u00e9 interne ?
- Le transfert est-il termin\u00e9, ou est-il toujours en cours (fen\u00eatre de confinement) ?

**Crit\u00e8res de classification de la s\u00e9v\u00e9rit\u00e9 :**
| S\u00e9v\u00e9rit\u00e9 | Crit\u00e8res |
|----------|-----------|
| Faible      | Anomalie signal\u00e9e mais donn\u00e9es non sensibles, faible volume, explicable par un besoin m\u00e9tier l\u00e9gitime |
| Moyenne     | Sch\u00e9ma de transfert inhabituel impliquant des donn\u00e9es internes (non r\u00e9glement\u00e9es), comportement du compte ambigu |
| \u00c9lev\u00e9e      | Transfert non autoris\u00e9 confirm\u00e9 de donn\u00e9es sensibles/internes, compte probablement compromis |
| Critique    | Exfiltration confirm\u00e9e de donn\u00e9es r\u00e9glement\u00e9es (PII, financi\u00e8res, sant\u00e9) ou de secrets commerciaux/PI |

## 4. Confinement

**Actions imm\u00e9diates (15 premi\u00e8res minutes) :**
1. Si le transfert est en cours : bloquer imm\u00e9diatement l'IP/domaine de destination au niveau du pare-feu/proxy pour stopper la perte de donn\u00e9es en cours.
2. D\u00e9sactiver ou suspendre le compte impliqu\u00e9 (si compromission ou menace interne est suspect\u00e9e).
3. Pr\u00e9server les preuves — ne pas supprimer les logs, les fichiers d'archive pr\u00e9par\u00e9s, ni alt\u00e9rer le syst\u00e8me affect\u00e9 avant la capture forensique.

**Confinement \u00e0 court terme :**
- Isoler la machine affect\u00e9e du r\u00e9seau si une exfiltration pilot\u00e9e par malware est suspect\u00e9e.
- R\u00e9voquer les sessions actives/tokens API du compte affect\u00e9.
- Identifier et bloquer toute infrastructure C2 ou canal d'exfiltration utilis\u00e9 (API de stockage cloud, tunneling DNS, email).

## 5. \u00c9radication

- D\u00e9terminer la cause racine : identifiants compromis, malware, permissions mal configur\u00e9es, ou action interne.
- Supprimer tout malware ou m\u00e9canisme de persistance impliqu\u00e9 dans l'automatisation de l'exfiltration.
- Examiner et renforcer les contr\u00f4les d'acc\u00e8s / permissions de moindre privil\u00e8ge sur les donn\u00e9es consult\u00e9es.
- Fermer le vecteur technique (partage expos\u00e9, identifiants faibles, couverture DLP manquante, application OAuth sur-permissionn\u00e9e).

## 6. R\u00e9cup\u00e9ration

- Restaurer l'acc\u00e8s normal du compte uniquement apr\u00e8s rotation des identifiants et fermeture du vecteur de compromission.
- R\u00e9activer progressivement les syst\u00e8mes avec une surveillance renforc\u00e9e du compte/de la machine pr\u00e9c\u00e9demment affect\u00e9(e).
- Valider que les r\u00e8gles DLP sont correctement ajust\u00e9es pour d\u00e9tecter des sch\u00e9mas similaires \u00e0 l'avenir.

## 7. Post-Incident

- Documenter le p\u00e9rim\u00e8tre complet : quelles donn\u00e9es, quel volume, vers o\u00f9, exfiltration confirm\u00e9e ou suspect\u00e9e.
- D\u00e9terminer les obligations de notification l\u00e9gale/r\u00e9glementaire (RGPD, r\u00e9glementations sectorielles) selon le type de donn\u00e9es et la juridiction — impliquer les \u00e9quipes juridique/conformit\u00e9.
- Mener une analyse des causes racines et mettre \u00e0 jour les r\u00e8gles de d\u00e9tection (politiques DLP, seuils d'anomalie).
- Si une menace interne est confirm\u00e9e, suivre les proc\u00e9dures d'escalade RH/juridique de l'organisation.

## 8. R\u00e9f\u00e9rences

- MITRE ATT&CK T1041 : https://attack.mitre.org/techniques/T1041/
- MITRE ATT&CK T1567 : https://attack.mitre.org/techniques/T1567/
- Playbook li\u00e9 : D\u00e9tection et Confinement Ransomware / Malware
