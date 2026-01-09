

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

## Introduction

L'interface en ligne de commande (CLI) de Cisco IOS utilise une **structure hiérarchique de modes** qui détermine quelles commandes sont disponibles et quel niveau de privilège est requis. Cette architecture à plusieurs niveaux protège le routeur contre les modifications accidentelles ou malveillantes tout en offrant une granularité fine dans l'administration.

> [!info] Pourquoi plusieurs modes ? Cette séparation permet de :
> 
> - **Sécuriser** l'accès aux commandes critiques
> - **Organiser** les commandes de façon logique et contextuelle
> - **Prévenir** les erreurs de configuration accidentelles
> - **Structurer** le workflow d'administration

---

## Mode utilisateur (User EXEC)

### 🎯 Caractéristiques

Le **mode utilisateur** est le premier niveau d'accès lorsqu'on se connecte à un routeur Cisco. C'est un mode en **lecture seule** avec des privilèges très limités.

**Prompt caractéristique :**

```bash
Router>
```

> [!warning] Limitations importantes
> 
> - Aucune commande de configuration n'est disponible
> - Impossible de voir les configurations complètes
> - Accès limité aux commandes de diagnostic basiques
> - Conçu pour une consultation minimale

### 📝 Commandes disponibles

Dans ce mode, vous avez accès uniquement à des **commandes de visualisation basiques** :

```bash
Router> show users              # Affiche les utilisateurs connectés
Router> show version            # Version d'IOS (limité)
Router> ping 192.168.1.1        # Test de connectivité
Router> traceroute 8.8.8.8      # Trace le chemin réseau
Router> show history            # Historique des commandes
Router> enable                  # Passe au mode privilégié
```

> [!example] Exemple d'utilisation Un technicien de niveau 1 peut se connecter en mode utilisateur pour vérifier si un routeur est accessible (ping) sans risquer de modifier la configuration.

### 🎓 Quand l'utiliser ?

- **Consultation rapide** de l'état du routeur
- **Tests réseau simples** (ping, traceroute)
- **Utilisateurs non-administrateurs** qui n'ont besoin que de vérifications basiques

> [!tip] Astuce Pour voir toutes les commandes disponibles dans un mode, tapez `?` à l'invite de commande. Pour voir les commandes commençant par une lettre, tapez cette lettre suivie de `?` (ex: `sh?`).

---

## Mode privilégié (Privileged EXEC)

### 🎯 Caractéristiques

Le **mode privilégié** (aussi appelé **mode enable** ou **mode exec privilégié**) offre un accès complet aux commandes de **visualisation et de diagnostic**, mais toujours **sans possibilité de modification**.

**Prompt caractéristique :**

```bash
Router#
```

> [!info] Différence clé Le symbole `#` remplace le `>` et indique que vous avez des privilèges élevés. C'est le mode où vous passerez le plus de temps pour diagnostiquer et analyser.

### 🔐 Accès au mode privilégié

Pour passer du mode utilisateur au mode privilégié :

```bash
Router> enable              # Commande pour entrer en mode privilégié
Password:                   # Saisie du mot de passe enable (si configuré)
Router#                     # Vous êtes maintenant en mode privilégié
```

Pour revenir au mode utilisateur :

```bash
Router# disable             # Retour au mode utilisateur
Router>
```

### 📝 Commandes disponibles

Le mode privilégié donne accès à **toutes les commandes show** et aux outils de diagnostic avancés :

```bash
Router# show running-config       # Affiche la config active en mémoire
Router# show startup-config       # Affiche la config sauvegardée en NVRAM
Router# show ip route             # Table de routage
Router# show ip interface brief   # Résumé des interfaces
Router# show interfaces           # Détails complets des interfaces
Router# show protocols            # Protocoles de routage actifs
Router# show version              # Informations système complètes
Router# show flash                # Contenu de la mémoire flash
Router# debug ip routing          # Active le débogage (à utiliser avec prudence)
Router# copy running-config startup-config  # Sauvegarde la configuration
Router# reload                    # Redémarre le routeur
Router# erase startup-config      # Efface la configuration sauvegardée
```

> [!warning] Commandes dangereuses Certaines commandes comme `reload`, `erase startup-config` ou `write erase` peuvent perturber le service. Utilisez-les avec précaution en production.

### 🎓 Quand l'utiliser ?

