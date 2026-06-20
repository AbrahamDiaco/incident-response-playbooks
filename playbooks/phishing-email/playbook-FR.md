# Playbook de R\u00e9ponse \u00e0 Incident — Email de Phishing

**Auteur :** Abraham Diaco
**Derni\u00e8re mise \u00e0 jour :** 2026-06-20
**S\u00e9v\u00e9rit\u00e9 :** Moyenne \u00e0 \u00c9lev\u00e9e (selon l'interaction utilisateur)
**MITRE ATT&CK :** T1566.001 / T1566.002 — Phishing: Spearphishing Attachment / Link

---

## 1. Vue d'ensemble

Le phishing reste l'un des vecteurs d'acc\u00e8s initial les plus courants. Ce playbook couvre la r\u00e9ponse lorsqu'un email de phishing est signal\u00e9 par un utilisateur ou d\u00e9tect\u00e9 par les outils de s\u00e9curit\u00e9 email, y compris les cas o\u00f9 l'utilisateur a pu cliquer sur un lien, ouvrir une pi\u00e8ce jointe ou soumettre des identifiants.

## 2. D\u00e9tection

**Indicateurs de compromission (IoC) :**
- Email suspect signal\u00e9 par un utilisateur (via le bouton "Signaler un phishing" ou un ticket helpdesk)
- Alerte de la passerelle email s\u00e9curis\u00e9e (SEG) — exp\u00e9diteur usurp\u00e9, pi\u00e8ce jointe suspecte, URL malveillante connue
- Sch\u00e9mas de flux de courrier anormaux (ex. envoi massif depuis une seule bo\u00eete mail, domaine similaire/typosquatt\u00e9)
- Anomalie de connexion peu apr\u00e8s l'ouverture de l'email (nouvelle localisation, nouvel appareil, d\u00e9placement impossible)

**Exemples d'indicateurs (anonymis\u00e9s) :**
```
Exp\u00e9diteur : support@micros0ft-secure.com  (domaine similaire)
Sujet : "Action requise : v\u00e9rifiez votre compte"
Pi\u00e8ce jointe : facture_2026.html.zip
```

**Objectif de temps de d\u00e9tection :** < 15 minutes entre le signalement utilisateur et le d\u00e9but du triage

## 3. Triage

Questions \u00e0 r\u00e9soudre avant escalade :
- L'utilisateur a-t-il cliqu\u00e9 sur le lien, ouvert la pi\u00e8ce jointe, ou saisi des identifiants ?
- S'agit-il d'un utilisateur cibl\u00e9 isol\u00e9 ou d'une campagne de masse \u00e0 l'\u00e9chelle de l'organisation ?
- L'email correspond-il \u00e0 une campagne de phishing connue (v\u00e9rifier les flux de threat intel) ?
- Le domaine/IP de l'exp\u00e9diteur est-il d\u00e9j\u00e0 signal\u00e9 ailleurs ?

**Crit\u00e8res de classification de la s\u00e9v\u00e9rit\u00e9 :**
| S\u00e9v\u00e9rit\u00e9 | Crit\u00e8res |
|----------|-----------|
| Faible      | Signal\u00e9 mais non ouvert/cliqu\u00e9 ; clairement identifiable comme phishing |
| Moyenne     | Lien cliqu\u00e9 mais aucun identifiant saisi, aucune pi\u00e8ce jointe ex\u00e9cut\u00e9e |
| \u00c9lev\u00e9e      | Identifiants saisis sur une page de phishing, ou pi\u00e8ce jointe ex\u00e9cut\u00e9e |
| Critique    | Compromission de compte confirm\u00e9e ou ex\u00e9cution de malware suite au clic |

## 4. Confinement

**Actions imm\u00e9diates (15 premi\u00e8res minutes) :**
1. Identifier tous les destinataires de l'email dans l'organisation (recherche dans les logs mail/SIEM).
2. Retirer l'email de toutes les bo\u00eetes de r\u00e9ception (mise en quarantaine via l'outil de s\u00e9curit\u00e9 email).
3. Bloquer le domaine/IP de l'exp\u00e9diteur et toute URL malveillante au niveau de la passerelle email et du proxy web/pare-feu.

**Confinement \u00e0 court terme :**
- Si des identifiants ont \u00e9t\u00e9 soumis : forcer une r\u00e9initialisation imm\u00e9diate du mot de passe et invalider les sessions/tokens actifs du compte affect\u00e9.
- Si une pi\u00e8ce jointe a \u00e9t\u00e9 ex\u00e9cut\u00e9e : isoler le poste affect\u00e9 du r\u00e9seau en attendant l'investigation (voir playbook Ransomware/Malware si applicable).
- Pr\u00e9venir les utilisateurs affect\u00e9s et leur donner des consignes (ne pas cliquer sur d'autres liens, signaler toute activit\u00e9 inhabituelle).

## 5. \u00c9radication

- Confirmer qu'aucun m\u00e9canisme de persistance n'a \u00e9t\u00e9 install\u00e9 (v\u00e9rifier les t\u00e2ches planifi\u00e9es, extensions de navigateur, r\u00e8gles de bo\u00eete mail — les attaquants cr\u00e9ent souvent des r\u00e8gles de transfert pour exfiltrer les futurs emails).
- Supprimer toute r\u00e8gle de bo\u00eete mail malveillante ou autorisation d'application OAuth cr\u00e9\u00e9e par l'attaquant.
- Bloquer l'infrastructure de phishing (domaine, IP, URL) \u00e0 tous les niveaux (email, web, DNS).

## 6. R\u00e9cup\u00e9ration

- Restaurer le poste affect\u00e9 \u00e0 partir d'une image saine si un malware a \u00e9t\u00e9 ex\u00e9cut\u00e9.
- R\u00e9activer le compte utilisateur avec de nouveaux identifiants et, si possible, imposer la MFA.
- Surveiller \u00e9troitement le compte/poste affect\u00e9 pendant 7 \u00e0 14 jours pour d\u00e9tecter une activit\u00e9 cons\u00e9cutive.

## 7. Post-Incident

- Documenter la chronologie compl\u00e8te : heure de livraison de l'email, heure de signalement, heure de confinement, nombre d'utilisateurs affect\u00e9s.
- Partager les IoC (domaine exp\u00e9diteur, URL, hash de pi\u00e8ce jointe) avec l'\u00e9quipe s\u00e9curit\u00e9 et mettre \u00e0 jour les r\u00e8gles de filtrage email.
- Envisager une formation de sensibilisation cibl\u00e9e pour les utilisateurs ou le service affect\u00e9.
- Suivre les m\u00e9triques au niveau campagne si plusieurs incidents de phishing sont li\u00e9s (infrastructure exp\u00e9ditrice r\u00e9currente, th\u00e9matiques).

## 8. R\u00e9f\u00e9rences

- MITRE ATT&CK T1566.001 : https://attack.mitre.org/techniques/T1566/001/
- MITRE ATT&CK T1566.002 : https://attack.mitre.org/techniques/T1566/002/
- Playbook li\u00e9 : D\u00e9tection et Confinement Ransomware / Malware
