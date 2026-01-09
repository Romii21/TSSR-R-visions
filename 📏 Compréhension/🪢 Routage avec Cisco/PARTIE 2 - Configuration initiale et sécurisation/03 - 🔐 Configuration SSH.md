

## 📋 Table des matières

```table-of-contents
title: 
style: nestedList # TOC style (nestedList|nestedOrderedList|inlineFirstLevel)
minLevel: 2 # Include headings from the specified level
maxLevel: 2 # Include headings up to the specified level
include: 
exclude: 
includeLinks: true # Make headings clickable
hideWhenEmpty: false # Hide TOC if no headings are found
debugInConsole: false # Print debug info in Obsidian console
```

---

## 🎯 Introduction

SSH (Secure Shell) est le protocole sécurisé standard pour l'administration distante des équipements réseau. Contrairement à Telnet qui transmet tout en clair, SSH chiffre l'intégralité de la communication, incluant les identifiants et les commandes.

> [!info] Pourquoi SSH ?
> 
> - **Chiffrement** : Toutes les données sont cryptées
> - **Authentification forte** : Vérification de l'identité du serveur
> - **Intégrité** : Protection contre la modification des données en transit
> - **Standard industriel** : Requis par la plupart des politiques de sécurité

> [!warning] Prérequis Pour activer SSH sur un routeur Cisco, vous devez avoir :
> 
> - Une image IOS avec support cryptographique (k9 dans le nom)
> - Un nom d'hôte configuré
> - Un nom de domaine configuré
> - Suffisamment de RAM pour générer les clés

---

## 🔑 Génération des clés RSA

### Concept

Les clés RSA sont des paires de clés cryptographiques (publique/privée) utilisées pour :

- Chiffrer la communication SSH
- Authentifier le routeur auprès des clients SSH
- Créer un canal sécurisé

### Commande de génération

```bash
Router(config)# crypto key generate rsa
```

Lors de l'exécution, le routeur demande la taille de la clé :

```bash
How many bits in the modulus [512]: 2048
```

> [!tip] Taille des clés recommandée
> 
> - **Minimum acceptable** : 1024 bits
> - **Recommandé** : 2048 bits (bon équilibre sécurité/performance)
> - **Haute sécurité** : 4096 bits (plus lent mais très sécurisé)
> 
> ⚠️ Les clés < 1024 bits sont considérées comme faibles et ne devraient plus être utilisées.

### Syntaxe détaillée

```bash
Router(config)# crypto key generate rsa [general-keys | usage-keys] [modulus modulus-size]
```

**Options disponibles** :

|Option|Description|
|---|---|
|`general-keys`|Une seule paire de clés pour signature et chiffrement (défaut)|
|`usage-keys`|Deux paires de clés distinctes (une pour chiffrement, une pour signature)|
|`modulus`|Spécifie directement la taille sans interaction|

### Exemples pratiques

```bash
# Génération interactive (demande la taille)
Router(config)# crypto key generate rsa

# Génération avec taille spécifiée directement
Router(config)# crypto key generate rsa modulus 2048

# Génération de clés séparées pour usage spécifique
Router(config)# crypto key generate rsa usage-keys modulus 2048
```

> [!example] Exemple de sortie
> 
> ```
> Router(config)# crypto key generate rsa
> The name for the keys will be: Router.example.com
> Choose the size of the key modulus in the range of 360 to 4096 for your
>   General Purpose Keys. Choosing a key modulus greater than 512 may take
>   a few minutes.
> 
> How many bits in the modulus [512]: 2048
> % Generating 2048 bit RSA keys, keys will be non-exportable...
> [OK] (elapsed time was 3 seconds)
> 
> *Mar 1 00:15:45.123: %SSH-5-ENABLED: SSH 1.99 has been enabled
> ```

> [!warning] Attention au temps de génération
> 
> - **1024 bits** : ~1-2 secondes
> - **2048 bits** : ~3-5 secondes
> - **4096 bits** : ~30-60 secondes (peut varier selon le matériel)
> 
> Ne pas interrompre le processus !

### Régénération des clés

Si vous devez régénérer les clés (compromission, changement de politique) :

```bash
# Supprimer les clés existantes
Router(config)# crypto key zeroize rsa

# Régénérer de nouvelles clés
Router(config)# crypto key generate rsa modulus 2048
```