- **Diagnostic réseau avancé** (analyse des routes, états des interfaces)
- **Consultation de la configuration** complète
- **Sauvegarde de configuration**
- **Vérifications avant modifications**

> [!tip] Bonne pratique - Vérification avant modification Toujours utiliser `show running-config` en mode privilégié avant d'entrer en mode configuration pour comprendre l'état actuel du routeur.

---

## Mode de configuration globale

### 🎯 Caractéristiques

Le **mode de configuration globale** (Global Configuration Mode) est le mode où vous **modifiez la configuration** du routeur. Les paramètres configurés ici affectent le **système dans son ensemble**.

**Prompt caractéristique :**

```bash
Router(config)#
```

> [!info] Différence fondamentale C'est le premier mode qui permet de **modifier** réellement la configuration. Tout ce qui est fait ici est stocké dans la `running-config` (configuration active en RAM).

### 🔐 Accès au mode de configuration globale

```bash
Router# configure terminal      # Commande complète
Router(config)#

# Ou version abrégée (très courante)
Router# conf t                  # Version courte acceptée
Router(config)#
```

Pour sortir du mode de configuration globale :

```bash
Router(config)# exit            # Retour au mode privilégié
Router#

# Ou raccourci
Router(config)# end             # Retour direct au mode privilégié
Router#
```

### 📝 Commandes typiques

Les commandes de ce mode affectent le routeur **globalement** :

```bash
Router(config)# hostname MonRouteur        # Change le nom du routeur
MonRouteur(config)#                        # Le prompt change immédiatement

MonRouteur(config)# enable secret cisco123  # Définit le mot de passe privilégié (chiffré)
MonRouteur(config)# enable password cisco   # Mot de passe privilégié (clair, déprécié)

MonRouteur(config)# banner motd #          # Message du jour
Entrez le texte, terminez par #
Acces reserve aux administrateurs autorises
#

MonRouteur(config)# no ip domain-lookup    # Désactive la résolution DNS (utile en lab)
MonRouteur(config)# ip domain-name exemple.com  # Définit le nom de domaine

MonRouteur(config)# service password-encryption  # Chiffre les mots de passe en clair
MonRouteur(config)# no service password-encryption  # Désactive le chiffrement type 7

MonRouteur(config)# clock timezone CET 1   # Définit le fuseau horaire
```

> [!example] Exemple pratique Configuration initiale d'un nouveau routeur :
> 
> ```bash
> Router# conf t
> Router(config)# hostname R1-Paris
> R1-Paris(config)# enable secret Admin2024!
> R1-Paris(config)# no ip domain-lookup
> R1-Paris(config)# banner motd #Acces Restreint#
> R1-Paris(config)# end
> R1-Paris#
> ```

### 🎓 Quand l'utiliser ?

- **Configuration système globale** (hostname, mots de passe, bannières)
- **Point d'entrée vers les sous-modes** de configuration
- **Paramètres affectant tout le routeur**

> [!tip] Sauvegarde immédiate Les modifications en mode configuration globale sont actives immédiatement dans la `running-config` mais seront **perdues au redémarrage** si vous ne sauvegardez pas avec `copy running-config startup-config` ou `write memory`.

---

## Modes de configuration spécifiques

Les modes de configuration spécifiques sont des **sous-modes** du mode de configuration globale. Chacun est dédié à la configuration d'un **élément particulier** du routeur.

### 🔧 Mode de configuration d'interface

**Prompt caractéristique :**

```bash
Router(config-if)#
```

Ce mode permet de configurer les paramètres d'une **interface réseau spécifique**.

#### Accès au mode

```bash
Router(config)# interface gigabitethernet 0/0    # Version complète
Router(config-if)#

# Versions abrégées acceptées
Router(config)# interface gi0/0                  # Abréviation
Router(config-if)#

Router(config)# int g0/0                         # Encore plus court
Router(config-if)#
```

#### Commandes typiques

```bash
Router(config-if)# ip address 192.168.1.1 255.255.255.0  # Adresse IP de l'interface
Router(config-if)# no shutdown                           # Active l'interface
Router(config-if)# shutdown                              # Désactive l'interface

Router(config-if)# description Lien vers Switch-Core    # Description de l'interface
Router(config-if)# bandwidth 100000                      # Bande passante (en Kbps)
Router(config-if)# speed 100                             # Force la vitesse (10/100/1000)
Router(config-if)# duplex full                           # Mode duplex (half/full/auto)

Router(config-if)# ip helper-address 192.168.10.5       # Serveur DHCP relay
Router(config-if)# clock rate 64000                      # Horloge sur interface série DCE
```

