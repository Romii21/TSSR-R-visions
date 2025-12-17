# Synthèse - Le Routage IP

## Sommaire

1. [Introduction au routage IP](https://claude.ai/chat/e6884661-2c9e-4711-a2c3-65e760f22ded#1-introduction-au-routage-ip)
2. [Principes fondamentaux du routage](https://claude.ai/chat/e6884661-2c9e-4711-a2c3-65e760f22ded#2-principes-fondamentaux-du-routage)
    - [Communication sur un même réseau](https://claude.ai/chat/e6884661-2c9e-4711-a2c3-65e760f22ded#communication-sur-un-m%C3%AAme-r%C3%A9seau)
    - [Communication entre réseaux différents](https://claude.ai/chat/e6884661-2c9e-4711-a2c3-65e760f22ded#communication-entre-r%C3%A9seaux-diff%C3%A9rents)
3. [Les tables de routage](https://claude.ai/chat/e6884661-2c9e-4711-a2c3-65e760f22ded#3-les-tables-de-routage)
    - [Structure d'une table de routage](https://claude.ai/chat/e6884661-2c9e-4711-a2c3-65e760f22ded#structure-dune-table-de-routage)
    - [Optimisation avec les préfixes de routage](https://claude.ai/chat/e6884661-2c9e-4711-a2c3-65e760f22ded#optimisation-avec-les-pr%C3%A9fixes-de-routage)
    - [Manipulation des tables de routage](https://claude.ai/chat/e6884661-2c9e-4711-a2c3-65e760f22ded#manipulation-des-tables-de-routage)
4. [Configuration d'un routeur Linux](https://claude.ai/chat/e6884661-2c9e-4711-a2c3-65e760f22ded#4-configuration-dun-routeur-linux)
5. [Routage dynamique](https://claude.ai/chat/e6884661-2c9e-4711-a2c3-65e760f22ded#5-routage-dynamique)
    - [Protocoles de routage](https://claude.ai/chat/e6884661-2c9e-4711-a2c3-65e760f22ded#protocoles-de-routage)
6. [Les protocoles de transport](https://claude.ai/chat/e6884661-2c9e-4711-a2c3-65e760f22ded#6-les-protocoles-de-transport)
    - [UDP (User Datagram Protocol)](https://claude.ai/chat/e6884661-2c9e-4711-a2c3-65e760f22ded#udp-user-datagram-protocol)
    - [TCP (Transmission Control Protocol)](https://claude.ai/chat/e6884661-2c9e-4711-a2c3-65e760f22ded#tcp-transmission-control-protocol)
7. [Résumé des concepts clés](https://claude.ai/chat/e6884661-2c9e-4711-a2c3-65e760f22ded#7-r%C3%A9sum%C3%A9-des-concepts-cl%C3%A9s)

---

## 1. Introduction au routage IP

Le **protocole IP** (Internet Protocol) permet l'interconnexion de réseaux. Son objectif principal est de faire communiquer des machines situées sur des **réseaux logiques différents**.

**Principe de base :** Les nœuds d'un même réseau IP doivent être sur le même lien physique. Pour communiquer entre réseaux différents, on utilise :

- Des **routeurs** (passerelles/gateways)
- Une technique : le **routage**

---

## 2. Principes fondamentaux du routage

### Communication sur un même réseau

Lorsqu'une machine veut envoyer un paquet à une autre machine **sur le même réseau** :

1. La machine calcule les réseaux de toutes ses interfaces
2. Elle vérifie si la destination appartient à l'un de ces réseaux
3. Si oui → envoi **direct** via la couche liaison (ex: Ethernet)

**Exemple pratique :**

```
Machine 11 : 192.168.0.11/24
Destination : 192.168.0.42/24

Réseau calculé : 192.168.0.0/24
→ Les deux sont sur le même réseau → Envoi direct
```

**Processus d'envoi :**

- Encapsulation du paquet IP dans une trame Ethernet
- Résolution de l'adresse MAC via **ARP** (IPv4) ou **NDP** (IPv6)
- Envoi de la trame avec l'adresse MAC de destination

### Communication entre réseaux différents

Lorsque la destination **n'est sur aucun réseau** de l'émetteur :

1. L'émetteur consulte sa **table de routage**
2. Il trouve l'adresse du **routeur** (next hop) approprié
3. Il envoie le paquet au routeur (qui est sur le même réseau que lui)

**Point crucial :**

- Le paquet IP conserve l'adresse de destination **finale**
- Mais la trame Ethernet est adressée au **routeur**

**Schéma conceptuel :**

```
[Machine A] ---réseau N0---> [Routeur] ---réseau N1---> [Machine B]
192.168.0.11              192.168.0.1                 192.168.1.42
                          192.168.1.1
```

---

## 3. Les tables de routage

### Structure d'une table de routage

Chaque entrée contient **au minimum** :

- **Destination** : adresse de réseau + masque (ex: 192.168.1.0/24)
- **Next hop** : adresse IP de la passerelle
- **Interface** : interface réseau à utiliser (optionnel)
- **Métrique** : qualité de la route (plus petit = meilleur)

**Important en IPv6 :** On utilise les **adresses link-local** des routeurs (fe80::...)

### Optimisation avec les préfixes de routage

Au lieu de stocker une route pour chaque adresse IP (2³² en IPv4, 2¹²⁸ en IPv6), on utilise des **adresses de réseau**.

**Principe d'agrégation :** On regroupe plusieurs réseaux accessibles via le même routeur en un seul **préfixe de routage**.

**Exemple d'agrégation :**

```
Réseaux via R1 :
- 192.168.192.0/20
- 192.168.208.0/22

Réseaux via R2 :
- 192.168.0.0/24
- 192.168.5.0/24
- 192.168.12.0/24

Table optimisée :
- 192.168.128.0/17 via R1  (3ème octet ≥ 128)
- 192.168.0.0/17 via R2    (3ème octet < 128)
```

### Manipulation des tables de routage

**Sous Linux (commande `ip route`) :**

```bash
# Afficher la table IPv4
ip route

# Afficher la table IPv6
ip -6 route

# Ajouter une route
ip route add 192.168.128.0/17 via 10.0.0.2

# Ajouter une passerelle par défaut
ip route add default via 10.0.0.254
```

**Sous Windows (commande `route`) :**

```cmd
# Afficher les tables
route print

# Ajouter une route
route add 192.168.128.0/17 10.0.0.2

# Supprimer une route
route delete 192.168.128.0/17
```

---

## 4. Configuration d'un routeur Linux

Par défaut, Linux est configuré en **hôte** (il jette les paquets qui ne lui sont pas destinés).

Pour activer le **routage**, modifier les paramètres système :

```bash
# Activer le routage IPv4 (temporaire)
sysctl -w net.ipv4.ip_forward=1

# Activer le routage IPv6 (temporaire)
sysctl -w net.ipv6.conf.all.forwarding=1

# Vérifier l'état
sysctl net.ipv4.ip_forward
sysctl net.ipv6.conf.all.forwarding
```

**Configuration persistante :**

- Éditer `/etc/sysctl.conf`
- Ajouter : `net.ipv4.ip_forward=1`
- Recharger : `sysctl -p /etc/sysctl.conf`

---

## 5. Routage dynamique

Le **routage statique** (tables configurées manuellement) convient aux petits réseaux stables. Pour les grands réseaux, on utilise le **routage dynamique**.

**Avantages du routage dynamique :**

- Mise à jour **automatique** des tables de routage
- Basculement vers des liaisons de secours en cas de panne
- Adaptation aux changements de topologie

### Protocoles de routage

|Protocole|Usage|Caractéristiques|
|---|---|---|
|**RIP**|Petits réseaux|Simple, max 15 sauts, UDP port 520|
|**OSPF**|Réseaux moyens/grands|Rapide, supporte VLSM, protocole 89|
|**EIGRP**|Réseaux complexes|Propriétaire Cisco, convergence rapide, protocole 88|
|**BGP**|Internet|Routage entre systèmes autonomes, TCP port 179|

**Note :** RIP existe en 3 versions : RIPv1, RIPv2 (IPv4) et RIPng (IPv6)

---

## 6. Les protocoles de transport

La couche transport (couche 4 OSI) permet la **communication entre processus** via les **ports**.

### UDP (User Datagram Protocol)

**Caractéristiques :**

- Protocole **léger** et **simple**
- Numéro de protocole IP : **17**
- **Non fiable** : pas de garantie de livraison
- Défini dans RFC 768 (1980)

**Structure de l'en-tête UDP :**

```
┌─────────────────┬─────────────────┐
│  Port source    │  Port dest      │  (16 bits chacun)
├─────────────────┼─────────────────┤
│  Longueur       │  Checksum       │  (16 bits chacun)
├─────────────────────────────────┤
│         Données (PDU)             │
└───────────────────────────────────┘
```

**Services rendus :**

- Communication entre processus (ports)
- Détection d'erreurs (checksum)
- **Pas de garantie** de livraison

**Cas d'usage :** DNS, streaming, diffusion d'informations

### TCP (Transmission Control Protocol)

**Caractéristiques :**

- Protocole **fiable** en **mode connecté**
- Numéro de protocole IP : **6**
- Garantit l'ordre et la livraison des données
- Défini dans RFC 793

**Établissement de connexion (Three-Way Handshake) :**

```
Client                          Serveur
  │                                │
  │  1. SYN, Seq=x                │
  │───────────────────────────────>│
  │                                │
  │  2. SYN+ACK, Seq=y, Ack=x+1   │
  │<───────────────────────────────│
  │                                │
  │  3. ACK, Seq=x+1, Ack=y+1     │
  │───────────────────────────────>│
  │                                │
  │    CONNEXION ÉTABLIE           │
```

**Structure de l'en-tête TCP (simplifié) :**

- Port source / destination (16 bits chacun)
- Numéro de séquence (32 bits)
- Numéro d'acquittement (32 bits)
- **Flags de contrôle** : SYN, ACK, FIN, RST, PSH, URG
- Taille de fenêtre (16 bits) : contrôle de flux
- Checksum (16 bits)

**Mécanisme de fiabilité :**

1. Chaque segment reçoit un **numéro de séquence**
2. Le destinataire envoie un **acquittement** (ACK)
3. Si pas d'ACK → **retransmission** après timeout
4. La **fenêtre TCP** permet d'envoyer plusieurs segments sans attendre l'ACK

**Services rendus :**

- Communication fiable entre processus
- Gestion des retransmissions
- Maintien de l'ordre des segments
- Optimisation de la bande passante
- Gestion de la congestion

**Inconvénient :** Protocole lourd, non adapté aux systèmes embarqués très contraints

---

## 7. Résumé des concepts clés

### Les ports

Les ports identifient les **processus** sur une machine :

|Plage|Type|Usage|
|---|---|---|
|**0-1023**|Ports système (Well Known)|Serveurs standards|
|**1024-49151**|Ports enregistrés (Registered)|Serveurs utilisateurs|
|**49152-65535**|Ports dynamiques (Ephemeral)|Clients|

**Exemples de ports standards :**

- DNS : port 53 (UDP/TCP)
- HTTP : port 80 (TCP)
- HTTPS : port 443 (TCP)

### Points essentiels à retenir

1. **Routage :** Permet la communication entre réseaux via des routeurs
2. **Table de routage :** Associe destinations et next hops
3. **Passerelle par défaut :** Route utilisée quand aucune autre ne correspond
4. **UDP :** Simple et rapide, mais non fiable
5. **TCP :** Fiable et ordonné, mais plus lourd
6. **Routage dynamique :** Indispensable pour les grands réseaux

---

## Sources

- **Document fourni :** Le routage IP.pdf (cours complet)
- RFC 768 - User Datagram Protocol (UDP)
- RFC 793 - Transmission Control Protocol (TCP)
- RFC 1918 - Adresses IPv4 privées
- RFC 4193 - Adresses IPv6 locales uniques
- RFC 7323 - Gestion de la fenêtre TCP