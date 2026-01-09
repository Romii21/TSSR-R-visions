

## 📑 Table des matières

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

## 📘 Introduction

Les interfaces réseau constituent les points de connexion physiques et logiques d'un routeur Cisco. Chaque type d'interface répond à des besoins spécifiques et nécessite une configuration appropriée pour fonctionner correctement.

> [!info] Pourquoi configurer les interfaces ? La configuration des interfaces permet de :
> 
> - Établir la connectivité réseau
> - Définir les adresses IP et les masques de sous-réseau
> - Activer ou désactiver des ports
> - Optimiser les performances réseau
> - Sécuriser les connexions

---

## 🔍 Types d'interfaces

### Interfaces FastEthernet/GigabitEthernet

Les interfaces Ethernet sont utilisées pour les connexions LAN (Local Area Network). Elles permettent de connecter le routeur aux switches, ordinateurs, ou autres équipements réseau locaux.

#### 📊 Différences principales

|Type|Vitesse|Nomenclature|Usage typique|
|---|---|---|---|
|FastEthernet|100 Mbps|Fa0/0, Fa0/1|Anciens réseaux LAN|
|GigabitEthernet|1000 Mbps (1 Gbps)|Gi0/0, Gi0/1, G0/0|Réseaux LAN modernes|

#### ⚙️ Configuration de base

```bash
Router> enable
Router# configure terminal
Router(config)# interface gigabitEthernet 0/0
Router(config-if)# ip address 192.168.1.1 255.255.255.0
Router(config-if)# description Connexion vers le LAN principal
Router(config-if)# no shutdown
Router(config-if)# exit
```

> [!tip] Astuce de nomenclature Les interfaces Ethernet utilisent la notation : `type slot/port`
> 
> - `GigabitEthernet 0/0` = première interface du slot 0
> - `GigabitEthernet 0/1` = deuxième interface du slot 0

#### 🎯 Commandes importantes

```bash
# Afficher l'état de l'interface
Router# show interface gigabitEthernet 0/0

# Vérifier l'adresse IP configurée
Router# show ip interface brief

# Configuration de la vitesse et du duplex (si nécessaire)
Router(config-if)# speed 100
Router(config-if)# duplex full
```

> [!warning] Piège courant : Interface "administratively down" Si après configuration l'interface reste down, vérifiez que vous avez bien exécuté la commande `no shutdown`. Par défaut, les interfaces Cisco sont désactivées.

#### 💡 Paramètres avancés

```bash
# Configuration du duplex et de la vitesse (généralement en auto)
Router(config-if)# duplex auto
Router(config-if)# speed auto

# Désactivation temporaire sans supprimer la config
Router(config-if)# shutdown

# Réactivation
Router(config-if)# no shutdown
```

---

### Interfaces série (WAN)

Les interfaces série sont principalement utilisées pour les connexions WAN (Wide Area Network) longue distance, typiquement pour relier des sites distants via des lignes louées ou des connexions point-à-point.

#### 🌐 Caractéristiques

> [!info] Spécificités des interfaces série
> 
> - Utilisées pour les connexions WAN point-à-point
> - Nécessitent une configuration d'horloge (clock rate) côté DCE
> - Supportent différents protocoles d'encapsulation (HDLC, PPP, Frame Relay)
> - Nomenclature : Serial 0/0/0, Serial 0/1/0, etc.

#### ⚙️ Configuration de base

```bash
Router(config)# interface serial 0/0/0
Router(config-if)# ip address 10.0.0.1 255.255.255.252
Router(config-if)# description Liaison WAN vers site distant
Router(config-if)# no shutdown
```

#### 🔄 Configuration DCE/DTE

Dans une liaison série, il existe deux types d'équipements :

- **DCE (Data Communication Equipment)** : Fournit l'horloge
- **DTE (Data Terminal Equipment)** : Reçoit l'horloge

```bash
# Vérifier si l'interface est DCE ou DTE
Router# show controllers serial 0/0/0

# Si DCE, configurer le clock rate
Router(config)# interface serial 0/0/0
Router(config-if)# clock rate 64000
```

> [!warning] Attention au clock rate Le `clock rate` ne doit être configuré QUE sur le côté DCE de la liaison. Si vous le configurez sur le DTE, cela n'aura aucun effet et peut créer de la confusion lors du dépannage.

#### 📋 Clock rates standards