> [!warning] Piège courant - Interface shutdown Par défaut, les interfaces Cisco sont en état `shutdown` (désactivées). Vous **devez** exécuter `no shutdown` pour les activer. C'est une erreur très fréquente chez les débutants.

#### Configuration de plusieurs interfaces

Pour configurer un **range d'interfaces** :

```bash
Router(config)# interface range gi0/0-3          # Sélectionne gi0/0, gi0/1, gi0/2, gi0/3
Router(config-if-range)#                         # Prompt spécifique

Router(config-if-range)# switchport mode access
Router(config-if-range)# switchport access vlan 10
Router(config-if-range)# exit
```

> [!example] Exemple complet - Configuration d'interface
> 
> ```bash
> Router# conf t
> Router(config)# interface gi0/1
> Router(config-if)# description Connexion LAN Entreprise
> Router(config-if)# ip address 10.0.1.1 255.255.255.0
> Router(config-if)# no shutdown
> Router(config-if)# exit
> Router(config)#
> ```

---

### 📞 Mode de configuration de ligne

**Prompt caractéristique :**

```bash
Router(config-line)#
```

Ce mode configure les **lignes d'accès** au routeur : console, VTY (Telnet/SSH), auxiliaire.

#### Types de lignes

|Type de ligne|Description|Nombre typique|
|---|---|---|
|**console 0**|Port console physique|1|
|**vty 0 4** (ou 0 15)|Lignes Telnet/SSH virtuelles|5 ou 16|
|**aux 0**|Port auxiliaire (modem)|1|

#### Accès au mode

```bash
# Configuration de la console
Router(config)# line console 0
Router(config-line)#

# Configuration des lignes VTY (accès distant)
Router(config)# line vty 0 4              # Lignes VTY 0 à 4 (5 connexions simultanées)
Router(config-line)#

# Configuration du port auxiliaire
Router(config)# line aux 0
Router(config-line)#
```

#### Commandes typiques

```bash
Router(config-line)# password cisco123         # Mot de passe pour la ligne
Router(config-line)# login                     # Active l'authentification par mot de passe

Router(config-line)# login local              # Authentification avec base locale (username/password)
Router(config-line)# transport input ssh      # Autorise uniquement SSH (sécurisé)
Router(config-line)# transport input telnet   # Autorise uniquement Telnet
Router(config-line)# transport input ssh telnet  # Autorise SSH et Telnet
Router(config-line)# transport input none     # Désactive tous les accès distants

Router(config-line)# exec-timeout 5 30        # Timeout : 5 min 30 sec d'inactivité
Router(config-line)# exec-timeout 0 0         # Désactive le timeout (déconseillé)

Router(config-line)# logging synchronous      # Empêche les messages de perturber la saisie
Router(config-line)# history size 50          # Taille de l'historique des commandes
```

> [!tip] Sécurisation SSH uniquement Pour un accès distant sécurisé, configurez :
> 
> ```bash
> Router(config)# line vty 0 4
> Router(config-line)# transport input ssh
> Router(config-line)# login local
> Router(config-line)# exit
> ```

> [!example] Exemple - Sécurisation de la console
> 
> ```bash
> Router(config)# line console 0
> Router(config-line)# password ConsolePwd2024
> Router(config-line)# login
> Router(config-line)# logging synchronous
> Router(config-line)# exec-timeout 10 0
> Router(config-line)# exit
> ```

---

### 🛣️ Mode de configuration de routeur (protocole de routage)

**Prompt caractéristique :**

```bash
Router(config-router)#
```

Ce mode configure les **protocoles de routage dynamique** comme RIP, EIGRP, OSPF, BGP.

> [!info] Note sur la portée Les détails des protocoles de routage seront abordés dans une partie ultérieure du cours. Nous nous concentrons ici uniquement sur le **mode de configuration** lui-même.

#### Accès au mode

```bash
# Configuration OSPF
Router(config)# router ospf 1              # Processus OSPF numéro 1
Router(config-router)#

# Configuration EIGRP
Router(config)# router eigrp 100           # Système autonome 100
Router(config-router)#

# Configuration RIP
Router(config)# router rip
Router(config-router)#
```

