

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

## 🎯 Introduction au Client DHCP

### Qu'est-ce qu'un client DHCP ?

Un routeur Cisco configuré en **client DHCP** obtient automatiquement ses paramètres IP (adresse IP, masque de sous-réseau, passerelle par défaut, serveurs DNS) depuis un serveur DHCP externe. Cette configuration est courante pour :

- **Interfaces WAN** connectées à un FAI
- **Connexions temporaires** ou environnements de test
- **Réseaux où l'adressage centralisé** est requis
- **Succursales** recevant leur configuration d'un siège central

> [!info] Différence avec le serveur DHCP
> 
> - **Serveur DHCP** : Le routeur attribue des adresses IP aux autres périphériques
> - **Client DHCP** : Le routeur reçoit sa propre adresse IP d'un serveur externe

### Pourquoi utiliser un client DHCP ?

|Avantage|Description|
|---|---|
|**Simplicité**|Pas besoin de configurer manuellement les paramètres IP|
|**Flexibilité**|Configuration automatique lors du changement de réseau|
|**Centralisation**|Gestion simplifiée depuis le serveur DHCP|
|**Rapidité**|Déploiement accéléré de nouveaux équipements|

---

## ⚙️ Configuration d'une interface en client DHCP

### Syntaxe de base

La configuration d'une interface en client DHCP est très simple sur un routeur Cisco.

```cisco
Router(config)# interface [type] [numéro]
Router(config-if)# ip address dhcp
Router(config-if)# no shutdown
Router(config-if)# exit
```

### Exemple pratique : Interface WAN

```cisco
! Configuration d'une interface WAN en client DHCP
Router(config)# interface GigabitEthernet0/0
Router(config-if)# description Connexion vers FAI
Router(config-if)# ip address dhcp
Router(config-if)# no shutdown
Router(config-if)# exit
```

> [!example] Scénario réel Vous connectez un routeur Cisco à une box Internet (modem FAI). L'interface WAN doit obtenir automatiquement son adresse IP publique ou privée depuis la box qui agit comme serveur DHCP.

### Options avancées du client DHCP

#### Définir un hostname pour l'identification

```cisco
Router(config)# interface GigabitEthernet0/0
Router(config-if)# ip address dhcp
Router(config-if)# ip dhcp client hostname ROUTER-PARIS
Router(config-if)# no shutdown
```

> [!tip] Astuce - Identification Le hostname permet au serveur DHCP d'identifier facilement le client dans ses logs et réservations. Utilisez des noms descriptifs et standardisés.

#### Demander un client-id spécifique

```cisco
Router(config)# interface GigabitEthernet0/0
Router(config-if)# ip address dhcp client-id GigabitEthernet0/0
Router(config-if)# no shutdown
```

> [!info] Client-ID Par défaut, le routeur utilise l'adresse MAC de l'interface. Le `client-id` peut être basé sur l'interface ou personnalisé pour des réservations DHCP spécifiques.

#### Configurer le délai de requête

```cisco
Router(config)# interface GigabitEthernet0/0
Router(config-if)# ip address dhcp
Router(config-if)# ip dhcp client request tftp-server-address
Router(config-if)# ip dhcp client request router
Router(config-if)# no shutdown
```

> [!info] Options demandées Vous pouvez spécifier quelles options DHCP le client doit demander au serveur (tftp-server, router, dns-server, etc.).

### Configuration complète avec toutes les options

```cisco
! Exemple complet d'interface WAN en client DHCP
Router(config)# interface GigabitEthernet0/0
Router(config-if)# description Connexion Internet - FAI XYZ
Router(config-if)# ip address dhcp
Router(config-if)# ip dhcp client hostname RTR-BRANCH-01
Router(config-if)# ip dhcp client client-id GigabitEthernet0/0
Router(config-if)# ip dhcp client request router
Router(config-if)# ip dhcp client request dns-nameserver
Router(config-if)# negotiation auto
Router(config-if)# no shutdown
Router(config-if)# exit
```

---

## 🔍 Vérification de l'attribution DHCP

### Commandes de vérification essentielles

#### 1. Afficher la configuration de l'interface

```cisco
Router# show ip interface GigabitEthernet0/0
```

**Sortie attendue :**

```
GigabitEthernet0/0 is up, line protocol is up
  Internet address is 192.168.1.100/24
  Broadcast address is 255.255.255.255
  Address determined by DHCP
  MTU is 1500 bytes
  Helper address is not set
  [...]
```

> [!tip] Indicateur clé Recherchez la ligne **"Address determined by DHCP"** qui confirme que l'adresse a été obtenue via DHCP.

#### 2. Afficher les informations DHCP détaillées

```cisco
Router# show dhcp lease
```

**Sortie exemple :**