> [!tip] Astuce professionnelle Après avoir zéroïsé les clés, SSH est automatiquement désactivé. Il sera réactivé après la génération de nouvelles clés.

---

## 🌐 Configuration du nom de domaine

### Concept

Le nom de domaine est **obligatoire** pour SSH car il est utilisé avec le nom d'hôte pour créer le FQDN (Fully Qualified Domain Name) qui identifie les clés RSA.

**Format du FQDN** : `hostname.domain-name`

### Configuration du hostname

```bash
Router(config)# hostname R1
R1(config)# 
```

> [!info] Importance du hostname Le hostname :
> 
> - Identifie le routeur sur le réseau
> - Fait partie du prompt CLI
> - Est utilisé dans le FQDN pour les clés SSH
> - Apparaît dans les logs et les messages système

### Configuration du domain-name

```bash
R1(config)# ip domain-name example.com
```

**Résultat** : Le FQDN devient `R1.example.com`

### Syntaxe complète

```bash
Router(config)# hostname [nom-du-routeur]
Router(config)# ip domain-name [nom-de-domaine]
```

> [!example] Exemples pratiques
> 
> ```bash
> # Configuration pour un réseau d'entreprise
> Router(config)# hostname CoreSwitch
> CoreSwitch(config)# ip domain-name entreprise.local
> # FQDN résultant : CoreSwitch.entreprise.local
> 
> # Configuration pour un site distant
> Router(config)# hostname Site-Paris-R1
> Site-Paris-R1(config)# ip domain-name paris.monentreprise.fr
> # FQDN résultant : Site-Paris-R1.paris.monentreprise.fr
> ```

### Vérification

```bash
R1# show running-config | include hostname|domain-name
hostname R1
ip domain-name example.com
```

> [!warning] Erreur commune Si vous tentez de générer les clés RSA sans avoir configuré le nom de domaine, vous obtiendrez :
> 
> ```
> Router(config)# crypto key generate rsa
> % Please define a domain-name first.
> ```

### Modification du nom de domaine

```bash
# Supprimer l'ancien domaine
R1(config)# no ip domain-name example.com

# Configurer le nouveau domaine
R1(config)# ip domain-name nouveau-domaine.com

# Important : Régénérer les clés RSA !
R1(config)# crypto key zeroize rsa
R1(config)# crypto key generate rsa modulus 2048
```

> [!tip] Convention de nommage **Bonnes pratiques** :
> 
> - Utiliser des noms descriptifs (fonction, emplacement)
> - Éviter les caractères spéciaux
> - Respecter une convention cohérente dans l'organisation
> - Exemples :
>     - `R1-Core.datacenter.corp`
>     - `SW-Access-Etage3.site-paris.local`
>     - `FW-DMZ.securite.entreprise.fr`

---

## 🔒 Activation de SSH version 2

### Concept

SSH existe en deux versions majeures :

|Version|Caractéristiques|Statut|
|---|---|---|
|**SSH v1**|- Vulnérabilités connues<br>- Algorithmes faibles<br>- Attaques MITM possibles|❌ Obsolète, déprécié|
|**SSH v2**|- Sécurité renforcée<br>- Meilleurs algorithmes<br>- Détection d'altération|✅ Standard actuel|

> [!warning] SSH v1 est vulnérable SSH version 1 présente des failles de sécurité connues et ne doit **jamais** être utilisé en production. De nombreuses normes de sécurité (PCI-DSS, ISO 27001) interdisent explicitement son utilisation.

### Commande de configuration

```bash
R1(config)# ip ssh version 2
```

Cette commande :

- Active uniquement SSH version 2
- Désactive automatiquement SSH version 1
- Est la configuration recommandée pour tous les environnements

### Vérification de la version

```bash
R1# show ip ssh
SSH Enabled - version 2.0
Authentication timeout: 120 secs; Authentication retries: 3
Minimum expected Diffie Hellman key size : 2048 bits
IOS Keys in SECSH format(ssh-rsa, base64 encoded):
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDxxxxx...
```

> [!info] Compatibilité SSH v2 est rétrocompatible et supporté par tous les clients SSH modernes :
> 
> - PuTTY, SecureCRT, MobaXterm
> - OpenSSH (Linux/Mac)
> - Clients intégrés Windows 10+

