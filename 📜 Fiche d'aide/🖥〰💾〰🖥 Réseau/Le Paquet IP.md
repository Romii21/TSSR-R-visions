## Définition

Un **paquet** est l'unité de données de la **couche 3 (réseau)** du modèle OSI. C'est le conteneur qui transporte les informations à travers différents réseaux interconnectés.

**Point clé** : Le paquet permet de faire communiquer des machines sur des réseaux physiques différents grâce au routage.

---

## Structure générale

Un paquet IP se compose de deux parties :

|Partie|Contenu|Rôle|
|---|---|---|
|**En-tête (header)**|Informations de contrôle|Routage, gestion, intégrité|
|**Charge utile (payload)**|Données à transmettre|Contient le segment de la couche supérieure (TCP/UDP)|

---

## IPv4 vs IPv6

### Taille de l'en-tête

|Version|Taille en-tête|Particularité|
|---|---|---|
|**IPv4**|20 à 60 octets (variable)|Options intégrées dans l'en-tête|
|**IPv6**|40 octets (fixe)|En-têtes d'extension séparés|

### Champs principaux

#### **IPv4**

|Champ|Taille|Description|
|---|---|---|
|**Version**|4 bits|Toujours "4"|
|**IHL** (Internet Header Length)|4 bits|Longueur de l'en-tête en mots de 32 bits (5 à 15)|
|**ToS/DSCP**|8 bits|Qualité de service (QoS)|
|**Total Length**|16 bits|Taille totale du paquet (max 65 535 octets)|
|**TTL** (Time To Live)|8 bits|Durée de vie, décrémenté à chaque routeur|
|**Protocol**|8 bits|Protocole encapsulé (6=TCP, 17=UDP, 1=ICMP)|
|**Source/Destination**|32 bits × 2|Adresses IPv4|
|**Header Checksum**|16 bits|Vérification d'intégrité de l'en-tête|

#### **IPv6**

|Champ|Taille|Description|
|---|---|---|
|**Version**|4 bits|Toujours "6"|
|**Traffic Class**|8 bits|QoS (équivalent ToS)|
|**Flow Label**|20 bits|Étiquette de flux pour traitement spécial|
|**Payload Length**|16 bits|Taille de la charge utile uniquement|
|**Next Header**|8 bits|Protocole suivant (équivalent Protocol)|
|**Hop Limit**|8 bits|Équivalent du TTL|
|**Source/Destination**|128 bits × 2|Adresses IPv6|

**Simplifications IPv6** : Pas de checksum, pas de fragmentation par les routeurs, traitement plus rapide.

---

## Fragmentation

### IPv4

**Principe** : Si un paquet est trop grand pour le lien suivant (> MTU), il est découpé.

|Champ|Rôle|
|---|---|
|**Identification**|ID commun à tous les fragments|
|**Flags**|DF (Don't Fragment), MF (More Fragments)|
|**Fragment Offset**|Position du fragment dans le paquet original|

**Réassemblage** : Uniquement au destinataire final.

**Exemple** : Paquet de 3000 octets, MTU 1500 → découpage en 2 fragments avec ajustement des champs.

### IPv6

**Principe** : Éviter la fragmentation !

- Utilise **PMTUD** (Path MTU Discovery)
- Si un routeur doit fragmenter : il **rejette le paquet** et prévient l'émetteur via ICMPv6
- L'émetteur ajuste la taille des paquets suivants

---

## TTL / Hop Limit

**Rôle** : Éviter les boucles infinies dans le réseau.

- Valeur initiale : généralement 64 ou 128
- Décrémenté à chaque passage par un routeur
- Si atteint 0 : paquet détruit, message ICMP "Time Exceeded" envoyé

**Utilisation pratique** : Outil `traceroute` pour découvrir le chemin réseau.

---

## Protocoles encapsulés

Le champ **Protocol** (IPv4) ou **Next Header** (IPv6) indique le type de données transportées :

| Valeur | Protocole | Usage                                    |
| ------ | --------- | ---------------------------------------- |
| 1      | ICMP      | Messages de contrôle et erreurs          |
| 6      | TCP       | Transport fiable                         |
| 17     | UDP       | Transport rapide                         |
| 58     | ICMPv6    | Contrôle IPv6 (inclut NDP, remplace ARP) |

---

## Points essentiels

✅ Le paquet permet l'**interconnexion de réseaux** différents  
✅ L'en-tête contient les **adresses IP source et destination**  
✅ Le **TTL/Hop Limit** empêche les boucles infinites  
✅ IPv4 : fragmentation possible par les routeurs  
✅ IPv6 : fragmentation évitée, en-tête simplifié et fixe  
✅ Le paquet encapsule un **segment** (TCP/UDP) ou **datagramme**