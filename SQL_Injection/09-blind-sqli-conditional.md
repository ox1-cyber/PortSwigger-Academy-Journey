# 💉 Lab : Blind SQL injection with conditional responses
**Catégorie :** Injection SQL (SQLi)
**Type :** Blind SQLi (Boolean-Based / Réponses conditionnelles)
**Plateforme :** PortSwigger Web Security Academy

---

## 📌 Résumé de la Vulnérabilité
L'application est vulnérable à une injection SQL en aveugle (Blind SQLi) via le cookie `TrackingId`. La base de données ne renvoie aucune donnée visible et aucune erreur. Cependant, l'application modifie son comportement (affichage du message "Welcome back") si la requête SQL injectée est évaluée comme vraie. Cette faille booléenne (Vrai/Faux) permet d'exfiltrer des données critiques caractère par caractère.

## 🛠️ Preuve de Concept (PoC)
L'extraction du mot de passe administrateur a été réalisée en automatisant des requêtes conditionnelles avec Burp Suite Intruder :

1. **Confirmation de la vulnérabilité :**
   - `Cookie: TrackingId=xyz' AND '1'='1` -> Le message "Welcome back" s'affiche (**VRAI**).
   - `Cookie: TrackingId=xyz' AND '1'='2` -> Le message disparaît (**FAUX**).

2. **Exfiltration automatisée (Brute-force) :**
   Utilisation d'une attaque "Cluster Bomb" dans Burp Intruder pour tester chaque position et chaque caractère du mot de passe ciblé, en se basant sur la fonction `SUBSTRING`.
   - **Payload :** `Cookie: TrackingId=xyz' AND SUBSTRING((SELECT password FROM users WHERE username='administrator'),§1§,1)='§a§`
   - **Position 1 :** Nombres de 1 à 20 (Index du caractère).
   - **Position 2 :** Liste de caractères (`a-z`, `0-9`).

3. **Résultat :**
   L'analyse de la longueur des réponses HTTP (11611 bytes pour un succès vs 11550 bytes pour un échec) a permis de reconstituer le mot de passe complet :
   - **Mot de passe :** `xavjkxw19ffoh8079zfq`

*La connexion avec ces identifiants a permis de prendre le contrôle du compte administrateur.*

## 🛡️ Recommandation de Sécurité
Les valeurs des cookies sont des entrées utilisateurs manipulables. Elles ne doivent jamais être concaténées dans une requête SQL. L'utilisation systématique de requêtes préparées (Prepared Statements) est la seule parade efficace contre ce type d'injection.