### Configuration avancée de SSH

#### Timeout d'authentification

Définit le temps maximum pour s'authentifier avant déconnexion :

```bash
R1(config)# ip ssh time-out 60
```

**Valeurs** :

- Par défaut : 120 secondes
- Recommandé : 60 secondes
- Minimum : 5 secondes

#### Nombre de tentatives d'authentification

```bash
R1(config)# ip ssh authentication-retries 3
```

**Valeurs** :

- Par défaut : 3 tentatives
- Recommandé : 2-3 tentatives
- Minimum : 1 tentative

> [!tip] Sécurité renforcée Réduire le nombre de tentatives et le timeout limite les attaques par force brute :
> 
> ```bash
> R1(config)# ip ssh time-out 30
> R1(config)# ip ssh authentication-retries 2
> R1(config)# ip ssh version 2
> ```

#### Version minimale de Diffie-Hellman

Pour une sécurité maximale, imposer une taille minimale de clé DH :

```bash
R1(config)# ip ssh dh min size 2048
```

> [!example] Configuration SSH complète sécurisée
> 
> ```bash
> R1(config)# ip ssh version 2
> R1(config)# ip ssh time-out 60
> R1(config)# ip ssh authentication-retries 2
> R1(config)# ip ssh dh min size 2048
> R1(config)# ip ssh source-interface GigabitEthernet0/0
> ```

#### Interface source pour SSH

Définir l'interface source améliore la sécurité et la gestion :

```bash
R1(config)# ip ssh source-interface Loopback0
```

**Avantages** :

- Adresse source cohérente dans les logs
- Simplification des ACLs
- Haute disponibilité (loopback toujours up)

### Logging SSH

Activer les logs pour tracer les connexions :

```bash
R1(config)# login on-success log
R1(config)# login on-failure log
```

Exemple de log généré :

```
%SEC_LOGIN-5-LOGIN_SUCCESS: Login Success [user: admin] [Source: 192.168.1.100]
```

---

## 🚫 Désactivation de Telnet

### Concept

Telnet est un protocole **non sécurisé** qui transmet toutes les données en clair, y compris :

- Les identifiants (username/password)
- Les commandes tapées
- Les sorties de commandes
- Les configurations

> [!warning] Danger de Telnet Telnet expose :
> 
> - **Capture d'identifiants** : Un attaquant sur le réseau peut intercepter les mots de passe
> - **Espionnage** : Toutes les commandes sont visibles
> - **Man-in-the-middle** : Possibilité d'injection de commandes
> - **Non-conformité** : Violation des standards de sécurité modernes

### Désactivation sur les lignes VTY

Les lignes VTY (Virtual TeletYpe) permettent les connexions distantes. Par défaut, elles acceptent Telnet.

#### Identification des lignes VTY

```bash
R1# show running-config | section line vty
line vty 0 4
 login local
 transport input telnet ssh
```

#### Configuration pour SSH uniquement

```bash
R1(config)# line vty 0 4
R1(config-line)# transport input ssh
R1(config-line)# exit
```

> [!info] Explication
> 
> - `line vty 0 4` : Configure les 5 premières lignes VTY (0 à 4)
> - `transport input ssh` : Autorise **uniquement** SSH en entrée
> - Cette commande remplace toute configuration précédente sur le transport

#### Désactivation complète de Telnet

```bash
R1(config)# line vty 0 4
R1(config-line)# transport input ssh
R1(config-line)# transport output ssh
R1(config-line)# exit
```

**Différence** :

- `transport input` : Protocoles acceptés pour se connecter **au** routeur
- `transport output` : Protocoles autorisés pour se connecter **depuis** le routeur

> [!example] Configuration VTY sécurisée complète
> 
> ```bash
> R1(config)# line vty 0 15
> R1(config-line)# transport input ssh
> R1(config-line)# login local
> R1(config-line)# exec-timeout 5 0
> R1(config-line)# logging synchronous
> R1(config-line)# exit
> ```

### Options de transport

|Commande|Description|
|---|---|
|`transport input all`|Autorise Telnet et SSH (❌ non recommandé)|
|`transport input telnet`|Telnet uniquement (❌ non sécurisé)|
|`transport input ssh`|SSH uniquement (✅ recommandé)|
|`transport input none`|Désactive toutes les connexions|

