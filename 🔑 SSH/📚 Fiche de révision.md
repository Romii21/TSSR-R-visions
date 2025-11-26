# SSH - Secure Shell

## 📋 Sommaire

1. [Introduction](https://claude.ai/chat/81d99fcf-3729-48f3-8530-10c6e13075d6#introduction)
2. [Fonctionnement technique](https://claude.ai/chat/81d99fcf-3729-48f3-8530-10c6e13075d6#fonctionnement-technique)
3. [Authentification](https://claude.ai/chat/81d99fcf-3729-48f3-8530-10c6e13075d6#authentification)
4. [Utilisation pratique](https://claude.ai/chat/81d99fcf-3729-48f3-8530-10c6e13075d6#utilisation-pratique)
5. [Configuration](https://claude.ai/chat/81d99fcf-3729-48f3-8530-10c6e13075d6#configuration)
6. [Protection contre les attaques](https://claude.ai/chat/81d99fcf-3729-48f3-8530-10c6e13075d6#protection-contre-les-attaques)
7. [Conclusion](https://claude.ai/chat/81d99fcf-3729-48f3-8530-10c6e13075d6#conclusion)

---

## Introduction

### Pourquoi SSH ?

Avant SSH, les protocoles de communication réseau comme **FTP**, **TELNET** et **RSH** ne garantissaient pas :

- ✗ L'authentification fiable
- ✗ La confidentialité des données
- ✗ L'intégrité des échanges

**SSH** (Secure Shell) résout ces problèmes en proposant une communication **sécurisée** entre deux machines.

### Qu'est-ce que SSH ?

SSH est un **protocole client-serveur** qui établit un tunnel de communication sécurisé. Il assure :

- ✅ **Confidentialité** : chiffrement des données
- ✅ **Authentification bidirectionnelle** : client et serveur se vérifient mutuellement
- ✅ **Intégrité** : vérification que les données n'ont pas été modifiées

**Informations clés :**

- Protocole de **couche 7** (OSI)
- Port standard : **TCP 22**
- Version recommandée : **SSH v2** (v1 obsolète)
- Implémentation principale : **OpenSSH** (version 9.9p2)

```
┌─────────────────────────────────────────────────────┐
│           TUNNEL SSH SÉCURISÉ (Port 22)             │
├────────────────┬─────────────────────────────────────┤
│   CLIENT       │  Chiffrement symétrique + clés     │   SERVEUR
│   (ssh)        │  Authentification bidirectionnelle  │   (sshd)
└────────────────┴─────────────────────────────────────┘
```

### Utilisations courantes

|Usage|Description|
|---|---|
|**Shell distant**|Exécuter des commandes sur une machine distante|
|**Transfert sécurisé**|SFTP, SCP pour copier des fichiers|
|**Tunnels**|Port forwarding, proxy SOCKS|
|**Accès graphique**|X11 forwarding pour des applications distantes|

---

## Fonctionnement technique

### Établissement d'une connexion SSH

Une connexion SSH suit ces étapes :

```
1. NÉGOCIATION
   ↓
   Client et serveur échangent leurs versions
   Accord sur les algorithmes de chiffrement
   ↓
2. ÉCHANGE DE CLÉS (Diffie-Hellman / ECDH)
   ↓
   Confidentialité Persistante (PFS - Perfect Forward Secrecy)
   ↓
3. CHIFFREMENT ACTIVÉ
   ↓
   La connexion devient confidentielle
   ↓
4. AUTHENTIFICATION
   ↓
   Vérification du serveur (client → serveur)
   Vérification du client (serveur → client)
```

### Cryptographie utilisée

SSH combine plusieurs techniques cryptographiques :

**Chiffrement symétrique** (rapidité)

- AES-128-GCM, AES-256-GCM
- ChaCha20-Poly1305

**Authentification de la clé** (intégrité)

- HMAC avec SHA-256, SHA-512

**Échange de clés** (sécurité)

- ECDH (Elliptic Curve Diffie-Hellman)

**Types de clés asymétriques**

- ED25519 (recommandé, moderne)
- RSA
- ECDSA

```
┌─────────────────────────────────────────┐
│     Cryptosystèmes SSH (OpenSSH)        │
├──────────────────┬──────────────────────┤
│  Symétrique      │  AES-GCM, ChaCha20   │
│  Authentification│  HMAC-SHA            │
│  Échange de clés │  ECDH                │
│  Clés publiques  │  ED25519, RSA, ECDSA │
└──────────────────┴──────────────────────┘
```

---

## Authentification

### Authentification bidirectionnelle

SSH sécurise les deux sens :

1. **Client authentifie le serveur** (lors de la connexion)
    
    - Vérification que je me connecte au bon serveur
    - Protection contre les attaques man-in-the-middle
2. **Serveur authentifie le client** (après)
    
    - Vérification de l'identité de l'utilisateur
    - Sur un canal confidentiel avec le bon serveur

### Authentification du serveur

#### Modèle TOFU (Trust On First Use)

À chaque serveur correspond une **paire de clés asymétriques** stockées dans `/etc/ssh/` :

```
/etc/ssh/ssh_host_ed25519_key      (clé privée - secret du serveur)
/etc/ssh/ssh_host_ed25519_key.pub  (clé publique - partagée)
/etc/ssh/ssh_host_rsa_key
/etc/ssh/ssh_host_rsa_key.pub
```

#### Première connexion

```
CLIENT                                    SERVEUR
   │                                        │
   ├──────────── Connexion ──────────────→ │
   │                                        │
   │ ← Serveur envoie sa clé publique ──── │
   │                                        │
   ├─ Affiche l'empreinte (fingerprint)    │
   │   SHA256:kPFlNQn+PHJ+PMOcHe4...      │
   │                                        │
   │   « Êtes-vous sûr ? (oui/non) »       │
   │─────────────────────────────────────→ │
   │   (Vérification manuelle théorique)    │
   │                                        │
   │   → Sauvegarde dans ~/.ssh/known_hosts │
```

**Action du client :**

```bash
$ ssh server
The authenticity of host 'server' can't be established.
ED25519 key fingerprint is SHA256:kPFlNQn+PHJ+PMOcHe4...
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added 'server' (ED25519) to the list of known hosts.
```

#### Connexions suivantes

```
CLIENT                                    SERVEUR
   │                                        │
   ├──────────── Connexion ──────────────→ │
   │                                        │
   │ ← Serveur envoie sa clé publique ──── │
   │                                        │
   ├─ Vérifie : clé = known_hosts ?
   │   ✅ OUI → Poursuivre
   │   ❌ NON → ALERTE USURPATION !
   │                                        │
   ├─ Envoie un "challenge" ───────────────→ │
   │   (je dois prouver que j'ai la clé)    │
   │                                        │
   │ ← Serveur vérifie la signature ────── │
   │   ✅ Signature valide → Authentification OK
```

**⚠️ Alerte usurpation :**

```
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@ WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED! @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@

Possible man-in-the-middle attack!
```

### Authentification du client

Après vérification du serveur, c'est au client de prouver son identité. Plusieurs méthodes possibles :

|Méthode|Description|Sécurité|
|---|---|---|
|**Password**|Mot de passe basé sur les comptes système|⭐ Faible|
|**PublicKey**|Clé privée/publique de l'utilisateur|⭐⭐⭐ Forte|
|**Hostbased**|Clés pour tous les utilisateurs d'un hôte|⭐⭐ Moyen|
|**GSSAPI**|Kerberos, outils tiers|⭐⭐⭐ Forte|

#### Authentification par clé (recommandée)

**Sur le client :**

Générer une paire de clés avec `ssh-keygen` :

```bash
$ ssh-keygen -t ed25519
Generating public/private ed25519 key pair.
Enter file: /home/user/.ssh/id_ed25519
Enter passphrase: [chiffre la clé privée]

# Résultat :
/home/user/.ssh/id_ed25519          (clé privée - garde-la secrète!)
/home/user/.ssh/id_ed25519.pub      (clé publique - à partager)
```

**Sur le serveur :**

Copier la clé publique du client vers le serveur :

```bash
$ ssh-copy-id -i ~/.ssh/id_ed25519.pub user@server
[On vous demande le mot de passe une dernière fois]

# La clé est copiée dans :
/home/user/.ssh/authorized_keys
```

Puis :

```bash
$ ssh server
[Pas de mot de passe demandé - connecté directement]
```

```
┌─ CLIENT ─────────────────────────────────┐
│  ~/.ssh/id_ed25519 (privée)              │
│  ~/.ssh/id_ed25519.pub (publique)        │
└──────────────────────────────────────────┘
                    │
                    │ ssh-copy-id
                    ↓
┌─ SERVEUR ────────────────────────────────┐
│  ~/.ssh/authorized_keys                  │
│  ├─ id_ed25519.pub (du client)           │
│  ├─ ...                                  │
└──────────────────────────────────────────┘
```

---

## Utilisation pratique

### 1. Accès à un terminal distant (ssh)

**Syntaxe basique :**

```bash
ssh [user@]host [-p port]

# Exemples :
ssh server
ssh user@192.168.1.10
ssh user@server -p 2222
ssh://user@server:22      # Format URI
```

**Options courantes :**

```bash
-p <port>        # Port personnalisé
-l <username>    # Nom d'utilisateur
-v[vv]          # Bavard (debug)
-X              # X11 forwarding (graphique)
```

### 2. Copie de fichiers sécurisée (scp)

**Transfert local → distant :**

```bash
$ scp mon_fichier.txt server:/chemin/distant/
$ scp -r mon_dossier/ server:/chemin/distant/
```

**Transfert distant → local :**

```bash
$ scp server:/chemin/distant/fichier.txt ~/Downloads/
$ scp -r user@server:/dossier/ ~/Downloads/
```

### 3. Transfert interactif (sftp)

Comme FTP, mais sécurisé :

```bash
$ sftp server
Connected to server.

sftp> ls                    # Lister les fichiers
sftp> cd /dossier           # Changer de répertoire
sftp> get distant_file.txt  # Télécharger
sftp> put local_file.txt    # Télécharger
sftp> bye                   # Quitter
```

### 4. Tunnels SSH

#### Port Forwarding (redirection)

**Local → Distant (accès indirect) :**

```bash
$ ssh -L 8080:intranet.local:80 serveur_bastion
# Accès à http://localhost:8080 = intranet.local:80
```

**Distant → Local (exposition) :**

```bash
$ ssh -R 9000:localhost:3000 serveur_distant
# Le serveur peut accéder à localhost:3000 via 9000
```

#### Proxy SOCKS

```bash
$ ssh -D 1080 serveur
# Lance un proxy SOCKS5 sur le port 1080
# Tout le trafic passe par le serveur
```

---

## Configuration

### Serveur SSH (sshd)

**Fichier de configuration :** `/etc/ssh/sshd_config`

#### Configurations recommandées

```bash
# Changer le port par défaut (évite les bots)
Port 2222

# Authentification
PasswordAuthentication no          # Désactiver mot de passe
PubkeyAuthentication yes           # Autoriser clés publiques
AuthenticationMethods publickey    # Ou publickey,password

# Sécurité
PermitRootLogin no                 # Pas de connexion directe root
X11Forwarding no                   # Désactiver X11 (si inutile)
PermitEmptyPasswords no            # Interdire clés vides

# Audit
LogLevel VERBOSE
```

**Vérifier la configuration :**

```bash
$ sshd -t              # Vérifier la syntaxe
$ sshd -T              # Afficher la configuration
$ systemctl restart ssh # Appliquer les changements
```

### Client SSH (ssh)

**Configuration système :** `/etc/ssh/ssh_config`

**Configuration utilisateur :** `~/.ssh/config` (prioritaire)

#### Exemple de configuration

```bash
# Configuration par défaut
Host *
    CheckHostIP yes          # Vérifier IP (éviter DNS spoofing)
    Port 22
    IdentityFile ~/.ssh/id_ed25519

# Configuration pour un serveur spécifique
Host serveur.com
    User alice
    Port 2222
    IdentityFile ~/.ssh/id_custom
    StrictHostKeyChecking accept-new

# Configuration pour un groupe de serveurs
Host prod-*.example.com
    User deployer
    IdentitiesOnly yes
    IdentityFile ~/.ssh/deployer_key
```

**Afficher la configuration pour un hôte :**

```bash
$ ssh -G serveur.com
```

---

## Protection contre les attaques

### Menaces courantes

**Attaques par force brute** : des milliers de tentatives avec différents mots de passe

```
Attaquant → ssh server (user, password123)    ❌
         → ssh server (user, password456)    ❌
         → ssh server (user, qwerty)         ❌
         → ssh server (admin, admin)         ✅ ACCÈS !
```

### Fail2Ban : protection automatique

**Fail2Ban** surveille les journaux système et bloque les adresses IP après trop d'échecs :

```
┌─ JOURNAUX SYSTÈME ────────┐
│ Failed password for user  │
│ Failed password for admin │
│ Failed password for root  │
└──────────────────────────┘
           ↓
      FAIL2BAN
      (surveillance)
           ↓
    3 tentatives échouées
           ↓
┌─ IP ADDRESS 192.168.1.50 ─┐
│  BANNIE pendant 10 minutes │
│  (ou plus selon config)    │
└──────────────────────────┘
```

#### Installation et utilisation

```bash
# Installation
$ sudo apt-get install fail2ban

# Activation
$ sudo systemctl start fail2ban
$ sudo systemctl enable fail2ban

# Vérification
$ sudo fail2ban-client status sshd
```

#### Configuration basique

**Fichier :** `/etc/fail2ban/jail.local`

```bash
[sshd]
enabled = true
port = ssh
logpath = /var/log/auth.log
maxretry = 3              # 3 tentatives
findtime = 600            # En 10 minutes
bantime = 900             # Bannie 15 minutes
```

**Commandes utiles :**

```bash
$ sudo fail2ban-client set sshd unbanip 192.168.1.50   # Débannir une IP
$ sudo fail2ban-client status sshd                      # Voir les IPs bannies
```

---

## Outils complémentaires

SSH fournit plusieurs utilitaires pratiques :

|Outil|Fonction|
|---|---|
|**ssh-keygen**|Générer et gérer les clés|
|**ssh-copy-id**|Copier la clé publique sur un serveur|
|**ssh-agent**|Stockage des clés en mémoire (sans passphrase à chaque fois)|
|**ssh-add**|Ajouter une clé à ssh-agent|
|**ssh-keyscan**|Récupérer les clés publiques d'un serveur|
|**sftp-server**|Sous-système SFTP (serveur)|

**Exemple avec ssh-agent :**

```bash
# Démarrer l'agent
$ eval "$(ssh-agent -s)"
Agent pid 12345

# Ajouter une clé (une seule fois)
$ ssh-add ~/.ssh/id_ed25519
Enter passphrase: ••••••

# Utiliser SSH sans taper la passphrase
$ ssh server  [pas de passphrase demandée]
```

---

## Schéma récapitulatif

```
                    SSH - SECURE SHELL
                            │
            ┌───────────────┼───────────────┐
            │               │               │
        VERSIONING      AUTHENTIFICATION   CHIFFREMENT
            │               │               │
        v1 (obsolète)   Serveur (TOFU)     Symétrique
        v2 (recommandé) Client (clés)      Asymétrique
        v3 (émergent)   Bidirectionnelle   Signature
            │               │               │
        ┌───┴────────────┬──┴────────────┬─┴───────────┐
        │                │              │              │
    OPENSSH           PROTOCOLE       USAGES      PROTECTION
        │          (TCP Port 22)          │
    Client (ssh)   Couche 7 (OSI)     Shell          Fail2Ban
    Serveur (sshd) RFC 4251-4254      SFTP/SCP    Authentif. clés
    v9.9p2                            Tunnels      Logs audit
                                      X11 FW
```

---

## Conclusion

**SSH en 5 points :**

1. **Sécurité** : Chiffrement + authentification bidirectionnelle
2. **Flexibilité** : Multiples usages (shell, transfert, tunnels)
3. **Authentification par clés** : Plus sûr que les mots de passe
4. **Configuration** : À adapter selon vos besoins de sécurité
5. **Surveillance** : Fail2Ban pour bloquer les attaques par force brute

SSH remplace complètement les protocoles non sécurisés de l'ère précédente (Telnet, FTP, RSH) et reste l'outil standard pour l'accès distant sécurisé.

---

## 📚 Sources

- **Cours fourni** : SSH.pdf - Présentation complète du protocole SSH
- **Documentation officielle** : [OpenSSH](https://www.openssh.com/)
- **RFC SSH** : RFC 4251, 4252, 4253, 4254 (IETF)
- **Fail2Ban** : [Documentation Ubuntu-fr](https://doc.ubuntu-fr.org/fail2ban)
- **Recommandations** : [ANSSI - Sécurisation d'OpenSSH](https://www.ssi.gouv.fr/)