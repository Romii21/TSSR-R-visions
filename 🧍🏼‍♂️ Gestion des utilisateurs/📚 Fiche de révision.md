# Les Utilisateurs dans les Systèmes d'Exploitation

## Synthèse Complète

---

## 📑 Sommaire

1. [Chapitre 1 : Fondamentaux](https://claude.ai/chat/b838ff63-7369-43a5-a8c2-998f210dec1c#chapitre-1--fondamentaux)
    
    - [Définition d'un utilisateur](https://claude.ai/chat/b838ff63-7369-43a5-a8c2-998f210dec1c#d%C3%A9finition-dun-utilisateur)
    - [Les groupes](https://claude.ai/chat/b838ff63-7369-43a5-a8c2-998f210dec1c#les-groupes)
    - [Identifiants uniques](https://claude.ai/chat/b838ff63-7369-43a5-a8c2-998f210dec1c#identifiants-uniques)
2. [Chapitre 2 : Les Droits d'Accès](https://claude.ai/chat/b838ff63-7369-43a5-a8c2-998f210dec1c#chapitre-2--les-droits-dacc%C3%A8s)
    
    - [Concepts généraux](https://claude.ai/chat/b838ff63-7369-43a5-a8c2-998f210dec1c#concepts-g%C3%A9n%C3%A9raux)
    - [Sur Linux](https://claude.ai/chat/b838ff63-7369-43a5-a8c2-998f210dec1c#sur-linux)
    - [Sur Windows](https://claude.ai/chat/b838ff63-7369-43a5-a8c2-998f210dec1c#sur-windows)
    - [Répertoire personnel](https://claude.ai/chat/b838ff63-7369-43a5-a8c2-998f210dec1c#r%C3%A9pertoire-personnel)
3. [Chapitre 3 : La Sécurité](https://claude.ai/chat/b838ff63-7369-43a5-a8c2-998f210dec1c#chapitre-3--la-s%C3%A9curit%C3%A9)
    
    - [Gestion des Identités et Accès (IAM)](https://claude.ai/chat/b838ff63-7369-43a5-a8c2-998f210dec1c#gestion-des-identit%C3%A9s-et-acc%C3%A8s-iam)
    - [Identification et authentification](https://claude.ai/chat/b838ff63-7369-43a5-a8c2-998f210dec1c#identification-et-authentification)
4. [Chapitre 4 : Gestion sur Linux](https://claude.ai/chat/b838ff63-7369-43a5-a8c2-998f210dec1c#chapitre-4--gestion-des-utilisateurs-sur-linux)
    
    - [Liste des utilisateurs](https://claude.ai/chat/b838ff63-7369-43a5-a8c2-998f210dec1c#liste-des-utilisateurs-linux)
    - [Base des mots de passe](https://claude.ai/chat/b838ff63-7369-43a5-a8c2-998f210dec1c#base-des-mots-de-passe)
    - [Base des groupes](https://claude.ai/chat/b838ff63-7369-43a5-a8c2-998f210dec1c#base-des-groupes)
    - [Commandes utiles](https://claude.ai/chat/b838ff63-7369-43a5-a8c2-998f210dec1c#commandes-utiles-linux)
5. [Chapitre 5 : Gestion sur Windows](https://claude.ai/chat/b838ff63-7369-43a5-a8c2-998f210dec1c#chapitre-5--gestion-des-utilisateurs-sur-windows)
    
    - [Liste des utilisateurs](https://claude.ai/chat/b838ff63-7369-43a5-a8c2-998f210dec1c#liste-des-utilisateurs-windows)
    - [Base SAM](https://claude.ai/chat/b838ff63-7369-43a5-a8c2-998f210dec1c#base-sam)
    - [Groupes](https://claude.ai/chat/b838ff63-7369-43a5-a8c2-998f210dec1c#groupes-windows)
    - [Commandes utiles](https://claude.ai/chat/b838ff63-7369-43a5-a8c2-998f210dec1c#commandes-utiles-windows)
6. [Synthèse comparative](https://claude.ai/chat/b838ff63-7369-43a5-a8c2-998f210dec1c#synth%C3%A8se-comparative)
    

---

## Chapitre 1 : Fondamentaux

### Définition d'un utilisateur

> **C'est quoi au juste ?**

En informatique, un **utilisateur** est une identité qui utilise un système d'exploitation. Ce n'est pas nécessairement une personne !

#### Trois formes d'utilisateurs :

```
┌─────────────────────────────────────────┐
│         LES UTILISATEURS                │
├─────────────────────────────────────────┤
│                                         │
│  👤 ÊTRES HUMAINS                      │
│     (Alice, Bob, administrateur)       │
│                                         │
│  🔧 SERVICES                           │
│     (www-data, mail, ftp, mysql)       │
│                                         │
│  👑 RÔLES                              │
│     (root, administrateur, invité)     │
│                                         │
└─────────────────────────────────────────┘
```

**Exemple concret :**

- L'utilisateur `www-data` sur Linux n'est pas une personne, mais un service web
- L'utilisateur `root` est le superadministrateur (UID = 0 par convention)

---

### Les groupes

> **Organiser les utilisateurs ensemble**

Les **groupes** sont des contenants qui permettent de regrouper les utilisateurs et de leur attribuer les mêmes droits d'accès.

#### Différences Linux / Windows :

|Aspect|Linux|Windows|
|---|---|---|
|**Contiennent**|Uniquement des utilisateurs|Utilisateurs OU d'autres groupes|
|**Hiérarchie**|Linéaire|Imbriquée possible|
|**Utilité**|Simplifier la gestion des droits|Gestion plus fine et flexible|

**Avantage principal :** Au lieu de donner des droits à chaque utilisateur individuellement, on les donne au groupe. C'est plus simple à maintenir !

```
Exemple de structure :
┌──────────────────┐
│  Groupe "Admin"  │
├──────────────────┤
│ • Alice          │
│ • Bob            │
└──────────────────┘
       ↓
   [Tous ces droits]
```

---

### Identifiants uniques

> **Comment l'OS reconnaît chaque utilisateur**

Chaque utilisateur (et groupe) reçoit un **identifiant unique (UID)** attribué par l'OS. C'est ce numéro qui est utilisé en interne.

#### Sur Linux : UID (User Identifier)

```bash
# Commande pour voir ses identifiants
$ id wilder
uid=1000(wilder) gid=1000(wilder) groupes=1000(wilder),4(adm),27(sudo)
```

**Convention Linux :**

- `root` : UID = 0
- Utilisateurs système : UID < 1000
- Utilisateurs normaux : UID ≥ 1000

#### Sur Windows : SID (Security Identifier)

```powershell
# Commande pour voir ses identifiants
PS> Get-WmiObject win32_useraccount | Where {$_.Name -eq "wilder"} | Select Name,SID

Name   SID
----   ---
wilder S-1-5-21-2973921518-4066818644-3297592939-500
```

**Particularité :** Les SID Windows sont beaucoup plus longs et complexes, calculés avec des éléments du domaine.

```
┌─────────────────────────────────────┐
│  IDENTIFIANTS UNIQUES               │
├─────────────────────────────────────┤
│                                     │
│  Linux      → UID simple (1000)    │
│                                     │
│  Windows    → SID complexe         │
│              (S-1-5-21-...)        │
│                                     │
│  → Permettent d'identifier         │
│     l'utilisateur auprès du        │
│     système et de déterminer       │
│     les ressources accessibles     │
│                                     │
└─────────────────────────────────────┘
```

---

## Chapitre 2 : Les Droits d'Accès

### Concepts généraux

> **Qu'est-ce qu'un droit d'accès ?**

Les **droits d'accès** sont des métadonnées qui décrivent ce qu'un utilisateur peut faire avec un fichier ou un dossier.

#### Cinq types de droits :

|Droit|Symbole|Signification|
|---|---|---|
|**Lecture**|R / read|Voir et copier le contenu|
|**Écriture**|W / write|Modifier ou supprimer le contenu|
|**Exécution**|X / execute|Lancer un programme ou accéder à un dossier|
|**Modification**|-|Changer les propriétés|
|**Suppression**|-|Effacer le fichier/dossier|

**Cas du refus/interdiction :** Empêcher explicitement un accès.

---

### Sur Linux

> **Le modèle classique à 3 niveaux**

Chaque fichier Linux a des droits définis par **3 catégories d'utilisateurs** :

```
┌──────────────────────────────────────┐
│  TROIS NIVEAUX DE DROITS (Linux)     │
├──────────────────────────────────────┤
│                                      │
│  👤 Propriétaire (UID)              │
│     → Droits du possesseur          │
│                                      │
│  👥 Groupe (GID)                    │
│     → Droits pour le groupe         │
│                                      │
│  🌍 Autres (other)                  │
│     → Droits pour le reste          │
│                                      │
└──────────────────────────────────────┘
```

#### Affichage des droits avec `ls -l`

```bash
$ ls -l /home/wilder/file.txt
-rw-rw-r-- 1 wilder wilder 512 sept. 23 15:45 file.txt
 ↓ ↓↓↓ ↓↓↓ ↓↓↓
 │  │   │   │
 │  │   │   └─ Autres (other)
 │  │   └───── Groupe (group)
 │  └───────── Propriétaire (user)
 └──────────── Type (- = fichier, d = dossier)
```

**Lecture du format :**

- Chaque niveau a 3 caractères : `rwx` (read, write, execute)
- Un `-` signifie l'absence du droit

**Exemple :** `-rw-rw-r--`

- Propriétaire : `rw-` → lecture + écriture
- Groupe : `rw-` → lecture + écriture
- Autres : `r--` → lecture uniquement

#### Droits avancés : ACL (Access Control List)

Pour des droits **plus fins**, on peut utiliser les **ACL** qui se superposent aux droits classiques.

```bash
# Donner des droits rwx à l'utilisateur wilder1
$ setfacl -m u:wilder1:rwx file.txt

# Vérifier les droits ACL
$ getfacl file.txt
user::rw-
user:wilder1:rwx
group::rw-
other::r--
```

**Commandes Linux essentielles :**

```bash
ls -l              # Voir les droits d'un répertoire
chmod 755 file     # Changer les droits (numérique)
chown alice file   # Changer le propriétaire
chgrp admin file   # Changer le groupe
```

---

### Sur Windows

> **Un modèle plus flexible avec Allow/Deny**

Windows offre une gestion des droits plus granulaire avec un système d'**autorisation explicite** (Allow/Deny).

#### Les 5 niveaux de droits Windows :

```
┌─────────────────────────────┐
│  DROITS WINDOWS             │
├─────────────────────────────┤
│ 1. Contrôle total           │
│ 2. Modification             │
│ 3. Lecture et exécution      │
│ 4. Écriture (W)             │
│ 5. Lecture (R)              │
└─────────────────────────────┘
```

#### Utilisateurs et Groupes

Contrairement à Linux, les droits peuvent s'appliquer à :

- Des **utilisateurs individuels**
- Des **groupes d'utilisateurs**
- Des **groupes imbriqués** (groupe dans groupe)

#### L'héritage des droits

Par défaut, un sous-dossier **hérite** des droits de son dossier parent. On peut le modifier si on a les permissions.

#### Affichage en PowerShell

```powershell
PS> Get-Acl -Path C:\Users\wilder\file.txt | Format-List

Owner  : PCLab\wilder
Access : BUILTIN\Administrators Allow FullControl
         BUILTIN\Utilisateurs Allow ReadAndExecute, Synchronize
```

**Attribuer des droits :**

```powershell
# Donner le contrôle total à wilder1
$acl = Get-Acl C:\Users\wilder\file.txt
$rule = New-Object System.Security.AccessControl.FileSystemAccessRule(
  "wilder1", "FullControl", "Allow"
)
$acl.SetAccessRule($rule)
$acl | Set-Acl C:\Users\wilder\file.txt
```

**Commandes Windows essentielles :**

```powershell
Get-Acl -Path C:\path\to\file       # Voir les droits
Set-Acl -Path C:\path\to\file       # Changer les droits
```

---

### Répertoire personnel

> **Chaque utilisateur a son espace privé**

Le **répertoire personnel** (home directory) est créé automatiquement pour chaque nouvel utilisateur. C'est son espace privé de stockage.

#### Sur Linux : `/home/xxx`

```bash
# Répertoire personnel de wilder
/home/wilder/

# Contenu typique :
Bureau/          # Desktop
Documents/       # Documents
Téléchargements/ # Downloads
Images/          # Pictures
Musique/         # Music
Vidéos/          # Videos
```

**Raccourcis pratiques :**

- `~` = répertoire personnel courant
- `cd` sans arguments = retour au home

**Exception :** L'utilisateur `root` a son home en `/root` (pas `/home/root`)

#### Sur Windows : `C:\Users\xxx`

```powershell
# Répertoire personnel de wilder
C:\Users\wilder\

# Contenu typique :
Desktop/      # Bureau
Documents/    # Documents
Downloads/    # Téléchargements
Pictures/     # Images
Music/        # Musique
Videos/       # Vidéos
Favorites/    # Favoris
```

#### Différence importante : "répertoire personnel" vs "répertoire individuel"

- **Répertoire personnel** : Espace de travail de l'utilisateur
- **Répertoire individuel** (notion juridique) : Espace protégé par des droits spécifiques

```
┌──────────────────────────────────┐
│  RÉPERTOIRE PERSONNEL            │
├──────────────────────────────────┤
│                                  │
│  Linux    : /home/wilder         │
│                                  │
│  Windows  : C:\Users\wilder      │
│                                  │
│  • Créé auto pour chaque user    │
│  • Espace de stockage personnel  │
│  • Protégé par les droits        │
│                                  │
└──────────────────────────────────┘
```

---

## Chapitre 3 : La Sécurité

### Gestion des Identités et Accès (IAM)

> **Le cœur de la sécurité informatique**

L'**IAM** (Identity and Access Management) est la pratique de **s'assurer que les bons utilisateurs ont accès aux bonnes ressources**.

```
┌─────────────────────────────────────┐
│         GESTION IAM                 │
├─────────────────────────────────────┤
│                                     │
│  ✓ Qui a accès ?                  │
│    → Identification de l'utilisateur│
│                                     │
│  ✓ À quoi exactement ?             │
│    → Attribution des rôles/droits   │
│                                     │
│  ✓ Est-ce vraiment lui ?           │
│    → Authentification sécurisée     │
│                                     │
└─────────────────────────────────────┘
```

**Principe fondamental :** Les rôles d'utilisateur et les privilèges d'accès doivent être **définis et gérés par un système IAM**.

---

### Identification et authentification

> **Les deux étapes de la connexion**

Ces deux concepts travaillent ensemble pour sécuriser l'accès.

#### ⓵ Identification

```
« Qui êtes-vous ? »
       ↓
Fournir un identifiant unique (login)
       ↓
C'est une information individuelle et non-répétable
```

**Exemples d'identifiants :**

- Login : `wilder`
- Email : `wilder@company.com`
- ID unique : `2453`

#### ⓶ Authentification

```
« Prouvez-le ! »
       ↓
Vérifier que la tentative de connexion est légitime
       ↓
Autorisation accordée APRÈS authentification réussie
```

**Méthodes d'authentification :**

- Mot de passe
- Empreinte digitale / Biométrie
- Clé de sécurité
- Code à usage unique (OTP)

```
┌──────────────────────────────────┐
│  PROCESSUS DE CONNEXION          │
├──────────────────────────────────┤
│                                  │
│  1️⃣  Identification              │
│     Enter username: wilder       │
│                                  │
│  2️⃣  Authentification            │
│     Enter password: ****         │
│                                  │
│  ✓ Accès autorisé !             │
│                                  │
└──────────────────────────────────┘
```

**⚠️ Point de sécurité critique :**

En matière de sécurité, **ne jamais réutiliser les anciens comptes utilisateur**. À chaque nouveau collaborateur = nouveau compte. À chaque départ = suppression après 90 jours.

---

## Chapitre 4 : Gestion des utilisateurs sur Linux

### Liste des utilisateurs (Linux)

> **Le fichier `/etc/passwd`**

Le fichier `/etc/passwd` contient **une ligne par utilisateur** avec 7 colonnes séparées par `:`.

```bash
$ cat /etc/passwd

root:x:0:0:root:/root:/bin/bash
wilder:x:1000:1000:Wilder User,,,:/home/wilder:/bin/bash
```

**Format des 7 colonnes :**

|Position|Nom|Exemple|Signification|
|---|---|---|---|
|1|Nom de connexion|`wilder`|Login unique|
|2|Mot de passe|`x`|`x` = en `/etc/shadow`, `*` = compte bloqué|
|3|UID|`1000`|Identifiant unique|
|4|GID|`1000`|Groupe primaire|
|5|GECOS|`Wilder User`|Description/commentaire|
|6|Répertoire home|`/home/wilder`|Chemin du home|
|7|Shell|`/bin/bash`|Interpréteur de commandes|

**Convention importante :**

- `root` a toujours UID = 0
- Les utilisateurs normaux ont UID ≥ 1000

---

### Base des mots de passe

> **Le fichier `/etc/shadow`**

Les mots de passe réels ne sont **pas stockés** dans `/etc/passwd` pour des raisons de sécurité. Ils sont dans `/etc/shadow` (accessible uniquement par root).

```bash
$ sudo cat /etc/shadow

root:!:19081:0:99999:7:::
wilder:$6$Je34t19kkZ2ZGs9f$PleinDeCaracteres...:19081:0:99999:7:::
```

**Format des 9 colonnes :**

|Position|Signification|
|---|---|
|1|Nom de connexion|
|2|**Mot de passe hashé avec salt**|
|3|Date de dernière modification (en jours depuis 1970)|
|4|Jours minimum avant changement|
|5|Jours maximum avant expiration|
|6|Jours d'avertissement avant expiration|
|7|Jours de tolérance après expiration|
|8|Date d'expiration du compte|
|9|Réservé (inutilisé)|

**Statuts du mot de passe :**

- `*` ou `!` → connexion impossible
- `$6$...` → mot de passe chiffré avec SHA-512

**Commandes utiles :**

```bash
passwd wilder              # Changer le mot de passe
chage -l wilder            # Voir l'expiration du mot de passe
chage -E 2025-12-31 wilder # Fixer une date d'expiration
pwck                       # Vérifier l'intégrité de passwd
```

---

### Base des groupes

> **Le fichier `/etc/group`**

Chaque groupe est stocké dans `/etc/group` avec **une ligne par groupe** et 4 colonnes séparées par `:`.

```bash
$ cat /etc/group

sudo:x:27:wilder
sambashare:x:135:wilder
```

**Format des 4 colonnes :**

|Position|Nom|Exemple|Signification|
|---|---|---|---|
|1|Nom du groupe|`sudo`|Identifiant du groupe|
|2|Mot de passe|`x` ou `*`|`x` = en `/etc/gshadow`|
|3|GID|`27`|Identifiant unique du groupe|
|4|Membres|`wilder`|Liste des utilisateurs séparés par `,`|

**Commandes utiles :**

```bash
groupadd admin             # Créer un groupe
groupmod -n newname admin  # Renommer un groupe
groupdel admin             # Supprimer un groupe
```

---

### Commandes utiles (Linux)

#### Gestion des utilisateurs

```bash
adduser wilder              # Créer un nouvel utilisateur
deluser wilder              # Supprimer un utilisateur
usermod -aG sudo wilder     # Ajouter à un groupe
chfn wilder                 # Modifier la description
chsh wilder                 # Changer le shell par défaut
chage -E 2025-12-31 wilder  # Gérer l'expiration
newusers users.txt          # Création par lot
pwck                        # Vérifier passwd et shadow
```

#### Interagir avec les utilisateurs

```bash
id wilder           # Voir ses UID/GID et groupes
whoami              # Voir l'utilisateur courant
who                 # Voir qui est connecté
su - wilder         # Changer d'utilisateur
sudo commande       # Exécuter avec droits élevés
exit                # Quitter la session
logout              # Déconnexion
```

---

## Chapitre 5 : Gestion des utilisateurs sur Windows

### Liste des utilisateurs (Windows)

> **Deux méthodes pour lister**

Les utilisateurs qui se sont connectés au moins une fois ont leur dossier dans `C:\Users`.

#### Méthode basique : `Get-LocalUser`

```powershell
PS> Get-LocalUser

Name           Enabled Description
----           ------- -----------
Administrateur False   Compte d'administration
wilder         True    Compte utilisateur de test
Invité         False   Compte utilisateur invité
```

**Colonnes :**

- Nom
- Activation (True/False)
- Description

#### Méthode détaillée : `Get-WmiObject Win32_UserAccount`

```powershell
PS> Get-WmiObject Win32_UserAccount -Filter "LocalAccount='True'" | Format-Table -AutoSize

AccountType  Caption      Domain  SID                              FullName
-----------  -------      ------  ---                              --------
512          PCLab\wilder PCLab   S-1-5-21-3909285403-...1001     wilder
```

**Colonnes supplémentaires :**

- Type de compte (512 = utilisateur normal)
- Légende (emplacement)
- Domaine
- SID (identifiant de sécurité)
- Nom complet

---

### Base SAM

> **Le cœur de la sécurité Windows**

La **SAM** (Security Account Manager / Gestionnaire de comptes de sécurité) est une base de données Windows cruciale.

```
Localisation :
%SystemRoot%\system32\Config\SAM
├─ Comptes d'utilisateur locaux
├─ Descripteurs de sécurité
├─ Mots de passe hashés
└─ Informations de groupe
```

**Rôle :** Valider les identifiants à chaque connexion.

**Gestion des mots de passe :**

```powershell
# Méthode graphique
Gestionnaire d'identification → Mots de passe Windows

# Méthode ligne de commande
rundll32.exe keymgr.dll,KRShowKeyMgr
```

---

### Groupes (Windows)

> **La gestion complète des groupes**

#### Lister les groupes

**Méthode rapide :**

```powershell
PS> Get-LocalGroup

Name              Description
----              -----------
Administrateurs   Les administrateurs...
Invités           Les utilisateurs invités...
Utilisateurs      Les utilisateurs normaux...
```

**Méthode détaillée :**

```powershell
PS> Get-WmiObject win32_group

Caption              Domain  Name               SID
-------              ------  ----               ---
PCLab\Administrateurs PCLab Administrateurs   S-1-5-32-544
```

#### Gestion des membres

```powershell
Add-LocalGroupMember -Group "Administrateurs" -Member "wilder"
Get-LocalGroupMember -Group "Administrateurs"
```

---

### Commandes utiles (Windows)

#### Utilisateurs

```powershell
Get-LocalUser                        # Lister les utilisateurs
New-LocalUser -Name "alice" -NoPassword  # Créer un utilisateur
Enable-LocalUser -Name "alice"       # Activer un compte
Disable-LocalUser -Name "alice"      # Désactiver un compte
Remove-LocalUser -Name "alice"       # Supprimer un utilisateur
```

#### Groupes

```powershell
Get-LocalGroup                          # Lister les groupes
New-LocalGroup -Name "DevTeam"         # Créer un groupe
Remove-LocalGroup -Name "DevTeam"      # Supprimer un groupe
Add-LocalGroupMember -Group "DevTeam" -Member "alice"     # Ajouter un membre
Get-LocalGroupMember -Group "DevTeam"  # Voir les membres
```

#### Droits d'accès

```powershell
Get-Acl -Path C:\Users\wilder\file.txt   # Voir les droits
Set-Acl -Path C:\Users\wilder\file.txt   # Changer les droits
```

---

## Synthèse comparative

> **Linux vs Windows en un coup d'œil**

```
┌─────────────────────┬──────────────────────┬──────────────────────┐
│     ASPECT          │       LINUX          │      WINDOWS         │
├─────────────────────┼──────────────────────┼──────────────────────┤
│ IDENTIFIANT         │ UID (simple)         │ SID (complexe)       │
│                     │ Ex: 1000             │ Ex: S-1-5-21-...     │
├─────────────────────┼──────────────────────┼──────────────────────┤
│ LISTE UTILISATEURS  │ /etc/passwd          │ C:\Users\*           │
│                     │ 7 colonnes           │ WMI queries          │
├─────────────────────┼──────────────────────┼──────────────────────┤
│ MOTS DE PASSE       │ /etc/shadow          │ SAM database         │
│                     │ 9 colonnes           │ %SystemRoot%\...     │
├─────────────────────┼──────────────────────┼──────────────────────┤
│ GROUPES             │ /etc/group           │ Get-LocalGroup       │
│                     │ Contient users       │ Peut contenir groupes│
├─────────────────────┼──────────────────────┼──────────────────────┤
│ DROITS ACCÈS        │ rwx (3 niveaux)      │ Allow/Deny (5 types) │
│                     │ user/group/other     │ Héritage possible    │
│                     │ ACL avancés          │                      │
├─────────────────────┼──────────────────────┼──────────────────────┤
│ RÉPERTOIRE PERSO    │ /home/xxx            │ C:\Users\xxx         │
├─────────────────────┼──────────────────────┼──────────────────────┤
│ INTERFACE           │ Terminal (puissant)  │ GUI ou PowerShell    │
│                     │ Commandes bas niveau │ Moderne et flexibl e │
├─────────────────────┼──────────────────────┼──────────────────────┤
│ PHILOSOPHIE         │ Minimaliste          │ Centralisé           │
│                     │ Fichiers texte       │ Base de données      │
└─────────────────────┴──────────────────────┴──────────────────────┘
```

---

## 🔑 Points clés à retenir

```
┌────────────────────────────────────────────────────┐
│ ✓ Un utilisateur = identification unique           │
│                                                    │
│ ✓ Les groupes = gestion simplifiée des droits     │
│                                                    │
│ ✓ IAM = sécurité complète (identification +       │
│           authentification + autorisation)         │
│                                                    │
│ ✓ Linux = simple et textuel                        │
│   Windows = centralisé et flexible                │
│                                                    │
│ ✓ Jamais réutiliser un ancien compte              │
│   Supprimer après 90 jours de départ              │
│                                                    │
│ ✓ Droits minimums nécessaires (moindre            │
│   privilège)                                       │
│                                                    │
└────────────────────────────────────────────────────┘
```

---

## 📚 Sources

- **Cours fourni** : "Les utilisateurs partie 1.pdf" et "Les utilisateurs partie 2.pdf"
- **Notes** : "Note" (résumé personnel)
- **Documentation officielle Ubuntu** : https://doc.ubuntu-fr.org/permissions
- **Linux ACL** : https://linux.goffinet.org/administration/securite-locale/access-control-lists-acls-linux/
- **PowerShell Microsoft** : https://petri.com/how-to-use-powershell-to-manage-folder-permissions/
- **IBM - Héritage des droits** : Concepts de base sur l'héritage Windows

---

**Créé à partir des cours "Les utilisateurs" (parties 1 & 2)**