### Vérification

```bash
R1# show running-config | section line vty
line vty 0 4
 login local
 transport input ssh
line vty 5 15
 login local
 transport input ssh
```

### Test de connexion

Depuis un autre équipement, tester :

```bash
# Telnet doit être refusé
PC> telnet 192.168.1.1
Trying 192.168.1.1 ...
% Connection refused by remote host

# SSH doit fonctionner
PC> ssh -l admin 192.168.1.1
Password: 
R1>
```

> [!tip] Extension du nombre de VTY Si vous avez besoin de plus de 5 connexions simultanées :
> 
> ```bash
> R1(config)# line vty 0 15
> R1(config-line)# transport input ssh
> ```
> 
> Cela permet jusqu'à 16 connexions SSH simultanées (0-15).

### Configuration avec ACL

Pour une sécurité accrue, limiter les adresses IP autorisées :

```bash
# Créer une ACL
R1(config)# access-list 10 permit 192.168.1.0 0.0.0.255
R1(config)# access-list 10 deny any log

# Appliquer aux lignes VTY
R1(config)# line vty 0 15
R1(config-line)# access-class 10 in
R1(config-line)# transport input ssh
```

> [!info] Layered Security Cette configuration combine :
> 
> - Chiffrement (SSH)
> - Filtrage d'adresses (ACL)
> - Authentification (login local)
> 
> C'est une approche de sécurité en profondeur (defense in depth).

---

## ⚙️ Configuration complète

### Séquence de configuration étape par étape

Voici la configuration complète pour activer SSH de manière sécurisée :

```bash
# 1. Configuration du hostname et domain-name
Router> enable
Router# configure terminal
Router(config)# hostname R1
R1(config)# ip domain-name example.com

# 2. Création d'un utilisateur local (si pas déjà fait)
R1(config)# username admin privilege 15 secret Cisco123!

# 3. Génération des clés RSA
R1(config)# crypto key generate rsa modulus 2048

# 4. Activation de SSH version 2
R1(config)# ip ssh version 2
R1(config)# ip ssh time-out 60
R1(config)# ip ssh authentication-retries 2

# 5. Configuration des lignes VTY
R1(config)# line vty 0 15
R1(config-line)# transport input ssh
R1(config-line)# login local
R1(config-line)# exec-timeout 5 0
R1(config-line)# logging synchronous
R1(config-line)# exit

# 6. Sauvegarde de la configuration
R1(config)# exit
R1# copy running-config startup-config
```

> [!example] Configuration production avec ACL
> 
> ```bash
> ! Configuration complète sécurisée pour production
> hostname R1-Core
> ip domain-name entreprise.local
> !
> ! Utilisateur administrateur
> username admin privilege 15 secret 8 $8$xyz...
> !
> ! ACL pour limiter l'accès SSH
> access-list 99 remark SSH Management Access
> access-list 99 permit 10.0.0.0 0.0.0.255
> access-list 99 permit host 192.168.1.100
> access-list 99 deny any log
> !
> ! Paramètres SSH
> ip ssh version 2
> ip ssh time-out 30
> ip ssh authentication-retries 2
> ip ssh source-interface Loopback0
> ip ssh dh min size 2048
> !
> ! Lignes VTY
> line vty 0 15
>  access-class 99 in
>  exec-timeout 5 0
>  logging synchronous
>  transport input ssh
>  login local
> !
> ! Logging des connexions
> login on-success log
> login on-failure log
> ```

### Script de déploiement rapide

Pour un déploiement rapide sur plusieurs routeurs :

```bash
conf t
hostname CHANGE-ME
ip domain-name mon-domaine.local
username admin privilege 15 secret MotDePasseSecurise!
crypto key generate rsa modulus 2048
ip ssh version 2
ip ssh time-out 60
ip ssh authentication-retries 2
line vty 0 15
 transport input ssh
 login local
 exec-timeout 5 0
end
copy run start
```

> [!tip] Automatisation Ce script peut être :
> 
> - Copié-collé dans un terminal
> - Intégré dans Ansible/Python pour automatisation
> - Sauvegardé comme template de configuration

---

## 🔍 Vérification et dépannage

### Commandes de vérification

#### Vérifier l'état SSH

