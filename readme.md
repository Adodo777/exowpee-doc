# Exowpee - Readme

## Vue d'ensemble

Exowpee est une plateforme de gestion d'entreprise multi-utilisateurs avec un système de profils et rôles granulaire. La plateforme permet la création d'entreprises, la gestion d'utilisateurs (standard et guest), et un système d'invitations complet.

## Architecture des utilisateurs

### Types d'utilisateurs
- **User** : Utilisateur standard lié à une adresse email, peut appartenir à plusieurs entreprises
- **Soft User (Guest)** : Utilisateur créé par l'entreprise, lié uniquement à une entreprise, authentification par username/mot de passe

### Système de profils et rôles
- Chaque utilisateur est nécessairement lié à une entreprise
- Un utilisateur peut avoir plusieurs profils
- Chaque profil contient un ensemble de rôles
- Chaque rôle contient un ensemble de permissions
- Les sessions sont basées sur un profil spécifique

---

## Flows principaux et appels API

### 1. Création d'entreprise

#### Flow UX
1. **Page d'accueil** → Clic sur "C'est parti"
2. **Step 1** : Création utilisateur principal
   - Si pas de compte : Nom, prénom, email → "Continuer"
   - Si compte existant : "Se connecter" → Connexion puis continuer
3. **Step 2** : Informations entreprise
   - Nom, adresse, téléphone, identifiant fiscal, registre de commerce
4. **Step 3** : Paramètres généraux
   - Langue, pays, fuseau horaire, monnaie → "Terminer"
5. **Finalisation** :
   - Si nouvelle inscription : Message confirmation → "Se connecter" (mot de passe envoyé par email)
   - Si connexion existante : "Accéder au dashboard"

#### Appels API
- **Utilisateur sans compte** :
  - `verifier si l'email existe`
  - `créer l'utilisateur et l'entreprise`
  - Redirection : Page de connexion
- **Utilisateur avec compte** :
  - `se connecter`
  - `créer l'entreprise`
  - Redirection : Dashboard

### 2. Rejoindre une entreprise (Invitations)

#### Flow UX
1. **Réception email d'invitation** → Clic sur le lien
2. **Page de connexion spéciale "join company"**
   - Si pas de compte : CTA "Créer un compte"
   - Si compte : Se connecter normalement
3. **Création de compte** (si nécessaire) :
   - Nom, prénom, nationalité, téléphone, email
   - Mot de passe envoyé par email
4. **Acceptation invitation** :
   - Bouton "Accepter l'invitation"
   - Redirection selon nombre de profils

#### Appels API
- **Utilisateur sans compte** :
  - `créer le compte et accepter l'invitation`
  - Redirection : Dashboard
- **Utilisateur avec compte** :
  - `se connecter`
  - `accepter l'invitation`
  - Redirection : Dashboard

### 3. Authentification

#### Flow UX Standard
1. **Page de connexion** :
   - Email + mot de passe → "Se connecter"
   - Lien "Mot de passe oublié ?"
   - Lien "Connexion Guest"

2. **Sélection de profil** (si plusieurs) :
   - Liste des entreprises et profils
   - Checkbox pour définir le choix par défaut
   - Si un seul profil : redirection directe au dashboard

