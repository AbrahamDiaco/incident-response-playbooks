# Playbook de Réponse à Incident — Exfiltration de Données

**Auteur :** Abraham Diaco
**Dernière mise à jour :** 2026-06-20
**Sévérité :** Critique
**MITRE ATT&CK :** T1041 — Exfiltration Over C2 Channel / T1567 — Exfiltration Over Web Service

---

## 1. Vue d'ensemble

L'exfiltration de données est le transfert non autorisé de données hors de l'environnement d'une organisation, qu'il soit réalisé par un attaquant externe, un compte compromis, ou une menace interne malveillante ou négligente. Ce playbook couvre la détection et la réponse lorsqu'une exfiltration est suspectée ou confirmée.

## 2. Détection

**Indicateurs de compromission (IoC) :**
- Alerte DLP (Data Loss Prevention) pour des données sensibles quittant le réseau (données personnelles, identifiants, propriété intellectuelle)
- Volume de données sortantes anormal depuis un hôte ou un compte (téléversements volumineux vers du stockage cloud, des sites de partage de fichiers, ou des IP externes inconnues)
- Utilisation de services de stockage cloud/partage de fichiers non autorisés (Mega, instances non sanctionnées de Dropbox/Drive) détectée via les logs proxy/pare-feu
- Compression d'ensembles de données volumineux peu avant un transfert sortant (archives `.zip`, `.rar`, `.7z` préparées dans des répertoires temporaires)
- Accès anormal à des partages de fichiers/bases de données en dehors des fonctions habituelles ou des horaires de travail de l'utilisateur

**Exemples d'indicateurs (anonymisés) :**
```
Transfert sortant : 2,3 Go vers unrecognized-cloud-storage[.]example via HTTPS
Utilisateur : jdupont — a accédé à plus de 400 fichiers sur le partage Finance en 10 minutes
Archive préparée : C:\Users\jdupont\AppData\Local\Temp\export_backup.zip
```

**Objectif de délai de détection :** moins de 10 minutes entre l'alerte DLP/anomalie et le début du triage

## 3. Triage

Questions à résoudre avant l'escalade :
- L'activité est-elle cohérente avec le rôle et le comportement habituel de l'utilisateur, ou clairement anormale ?
- Quelles données ont été consultées/transférées — incluent-elles des données réglementées (données personnelles, financières, de santé, propriété intellectuelle) ?
- Le compte est-il confirmé comme compromis, ou s'agit-il d'une activité interne potentielle ?
- Le transfert est-il terminé, ou toujours en cours (fenêtre de confinement) ?

**Critères de classification de la sévérité :**

| Sévérité | Critères |
|----------|----------|
| Faible | Anomalie signalée mais données non sensibles, faible volume, explicable par un besoin métier légitime |
| Moyenne | Schéma de transfert inhabituel impliquant des données internes (non réglementées), comportement du compte incertain |
| Élevée | Transfert non autorisé confirmé de données sensibles/internes, compte probablement compromis |
| Critique | Exfiltration confirmée de données réglementées (données personnelles, financières, de santé) ou de secrets commerciaux/propriété intellectuelle |

## 4. Confinement

**Actions immédiates (15 premières minutes) :**
1. Si le transfert est en cours : bloquer immédiatement l'IP/domaine de destination au niveau du pare-feu/proxy pour arrêter la perte de données en cours.
2. Désactiver ou suspendre le compte concerné (si une compromission ou une menace interne est suspectée).
3. Préserver les preuves — ne pas supprimer les logs, les fichiers d'archive préparés, ni altérer le système affecté avant la capture forensique.

**Confinement à court terme :**
- Isoler l'hôte affecté du réseau si l'exfiltration est suspectée d'être pilotée par un malware.
- Révoquer les sessions actives/jetons API du compte concerné.
- Identifier et bloquer toute infrastructure C2 ou canal d'exfiltration utilisé (API de stockage cloud, tunneling DNS, email).

## 5. Éradication

- Déterminer la cause racine : identifiants compromis, malware, permissions mal configurées, ou action d'un utilisateur interne.
- Supprimer tout malware ou mécanisme de persistance ayant servi à automatiser l'exfiltration.
- Examiner et renforcer les contrôles d'accès/permissions au moindre privilège sur les données consultées.
- Fermer le vecteur technique (partage exposé, identifiants faibles, couverture DLP manquante, application OAuth disposant de trop de permissions).

## 6. Récupération

- Restaurer l'accès normal au compte uniquement après rotation des identifiants et fermeture du vecteur de compromission.
- Réactiver les systèmes progressivement avec une surveillance renforcée sur le compte/hôte précédemment affecté.
- Valider que les règles DLP sont correctement ajustées pour détecter des schémas similaires à l'avenir.

## 7. Post-Incident

- Documenter le périmètre complet : quelles données, quel volume, où elles ont été transférées, exfiltration confirmée ou suspectée.
- Déterminer les obligations légales/réglementaires de notification (RGPD, réglementations sectorielles) selon le type de données et la juridiction — impliquer les équipes juridique/conformité.
- Mener une analyse de cause racine et mettre à jour les règles de détection (politiques DLP, seuils d'anomalie).
- Si une menace interne est confirmée, suivre les procédures d'escalade RH/juridique de l'organisation.

## 8. Références

- MITRE ATT&CK T1041 : https://attack.mitre.org/techniques/T1041/
- MITRE ATT&CK T1567 : https://attack.mitre.org/techniques/T1567/
- Playbook lié : Détection et Confinement Ransomware / Malware
