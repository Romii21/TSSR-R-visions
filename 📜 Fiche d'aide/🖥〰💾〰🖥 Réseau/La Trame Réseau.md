## Définition

Une **trame** est le PDU (Protocol Data Unit - unité de données de protocole) de la **couche 2 (Liaison)** du modèle OSI. C'est le conteneur qui encapsule les données pour les transmettre sur un réseau physique local.

## La Trame Ethernet

Ethernet est la norme IEEE 802.3 la plus utilisée pour les réseaux locaux filaires. La trame Ethernet a une taille comprise entre **64 et 1518 octets**.

### Structure complète

|Composant|Taille|Description|
|---|---|---|
|**Préambule**|7 octets|Séquence de synchronisation (0xAA répété)|
|**SFD**|1 octet|Start Frame Delimiter (0xAB) - signale le début de la trame|
|**MAC Destination**|6 octets|Adresse physique du destinataire|
|**MAC Source**|6 octets|Adresse physique de l'émetteur|
|**EtherType**|2 octets|Type de protocole encapsulé (≥1536) ou longueur (≤1500)|
|**Payload**|46-1500 octets|Données utiles (paquet IP, etc.)|
|**FCS/CRC**|4 octets|Frame Check Sequence - détection d'erreurs|
|**Gap inter-trame**|96 bits|Silence de 96 bits entre deux trames|

### Le champ EtherType

L'EtherType indique le protocole encapsulé dans la trame :

|Valeur|Protocole|
|---|---|
|0x0800|IPv4|
|0x0806|ARP|
|0x86DD|IPv6|
|0x8100|VLAN (802.1Q)|

**Règle** : Si la valeur est ≤1500, c'est une longueur. Si ≥1536, c'est un type de protocole.

## Adresses MAC

Une **adresse MAC** (Media Access Control) est l'identifiant physique unique d'une interface réseau.

- **Format** : 48 bits (6 octets) en hexadécimal
- **Exemple** : `D4:93:90:05:2C:1C`
- **Structure** : OUI (3 octets, constructeur) + Suffixe (3 octets, unique)

### Bits spéciaux

|Bit|Nom|Signification|
|---|---|---|
|7ème bit|U/L|0 = Universelle (IEEE), 1 = Locale|
|8ème bit|I/G|0 = Individuelle (unicast), 1 = Groupe (multicast)|

**Adresse broadcast** : `FF:FF:FF:FF:FF:FF` (envoi à tous)

## Le Payload

Le payload contient les données de la couche supérieure (couche 3 - Réseau), généralement un **paquet IP**.

- **Taille** : 46 à 1500 octets
- **MTU** (Maximum Transmission Unit) : 1500 octets maximum
- Si les données font moins de 46 octets, un **padding** (remplissage de zéros) est ajouté

### Contenu typique du payload

Le payload encapsule l'**en-tête IP** qui contient :

- Adresse IP source
- Adresse IP destination
- Données de la couche transport (TCP/UDP)

## Contrôle d'erreurs

Le **FCS** (Frame Check Sequence) ou **CRC** (Cyclic Redundancy Check) permet de détecter les erreurs de transmission.

- Si une erreur est détectée → la trame est **détruite** (best effort)
- Pas de retransmission au niveau Ethernet
- La fiabilité doit être gérée par les couches supérieures (TCP)

## VLAN et Trame 802.1Q

Pour segmenter un réseau, on peut ajouter un **tag VLAN** à la trame.

### En-tête 802.1Q (4 octets)

Inséré après les adresses MAC, avant l'EtherType original :

|Champ|Taille|Description|
|---|---|---|
|**TPID**|2 octets|Tag Protocol Identifier (0x8100)|
|**TCI**|2 octets|Tag Control Information|

Le TCI contient :

- **PCP** (3 bits) : Priority Code Point (QoS)
- **DEI** (1 bit) : Drop Eligible Indicator
- **VLAN ID** (12 bits) : Identifiant du VLAN (0-4095)

**Important** : Le CRC doit être recalculé après ajout du tag VLAN.

## Points clés à retenir

- La trame est l'unité de transmission de la **couche 2**
- Elle fonctionne uniquement sur un **réseau local** (même segment)
- L'adresse MAC identifie l'équipement **physiquement**
- L'EtherType indique quel protocole est encapsulé
- Le CRC détecte les erreurs mais ne les corrige pas
- Les VLANs permettent de segmenter logiquement un réseau physique