```bash
R1# show ip ssh
SSH Enabled - version 2.0
Authentication timeout: 60 secs; Authentication retries: 2
Minimum expected Diffie Hellman key size : 2048 bits
IOS Keys in SECSH format(ssh-rsa, base64 encoded):
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCxx...
```

#### Vérifier les clés RSA

```bash
R1# show crypto key mypubkey rsa
% Key pair was generated at: 15:23:45 UTC Mar 1 2024
Key name: R1.example.com
 Storage Device: not specified
 Usage: General Purpose Key
 Key is not exportable.
 Key Data:
  30820122 300D0609 2A864886 F70D0101 01050003 82010F00 3082010A 02820101
  [...]
```

#### Vérifier les sessions SSH actives

```bash
R1# show ssh
Connection Version Mode Encryption  Hmac         State          Username
0          2.0     IN   aes256-cbc  hmac-sha1    SessionStarted admin
1          2.0     IN   aes256-cbc  hmac-sha1    SessionStarted technicien

%No SSHv1 server connections running.
```

#### Vérifier la configuration des VTY

```bash
R1# show line vty 0 4
Tty Line Typ     Tx/Rx    A Modem  Roty AccO AccI   Uses   Noise  Overruns   Int
*  2 VTY              -    -    -     -    -    -      0       0     0/0       -
   3 VTY              -    -    -     -    -    -      0       0     0/0       -
   4 VTY              -    -    -     -    -    -      1       0     0/0       -
   5 VTY              -    -    -     -    -    -      0       0     0/0       -
   6 VTY              -    -    -     -    -    -      0       0     0/0       -

Line(s) not in async mode -or- with no hardware support:
1-2, 7-17
```

### Tests de connectivité

#### Test depuis un client SSH

```bash
# Linux/Mac
ssh admin@192.168.1.1

# Avec verbosité pour debug
ssh -v admin@192.168.1.1

# Windows PowerShell
ssh admin@192.168.1.1
```

#### Vérifier depuis le routeur

```bash
# Vérifier que le service écoute
R1# show control-plane host open-ports
Active internet connections (servers and established)
Prot        Local Address      Foreign Address                  Service    State
 tcp                 *:22                  *:0                  SSH-Server LISTEN
 tcp                 *:23                  *:0                    Telnet LISTEN
```

> [!warning] Si Telnet apparaît encore Cela signifie que Telnet n'est pas complètement désactivé. Vérifiez :
> 
> ```bash
> R1# show running-config | include transport
> ```

### Debug SSH (uniquement si problème)

> [!warning] Attention avec debug Les commandes debug sont intensives en CPU. Ne les utilisez qu'en dépannage et désactivez-les ensuite.

```bash
# Activer le debug SSH
R1# debug ip ssh

# Observer les tentatives de connexion
# ...logs apparaissent...

# Désactiver le debug
R1# undebug all
```

### Problèmes courants et solutions

#### Problème 1 : "% Please define a domain-name first"

**Cause** : Nom de domaine non configuré

**Solution** :

```bash
R1(config)# ip domain-name example.com
R1(config)# crypto key generate rsa modulus 2048
```

#### Problème 2 : "Connection refused"

**Causes possibles** :

1. SSH non activé
2. Lignes VTY mal configurées
3. ACL bloquante

**Diagnostic** :

```bash
R1# show ip ssh
R1# show running-config | section line vty
R1# show access-lists
```

#### Problème 3 : "Permission denied"

**Causes possibles** :

1. Mauvais identifiants
2. `login local` non configuré
3. Base d'utilisateurs locale vide

**Solution** :

```bash
R1(config)# username admin privilege 15 secret MotDePasse
R1(config)# line vty 0 15
R1(config-line)# login local
```

#### Problème 4 : Déconnexion après timeout

**Cause** : Timeout exec trop court

**Solution** :

```bash
R1(config)# line vty 0 15
R1(config-line)# exec-timeout 10 0
```

#### Problème 5 : "No RSA key pair"

**Cause** : Clés RSA supprimées ou jamais générées

**Solution** :

```bash
R1(config)# crypto key generate rsa modulus 2048
```

### Tableau de diagnostic rapide