|Vitesse|Clock Rate|Usage|
|---|---|---|
|64 Kbps|64000|Ligne lente|
|128 Kbps|128000|Ligne basique|
|1.544 Mbps|1544000|T1|
|2.048 Mbps|2048000|E1|

#### 🔧 Protocoles d'encapsulation

```bash
# HDLC (par défaut sur Cisco)
Router(config-if)# encapsulation hdlc

# PPP (plus standard, interopérable)
Router(config-if)# encapsulation ppp

# Vérifier l'encapsulation
Router# show interface serial 0/0/0
```

> [!tip] Choisir l'encapsulation
> 
> - **HDLC** : Propriétaire Cisco, par défaut, simple
> - **PPP** : Standard, authentification possible, meilleur pour multi-vendeurs

#### 💡 Configuration complète d'une liaison série

```bash
# Côté DCE (fournit l'horloge)
Router-DCE(config)# interface serial 0/0/0
Router-DCE(config-if)# ip address 10.1.1.1 255.255.255.252
Router-DCE(config-if)# clock rate 128000
Router-DCE(config-if)# encapsulation ppp
Router-DCE(config-if)# description Liaison vers Router-DTE
Router-DCE(config-if)# no shutdown

# Côté DTE (reçoit l'horloge)
Router-DTE(config)# interface serial 0/0/0
Router-DTE(config-if)# ip address 10.1.1.2 255.255.255.252
Router-DTE(config-if)# encapsulation ppp
Router-DTE(config-if)# description Liaison vers Router-DCE
Router-DTE(config-if)# no shutdown
```

---

### Interfaces auxiliaires

L'interface auxiliaire (AUX) est un port série utilisé principalement pour l'accès administratif à distance via modem ou pour des connexions de secours.

#### 📞 Caractéristiques

> [!info] Utilisation de l'interface AUX
> 
> - Accès distant d'urgence au routeur
> - Connexion via modem pour administration hors-bande
> - Backup pour la connexion console
> - Rarement utilisée dans les réseaux modernes

#### ⚙️ Configuration de base

```bash
Router(config)# line aux 0
Router(config-line)# password cisco123
Router(config-line)# login
Router(config-line)# exec-timeout 10 0
```

> [!warning] Sécurité de l'interface AUX Si l'interface auxiliaire n'est pas utilisée, il est recommandé de la désactiver pour éviter tout accès non autorisé :
> 
> ```bash
> Router(config)# line aux 0
> Router(config-line)# no exec
> Router(config-line)# transport input none
> ```

#### 🔧 Configuration pour accès modem

```bash
Router(config)# line aux 0
Router(config-line)# password SecurePass123!
Router(config-line)# login
Router(config-line)# modem InOut
Router(config-line)# transport input all
Router(config-line)# speed 115200
Router(config-line)# flowcontrol hardware
```

> [!tip] Modernité Avec l'avènement des connexions VPN et SSH, l'interface auxiliaire est de moins en moins utilisée. Dans les environnements modernes, privilégiez les connexions sécurisées via le réseau.

---

### Loopback

Une interface loopback est une interface virtuelle (logique) qui n'a pas de composant physique. Elle reste toujours active tant que le routeur fonctionne.

#### 🔄 Pourquoi utiliser des loopbacks ?

> [!info] Avantages des interfaces loopback
> 
> - **Toujours UP** : Indépendante de l'état des interfaces physiques
> - **Identifiant unique** : Parfait pour l'identité du routeur (Router ID)
> - **Tests et diagnostics** : Simulation de réseaux pour les tests
> - **Protocoles de routage** : Point d'ancrage stable pour OSPF, BGP, etc.
> - **Management** : Adresse stable pour l'administration

#### ⚙️ Configuration d'une interface loopback

```bash
# Création de la loopback
Router(config)# interface loopback 0
Router(config-if)# ip address 1.1.1.1 255.255.255.255
Router(config-if)# description Interface de management
Router(config-if)# no shutdown
```

> [!tip] Convention de nommage
> 
> - Numérotation : Loopback 0, Loopback 1, etc.
> - Pour l'identité du routeur, utilisez généralement **Loopback 0**
> - Masque /32 (255.255.255.255) pour une adresse d'hôte unique

#### 💡 Utilisations courantes

**1. Router ID pour OSPF**

