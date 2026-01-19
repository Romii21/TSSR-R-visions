## Définition

Un **VLAN** est une segmentation **logique** d'un réseau physique qui permet de créer plusieurs réseaux locaux distincts sur la même infrastructure matérielle, sans nécessiter de câblage séparé.

**Acronyme :** VLAN = Virtual Local Area Network (Réseau Local Virtuel)

## Principe de fonctionnement

Les VLANs fonctionnent au **niveau de la couche 2** (liaison de données) du modèle OSI et permettent de séparer les domaines de diffusion (broadcast).

### Caractéristiques principales

|Aspect|Description|
|---|---|
|**Isolation**|Les hôtes de VLANs différents ne peuvent pas communiquer directement|
|**Domaine de broadcast**|Chaque VLAN constitue son propre domaine de diffusion|
|**Identification**|Chaque VLAN possède un ID unique (1 à 4094)|
|**Flexibilité**|Réorganisation sans modification physique du câblage|

## Configuration sur un switch

### Affectation des ports

Le switch associe ses ports physiques à des VLANs spécifiques :

```
Switch A :
- Ports 1-4  → VLAN 10 (Comptabilité)
- Ports 5-8  → VLAN 20 (Informatique)
- Ports 9-12 → VLAN 30 (Direction)
```

### Types de ports

|Type de port|Fonction|VLAN|
|---|---|---|
|**Access**|Connexion d'un hôte terminal|1 seul VLAN non tagué|
|**Trunk**|Liaison entre switches|Plusieurs VLANs tagués (802.1Q)|

## Le protocole 802.1Q

Norme **IEEE** permettant le transport de plusieurs VLANs sur une même liaison physique entre switches.

### En-tête 802.1Q

La trame Ethernet est modifiée avec l'ajout de **4 octets** après l'adresse MAC source :

|Champ|Taille|Description|
|---|---|---|
|**TPID**|2 octets|Tag Protocol ID = 0x8100 (identifie 802.1Q)|
|**PCP**|3 bits|Priority Code Point (QoS)|
|**DEI**|1 bit|Drop Eligible Indicator|
|**VID**|12 bits|VLAN ID (1-4094)|

**Important :** Le CRC/FCS est recalculé après ajout de l'en-tête.

## Communication inter-VLAN

Par défaut, les VLANs sont **isolés**. Pour communiquer entre VLANs différents :

### Solution 1 : Routeur externe

```
VLAN 10 ← → Switch ← → Routeur ← → Switch ← → VLAN 20
```

Le routeur effectue le routage entre les deux réseaux logiques.

### Solution 2 : Switch Layer 3

Un **switch L3** combine les fonctions de commutation (L2) et de routage (L3) :

- Gère les VLANs (fonction switch L2)
- Assure le routage inter-VLAN (fonction routeur L3)
- Plus performant qu'un routeur externe

## Avantages des VLANs

|Avantage|Explication|
|---|---|
|**Sécurité**|Isolation du trafic entre départements|
|**Performance**|Réduction des domaines de broadcast|
|**Flexibilité**|Modification sans recâblage physique|
|**Organisation**|Regroupement logique des utilisateurs|
|**Économie**|Utilisation optimale du matériel existant|

## Exemple pratique

Une entreprise avec 3 services sur un seul switch :

|Service|VLAN ID|Ports|Réseau IP|
|---|---|---|---|
|Comptabilité|10|1-8|192.168.10.0/24|
|RH|20|9-16|192.168.20.0/24|
|Direction|30|17-24|192.168.30.0/24|

- Les PC du VLAN 10 ne voient pas le trafic des VLAN 20 et 30
- Communication inter-VLAN possible via un routeur ou switch L3
- Sécurité accrue : isolation des données sensibles

## Points clés à retenir

- **VLAN = segmentation logique** sans modification physique
- **802.1Q** permet de transporter plusieurs VLANs sur un lien trunk
- **Isolation par défaut** : besoin d'un routeur/switch L3 pour communiquer entre VLANs
- **Améliore** la sécurité, les performances et la gestion du réseau