## Définition

Le **trunk** (ou liaison agrégée) est une connexion réseau qui **transporte le trafic de plusieurs VLANs simultanément** entre deux équipements réseau (généralement entre switches ou entre un switch et un routeur).

Sans trunk, il faudrait un câble physique distinct par VLAN entre les équipements.

## Le Protocole 802.1Q

**IEEE 802.1Q** est le standard qui définit comment encapsuler les trames Ethernet pour identifier leur VLAN d'appartenance.

### Fonctionnement

Lorsqu'une trame traverse un port trunk, le switch **ajoute un tag 802.1Q** de 4 octets dans l'en-tête Ethernet :

| Champ         | Taille   | Description                                                     |
| ------------- | -------- | --------------------------------------------------------------- |
| **TPID**      | 2 octets | Tag Protocol Identifier = `0x8100` (identifie une trame 802.1Q) |
| **TCI**       | 2 octets | Tag Control Information, contient :                             |
| **PCP**       | 3 bits   | Priority Code Point (QoS selon 802.1p)                          |
| **DEI**       | 1 bit    | Drop Eligible Indicator                                         |
| - **VLAN ID** | 12 bits  | Identifiant du VLAN (1 à 4094)                                  |

### Processus d'encapsulation

```
Trame entrante (VLAN 10) → Switch A (port access) 
                           ↓
                   Ajout du tag 802.1Q
                           ↓
Switch A (port trunk) → [Trame + Tag VLAN 10] → Switch B (port trunk)
                           ↓
                   Retrait du tag 802.1Q
                           ↓
Switch B (port access) → Trame sortante (VLAN 10)
```

**Points importants** :

- Le tag est **ajouté** à l'entrée du trunk
- Le tag est **retiré** à la sortie du trunk
- Le **CRC est recalculé** après ajout/retrait du tag

## Native VLAN

Le **Native VLAN** (VLAN natif) est un VLAN spécial sur un port trunk dont les trames **ne sont PAS taguées**.

|Caractéristique|Description|
|---|---|
|**Par défaut**|VLAN 1|
|**Trafic**|Non tagué sur le trunk|
|**Usage**|Compatibilité avec équipements non-VLAN, gestion du switch|
|**Sécurité**|Doit être identique aux deux extrémités du trunk|

**Recommandation de sécurité** : Changer le Native VLAN par défaut et ne pas l'utiliser pour du trafic utilisateur.

## Types de Ports

|Type de port|Description|Usage|Tagging|
|---|---|---|---|
|**Access**|Appartient à un seul VLAN|Connexion d'équipements finaux (PC, imprimante)|Non tagué|
|**Trunk**|Transporte plusieurs VLANs|Liaison inter-switches, vers routeurs|Tagué (sauf Native VLAN)|

## Cas d'Usage

### 1. Interconnexion de switches

```
Switch A (VLAN 10, 20, 30) ←[TRUNK]→ Switch B (VLAN 10, 20, 30)
```

Les trois VLANs circulent sur un seul câble entre les deux switches.

### 2. Routage inter-VLAN (Router-on-a-Stick)

```
Switch ←[TRUNK]→ Routeur
```

Le routeur reçoit le trafic tagué de tous les VLANs et peut router entre eux.

## Acronymes Récapitulatifs

- **VLAN** : Virtual Local Area Network (réseau local virtuel)
- **TPID** : Tag Protocol Identifier (identifiant du protocole de tag)
- **TCI** : Tag Control Information (information de contrôle du tag)
- **PCP** : Priority Code Point (priorité QoS)
- **DEI** : Drop Eligible Indicator (indicateur d'éligibilité au rejet)
- **QoS** : Quality of Service (qualité de service)

## Points Clés

✅ Le trunk permet de transporter **plusieurs VLANs sur un seul lien physique**  
✅ Le standard **802.1Q** ajoute un tag de 4 octets contenant le VLAN ID  
✅ Le **Native VLAN** circule sans tag (par défaut VLAN 1)  
✅ Les ports **access** sont pour les équipements finaux, les ports **trunk** pour les liaisons entre équipements réseau  
✅ Le tag est **transparent** pour les équipements finaux (ajouté/retiré par le switch)