#### Exemple de structure (sans détails)

```bash
Router(config)# router ospf 1
Router(config-router)# network 192.168.1.0 0.0.0.255 area 0
Router(config-router)# network 10.0.0.0 0.0.0.255 area 0
Router(config-router)# exit
Router(config)#
```

> [!info] À noter Chaque protocole de routage a ses propres commandes spécifiques. Le mode `config-router` est simplement le contexte où ces commandes sont appliquées.

---

### 🔑 Autres modes spécifiques importants

#### Mode DHCP Pool

```bash
Router(config)# ip dhcp pool LAN_POOL
Router(dhcp-config)# network 192.168.1.0 255.255.255.0
Router(dhcp-config)# default-router 192.168.1.1
Router(dhcp-config)# dns-server 8.8.8.8
Router(dhcp-config)# exit
```

#### Mode ACL (Access Control List)

```bash
Router(config)# ip access-list standard ADMIN_ONLY
Router(config-std-nacl)# permit 192.168.1.0 0.0.0.255
Router(config-std-nacl)# deny any
Router(config-std-nacl)# exit
```

> [!tip] Reconnaissance du mode Le prompt change toujours pour indiquer dans quel contexte vous êtes :
> 
> - `(config)#` : configuration globale
> - `(config-if)#` : interface
> - `(config-line)#` : ligne
> - `(config-router)#` : protocole de routage
> - `(dhcp-config)#` : pool DHCP
> - etc.

---

## Navigation entre les modes

### 🗺️ Hiérarchie des modes

```
Mode Utilisateur (Router>)
    ↓ enable
Mode Privilégié (Router#)
    ↓ configure terminal
Mode Configuration Globale (Router(config)#)
    ↓ interface gi0/0          ↓ line vty 0 4          ↓ router ospf 1
Config Interface         Config Ligne            Config Routeur
(config-if)#            (config-line)#         (config-router)#
```

### 🔄 Commandes de navigation

#### Montée dans la hiérarchie

```bash
# Depuis n'importe quel mode de configuration vers le mode privilégié
Router(config-if)# end              # Retour direct au mode #
Router#

Router(config-line)# end
Router#

# Raccourci clavier (plus rapide)
Router(config-if)# Ctrl+Z           # Équivalent à "end"
Router#
```

#### Descente d'un niveau

```bash
# Sortir du sous-mode actuel vers le mode parent
Router(config-if)# exit             # Retour au mode (config)#
Router(config)#

Router(config-line)# exit
Router(config)#

Router(config)# exit                # Retour au mode #
Router#
```

#### Passage entre modes privilégié et utilisateur

```bash
# Descente
Router# disable                     # Passe en mode >
Router>

# Montée
Router> enable                      # Passe en mode #
Router#
```

### 📊 Tableau récapitulatif des commandes

|Commande|Action|Depuis|Vers|
|---|---|---|---|
|`enable`|Monte en privilégié|`>`|`#`|
|`disable`|Descend en utilisateur|`#`|`>`|
|`configure terminal` (ou `conf t`)|Entre en config globale|`#`|`(config)#`|
|`exit`|Remonte d'un niveau|N'importe quel mode|Mode parent|
|`end` ou `Ctrl+Z`|Retour au mode privilégié|N'importe quel mode config|`#`|
|`interface [type] [numéro]`|Entre en config interface|`(config)#`|`(config-if)#`|
|`line [type] [numéro]`|Entre en config ligne|`(config)#`|`(config-line)#`|
|`router [protocole] [id]`|Entre en config routeur|`(config)#`|`(config-router)#`|

> [!tip] Raccourcis pratiques
> 
> - `Ctrl+Z` : retour immédiat au mode `#` depuis n'importe quel mode de configuration
> - `exit` répété : remonte progressivement la hiérarchie
> - `end` : équivalent à `Ctrl+Z`, retour direct au mode `#`

### 🎯 Exemple de navigation complète

```bash
# Connexion initiale
Router>                                    # Mode utilisateur

Router> enable                             # Passage en mode privilégié
Password:
Router#

Router# configure terminal                 # Entrée en configuration globale
Router(config)#

Router(config)# interface gi0/0            # Sous-mode interface
Router(config-if)#

Router(config-if)# exit                    # Retour en config globale
Router(config)#

Router(config)# line vty 0 4               # Sous-mode ligne
Router(config-line)#

Router(config-line)# end                   # Retour direct en mode privilégié
Router#

Router# disable                            # Retour en mode utilisateur
Router>
```

