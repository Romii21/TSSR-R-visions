

## 📚 Table des matières

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

## 🎯 Introduction au DHCP Relay

Le **DHCP Relay** (ou agent relais DHCP) est un mécanisme qui permet de transmettre des requêtes DHCP entre des clients situés sur un réseau et un serveur DHCP situé sur un autre réseau. Par défaut, les messages DHCP sont des broadcasts qui ne traversent pas les routeurs. Le DHCP Relay résout ce problème.

> [!info] Pourquoi utiliser un DHCP Relay ?
> 
> - **Centralisation** : Un seul serveur DHCP peut gérer plusieurs sous-réseaux
> - **Économie** : Pas besoin de déployer un serveur DHCP sur chaque VLAN/sous-réseau
> - **Maintenance simplifiée** : Administration centralisée des plages d'adresses
> - **Scalabilité** : Facilite la gestion dans des environnements multi-sites

---

## 🔄 Principe du DHCP Relay

### Fonctionnement sans DHCP Relay

Lorsqu'un client envoie une requête DHCP DISCOVER, il s'agit d'un **broadcast** (destination 255.255.255.255) qui ne peut pas traverser un routeur. Sans relay, chaque réseau nécessite son propre serveur DHCP.

```
Client DHCP ----[DISCOVER broadcast]----> X Routeur (bloque)
                                          
Serveur DHCP (sur un autre réseau) ne reçoit rien
```

### Fonctionnement avec DHCP Relay

