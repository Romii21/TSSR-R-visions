

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

## 🚀 Activation d'IPv6 {#activation-ipv6}

### Pourquoi activer IPv6 ?

Par défaut, le routage IPv6 est **désactivé** sur les routeurs Cisco. Avant de pouvoir configurer des adresses IPv6 et router des paquets IPv6, vous devez explicitement activer cette fonctionnalité au niveau global du routeur.

> [!info] Comportement par défaut Sans la commande `ipv6 unicast-routing`, le routeur peut avoir des adresses IPv6 sur ses interfaces mais ne pourra **pas router** les paquets IPv6 entre ses interfaces. Il agira simplement comme un hôte IPv6.

### Configuration

```bash
Router> enable
Router# configure terminal
Router(config)# ipv6 unicast-routing
```

> [!tip] Bonne pratique Cette commande doit être la **première étape** de toute configuration IPv6 sur un routeur Cisco. Sans elle, aucun routage IPv6 ne sera possible.

### Effets de l'activation

Une fois activé, le routeur :

- ✅ Peut router les paquets IPv6 entre ses interfaces
- ✅ Génère automatiquement des adresses link-local sur les interfaces configurées
- ✅ Peut participer aux protocoles de routage IPv6 (RIPng, OSPFv3, EIGRP pour IPv6)
- ✅ Envoie des messages Router Advertisement (RA) sur les segments réseau

---

## 📝 Adressage IPv6 statique {#adressage-ipv6-statique}

### Comprendre l'adressage statique IPv6

L'adressage statique consiste à configurer manuellement une adresse IPv6 globale unicast (GUA - Global Unicast Address) sur une interface. Cette adresse est routable sur Internet et unique au niveau mondial.

> [!info] Types d'adresses IPv6
> 
> - **Global Unicast Address (GUA)** : Adresse routable publiquement (équivalent IPv4 publique)
> - **Unique Local Address (ULA)** : Adresse privée routable localement (équivalent IPv4 privée)
> - **Link-Local Address** : Adresse utilisée uniquement sur le segment local (abordée dans la section suivante)

### Syntaxe de configuration

```bash
Router(config)# interface <type> <numéro>
Router(config-if)# ipv6 address <adresse-ipv6>/<préfixe>
Router(config-if)# no shutdown
```

### Exemple pratique

```bash
Router(config)# interface gigabitEthernet 0/0
Router(config-if)# ipv6 address 2001:db8:acad:1::1/64
Router(config-if)# no shutdown
Router(config-if)# exit
```

> [!example] Décomposition de l'adresse Dans `2001:db8:acad:1::1/64` :
> 
> - **2001:db8:acad:1** : Préfixe réseau (64 premiers bits)
> - **::1** : Identifiant d'interface (64 derniers bits)
> - **/64** : Longueur du préfixe (standard pour les réseaux IPv6)

### Format EUI-64 (Extended Unique Identifier)

Au lieu de spécifier manuellement l'identifiant d'interface, vous pouvez utiliser le format EUI-64 qui génère automatiquement cette partie à partir de l'adresse MAC de l'interface.

```bash
Router(config)# interface gigabitEthernet 0/1
Router(config-if)# ipv6 address 2001:db8:acad:2::/64 eui-64
Router(config-if)# no shutdown
```