|Symptôme|Commande de vérification|Solution probable|
|---|---|---|
|Connexion refusée|`show ip ssh`|Activer SSH v2|
|Permission denied|`show running-config \| section username`|Créer utilisateur local|
|Timeout rapide|`show running-config \| section line vty`|Augmenter exec-timeout|
|Pas de clés|`show crypto key mypubkey rsa`|Générer clés RSA|
|ACL bloque|`show access-lists`|Modifier ACL|

---

## ⚠️ Pièges courants

### 1. Oublier le nom de domaine

> [!warning] Erreur fréquente
> 
> ```bash
> Router(config)# crypto key generate rsa
> % Please define a domain-name first.
> ```
> 
> **Toujours configurer** le hostname ET le domain-name avant de générer les clés.

### 2. Taille de clé insuffisante

```bash
# ❌ Mauvais - Clé trop faible
Router(config)# crypto key generate rsa modulus 512

# ✅ Bon - Clé sécurisée
Router(config)# crypto key generate rsa modulus 2048
```

### 3. Ne pas sauvegarder la configuration

> [!tip] Toujours sauvegarder Après toute modification SSH :
> 
> ```bash
> R1# copy running-config startup-config
> ```
> 
> Sinon, la configuration est perdue au redémarrage !

### 4. Bloquer sa propre connexion

Si vous êtes connecté en SSH et que vous modifiez mal les VTY :

```bash
# ❌ Dangereux si vous êtes connecté en SSH
R1(config)# line vty 0 15
R1(config-line)# transport input none
```

> [!warning] Protection Gardez toujours une session console ouverte ou une autre connexion SSH active avant de modifier les VTY.

### 5. Désactiver SSH sans Telnet de secours

```bash
# ❌ Vous perdez tout accès distant
R1(config)# line vty 0 15
R1(config-line)# transport input none
```

**Récupération** : Nécessite un accès console physique.

### 6. Oublier `login local`

```bash
# ❌ SSH activé mais pas d'authentification configurée
R1(config)# line vty 0 15
R1(config-line)# transport input ssh
# Manque : login local
```

**Résultat** : Connexions refusées car pas de méthode d'authentification.

### 7. ACL trop restrictive

```bash
# ❌ Bloque tout le monde, y compris les admins
R1(config)# access-list 10 deny any
R1(config)# line vty 0 15
R1(config-line)# access-class 10 in
```

> [!tip] Testez vos ACL Testez toujours avec une IP autorisée avant d'appliquer aux VTY.

### 8. Ne pas vérifier après configuration

**Bonne pratique** : Toujours vérifier immédiatement :

```bash
R1# show ip ssh
R1# show running-config | section line vty
R1# show crypto key mypubkey rsa
```

### 9. Modifier domain-name sans régénérer les clés

```bash
# ❌ Changement de domaine
R1(config)# ip domain-name nouveau-domaine.com
# Les clés RSA gardent l'ancien nom !
```

**Solution** : Régénérer les clés après changement :

```bash
R1(config)# crypto key zeroize rsa
R1(config)# crypto key generate rsa modulus 2048
```

### 10. Debug SSH laissé actif

```bash
# ❌ Laissé actif, charge le CPU
R1# debug ip ssh
```

> [!warning] Impact performance Les debug sont intensifs. Toujours les désactiver après usage :
> 
> ```bash
> R1# undebug all
> ```

---

## 📝 Checklist de configuration SSH

Avant de considérer la configuration terminée, vérifiez :

- [ ] Hostname configuré
- [ ] Domain-name configuré
- [ ] Clés RSA générées (2048 bits minimum)
- [ ] SSH version 2 activé
- [ ] Telnet désactivé sur VTY
- [ ] Utilisateur local créé
- [ ] `login local` configuré sur VTY
- [ ] Timeout SSH configuré (≤ 60 secondes)
- [ ] Nombre de tentatives limité (≤ 3)
- [ ] Configuration sauvegardée
- [ ] Test de connexion SSH réussi
- [ ] Telnet bloqué vérifié
- [ ] Logs de connexion activés (optionnel)
- [ ] ACL configurée si nécessaire (optionnel)

> [!tip] Documentation Documentez toujours :
> 
> - Le mot de passe du compte admin
> - Les adresses IP autorisées (si ACL)
> - La date de génération des clés RSA
> - Les paramètres SSH spécifiques

---