Le routeur, configuré comme agent relais, intercepte les broadcasts DHCP et les transmet en **unicast** au serveur DHCP configuré. Le routeur ajoute également des informations importantes (comme l'adresse IP de l'interface d'entrée) pour que le serveur sache quelle plage d'adresses attribuer.

```
1. Client ----[DISCOVER broadcast]----> Routeur (relay)
2. Routeur ----[DISCOVER unicast]----> Serveur DHCP
3. Serveur DHCP ----[OFFER unicast]----> Routeur
4. Routeur ----[OFFER]----> Client
```

> [!warning] Attention Le champ **GIADDR** (Gateway IP Address) dans le paquet DHCP est rempli par le routeur avec l'adresse IP de l'interface qui a reçu la requête. C'est ainsi que le serveur DHCP sait de quel réseau provient la demande.

---

## ⚙️ Configuration de base

### Commande principale : `ip helper-address`

La configuration du DHCP Relay sur un routeur Cisco se fait avec la commande `ip helper-address` sur l'interface qui reçoit les requêtes DHCP des clients.

```bash
Router(config)# interface [interface-id]
Router(config-if)# ip helper-address [adresse-IP-serveur-DHCP]
```

### Exemple de configuration simple

```bash
! Configuration d'une interface de sous-réseau client
Router(config)# interface GigabitEthernet0/0
Router(config-if)# ip address 192.168.10.1 255.255.255.0
Router(config-if)# ip helper-address 10.1.1.100
Router(config-if)# no shutdown

! Le serveur DHCP est à l'adresse 10.1.1.100
! Les clients du réseau 192.168.10.0/24 pourront obtenir des adresses IP
```

> [!example] Explication
> 
> - L'interface `GigabitEthernet0/0` est la passerelle du réseau client 192.168.10.0/24
> - Tous les broadcasts DHCP reçus sur cette interface seront transférés en unicast vers 10.1.1.100
> - Le serveur DHCP doit avoir une plage configurée pour le réseau 192.168.10.0/24

### Protocoles relayés par défaut

La commande `ip helper-address` ne relaie pas uniquement DHCP. Par défaut, elle transfère 8 types de broadcasts UDP :

|Port UDP|Service|
|---|---|
|37|Time|
|49|TACACS|
|53|DNS|
|67|DHCP/BOOTP Server|
|68|DHCP/BOOTP Client|
|69|TFTP|
|137|NetBIOS Name Service|
|138|NetBIOS Datagram Service|

> [!tip] Astuce : Limiter les protocoles relayés Pour ne relayer que DHCP et éviter du trafic inutile :
> 
> ```bash
> Router(config)# no ip forward-protocol udp 37
> Router(config)# no ip forward-protocol udp 49
> Router(config)# no ip forward-protocol udp 53
> Router(config)# no ip forward-protocol udp 69
> Router(config)# no ip forward-protocol udp 137
> Router(config)# no ip forward-protocol udp 138
> ```
> 
> Seuls les ports 67 et 68 (DHCP) resteront actifs.

---

## 🏢 Configuration multi-sites

### Scénario : Plusieurs VLANs, un seul serveur DHCP

Dans un environnement avec plusieurs VLANs, chaque interface VLAN doit être configurée avec `ip helper-address` pointant vers le serveur DHCP centralisé.

```bash
! VLAN 10 - Département Commercial
Router(config)# interface Vlan10
Router(config-if)# ip address 192.168.10.1 255.255.255.0
Router(config-if)# ip helper-address 10.1.1.100
Router(config-if)# no shutdown

! VLAN 20 - Département Technique
Router(config)# interface Vlan20
Router(config-if)# ip address 192.168.20.1 255.255.255.0
Router(config-if)# ip helper-address 10.1.1.100
Router(config-if)# no shutdown

! VLAN 30 - Département RH
Router(config)# interface Vlan30
Router(config-if)# ip address 192.168.30.1 255.255.255.0
Router(config-if)# ip helper-address 10.1.1.100
Router(config-if)# no shutdown
```

> [!info] Configuration du serveur DHCP Le serveur DHCP (10.1.1.100) doit avoir trois plages d'adresses configurées :
> 
> - Une pour 192.168.10.0/24 (VLAN 10)
> - Une pour 192.168.20.0/24 (VLAN 20)
> - Une pour 192.168.30.0/24 (VLAN 30)
> 
> Le serveur identifie le réseau grâce au champ GIADDR dans les requêtes DHCP.

### Scénario : Redondance avec plusieurs serveurs DHCP

Pour assurer la haute disponibilité, vous pouvez configurer plusieurs serveurs DHCP. Ajoutez simplement plusieurs commandes `ip helper-address` sur la même interface.

```bash
Router(config)# interface Vlan10
Router(config-if)# ip address 192.168.10.1 255.255.255.0
Router(config-if)# ip helper-address 10.1.1.100
Router(config-if)# ip helper-address 10.1.1.200
Router(config-if)# no shutdown
```

> [!warning] Comportement avec plusieurs serveurs
> 
> - Les requêtes DHCP seront envoyées à **tous** les serveurs configurés
> - Le client acceptera la première réponse reçue (généralement la plus rapide)
> - Assurez-vous que les plages d'adresses ne se chevauchent pas entre serveurs pour éviter les conflits

### Scénario : Sites distants avec liaison WAN

Dans une architecture multi-sites reliés par WAN, le DHCP Relay permet de centraliser le serveur DHCP au siège tout en desservant les sites distants.

```bash
! === Configuration Routeur Site Distant ===

! Interface vers le LAN local
Router-Site-A(config)# interface GigabitEthernet0/0
Router-Site-A(config-if)# ip address 192.168.100.1 255.255.255.0
Router-Site-A(config-if)# ip helper-address 10.0.0.50
Router-Site-A(config-if)# no shutdown

! Interface WAN vers le siège
Router-Site-A(config)# interface Serial0/0/0
Router-Site-A(config-if)# ip address 172.16.1.2 255.255.255.252
Router-Site-A(config-if)# no shutdown

! Route vers le serveur DHCP au siège (si nécessaire)
Router-Site-A(config)# ip route 10.0.0.0 255.255.255.0 172.16.1.1
```

> [!tip] Optimisation WAN Dans un contexte WAN avec bande passante limitée :
> 
> - Désactivez les protocoles UDP inutiles (voir section précédente)
> - Envisagez un serveur DHCP local pour réduire la latence
> - Utilisez le DHCP Relay uniquement si la centralisation est critique

---

## 🔍 Vérification et dépannage

### Commandes de vérification

```bash
! Afficher la configuration des interfaces
Router# show running-config interface [interface-id]

! Afficher les statistiques de relay DHCP
Router# show ip dhcp relay information

! Vérifier les broadcasts reçus
Router# show ip interface [interface-id]

! Afficher les protocoles UDP transférés
Router# show ip forward-protocol
```

### Exemple de sortie

```bash
Router# show ip interface GigabitEthernet0/0
GigabitEthernet0/0 is up, line protocol is up
  Internet address is 192.168.10.1/24
  Broadcast address is 255.255.255.255
  Address determined by setup command
  MTU is 1500 bytes
  Helper address is 10.1.1.100
  [...]
```

### Dépannage courant

|Problème|Cause possible|Solution|
|---|---|---|
|Les clients n'obtiennent pas d'IP|Helper-address mal configurée|Vérifier l'adresse IP du serveur DHCP|
||Serveur DHCP injoignable|Vérifier la connectivité (ping) et le routage|
||Serveur DHCP n'a pas de plage pour ce réseau|Configurer une plage correspondant au GIADDR|
|Lenteur d'attribution|Multiples helper-address avec délais|Réduire le nombre de serveurs ou optimiser|
||Latence WAN élevée|Envisager un serveur DHCP local|

> [!warning] Pièges courants
> 
> 1. **Oubli de configurer le serveur DHCP** : Le relay fonctionne, mais le serveur n'a pas de plage pour le sous-réseau concerné
> 2. **Problèmes de routage** : Le routeur peut atteindre le serveur, mais le serveur ne peut pas répondre (route de retour manquante)
> 3. **Firewall/ACL bloquant** : Les ports UDP 67/68 doivent être autorisés
> 4. **GIADDR dans le mauvais réseau** : Si l'interface relay a une IP dans le mauvais réseau, le serveur ne saura pas quelle plage utiliser

### Debug (à utiliser avec précaution en production)

```bash
! Activer le debug des paquets DHCP
Router# debug ip dhcp server packet

! Activer le debug des événements DHCP
Router# debug ip dhcp server events

! Désactiver le debug
Router# no debug all
ou
Router# undebug all
```

> [!warning] Attention au debug Les commandes `debug` génèrent beaucoup de messages sur la console et peuvent impacter les performances du routeur. À utiliser uniquement lors du dépannage et à désactiver immédiatement après.

---

## ✅ Bonnes pratiques

### 1. Documentation

- **Documentez chaque helper-address** : Notez quel serveur DHCP est utilisé et pourquoi
- **Maintenez un inventaire** : Listez tous les réseaux et leurs serveurs DHCP associés
- **Commentaires dans la config** : Utilisez des commentaires pour clarifier

```bash
Router(config)# interface Vlan10
Router(config-if)# description VLAN Commercial - DHCP via 10.1.1.100
Router(config-if)# ip address 192.168.10.1 255.255.255.0
Router(config-if)# ip helper-address 10.1.1.100
```

### 2. Sécurité

- **Limitez les protocoles relayés** : N'autorisez que DHCP (ports 67/68) si possible
- **ACLs sur les interfaces** : Protégez le serveur DHCP avec des ACLs appropriées
- **DHCP Snooping** : Sur les switches, activez DHCP snooping pour éviter les serveurs DHCP malveillants

### 3. Redondance et disponibilité

- **Serveurs DHCP multiples** : Configurez au moins deux serveurs pour la redondance
- **Plages non-chevauchantes** : Si vous utilisez plusieurs serveurs, assurez-vous que les plages ne se chevauchent pas
- **Monitoring** : Surveillez la disponibilité de vos serveurs DHCP et les taux d'attribution

### 4. Performance

- **Proximité géographique** : Dans les environnements distribués, envisagez des serveurs DHCP locaux
- **Optimisation WAN** : Désactivez les protocoles UDP inutiles pour réduire la charge réseau
- **Dimensionnement des plages** : Prévoyez suffisamment d'adresses mais pas trop (gaspillage d'espace d'adressage)

### 5. Maintenance

- **Tests réguliers** : Vérifiez périodiquement que les clients obtiennent bien des adresses
- **Logs du serveur DHCP** : Analysez les logs pour détecter les problèmes d'attribution
- **Mise à jour de la configuration** : Lors de changements de topologie, pensez à mettre à jour les helper-address

> [!tip] Astuce professionnelle Créez un document de procédure standardisé pour la configuration du DHCP Relay dans votre organisation. Incluez :
> 
> - Template de configuration
> - Checklist de vérification
> - Procédure de test
> - Procédure de rollback en cas de problème

---

## 📋 Récapitulatif des commandes essentielles

```bash
! Configuration de base
Router(config)# interface [interface-id]
Router(config-if)# ip helper-address [IP-serveur-DHCP]

! Désactiver un protocole UDP relayé
Router(config)# no ip forward-protocol udp [port]

! Vérification
Router# show running-config interface [interface-id]
Router# show ip interface [interface-id]
Router# show ip forward-protocol

! Debug (avec précaution)
Router# debug ip dhcp server packet
Router# undebug all
```

---

> [!info] Résumé Le **DHCP Relay** est un élément essentiel dans les architectures réseau modernes permettant la centralisation et la scalabilité de l'attribution d'adresses IP. La commande `ip helper-address` sur les interfaces passerelles transforme les broadcasts DHCP en unicasts vers les serveurs configurés, permettant ainsi à un serveur DHCP centralisé de gérer plusieurs sous-réseaux ou VLANs. Une configuration correcte, associée à une documentation rigoureuse et un monitoring adéquat, garantit un service DHCP fiable et performant.