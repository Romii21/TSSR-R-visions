# SSH - Secure Shell

## 📋 Sommaire

1. [Introduction](https://claude.ai/chat/3cb4cdbb-3545-4f6d-b060-059131bed0df#introduction)
2. [À quoi sert SSH ?](https://claude.ai/chat/3cb4cdbb-3545-4f6d-b060-059131bed0df#%C3%A0-quoi-sert-ssh)
3. [Fonctionnement techniques](https://claude.ai/chat/3cb4cdbb-3545-4f6d-b060-059131bed0df#fonctionnement-techniques)
4. [Authentification](https://claude.ai/chat/3cb4cdbb-3545-4f6d-b060-059131bed0df#authentification)
5. [Usages pratiques](https://claude.ai/chat/3cb4cdbb-3545-4f6d-b060-059131bed0df#usages-pratiques)
6. [Configuration](https://claude.ai/chat/3cb4cdbb-3545-4f6d-b060-059131bed0df#configuration)
7. [Sécurisation avec Fail2Ban](https://claude.ai/chat/3cb4cdbb-3545-4f6d-b060-059131bed0df#s%C3%A9curisation-avec-fail2ban)
8. [Schéma récapitulatif](https://claude.ai/chat/3cb4cdbb-3545-4f6d-b060-059131bed0df#sch%C3%A9ma-r%C3%A9capitulatif)

---

## Introduction

### Qu'est-ce que SSH ?

**SSH (Secure Shell)** est un protocole de communication sécurisé de couche 7 (OSI) qui permet l'établissement d'une session chiffrée entre deux machines via un tunnel sécurisé.

**Informations clés :**

- **Initiateur** : Tatu Ylönen (1995)
- **Standards** : RFC 4251, 4252, 4253, 4254
- **Port par défaut** : 22 (TCP)
- **Versions** : SSH1 (obsolète) et SSH2 (recommandée)
- **Utilisable sur** : Linux, Unix, Windows

### Pourquoi SSH existe-t-il ?

Avant SSH, les protocoles existants (FTP, TELNET, RSH) présentaient des graves failles de sécurité :

- ❌ Pas de chiffrement (données en clair)
- ❌ Authentification faible
- ❌ Pas d'intégrité des données
- ❌ Pas d'authentification bidirectionnelle

**SSH résout tous ces problèmes en assurant :**

- ✅ **Confidentialité** : Chiffrement des données
- ✅ **Authentification bidirectionnelle** : Client ET serveur s'authentifient mutuellement
- ✅ **Intégrité** : Vérification que les données n'ont pas été modifiées

---

## À quoi sert SSH ?

SSH a plusieurs usages pratiques :

### 1️⃣ Terminal distant et exécution de commandes

Ouvrir un terminal sur une machine distante et exécuter des commandes.

```
ssh user@serveur
```

### 2️⃣ Transfert de fichiers

- **SFTP** (SSH File Transfer Protocol) : Transfert sécurisé interactif de fichiers
- **SCP** (Secure Copy) : Copie sécurisée de fichiers
- **SSHFS** : Montage d'un dossier distant

### 3️⃣ Tunnelisation (Port Forwarding)

Encapsuler du trafic via SSH pour le sécuriser.

### 4️⃣ Interface graphique distante

**X11 Forwarding** : Lancer des applications graphiques distantes.

### 5️⃣ Gestion de dépôts Git

Utiliser SSH pour synchroniser avec GitHub, GitLab, etc.

---

## Fonctionnement techniques

### Architecture SSH

SSH fonctionne selon le modèle **Client-Serveur** :

```
┌─────────────────┐    SSH (Port 22)    ┌──────────────────┐
│   Client SSH    │◄────────────────────►│  Serveur sshd    │
│  (openssh)      │   Tunnel sécurisé    │  (openssh-server)│
└─────────────────┘                      └──────────────────┘
```

### Les protagonistes

**OpenSSH** est l'implémentation de référence (maintenue par openBSD) :

- **Client** : `ssh` (openssh-client)
- **Serveur** : `sshd` (openssh-server)
- **Version actuelle** : 9.9p2 (juillet 2025)

### Étapes de la connexion SSH

Quand vous vous connectez à un serveur, plusieurs étapes se déroulent :

```
1. NÉGOCIATION
   ├─ Échange des versions SSH
   └─ Négociation des algorithmes cryptographiques

2. AUTHENTIFICATION DU SERVEUR (TOFU)
   ├─ Serveur envoie sa clé publique
   ├─ Client compare avec known_hosts
   └─ Première fois ? Demande de confirmation

3. CHIFFREMENT DU CANAL
   ├─ Échange de clés Diffie-Hellman (ECDH en pratique)
   └─ Établissement d'une clé de session symétrique (PFS)

4. AUTHENTIFICATION DU CLIENT
   ├─ Par clé asymétrique (recommandé)
   ├─ Par mot de passe
   └─ Par hôte ou autres méthodes

5. SESSION ÉTABLIE
   └─ Tunnel sécurisé opérationnel
```

### Chiffrement et cryptographie

SSH utilise plusieurs couches cryptographiques :

|Type|Rôle|Exemples|
|---|---|---|
|**Chiffrement symétrique**|Confidentialité du canal|AES-128-GCM, ChaCha20-Poly1305|
|**Chiffrement asymétrique**|Authentification + échange de clés|ED25519, RSA, ECDSA|
|**Hachage (MAC)**|Intégrité des données|SHA256, SHA512|
|**Échange de clés**|Génération de clés sécurisées|ECDH|

**Important** : SSH utilise **Perfect Forward Secrecy (PFS)**, ce qui signifie que même si votre clé privée est compromise demain, les conversations passées restent sécurisées.

---

## Authentification

### Principe fondamental : TOFU

**TOFU = Trust On First Use** : À la première connexion, vous faites confiance à la clé du serveur. Les connexions suivantes vérifient que c'est la même clé.

```
Première connexion :
Voulez-vous continuer ? YES
→ Clé stockée dans ~/.ssh/known_hosts

Connexions suivantes :
La clé correspond-elle ? OUI → Connexion
La clé ne correspond pas ? → ALERTE POSSIBLE USURPATION !
```

### Authentification du serveur

Le serveur SSH possède une paire de clés asymétriques qui l'identifient :

**Localisation** : `/etc/ssh/`

```
ssh_host_ed25519_key       (clé privée)
ssh_host_ed25519_key.pub   (clé publique)
ssh_host_rsa_key
ssh_host_rsa_key.pub
```

**À la première connexion :**

```
The authenticity of host 'server (IP)' can't be established.
ED25519 key fingerprint is SHA256:kPFlNQn+PHJ+...
Are you sure you want to continue connecting (yes/no)? yes
```

**Afficher l'empreinte d'un serveur :**

```bash
ssh-keygen -lf /etc/ssh/ssh_host_ed25519_key.pub
```

### Authentification du client

Après vérification du serveur, le serveur authentifie le client. **Plusieurs méthodes possibles :**

#### ✅ Par clé asymétrique (RECOMMANDÉ)

- Client possède une paire de clés (privée + publique)
- La clé publique est sur le serveur dans `~/.ssh/authorized_keys`
- Client prouve sa possession de la clé privée
- **Avantage** : Plus sécurisé, automatisable, sans mot de passe

#### ⚠️ Par mot de passe

- Authentification par mot de passe système
- **Inconvénient** : Sujet aux attaques par force brute

#### 🔑 Par hôte (hostbased)

- Authentification de tous les utilisateurs d'un hôte donné
- Moins courant

#### 🔗 Par outils tiers

- PAM, Kerberos via GSSAPI
- Configuration avancée

### Générer ses clés SSH

**Sur la machine locale :**

```bash
ssh-keygen -t ecdsa -b 256
# ou
ssh-keygen -t ed25519  (recommandé, plus rapide)
```

**Ce que cela crée :**

```
~/.ssh/id_ed25519          (clé privée - À PROTÉGER)
~/.ssh/id_ed25519.pub      (clé publique - À partager)
```

**Important** : Protégez votre clé privée avec une **passphrase** (phrase secrète). Elle sera chiffrée localement.

```
Enter passphrase (empty for no passphrase): ✓ ENTREZ UNE PASSPHRASE
```

### Copier la clé publique sur le serveur

**Méthode recommandée (simple et sécurisée) :**

```bash
ssh-copy-id user@serveur
```

Cela :

1. Demande votre mot de passe une dernière fois
2. Copie votre clé publique dans `~/.ssh/authorized_keys` sur le serveur
3. Vous pouvez ensuite vous connecter sans mot de passe

**Résultat :**

```
wilder@host:~$ ssh server
Linux debian 5.10.0-15-amd64 #1 SMP Debian 5.10.120-1
```

---

## Usages pratiques

### 1. Shell distant

```bash
# Connexions équivalentes
ssh server
ssh wilder@server
ssh -l wilder -p 22 server
ssh ssh://wilder@server:22
```

### 2. Copie sécurisée (SCP)

```bash
# Local vers distant
scp fichier_local serveur:chemin_distant

# Distant vers local
scp user@serveur:chemin_distant ~/Downloads/

# Copie récursive (dossier)
scp -r dossier serveur:
```

### 3. Transfert interactif (SFTP)

```bash
sftp serveur
sftp> ls
sftp> get fichier_distant
sftp> put fichier_local
sftp> bye
```

**Clients graphiques** : FileZilla, WinSCP, etc.

### 4. Tunnelisation et Port Forwarding

#### Redirection locale (Client → Serveur)

```bash
ssh -L 8080:web.interne:80 serveur
# Accéder à un service interne via localhost:8080
```

#### Redirection distante (Serveur → Client)

```bash
ssh -R 9000:localhost:3000 serveur
# Exposer un service local sur le serveur distant
```

#### Proxy SOCKS

```bash
ssh -D 1080 serveur
# Tunnel tout le trafic via le serveur
```

### 5. Interface graphique distante (X11 Forwarding)

```bash
ssh -X user@serveur
# Lancer des applications graphiques distantes
```

### 6. Gestion de dépôt Git

```bash
# Générer une clé SSH
ssh-keygen -t ed25519

# Ajouter la clé publique à GitHub/GitLab
cat ~/.ssh/id_ed25519.pub

# Cloner un dépôt via SSH
git clone git@github.com:user/repo.git
```

---

## Configuration

### Configuration du serveur (sshd)

**Fichier de configuration** : `/etc/ssh/sshd_config`

**Bonnes pratiques essentielles :**

```bash
# Changer le port (évite les bots)
Port 2222

# Privilégier la clé publique
PubkeyAuthentication yes
PasswordAuthentication no

# Ou demander clé + mot de passe
AuthenticationMethods publickey,password

# Interdire la connexion du root
PermitRootLogin no

# Interdire les connexions vides
PermitEmptyPasswords no
```

**Tester la configuration (avant de redémarrer) :**

```bash
sshd -t           # Teste la syntaxe
sshd -T           # Teste et affiche la configuration
```

**Appliquer les changements :**

```bash
systemctl restart ssh
```

### Configuration du client (ssh)

**Fichiers de configuration :**

- `/etc/ssh/ssh_config` (global, pour tous les utilisateurs)
- `~/.ssh/config` (utilisateur, prioritaire)

**Exemple de configuration utilisateur :**

```bash
# ~/.ssh/config
Host monserveur
    HostName serveur.example.com
    User wilder
    Port 2222
    IdentityFile ~/.ssh/id_ed25519
    CheckHostIP yes

# Utilisation : ssh monserveur
```

**Vérifier la configuration :**

```bash
ssh -G monserveur
```

### Outils additionnels OpenSSH

|Outil|Rôle|
|---|---|
|`ssh-keygen`|Génération et gestion de clés|
|`ssh-agent`|Stockage de clés en mémoire|
|`ssh-add`|Ajout de clés à ssh-agent|
|`ssh-keyscan`|Récupération de clés publiques|
|`sftp-server`|Sous-système SFTP|

---

## Sécurisation avec Fail2Ban

### Le problème : Attaques par force brute

Les serveurs SSH reçoivent **constamment** des tentatives de connexion automatisées :

- Des bots qui testent des mots de passe courants
- Risque d'intrusion si un mot de passe faible est trouvé

### Solution : Fail2Ban

**Fail2Ban** est un outil de protection qui :

1. **Surveille** les journaux de connexion (`/var/log/auth.log`)
2. **Détecte** les tentatives répétées échouées
3. **Bloque automatiquement** l'adresse IP offensante (pare-feu)
4. **Débloque** après une période définie

**Exemple :**

```
IP 192.168.1.100 : 5 tentatives échouées en 10 minutes
→ Blocage de 192.168.1.100 pendant 1 heure
```

**Avantage** : Fonctionne pour tout service (SSH, HTTP, FTP, etc.)

**Installation :**

```bash
sudo apt install fail2ban
sudo systemctl start fail2ban
```

---

## Schéma récapitulatif

### Flux de connexion SSH complet

```
┌──────────────────┐
│    UTILISATEUR   │
└────────┬─────────┘
         │
         ▼
    ssh server
         │
         ▼
┌─────────────────────────────────┐
│  1. NÉGOCIATION                 │
│  • Versions SSH                 │
│  • Algorithmes cryptographiques │
└────────┬────────────────────────┘
         │
         ▼
┌─────────────────────────────────┐
│  2. AUTH SERVEUR (TOFU)         │
│  • Serveur envoie clé publique  │
│  • Vérification known_hosts     │
│  • Première fois = Confirmation │
└────────┬────────────────────────┘
         │
         ▼
┌─────────────────────────────────┐
│  3. CHIFFREMENT ÉTABLI          │
│  • Échange Diffie-Hellman       │
│  • Clé session symétrique (PFS) │
└────────┬────────────────────────┘
         │
         ▼
┌─────────────────────────────────┐
│  4. AUTH CLIENT                 │
│  • Clé publique/privée ✓        │
│  • ou Mot de passe              │
└────────┬────────────────────────┘
         │
         ▼
┌─────────────────────────────────┐
│  5. SESSION SÉCURISÉE           │
│  Terminal distant opérationnel  │
└─────────────────────────────────┘
```

### Où se trouvent les clés ?

```
SERVEUR
└── /etc/ssh/
    ├── ssh_host_ed25519_key      ← Clé privée serveur (À PROTÉGER)
    └── ssh_host_ed25519_key.pub  ← Clé publique serveur
    
CLIENT
└── ~/.ssh/
    ├── id_ed25519                ← Clé privée client (À PROTÉGER)
    ├── id_ed25519.pub            ← Clé publique client (À partager)
    └── known_hosts               ← Clés connues des serveurs
    
SERVEUR (fichier des accès)
└── ~/.ssh/
    └── authorized_keys           ← Clés publiques autorisées
```

### Vue d'ensemble des usages

```
SSH
├── 1. Terminal distant (ssh)
├── 2. Copie de fichiers
│   ├── SCP
│   ├── SFTP (interactif)
│   └── SSHFS (montage)
├── 3. Tunnelisation
│   ├── Port Forwarding Local (-L)
│   ├── Port Forwarding Distant (-R)
│   └── Proxy SOCKS (-D)
├── 4. Interface graphique (X11 -X)
└── 5. Git et dépôts distants
```

---

## Points clés à retenir

### ✅ À FAIRE

1. **Générer une paire de clés** (`ssh-keygen`)
2. **Protéger la clé privée** par une passphrase
3. **Copier la clé publique** sur les serveurs (`ssh-copy-id`)
4. **Vérifier les empreintes** à la première connexion
5. **Désactiver l'authentification par mot de passe** (sur le serveur)
6. **Utiliser un port non-standard** (évite les bots)
7. **Installer Fail2Ban** pour bloquer les attaques

### ❌ À ÉVITER

1. Ne **JAMAIS partager** votre clé privée
2. Ne **JAMAIS accepter** sans vérifier l'empreinte (première connexion)
3. Ne pas utiliser SSH sans passphrase (local)
4. Ne pas laisser le port 22 ouvert sur Internet (trop de bots)
5. Ne pas ignorer les alertes d'usurpation de clé
6. Ne pas désactiver l'authentification du serveur

---

## Sources

- **Documents fournis** : Cours SSH & Notes personnelles
- **RFC officielles** : 4251, 4252, 4253, 4254
- **OpenSSH officiel** : https://www.openssh.com/
- **SSH Academy** : https://ssh.com/
- **Fail2Ban** : https://doc.ubuntu-fr.org/fail2ban
- **ANSSI** : Recommandations de sécurisation OpenSSH