> [!tip] Quand utiliser EUI-64 ?
> 
> - ✅ Utile pour une configuration rapide
> - ✅ Garantit l'unicité de l'adresse
> - ❌ Moins lisible et prévisible
> - ❌ Peut poser des problèmes de confidentialité (l'adresse MAC est exposée)

### Comparaison : Statique vs EUI-64

|Critère|Adressage statique manuel|EUI-64|
|---|---|---|
|**Simplicité**|Nécessite de connaître l'adresse complète|Génération automatique|
|**Lisibilité**|Adresses organisées et prévisibles|Adresses moins lisibles|
|**Sécurité**|N'expose pas l'adresse MAC|Expose l'adresse MAC dans l'IPv6|
|**Usage recommandé**|Interfaces de routeur, serveurs|Prototypage rapide|

> [!warning] Attention au préfixe /64 Le préfixe **/64** est le standard pour les réseaux IPv6. Ne pas utiliser /64 peut causer des problèmes avec certains protocoles (notamment SLAAC et Neighbor Discovery).

---

## 🔗 Adressage IPv6 link-local {#adressage-ipv6-link-local}

### Qu'est-ce qu'une adresse link-local ?

Une adresse link-local est une adresse IPv6 qui n'est valide que sur un segment réseau local (un lien). Elle n'est **jamais routée** au-delà du segment local et commence toujours par le préfixe **fe80::/10**.

> [!info] Caractéristiques des adresses link-local
> 
> - Préfixe : **fe80::/10** (en pratique, toujours **fe80::/64**)
> - Portée : Segment réseau local uniquement
> - Obligatoire : Toute interface IPv6 active doit avoir une adresse link-local
> - Utilisation : Communication entre voisins, protocoles de routage, Neighbor Discovery

### Génération automatique

Lorsque vous activez IPv6 sur une interface (avec `ipv6 unicast-routing` activé globalement), une adresse link-local est **automatiquement générée** :

```bash
Router(config)# interface gigabitEthernet 0/0
Router(config-if)# ipv6 address 2001:db8:acad:1::1/64
Router(config-if)# no shutdown
# Une adresse link-local est automatiquement créée : fe80::xxxx:xxxx:xxxx:xxxx
```

L'identifiant d'interface (les 64 derniers bits) est généré via EUI-64 à partir de l'adresse MAC.

### Configuration manuelle d'une adresse link-local

Pour des raisons de lisibilité ou de configuration spécifique, vous pouvez définir manuellement l'adresse link-local :

```bash
Router(config)# interface gigabitEthernet 0/0
Router(config-if)# ipv6 address fe80::1 link-local
Router(config-if)# no shutdown
```

> [!tip] Bonnes pratiques pour les adresses link-local manuelles Utilisez des adresses simples et mémorables pour les routeurs :
> 
> - `fe80::1` pour le premier routeur
> - `fe80::2` pour le deuxième routeur
> - `fe80::a`, `fe80::b`, `fe80::c` pour identifier facilement les équipements

### Pourquoi configurer manuellement les link-local ?

|Avantage|Description|
|---|---|
|**Lisibilité**|Adresses courtes et faciles à retenir|
|**Documentation**|Facilite le dépannage et la documentation réseau|
|**Protocoles de routage**|Certains protocoles (OSPFv3) utilisent la link-local comme router-id|
|**Consistance**|Garantit que l'adresse reste identique même après un changement de carte réseau|

### Exemple de configuration complète

```bash
Router(config)# interface gigabitEthernet 0/0
Router(config-if)# description Liaison vers R2
Router(config-if)# ipv6 address 2001:db8:acad:1::1/64
Router(config-if)# ipv6 address fe80::1 link-local
Router(config-if)# no shutdown
Router(config-if)# exit

Router(config)# interface gigabitEthernet 0/1
Router(config-if)# description Liaison vers LAN
Router(config-if)# ipv6 address 2001:db8:acad:2::1/64
Router(config-if)# ipv6 address fe80::1 link-local
Router(config-if)# no shutdown
Router(config-if)# exit
```

> [!warning] Unicité locale L'adresse link-local doit être unique **sur le segment réseau local**. Vous pouvez réutiliser la même adresse link-local (ex: fe80::1) sur plusieurs interfaces du même routeur, car chaque interface est sur un segment différent.

> [!example] Utilisation des adresses link-local Les adresses link-local sont utilisées pour :
> 
> - **Next-hop dans les routes statiques IPv6**
> - **Communication entre routeurs adjacents** dans les protocoles de routage
> - **Neighbor Discovery Protocol (NDP)** : découverte de voisins, résolution d'adresses
> - **Router Advertisement** : annonce des préfixes réseau

---

## ✅ Vérification IPv6 {#verification-ipv6}

### Commandes de vérification essentielles

Une fois la configuration IPv6 effectuée, il est crucial de vérifier que tout fonctionne correctement. Voici les commandes principales.

#### 1. Vérifier l'état général d'IPv6

```bash
Router# show ipv6 interface brief
```

> [!example] Sortie type
> 
> ```
> GigabitEthernet0/0     [up/up]
>     FE80::1
>     2001:DB8:ACAD:1::1
> GigabitEthernet0/1     [up/up]
>     FE80::1
>     2001:DB8:ACAD:2::1
> ```

Cette commande affiche :

- ✅ L'état de chaque interface (up/up, down/down)
- ✅ Les adresses IPv6 configurées (link-local et globales)
- ✅ Vue d'ensemble rapide de la configuration

#### 2. Vérifier les détails d'une interface

```bash
Router# show ipv6 interface <type> <numéro>
```

Exemple :

```bash
Router# show ipv6 interface gigabitEthernet 0/0
```

> [!info] Informations affichées
> 
> - Adresses IPv6 (globale, link-local)
> - État de l'interface
> - Groupes multicast rejoints
> - Paramètres MTU
> - Statut ICMP redirects
> - Informations ND (Neighbor Discovery)

#### 3. Vérifier le routage IPv6 activé

```bash
Router# show ipv6 interface
```

Sans arguments, cette commande affiche les détails de **toutes** les interfaces IPv6.

> [!tip] Vérifier que le routage est activé Dans la sortie, cherchez la ligne :
> 
> ```
> IPv6 is enabled, link-local address is FE80::1
> ```
> 
> Si IPv6 n'est pas activé globalement, vous verrez un message différent.

#### 4. Afficher la table de routage IPv6

```bash
Router# show ipv6 route
```

> [!example] Sortie type
> 
> ```
> C   2001:DB8:ACAD:1::/64 [0/0]
>      via GigabitEthernet0/0, directly connected
> L   2001:DB8:ACAD:1::1/128 [0/0]
>      via GigabitEthernet0/0, receive
> ```

Codes importants :

- **C** : Réseau directement connecté
- **L** : Adresse locale (l'adresse de l'interface elle-même)
- **S** : Route statique (sera vue dans d'autres parties du cours)

#### 5. Tester la connectivité IPv6

```bash
Router# ping ipv6 <adresse-ipv6>
```

Exemple :

```bash
Router# ping ipv6 2001:db8:acad:2::2
```

> [!tip] Ping avec adresse link-local Pour pinguer une adresse link-local, vous devez spécifier l'interface de sortie :
> 
> ```bash
> Router# ping ipv6 fe80::2 source gigabitEthernet 0/0
> ```

#### 6. Afficher les voisins IPv6 (cache NDP)

```bash
Router# show ipv6 neighbors
```

> [!example] Sortie type
> 
> ```
> IPv6 Address                 Age Link-layer Addr State Interface
> FE80::2                       0  0011.2233.4455  REACH Gi0/0
> 2001:DB8:ACAD:1::2           15  0011.2233.4455  STALE Gi0/0
> ```

Cette commande est l'équivalent IPv6 de `show arp` en IPv4. Elle affiche :

- Adresses IPv6 des voisins
- Adresses MAC associées
- État de la résolution (REACH, STALE, DELAY, etc.)
- Interface concernée

### Tableau récapitulatif des commandes

|Commande|Usage|Informations clés|
|---|---|---|
|`show ipv6 interface brief`|Vue d'ensemble rapide|État et adresses de toutes les interfaces|
|`show ipv6 interface <if>`|Détails d'une interface|Configuration complète d'une interface|
|`show ipv6 route`|Table de routage|Routes IPv6 connues du routeur|
|`ping ipv6 <addr>`|Test de connectivité|Vérifier la joignabilité d'une adresse|
|`show ipv6 neighbors`|Cache NDP|Résolution adresse IPv6 ↔ MAC|
|`show running-config interface <if>`|Configuration active|Configuration appliquée sur l'interface|

> [!warning] Pièges courants
> 
> - ❌ Oublier d'activer `ipv6 unicast-routing` : le routeur ne routera pas les paquets IPv6
> - ❌ Oublier `no shutdown` : l'interface reste désactivée même avec une adresse IPv6
> - ❌ Utiliser un préfixe autre que /64 sur un segment LAN : peut casser SLAAC
> - ❌ Ne pas spécifier l'interface source lors du ping d'une link-local

> [!tip] Astuces de dépannage
> 
> 1. **Vérifiez toujours l'état physique** : `show ipv6 interface brief` doit afficher `[up/up]`
> 2. **Vérifiez le routage global** : `ipv6 unicast-routing` doit être activé
> 3. **Utilisez des adresses link-local simples** : `fe80::1`, `fe80::2` pour faciliter le dépannage
> 4. **Documentez vos adresses** : gardez un plan d'adressage IPv6 à jour

---

## 📌 Points clés à retenir

> [!info] Résumé de la configuration IPv6
> 
> 1. **Activer le routage IPv6 globalement** : `ipv6 unicast-routing`
> 2. **Configurer une adresse IPv6 globale** : `ipv6 address <adresse>/<préfixe>`
> 3. **Optionnel : définir une link-local manuelle** : `ipv6 address fe80::x link-local`
> 4. **Activer l'interface** : `no shutdown`
> 5. **Vérifier la configuration** : `show ipv6 interface brief`

### Workflow de configuration type

```bash
# Étape 1 : Activer IPv6 globalement
Router(config)# ipv6 unicast-routing

# Étape 2 : Configurer chaque interface
Router(config)# interface gigabitEthernet 0/0
Router(config-if)# ipv6 address 2001:db8:acad:1::1/64
Router(config-if)# ipv6 address fe80::1 link-local
Router(config-if)# no shutdown
Router(config-if)# exit

# Étape 3 : Vérifier
Router# show ipv6 interface brief
Router# show ipv6 route
Router# ping ipv6 <destination>
```

---

_Configuration IPv6 des interfaces routeur Cisco - Formation réseau_