```bash
Router(config)# interface loopback 0
Router(config-if)# ip address 10.0.0.1 255.255.255.255

# OSPF utilisera automatiquement cette adresse comme Router ID
# (l'adresse loopback la plus élevée)
```

**2. Management et administration**

```bash
# Interface stable pour SSH/Telnet
Router(config)# interface loopback 100
Router(config-if)# ip address 192.168.100.1 255.255.255.255
Router(config-if)# description Interface de management
```

**3. Tests et simulations**

```bash
# Simulation de réseaux pour les tests
Router(config)# interface loopback 10
Router(config-if)# ip address 172.16.10.1 255.255.255.0
Router(config-if)# description Reseau de test
```

> [!example] Exemple pratique Dans un réseau d'entreprise avec 3 routeurs interconnectés, chaque routeur possède une loopback avec une adresse unique (1.1.1.1, 2.2.2.2, 3.3.3.3). Ces adresses servent de Router ID pour OSPF et d'adresses de management, garantissant une identification stable même si des liens physiques tombent.

#### 🔍 Vérification des loopbacks

```bash
# Afficher toutes les interfaces loopback
Router# show ip interface brief | include Loopback

# Détails d'une loopback spécifique
Router# show interface loopback 0

# Vérifier le Router ID utilisé
Router# show ip protocols
```

> [!warning] Les loopbacks ne peuvent pas être shutdown Même si vous exécutez `shutdown` sur une loopback, elle reste dans l'état "up" car c'est une interface virtuelle. Pour la désactiver réellement, vous devez la supprimer complètement :
> 
> ```bash
> Router(config)# no interface loopback 0
> ```

---

## 🛠️ Configuration de base des interfaces

### Processus standard de configuration

```bash
# 1. Entrer en mode privilégié
Router> enable

# 2. Entrer en mode de configuration globale
Router# configure terminal

# 3. Sélectionner l'interface
Router(config)# interface gigabitEthernet 0/0

# 4. Configurer l'adresse IP
Router(config-if)# ip address 192.168.1.1 255.255.255.0

# 5. Ajouter une description (bonne pratique)
Router(config-if)# description Connexion LAN Bureau Principal

# 6. Activer l'interface
Router(config-if)# no shutdown

# 7. Sortir du mode de configuration
Router(config-if)# exit
Router(config)# exit
```

### Commandes essentielles par interface

```bash
# Description (toujours recommandée)
Router(config-if)# description [texte descriptif]

# Adresse IP
Router(config-if)# ip address [adresse] [masque]

# Activation/Désactivation
Router(config-if)# no shutdown
Router(config-if)# shutdown

# Suppression d'une configuration IP
Router(config-if)# no ip address
```

> [!tip] Descriptions significatives Utilisez des descriptions claires et standardisées :
> 
> - `description WAN vers Site Paris - Lien principal`
> - `description LAN VLAN10 - Departement RH`
> - `description Backup vers ISP2`

---

## 📊 Commandes de vérification

### Vérification rapide

```bash
# Vue d'ensemble de toutes les interfaces
Router# show ip interface brief

# Sortie typique :
# Interface              IP-Address      OK? Method Status                Protocol
# GigabitEthernet0/0     192.168.1.1     YES manual up                    up
# GigabitEthernet0/1     unassigned      YES unset  administratively down down
# Serial0/0/0            10.1.1.1        YES manual up                    up
# Loopback0              1.1.1.1         YES manual up                    up
```

### Vérification détaillée d'une interface

```bash
# Informations complètes
Router# show interface gigabitEthernet 0/0

# Informations IP spécifiques
Router# show ip interface gigabitEthernet 0/0

# Statistiques et compteurs
Router# show interface gigabitEthernet 0/0 | include error|drop|packet
```

### Commandes de diagnostic

```bash
# Vérifier la configuration en cours
Router# show running-config interface gigabitEthernet 0/0

# Vérifier les contrôleurs (pour les interfaces série)
Router# show controllers serial 0/0/0

# Statistiques d'erreurs
Router# show interface gigabitEthernet 0/0 stats
```

> [!tip] Interpréter "Status" et "Protocol"
> 
> - **Status** : État de la couche physique (Layer 1)
>     - `up` : Câble connecté, signal présent
>     - `down` : Problème physique (câble, connecteur)
>     - `administratively down` : Interface désactivée par commande
> - **Protocol** : État de la couche liaison (Layer 2)
>     - `up` : Protocole fonctionnel
>     - `down` : Problème de configuration ou de protocole