```
Temp IP addr: 192.168.1.100  for peer on Interface: GigabitEthernet0/0
Temp sub net mask: 255.255.255.0
   DHCP Lease server: 192.168.1.1, state: 5 Bound
   DHCP transaction id: 1A2B
   Lease: 86400 secs,  Renewal: 43200 secs,  Rebind: 75600 secs
   Next timer fires after: 11:59:45
   Retry count: 0   Client-ID: cisco-0011.2233.4455-Gi0/0
   Client-ID hex dump: 636973636F2D303031312E323233332E
                       343435352D4769302F30
   Hostname: ROUTER-PARIS
```

> [!info] Informations importantes
> 
> - **State: 5 Bound** : Le client a obtenu et maintient un bail DHCP actif
> - **Lease** : Durée totale du bail
> - **Renewal** : Moment où le client tentera de renouveler
> - **Rebind** : Moment où le client contactera n'importe quel serveur si le renouvellement échoue

#### 3. Vérifier la table de routage

```cisco
Router# show ip route
```

**Sortie exemple :**

```
Gateway of last resort is 192.168.1.1 to network 0.0.0.0

S*    0.0.0.0/0 [254/0] via 192.168.1.1
      192.168.1.0/24 is variably subnetted, 2 subnets, 2 masks
C        192.168.1.0/24 is directly connected, GigabitEthernet0/0
L        192.168.1.100/32 is directly connected, GigabitEthernet0/0
```

> [!tip] Route par défaut Le serveur DHCP fournit généralement une passerelle par défaut qui apparaît comme route par défaut (0.0.0.0/0) dans la table de routage.

#### 4. Vérifier les serveurs DNS reçus

```cisco
Router# show ip dns view
```

Ou vérifier dans la configuration running :

```cisco
Router# show running-config | include name-server
```

**Sortie exemple :**

```
ip name-server 8.8.8.8
ip name-server 8.8.4.4
```

#### 5. Test de connectivité

```cisco
! Test ping vers la passerelle par défaut
Router# ping 192.168.1.1

! Test ping vers un DNS public
Router# ping 8.8.8.8

! Test de résolution DNS
Router# ping google.com
```

### Tableau récapitulatif des commandes de vérification

|Commande|Information fournie|Utilisation|
|---|---|---|
|`show ip interface brief`|Statut et adresses IP de toutes les interfaces|Vue d'ensemble rapide|
|`show ip interface [interface]`|Détails complets d'une interface|Vérification approfondie|
|`show dhcp lease`|Informations du bail DHCP actif|Dépannage DHCP|
|`show ip route`|Table de routage incluant la passerelle|Vérification du routage|
|`show running-config interface [interface]`|Configuration active de l'interface|Audit de configuration|

---

## 🔧 Dépannage et bonnes pratiques

### Problèmes courants et solutions

#### ❌ Problème 1 : Aucune adresse IP obtenue

**Symptômes :**

```cisco
Router# show ip interface brief
Interface              IP-Address      OK? Method Status                Protocol
GigabitEthernet0/0     unassigned      YES DHCP   up                    up
```

**Solutions :**

```cisco
! 1. Vérifier que l'interface est activée
Router(config)# interface GigabitEthernet0/0
Router(config-if)# no shutdown

! 2. Libérer et renouveler le bail DHCP
Router# release dhcp GigabitEthernet0/0
Router# renew dhcp GigabitEthernet0/0

! 3. Vérifier la connectivité physique
Router# show interface GigabitEthernet0/0
! Rechercher "line protocol is up"

! 4. Activer le debug DHCP (attention en production !)
Router# debug dhcp detail
! Puis désactiver après diagnostic
Router# no debug dhcp detail
```

> [!warning] Debug en production Les commandes `debug` génèrent beaucoup de trafic console. Utilisez-les avec précaution sur des équipements en production et désactivez-les immédiatement après le diagnostic.

#### ❌ Problème 2 : Adresse IP obtenue mais pas de connectivité Internet

**Diagnostic :**

```cisco
! Vérifier l'adresse IP obtenue
Router# show dhcp lease

! Vérifier la passerelle par défaut
Router# show ip route | include 0.0.0.0

! Tester la passerelle
Router# ping [adresse-passerelle]

! Vérifier les DNS
Router# show running-config | include name-server

! Tester un DNS externe
Router# ping 8.8.8.8
```

**Solution possible :**

```cisco
! Si pas de passerelle par défaut, l'ajouter manuellement
Router(config)# ip route 0.0.0.0 0.0.0.0 [adresse-passerelle]

! Si pas de DNS, les configurer manuellement
Router(config)# ip name-server 8.8.8.8 8.8.4.4
```

#### ❌ Problème 3 : Bail DHCP expire trop rapidement

**Vérification :**

```cisco
Router# show dhcp lease
! Vérifier les valeurs de Lease, Renewal, Rebind
```