#### Flow UX Guest
1. **Connexion via lien personnalisé** :
   - Username + mot de passe (ID entreprise dans l'URL)
2. **Connexion sans lien** :
   - Username + mot de passe + ID entreprise

#### Récupération mot de passe
1. **Modal "Mot de passe oublié"** :
   - Saisie email → Envoi OTP
2. **Écran de réinitialisation** :
   - Code OTP + nouveau mot de passe + confirmation

#### Appels API
- `se connecter`
- `se connecter en tant que soft_user` (pour guests)
- `choisir un profil` (si nécessaire)
- `envoyer otp` (récupération mot de passe)
- `modifier mot de passe`

### 4. Gestion du compte utilisateur

#### Flow UX
1. **Page User Account** (uniquement pour Users, pas les Guests) :
   - Modification : nom, prénom, nationalité, téléphone, email
   - Boutons : "Mot de passe" et "Enregistrer"

2. **Modification avec changement d'email** :
   - Modal de confirmation
   - Envoi de 2 codes OTP (ancien et nouveau email)
   - Saisie des deux codes + validation

3. **Modification mot de passe** :
   - Ancien + nouveau + confirmation
   - Si oublié ancien : Envoi OTP → Code + nouveau + confirmation

4. **Suppression de compte** :
   - Envoi OTP → Confirmation

#### Appels API
- `envoyer otps` (si email modifié)
- `mettre à jour user`
- `mettre à jour mot de passe`
- `supprimer user`

### 5. Gestion des utilisateurs

#### Flow UX
1. **Page principale avec 2 tableaux** :

   **Tableau 1 - Utilisateurs** :
   - Colonnes : Username, Email (vide si guest), Type, Profils, Actions
   - Actions : Voir, Activités, Modifier, Supprimer

   **Tableau 2 - Invitations** :
   - Colonnes : Email, Profils, Statut (en cours/expiré), Actions
   - Actions en cours : Modifier, Supprimer, Renvoyer email
   - Actions expirées : Actualiser, Supprimer

2. **Création d'utilisateur** :
   - Checkbox type (User/Guest)
   - Si User : email, username
   - Si Guest : username, mot de passe
   - Assignation profils avec options par profil :
     - Désactivation utilisateur
     - Restriction magasin
     - Permissions extra

#### Appels API
- `récupérer utilisateurs de l'entreprise`
- `récupérer invitations`
- **Soft User** : `récupérer ses données`, `mettre à jour`, `supprimer`
- **User** : Même chose sauf accès au mot de passe

### 6. Gestion des invitations

#### Flow UX
- Création depuis la page utilisateurs
- Modification des invitations en cours
- Renouvellement des invitations expirées
- Suppression d'invitations

#### Appels API
- `récupérer invitations non acceptées`
- `créer un soft_user`
- `inviter un user dans l'entreprise`
- `supprimer une invitation`
- `renvoyer email d'invitation`
- `modifier une invitation en cours`
- `renouveler une invitation expirée`

### 7. Gestion des profils et rôles

#### Flow UX Rôles
1. **Page de création de rôle** :
   - Nom unique + sélection permissions
2. **Tableau des rôles** :
   - Colonnes : Nom, Actions (Modifier, Supprimer)

#### Flow UX Profils
1. **Page de création de profil** :
   - Nom + description + sélection des rôles
2. **Tableau des profils** :
   - Colonnes : Nom, Description, Actions (Modifier, Supprimer)

#### Appels API
- `récupérer la liste`
- `créer`
- `modifier`
- `supprimer`

---

## Règles de redirection

### Après connexion
- **Un seul profil** : Dashboard direct
- **Plusieurs profils** : Page de sélection de profil
- **Choix par défaut défini** : Dashboard avec profil par défaut

### Switch entre profils
- Bouton dans le popover avatar (navbar)
- Changement de session instantané

---

## Spécificités techniques

### Authentification
- Les Users utilisent email/mot de passe
- Les Guests utilisent username/mot de passe + ID entreprise
- Sessions liées à un profil spécifique

### Sécurité
- Système OTP pour modifications sensibles
- Vérification double email lors des changements
- Permissions granulaires par rôle

### Contraintes
- Un Guest ne peut appartenir qu'à une seule entreprise
- Un User peut appartenir à plusieurs entreprises
- Chaque utilisateur doit avoir au moins un profil
- Les profils sont spécifiques à une entreprise

---

## États et statuts

### Utilisateurs
- Actif/Désactivé (global et par profil)
- Restrictions par magasin
- Permissions extra individuelles

### Invitations
- **En cours** : Peut être modifiée, renvoyée, supprimée
- **Expirée** : Peut être renouvelée ou supprimée

### Sessions
- Liée à un profil spécifique
- Changement de profil = changement de session

---

Cette documentation couvre l'ensemble des flows et fonctionnalités de la plateforme Exowpee, en combinant les spécifications techniques backend avec l'expérience utilisateur conçue.