### Réinitialisation des compteurs

```bash
# Réinitialiser les statistiques d'une interface
Router# clear counters gigabitEthernet 0/0

# Confirmer avec "yes" ou appuyer sur Entrée
```

---

## ✅ Bonnes pratiques

### 1. Documentation et descriptions

> [!tip] Toujours documenter
> 
> ```bash
> Router(config-if)# description [Fonction] - [Destination] - [Infos complementaires]
> ```
> 
> Exemples :
> 
> - `description WAN - Vers Site Lyon - Lien 100Mbps Fibre`
> - `description LAN - VLAN 20 Production - Switch Core`
> - `description MGMT - Acces administration distance`

### 2. Standardisation des adresses IP

> [!info] Conventions recommandées
> 
> - **LAN** : 192.168.x.0/24
> - **WAN** : 10.x.x.0/30 (connexions point-à-point)
> - **Loopbacks** : x.x.x.x/32 (adresse unique)
> - **Management** : Réseau dédié séparé

### 3. Sécurité de base

```bash
# Désactiver les interfaces non utilisées
Router(config)# interface gigabitEthernet 0/1
Router(config-if)# shutdown
Router(config-if)# description Interface inutilisee - DESACTIVEE

# Désactiver CDP si non nécessaire (révèle infos du réseau)
Router(config-if)# no cdp enable
```

### 4. Sauvegarde régulière

```bash
# Sauvegarder la configuration après chaque modification
Router# copy running-config startup-config

# Ou version courte
Router# write memory
# Ou encore plus court
Router# wr
```

> [!warning] Ne jamais oublier de sauvegarder ! Sans sauvegarde, toute configuration est perdue au redémarrage du routeur. Prenez l'habitude de sauvegarder immédiatement après chaque modification importante.

### 5. Vérification systématique

> [!tip] Checklist de vérification Après chaque configuration d'interface :
> 
> 1. ✅ Vérifier que l'interface est UP : `show ip interface brief`
> 2. ✅ Tester la connectivité : `ping [adresse]`
> 3. ✅ Vérifier les erreurs : `show interface [nom]`
> 4. ✅ Contrôler la config : `show running-config interface [nom]`
> 5. ✅ Sauvegarder : `write memory`

### 6. Nommage cohérent des loopbacks

```bash
# Loopback 0 : Router ID / Identité
Router(config)# interface loopback 0
Router(config-if)# ip address 1.1.1.1 255.255.255.255

# Loopback 100-199 : Management
Router(config)# interface loopback 100
Router(config-if)# ip address 192.168.100.1 255.255.255.255

# Loopback 200-254 : Tests et simulations
Router(config)# interface loopback 200
Router(config-if)# ip address 172.16.200.1 255.255.255.0
```

### 7. Utilisation des ranges pour configurations multiples

```bash
# Configurer plusieurs interfaces similaires en une fois
Router(config)# interface range gigabitEthernet 0/0 - 3
Router(config-if-range)# shutdown
Router(config-if-range)# description Interfaces desactivees
```

---

## 🎯 Résumé des commandes clés

|Commande|Description|
|---|---|
|`interface [type] [numéro]`|Entrer en mode de configuration d'interface|
|`ip address [IP] [masque]`|Configurer l'adresse IP|
|`description [texte]`|Ajouter une description|
|`no shutdown`|Activer l'interface|
|`shutdown`|Désactiver l'interface|
|`clock rate [vitesse]`|Configurer l'horloge (série DCE)|
|`encapsulation [type]`|Définir le protocole d'encapsulation|
|`show ip interface brief`|Vue d'ensemble des interfaces|
|`show interface [nom]`|Détails d'une interface|
|`show controllers serial [num]`|Vérifier DCE/DTE|

---

> [!success] Points clés à retenir
> 
> - Chaque type d'interface a un rôle spécifique dans l'architecture réseau
> - La commande `no shutdown` est ESSENTIELLE pour activer une interface
> - Les loopbacks sont des interfaces virtuelles toujours actives
> - Les interfaces série nécessitent un `clock rate` côté DCE
> - Toujours documenter avec des descriptions claires
> - Vérifier systématiquement après configuration
> - Sauvegarder la configuration après chaque modification importante