> [!info] Durées de bail
> 
> - **Durée courte (< 1 heure)** : Appropriée pour réseaux invités ou BYOD
> - **Durée moyenne (1-24 heures)** : Standard pour la plupart des réseaux
> - **Durée longue (> 1 jour)** : Pour équipements fixes ou serveurs
> 
> Nota : Ce paramètre est configuré côté serveur DHCP, pas côté client.

### Bonnes pratiques

> [!tip] Recommandation 1 : Documentation Documentez toujours quelle interface utilise DHCP et pourquoi. Utilisez la commande `description` :
> 
> ```cisco
> Router(config-if)# description WAN - DHCP depuis FAI-ORANGE
> ```

> [!tip] Recommandation 2 : Sauvegarde de configuration Avant de configurer une interface en DHCP (surtout si c'est votre interface de management !), sauvegardez votre configuration :
> 
> ```cisco
> Router# copy running-config startup-config
> ```

> [!tip] Recommandation 3 : Interface de secours Si vous configurez votre interface de management en DHCP, assurez-vous d'avoir un accès console physique en cas de problème :
> 
> ```cisco
> ! NE JAMAIS faire ceci sans accès console :
> Router(config)# interface GigabitEthernet0/0
> Router(config-if)# ip address dhcp
> ! Si le DHCP échoue, vous perdez l'accès distant !
> ```

> [!warning] Attention - Interface de management **Ne configurez JAMAIS votre unique interface de management en client DHCP sans avoir :**
> 
> - Un accès console physique à l'équipement
> - Une autre interface de management configurée en statique
> - La certitude que le serveur DHCP est fiable

> [!tip] Recommandation 4 : Monitoring Dans un environnement de production, surveillez l'état des baux DHCP avec des scripts ou des outils de monitoring pour détecter les problèmes avant qu'ils n'impactent les utilisateurs.

### Commandes de dépannage avancées

```cisco
! Réinitialiser complètement l'interface
Router(config)# interface GigabitEthernet0/0
Router(config-if)# shutdown
Router(config-if)# no ip address
Router(config-if)# ip address dhcp
Router(config-if)# no shutdown

! Forcer une nouvelle requête DHCP
Router# release dhcp GigabitEthernet0/0
Router# renew dhcp GigabitEthernet0/0

! Afficher les statistiques d'erreurs sur l'interface
Router# show interface GigabitEthernet0/0 | include error

! Vérifier les logs système
Router# show logging | include DHCP
```

### Cas d'usage spécifiques

#### Configuration pour connexion FAI

```cisco
! Interface WAN vers fournisseur d'accès Internet
Router(config)# interface GigabitEthernet0/0
Router(config-if)# description WAN - Connexion FAI
Router(config-if)# ip address dhcp
Router(config-if)# ip dhcp client hostname RTR-SITE-PRINCIPAL
Router(config-if)# ip nat outside
Router(config-if)# no shutdown
```

> [!info] NAT et DHCP L'interface WAN en DHCP est souvent associée à la configuration NAT (`ip nat outside`), mais le NAT appartient à une autre partie du cours et ne sera pas détaillé ici.

#### Configuration pour réseau de lab/test

```cisco
! Interface dans un environnement de test
Router(config)# interface GigabitEthernet0/1
Router(config-if)# description Lab - Configuration dynamique
Router(config-if)# ip address dhcp
Router(config-if)# no shutdown
```

---

## 📊 Comparaison : IP statique vs DHCP client

|Critère|IP Statique|DHCP Client|
|---|---|---|
|**Complexité**|Configuration manuelle requise|Configuration automatique|
|**Flexibilité**|Fixe, nécessite reconfiguration|S'adapte automatiquement|
|**Fiabilité**|Indépendant d'un serveur externe|Dépend du serveur DHCP|
|**Cas d'usage**|Serveurs, équipements critiques|WAN, environnements dynamiques|
|**Traçabilité**|Adresse prévisible|Adresse peut changer|
|**Temps de déploiement**|Plus long|Très rapide|

> [!info] Choix de configuration
> 
> - **Utilisez IP statique** : pour les interfaces LAN, serveurs, équipements d'infrastructure
> - **Utilisez DHCP client** : pour les interfaces WAN vers FAI, environnements temporaires, labs

---

## 🎯 Points clés à retenir

1. **Configuration simple** : `ip address dhcp` sur l'interface concernée
2. **Vérification essentielle** : `show dhcp lease` et `show ip interface`
3. **Attention au management** : Ne jamais perdre l'accès à l'équipement
4. **Documentation** : Toujours utiliser `description` pour identifier les interfaces DHCP
5. **Dépannage** : Commandes `release` et `renew` pour forcer le renouvellement

> [!warning] Rappel important Le client DHCP sur routeur Cisco est principalement utilisé pour les interfaces WAN. Pour les réseaux LAN internes, privilégiez toujours une configuration IP statique pour garantir la stabilité et la prévisibilité de l'infrastructure.

---

_Configuration Client DHCP - Routeur Cisco IOS_