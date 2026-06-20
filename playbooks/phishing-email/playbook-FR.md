# Playbook de Réponse à Incident — Email de Phishing

**Auteur :** Abraham Diaco
**Dernière mise à jour :** 2026-06-20
**Sévérité :** Moyenne à Élevée (selon l'interaction de l'utilisateur)
**MITRE ATT&CK :** T1566.001 / T1566.002 — Phishing: Spearphishing Attachment / Link

---

## 1. Vue d'ensemble

Le phishing reste l'un des vecteurs d'accès initial les plus courants. Ce playbook couvre la réponse lorsqu'un email de phishing est signalé par un utilisateur ou détecté par les outils de sécurité email, y compris les cas où l'utilisateur a pu cliquer sur un lien, ouvrir une pièce jointe, ou soumettre des identifiants.

## 2. Détection

**Indicateurs de compromission (IoC) :**
- Email suspect signalé par un utilisateur (via le bouton « Signaler un phishing » ou un ticket helpdesk)
- Alerte de la passerelle email sécurisée (SEG) — expéditeur usurpé, pièce jointe suspecte, URL malveillante connue
- Schémas de flux de courrier anormaux (par ex. envoi massif depuis une seule boîte mail, domaine similaire/typosquatté)
- Anomalie de connexion peu après l'ouverture de l'email (nouvelle localisation, nouvel appareil, déplacement impossible)

**Exemples d'indicateurs (anonymisés) :**
```
Expéditeur : support@micros0ft-secure.com  (domaine similaire)
Sujet : « Action requise : vérifiez votre compte »
Pièce jointe : facture_2026.html.zip
```

**Objectif de délai de détection :** moins de 15 minutes entre le signalement de l'utilisateur et le début du triage

## 3. Triage

Questions à résoudre avant l'escalade :
- L'utilisateur a-t-il cliqué sur le lien, ouvert la pièce jointe, ou saisi des identifiants ?
- S'agit-il d'un utilisateur isolé visé spécifiquement, ou d'une campagne de masse à l'échelle de l'organisation ?
- L'email correspond-il à une campagne de phishing connue (vérifier les flux de threat intel) ?
- Le domaine/IP de l'expéditeur est-il déjà signalé ailleurs ?

**Critères de classification de la sévérité :**

| Sévérité | Critères |
|----------|----------|
| Faible | Signalé mais non ouvert/cliqué ; clairement identifiable comme phishing |
| Moyenne | Lien cliqué mais aucun identifiant saisi, aucune pièce jointe exécutée |
| Élevée | Identifiants saisis sur une page de phishing, ou pièce jointe exécutée |
| Critique | Compromission de compte confirmée ou exécution de malware suite au clic |

## 4. Confinement

**Actions immédiates (15 premières minutes) :**
1. Identifier tous les destinataires de l'email à travers l'organisation (recherche dans les logs mail/SIEM).
2. Retirer l'email de toutes les boîtes de réception (mise en quarantaine via l'outil de sécurité email).
3. Bloquer le domaine/IP de l'expéditeur ainsi que toute URL malveillante au niveau de la passerelle email et du proxy web/pare-feu.

**Confinement à court terme :**
- Si des identifiants ont été soumis : forcer une réinitialisation immédiate du mot de passe et invalider les sessions/jetons actifs du compte concerné.
- Si une pièce jointe a été exécutée : isoler le poste affecté du réseau en attendant l'investigation (voir le playbook Ransomware/Malware si applicable).
- Avertir les utilisateurs concernés et leur fournir des consignes (ne pas cliquer sur d'autres liens, signaler toute activité inhabituelle).

## 5. Éradication

- Confirmer qu'aucun mécanisme de persistance n'a été installé (vérifier les tâches planifiées, les extensions de navigateur, les règles de boîte mail — les attaquants créent souvent des règles de transfert pour exfiltrer les futurs emails).
- Supprimer toute règle de boîte mail malveillante ou autorisation d'application OAuth créée par l'attaquant.
- Bloquer l'infrastructure de phishing (domaine, IP, URL) à tous les niveaux (email, web, DNS).

## 6. Récupération

- Restaurer le poste affecté à partir d'une image saine si un malware a été exécuté.
- Réactiver le compte de l'utilisateur avec de nouveaux identifiants et, si possible, imposer la MFA.
- Surveiller étroitement le compte/poste affecté pendant 7 à 14 jours pour détecter une activité consécutive.

## 7. Post-Incident

- Documenter la chronologie complète : heure de livraison de l'email, heure de signalement, heure de confinement, nombre d'utilisateurs affectés.
- Partager les IoC (domaine expéditeur, URL, hash de pièce jointe) avec l'équipe sécurité et mettre à jour les règles de filtrage email.
- Envisager une formation de sensibilisation ciblée pour les utilisateurs ou le service concerné.
- Suivre les métriques au niveau de la campagne si plusieurs incidents de phishing sont liés (infrastructure expéditrice récurrente, thématiques communes).

## 8. Références

- MITRE ATT&CK T1566.001 : https://attack.mitre.org/techniques/T1566/001/
- MITRE ATT&CK T1566.002 : https://attack.mitre.org/techniques/T1566/002/
- Playbook lié : Détection et Confinement Ransomware / Malware
