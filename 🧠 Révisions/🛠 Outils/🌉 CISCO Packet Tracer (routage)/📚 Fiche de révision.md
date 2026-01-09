# Guide de révision - Cisco Packet Tracer

## Sommaire

1. [Introduction à Packet Tracer](#introduction)
2. [Concepts de base du réseau](https://claude.ai/chat/4148f4d3-cff3-4870-8114-a1adb7cfbf99#concepts-base)
3. [Configuration des PC](https://claude.ai/chat/4148f4d3-cff3-4870-8114-a1adb7cfbf99#config-pc)
4. [Configuration des routeurs](https://claude.ai/chat/4148f4d3-cff3-4870-8114-a1adb7cfbf99#config-routeurs)
5. [Routage statique](https://claude.ai/chat/4148f4d3-cff3-4870-8114-a1adb7cfbf99#routage-statique)
6. [Commandes essentielles](https://claude.ai/chat/4148f4d3-cff3-4870-8114-a1adb7cfbf99#commandes-essentielles)
7. [Dépannage](https://claude.ai/chat/4148f4d3-cff3-4870-8114-a1adb7cfbf99#depannage)
8. [Plan d'adressage type](https://claude.ai/chat/4148f4d3-cff3-4870-8114-a1adb7cfbf99#plan-adressage)

---

## <a id="introduction"></a>1. Introduction à Packet Tracer

**Packet Tracer** est un simulateur de réseau Cisco permettant de créer et tester des topologies réseau virtuelles.

### Objectifs principaux

- Comprendre les réseaux LAN et WAN
- Configurer des équipements réseau (routeurs, switches, PC)
- Mettre en place du routage statique IPv4 et IPv6
- Tester la connectivité avec ICMP (ping)

---

## <a id="concepts-base"></a>2. Concepts de base du réseau

### Types de réseaux

- **LAN (Local Area Network)** : Réseau local (ex: 192.168.1.0/24)
- **WAN (Wide Area Network)** : Réseau étendu interconnectant plusieurs LAN

### Composants réseau

- **Switch (commutateur)** : Connecte les machines dans un LAN
- **Routeur** : Interconnecte différents réseaux et route les paquets
- **PC** : Ordinateurs configurés avec IP, masque et passerelle

### Table de routage

Un routeur connaît les réseaux de trois façons :

- **C (Connected)** : Réseaux directement connectés
- **L (Local)** : Adresse de l'interface du routeur
- **S (Static)** : Routes configurées manuellement
- **O (OSPF), R (RIP)** : Routes apprises dynamiquement

### Passerelle (Gateway)

La passerelle est l'adresse IP du routeur que les PC utilisent pour communiquer en dehors de leur réseau local.

---

## <a id="config-pc"></a>3. Configuration des PC

### Configuration IPv4

1. Cliquer sur le PC
2. Aller dans **Desktop > IP Configuration**
3. Configurer :
    - **IP Address** : ex. 192.168.1.10
    - **Subnet Mask** : ex. 255.255.255.0
    - **Default Gateway** : ex. 192.168.1.1 (adresse du routeur)

### Configuration IPv6

1. Même chemin : **Desktop > IP Configuration**
2. Configurer :
    - **IPv6 Address** : ex. 2001:db8:cafe:cafe::10/64
    - **IPv6 Gateway** : ex. 2001:db8:cafe:cafe::1

---

## <a id="config-routeurs"></a>4. Configuration des routeurs

### Accès au routeur

Cliquer sur le routeur → Onglet **CLI**

### Configuration de base

```cisco
Router> enable                          # Passe en mode privilégié
Router# configure terminal              # Entre en mode configuration
Router(config)# hostname R1             # Nomme le routeur
R1(config)# ipv6 unicast-routing       # Active le routage IPv6
```

### Configuration d'une interface

```cisco
R1(config)# interface gigabitEthernet 0/0
R1(config-if)# description Gateway_LAN
R1(config-if)# ip address 192.168.1.1 255.255.255.0
R1(config-if)# ipv6 enable
R1(config-if)# ipv6 address 2001:db8:cafe:cafe::1/64
R1(config-if)# no shutdown              # Active l'interface
R1(config-if)# exit
```

⚠️ **Important** : Les interfaces Cisco sont désactivées par défaut, toujours faire `no shutdown` !

### Sauvegarder la configuration

```cisco
R1(config)# do write memory            # Ou "do wr"
# Alternative :
R1# copy running-config startup-config
```

---

## <a id="routage-statique"></a>5. Routage statique

### Principe

Par défaut, un routeur ne connaît **que les réseaux directement connectés**. Pour joindre des réseaux distants, il faut configurer des routes statiques.

### Syntaxe IPv4

```cisco
ip route [réseau_destination] [masque] [passerelle_next_hop]
```

**Exemple** : Pour que R1 joigne le réseau 192.168.2.0/24 via le routeur Internet (198.51.100.2) :

```cisco
R1(config)# ip route 192.168.2.0 255.255.255.0 198.51.100.2
```

### Syntaxe IPv6

```cisco
ipv6 route [réseau_destination/préfixe] [passerelle_next_hop]
```

**Exemple** :

```cisco
R1(config)# ipv6 route 2001:db8:babe:babe::/64 2001:db8:cafe:feed::2
```

### Erreurs courantes

#### "Invalid next hop address (it's this router)"

Tu essaies d'utiliser une adresse IP du routeur lui-même comme passerelle. Utilise l'adresse du **routeur suivant** (next hop).

#### "Overlaps with interface X"

Deux interfaces du même routeur ne peuvent pas avoir le même sous-réseau. Change l'un des deux réseaux.

---

## <a id="commandes-essentielles"></a>6. Commandes essentielles

### Consultation

|Commande|Description|
|---|---|
|`show ip interface brief` ou `sh ip int br`|Affiche toutes les interfaces et leurs IP (IPv4)|
|`show ipv6 interface brief` ou `sh ipv6 int br`|Affiche toutes les interfaces IPv6|
|`show ip route` ou `sh ip route`|Affiche la table de routage IPv4|
|`show ipv6 route` ou `sh ipv6 route`|Affiche la table de routage IPv6|
|`show running-config` ou `sh run`|Affiche toute la configuration active|
|`show running-config interface gi0/0`|Affiche la config d'une interface spécifique|

### Navigation

|Commande|Description|
|---|---|
|`enable` ou `en`|Mode privilégié|
|`configure terminal` ou `conf t`|Mode configuration globale|
|`interface gigabitEthernet 0/0` ou `int gi0/0`|Configure une interface|
|`exit`|Revient au niveau précédent|
|`end`|Revient au mode privilégié depuis n'importe où|
|`do [commande]`|Exécute une commande du mode privilégié depuis le mode config|

### Configuration

|Commande|Description|
|---|---|
|`hostname [nom]`|Change le nom du routeur|
|`ip address [ip] [masque]`|Configure une IP sur une interface|
|`ipv6 address [ipv6/préfixe]`|Configure une IPv6 sur une interface|
|`ipv6 enable`|Active IPv6 sur l'interface|
|`ipv6 unicast-routing`|Active le routage IPv6 (global)|
|`no shutdown` ou `no sh`|Active une interface|
|`description [texte]`|Ajoute une description à l'interface|

### Suppression

|Commande|Description|
|---|---|
|`no ip address [ip] [masque]`|Supprime une adresse IPv4|
|`no ipv6 address [ipv6/préfixe]`|Supprime une adresse IPv6|
|`no ipv6 address`|Supprime toutes les IPv6 de l'interface|
|`no ip route [réseau] [masque] [gateway]`|Supprime une route statique IPv4|

### Tests de connectivité

|Commande|Description|
|---|---|
|`ping [ip ou hostname]`|Test de connectivité ICMP|
|`ping [ipv6]`|Test de connectivité IPv6|
|`traceroute [ip]`|Trace le chemin vers une destination|

---

## <a id="depannage"></a>7. Dépannage

### Méthodologie

1. **Vérifier la configuration physique**
    
    - Les câbles sont-ils connectés ?
    - Les interfaces sont-elles activées (`no shutdown`) ?
    - Les voyants sont-ils verts ?
2. **Vérifier les adresses IP**
    
    ```cisco
    show ip interface brief
    show ipv6 interface brief
    ```
    
3. **Vérifier les tables de routage**
    
    ```cisco
    show ip route
    show ipv6 route
    ```
    
    - Le réseau de destination apparaît-il ?
    - Quelle est la passerelle (next hop) ?
4. **Tester la connectivité étape par étape**
    
    - Ping vers la passerelle locale
    - Ping vers l'interface WAN du routeur suivant
    - Ping vers la destination finale

### Problèmes fréquents

|Symptôme|Cause probable|Solution|
|---|---|---|
|"Destination host unreachable"|Pas de route vers la destination|Ajouter une route statique|
|"Request timed out"|Interface down ou problème physique|Vérifier `no shutdown` et câblage|
|Interface "administratively down"|Interface désactivée|Faire `no shutdown`|
|"Invalid next hop address"|Next hop = adresse du routeur lui-même|Utiliser l'adresse du routeur suivant|
|"Overlaps with interface X"|Même sous-réseau sur 2 interfaces|Changer l'un des deux réseaux|

---

## <a id="plan-adressage"></a>8. Plan d'adressage type

### Exemple de topologie 3 sites

#### Site 1 (Réseau N°1)

|Machine|IPv4|Masque|Passerelle|IPv6|
|---|---|---|---|---|
|PC0|192.168.1.10|/24|192.168.1.1|2001:db8:cafe:cafe::10/64|
|PC1|192.168.1.11|/24|192.168.1.1|2001:db8:cafe:cafe::11/64|
|R1 (LAN)|192.168.1.1|/24|-|2001:db8:cafe:cafe::1/64|
|R1 (WAN)|198.51.100.1|/30|-|2001:db8:cafe:feed::1/64|

#### Site 2 (Réseau N°2)

|Machine|IPv4|Masque|Passerelle|IPv6|
|---|---|---|---|---|
|PC2|192.168.2.10|/24|192.168.2.1|2001:db8:babe:babe::10/64|
|PC3|192.168.2.11|/24|192.168.2.1|2001:db8:babe:babe::11/64|
|R2 (LAN)|192.168.2.1|/24|-|2001:db8:babe:babe::1/64|
|R2 (WAN)|198.51.100.5|/30|-|2001:db8:babe:feed::1/64|

#### Site 3 (Réseau N°3)

|Machine|IPv4|Masque|Passerelle|IPv6|
|---|---|---|---|---|
|PC4|192.168.3.10|/24|192.168.3.1|2001:db8:face:face::10/64|
|PC5|192.168.3.11|/24|192.168.3.1|2001:db8:face:face::11/64|
|R3 (LAN)|192.168.3.1|/24|-|2001:db8:face:face::1/64|
|R3 (WAN)|198.51.100.9|/30|-|2001:db8:face:feed::1/64|

#### Routeur Internet

|Interface|IPv4|Masque|IPv6|
|---|---|---|---|
|Gi0/0 (vers R1)|198.51.100.2|/30|2001:db8:cafe:feed::2/64|
|Gi0/1 (vers R2)|198.51.100.6|/30|2001:db8:babe:feed::2/64|
|Gi0/2 (vers R3)|198.51.100.10|/30|2001:db8:face:feed::2/64|

### Routes statiques nécessaires

#### Sur R1

```cisco
ip route 192.168.2.0 255.255.255.0 198.51.100.2
ip route 192.168.3.0 255.255.255.0 198.51.100.2
ipv6 route 2001:db8:babe:babe::/64 2001:db8:cafe:feed::2
ipv6 route 2001:db8:face:face::/64 2001:db8:cafe:feed::2
```

#### Sur R2

```cisco
ip route 192.168.1.0 255.255.255.0 198.51.100.6
ip route 192.168.3.0 255.255.255.0 198.51.100.6
ipv6 route 2001:db8:cafe:cafe::/64 2001:db8:babe:feed::2
ipv6 route 2001:db8:face:face::/64 2001:db8:babe:feed::2
```

#### Sur R3

```cisco
ip route 192.168.1.0 255.255.255.0 198.51.100.10
ip route 192.168.2.0 255.255.255.0 198.51.100.10
ipv6 route 2001:db8:cafe:cafe::/64 2001:db8:face:feed::2
ipv6 route 2001:db8:babe:babe::/64 2001:db8:face:feed::2
```

#### Sur Routeur Internet

```cisco
ip route 192.168.1.0 255.255.255.0 198.51.100.1
ip route 192.168.2.0 255.255.255.0 198.51.100.5
ip route 192.168.3.0 255.255.255.0 198.51.100.9
ipv6 route 2001:db8:cafe:cafe::/64 2001:db8:cafe:feed::1
ipv6 route 2001:db8:babe:babe::/64 2001:db8:babe:feed::1
ipv6 route 2001:db8:face:face::/64 2001:db8:face:feed::1
```

---

## Points clés à retenir 🔑

1. **Toujours activer les interfaces** avec `no shutdown`
2. **Activer le routage IPv6** avec `ipv6 unicast-routing` (global)
3. **Sauvegarder** avec `do write memory` ou `copy run start`
4. **Le next hop** doit être l'adresse IP du **routeur suivant**, pas du routeur actuel
5. **Vérifier les tables de routage** avant de tester
6. **Un routeur ne connaît que** :
    - Les réseaux directement connectés (C)
    - Les routes configurées manuellement (S)
    - Les routes apprises dynamiquement (O, R, etc.)

---

## Sources

- Document de cours : "Atelier Packet Tracer - Routage statique IPv4/IPv6"
- Captures d'écran et configurations fournies
- Documentation Cisco IOS pour les commandes de routage statique