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
- [ ] 🔴 **Expert** : Blind SQL injection with conditional responses

---

## ⚔️ L'Arsenal Tactique (Cheatsheet)
*Mes munitions prêtes à l'emploi pour le Bug Bounty.*

### 1. Contournement & Manipulation de Logique
*   **Payload universel (Vrai absolu) :** `' OR 1=1--`
    *   *Explication :* Ferme la chaîne de caractères initiale, force une condition validée, et commente le reste de la requête backend.