> [!warning] Erreur courante Taper `exit` plusieurs fois rapidement peut vous déconnecter complètement de la session. Si vous êtes en `mode >` et tapez `exit`, vous **fermez la connexion**. Utilisez plutôt `enable` pour remonter.

---

## Récapitulatif des commandes

### 🎯 Commandes essentielles par mode

#### Mode Utilisateur (`>`)

```bash
enable              # Passe en mode privilégié
show users          # Utilisateurs connectés
show version        # Version système (limitée)
ping [IP]           # Test de connectivité
exit                # Déconnexion
```

#### Mode Privilégié (`#`)

```bash
configure terminal  # Entre en mode configuration globale
show running-config # Configuration active
show startup-config # Configuration sauvegardée
show ip route       # Table de routage
show ip interface brief  # Résumé des interfaces
copy running-config startup-config  # Sauvegarde
reload              # Redémarrage
disable             # Retour en mode utilisateur
exit                # Déconnexion
```

#### Mode Configuration Globale (`(config)#`)

```bash
hostname [nom]      # Change le nom du routeur
enable secret [pwd] # Mot de passe privilégié (chiffré)
interface [type] [num]  # Entre en config interface
line [type] [num]   # Entre en config ligne
router [protocole]  # Entre en config routeur
banner motd [délim] # Configure un message du jour
no ip domain-lookup # Désactive la résolution DNS
exit                # Retour en mode privilégié
end                 # Retour en mode privilégié
```

#### Mode Configuration Interface (`(config-if)#`)

```bash
ip address [IP] [masque]  # Adresse IP
no shutdown         # Active l'interface
shutdown            # Désactive l'interface
description [texte] # Description
exit                # Retour en config globale
end                 # Retour en mode privilégié
```

#### Mode Configuration Ligne (`(config-line)#`)

```bash
password [pwd]      # Mot de passe
login               # Active l'authentification
login local         # Authentification avec base locale
transport input [ssh/telnet]  # Protocoles autorisés
logging synchronous # Messages non intrusifs
exec-timeout [min] [sec]  # Timeout d'inactivité
exit                # Retour en config globale
end                 # Retour en mode privilégié
```

### 🔑 Raccourcis et astuces

|Raccourci|Action|
|---|---|
|`Ctrl+Z`|Retour direct au mode `#`|
|`Tab`|Autocomplétion de commande|
|`?`|Liste des commandes disponibles|
|`[commande] ?`|Aide sur une commande spécifique|
|`Ctrl+C`|Interrompt une commande en cours|
|`Ctrl+Shift+6`|Interrompt un ping/traceroute|
|`↑` / `↓`|Navigation dans l'historique|
|`Ctrl+A`|Début de ligne|
|`Ctrl+E`|Fin de ligne|

> [!tip] Aide contextuelle À tout moment, tapez `?` pour voir les commandes disponibles dans le mode actuel. C'est votre meilleur allié pour découvrir les options disponibles.

### 📌 Points clés à retenir

> [!info] Les essentiels
> 
> 1. **Hiérarchie stricte** : `>` → `#` → `(config)#` → sous-modes
> 2. Le **prompt change** pour indiquer le mode actuel
> 3. **Deux façons de remonter** : `exit` (niveau par niveau) ou `end`/`Ctrl+Z` (direct vers `#`)
> 4. Les modifications sont **immédiatement actives** mais **volatiles** sans sauvegarde
> 5. Toujours **sauvegarder** avec `copy run start` après les changements
> 6. Le symbole **`#`** indique un mode avec privilèges élevés

---

> [!warning] Sécurité et bonnes pratiques
> 
> - **Toujours** configurer un mot de passe `enable secret` (pas `enable password`)
> - **Toujours** sauvegarder après modification : `copy running-config startup-config`
> - **Utiliser SSH** plutôt que Telnet pour les accès distants
> - **Activer** `service password-encryption` pour chiffrer les mots de passe en clair
> - **Documenter** vos interfaces avec la commande `description`
> - **Tester** avant de déconnecter : vérifier qu'une interface de management reste accessible

---

_Ce document couvre les modes de configuration CLI essentiels. La maîtrise de cette navigation est fondamentale pour toute administration de routeur Cisco._