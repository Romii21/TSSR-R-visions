# Synthèse sur Ethernet

Cette synthèse est basée sur les notes fournies et le cours du PDF "Ethernet.pdf". Elle est organisée en chapitres pour une compréhension simple, en utilisant les termes du cours. J'ai ajouté des schémas graphiques en ASCII ou Markdown pour illustrer les points clés, et un schéma récapitulatif à la fin. Les points importants sont mis en **gras**.

## Sommaire

- [Différence entre Internet et Ethernet](#différence-entre-internet-et-ethernet)
- [La norme Ethernet](#la-norme-ethernet)
- [Câblage et équipements](#câblage-et-équipements)
- [L'adresse MAC](#ladresse-mac)
- [La trame Ethernet](#la-trame-ethernet)
- [Switch et VLAN](#switch-et-vlan)
- [Résumé global](#résumé-global)

## Différence entre Internet et Ethernet
<span id="différence-entre-internet-et-ethernet"></span>

**Internet** est un réseau mondial interconnectant des millions d'appareils via divers protocoles (comme TCP/IP). Il permet l'accès à des services distants comme le web ou les emails.

**Ethernet** est une **norme IEEE 802.3** pour les réseaux locaux filaires (**LAN** - Local Area Network). C'est un ensemble de protocoles pour connecter des machines sur de courtes distances, avec des débits de 10 Mbps à 400 Gbps. Ethernet utilise des protocoles comme TCP, IP, mais se concentre sur la couche physique et liaison.

**Point important** : Ethernet est souvent utilisé pour accéder à Internet, mais ce sont deux concepts distincts – Ethernet est local, Internet est global.

### Schéma simple
```
Internet (global)
   |
   | (via routeurs)
   v
Ethernet (LAN local)
  - Machine 1 -- Câble -- Machine 2
```

## La norme Ethernet

Ethernet est défini par la **norme IEEE 802.3**, née dans les années 70. Il s'agit du protocole le plus utilisé pour les réseaux filaires locaux.

### Architecture en couches IEEE
- **Couche PHYsique** (couche 1 OSI) : Gère le médium, débit, portée, taux d'erreur.
- **Couche Liaison** (couche 2 OSI) :
  - Sous-couche **MAC** (Medium Access Control) : Spécifique à Ethernet.
  - Sous-couche **LLC** (Logical Link Control) : Commune aux protocoles IEEE (mais souvent non utilisée avec IP).

**Normes IEEE liées** :
- IEEE 802.2 : LLC.
- IEEE 802.3 : Ethernet.
- Autres : WiFi (802.11), Bluetooth (802.15), etc.

### Débits
Débit en bit/s (b/s), convertible en octet/s (o/s) par division par 8.  
Exemples : 100 Mbps = 12.5 Mo/s.  
**Point important** : Débits de 2.3 Mbps (expérimental) à 400 Gbps, objectif 1.6 Tbps.

### Schéma des couches
```
Couche 2 (Liaison)
  - LLC (commune)
  - MAC (spécifique Ethernet)
Couche 1 (Physique)
  - PHY
```

## Câblage et équipements

### Types de câbles
- **Câble coaxial** : Utilisé dans les premières versions d'Ethernet (obsolète).
- **Paire torsadée** (câble RJ45) : 4 paires (8 fils), le plus utilisé pour courtes distances.
  - **Câbles droits** (MDI) : Entre hôtes (ex. : ordinateur à switch).
  - **Câbles croisés** (MDI-X) : Entre équipements similaires (ex. : switch à switch), mais rare avec Auto MDI/MDIX.
- **Fibre optique** : Pour débits ultra-rapides et longues distances.
  - **Fibre multimode (MMF)** : Moins chère, moins fine, moins longue distance.
  - **Fibre monomode (SMF)** : Plus chère, plus fine (fragile), plus longue distance.

**Point important** : RJ45 est le connecteur pour paires torsadées.

### Catégories de câbles RJ45
| Catégorie | Débit       | Distance max |
|-----------|-------------|--------------|
| CAT 8     | 25-40 Gb/s | 30 m        |
| CAT 7     | 10 Gb/s    | 100 m       |
| CAT 6a    | 10 Gb/s    | 100 m       |
| CAT 6     | 1/10 Gb/s  | 100/50 m    |
| CAT 5e    | 1 Gb/s     | 100 m       |
| CAT 5     | 100 Mb/s   | 100 m       |

### Types de blindage RJ45
- U/UTP : Non blindé.
- F/UTP : Blindage général.
- U/FTP : Blindage sur paires.
- F/FTP : Blindage général + paires.
- S/FTP : Blindage en tresse + paires.
- SF/UTP : Double blindage général.

### Équipements
- **Carte réseau** : Liée à l'**adresse MAC**, dans tous les appareils connectés. Une adresse MAC par port.
- **Émetteur-récepteur** (pour fibre) : GBIC, SFP, QSFP, etc.
- **Concentrateur/Hub** : Répéteur multiports (couche 1), obsolète.
- **Commutateur/Switch** : Pont multiports (couche 2), standard aujourd'hui.
- **Ports** : Avec diodes link (lien opérationnel) et activity (activité). **Vérifier link en premier !**

### Schéma câblage RJ45
```
Ordinateur (MDI) -- Câble droit -- Switch (MDI-X)
Switch -- Câble croisé -- Switch (si sans auto MDI/MDIX)
```

## L'adresse MAC

**Adresse MAC** : 48 bits (6 octets), notée en hexadécimal (ex. : 0F:B4:AA:07:F4:1A). Une par port de carte réseau.

- **7ème bit (U/L)** : 0 = Universelle (globale, par constructeur IEEE), 1 = Locale (gérée localement).
- **8ème bit (I/G)** : 0 = Individuelle (un hôte), 1 = Groupe (multicast).
- Adresse broadcast : FF:FF:FF:FF:FF:FF.

**Point important** : Assure unicité physique sur le réseau. Peut être changée, mais IEEE assure unicité globale.

### Consultation
- Linux : `ip l` (ex. : link/ether d4:93:90:05:2c:1c).
- Windows : `Get-NetAdapter` (PowerShell).

### Schéma format MAC
```
48 bits: [OUI (3 octets, constructeur)] [Suffixe (3 octets, unique)]
Ex: D4:93:90:05:2C:1C
```

## La trame Ethernet

La **trame Ethernet** est le PDU de la couche 2. Taille : 64 à 1518 octets.

### Structure
- **Début de trame** (préambule) : 7 octets 0xAA + 1 octet 0xAB (SFD). Pour synchronisation.
- **MAC header** (14 octets) :
  - Adresse MAC destinataire (6 octets).
  - Adresse MAC émetteur (6 octets).
  - **EtherType** (2 octets) : Type de payload (ex. : 0x0800 = IPv4). Si ≤1500 = longueur, ≥1536 = type.
- **Payload** (données) : 46 à 1500 octets. Encapsule PDU supérieur (ex. : paquet IP). Si <46, padding avec 0. MTU = 1500 max.
- **En-tête IP** (dans payload) : Adresse IP source + destinataire (couche 3).
- **CRC/FCS** (4 octets) : Somme de contrôle. Si erreur, trame détruite (best effort).
- **Gap inter trame** : 96 bits de silence pour fin de trame.

**Point important** : EtherType indique le protocole (ex. IPv4, ARP). Trame Ethernet 2 pour EtherType >1536.

### Schéma trame
```
Préambule (7 octets AA) | SFD (AB) | MAC Dest (6) | MAC Src (6) | EtherType (2) | Payload (46-1500) | FCS (4)
```

## Switch et VLAN

### Switch (commutateur)
Équipement couche 2. Crée canaux spécifiques via table MAC <-> ports.

Fonctions :
- **Apprentissage** : Enregistre MAC source/port.
- **Inondation** : Envoie à tous si destination inconnue.
- **Réexpédition** : Envoie seulement au port connu.
- **Filtrage** : Pas d'envoi si source/dest même port.
- **Vieillissement** : Vide table inactive.

Autres : Stack, LACP, SNMP, mirroring, 802.1x, VLAN, QoS (802.1p).

### VLAN (Virtual LAN)
Segmente réseau sans physique séparé. Sépare broadcast. Hôtes VLAN différents ne communiquent pas.

- Sur switch : Affecte ports à VLAN.
- Inter-switch : IEEE 802.1Q ajoute en-tête (4 octets) après MAC header : TPID (EtherType 802.1Q) + TCI (QoS + VLAN ID). Ajouté/ôté par switch, CRC recalculé.

**Point important** : Améliore sécurité et performance.

### Schéma VLAN
```
Switch
Port 1-4: VLAN 10
Port 5-8: VLAN 20
Inter-switch: Trame avec 802.1Q header
```

## Résumé global

Ethernet est une norme IEEE 802.3 pour LAN filaires, avec adresses MAC uniques, trames structurées (préambule, header, payload, FCS), et équipements comme switches/VLAN pour gestion efficace.

### Schéma récapitulatif
```
Norme 802.3 -> Câbles (RJ45/Fibre) -> Carte réseau (MAC) -> Trame (EtherType/Payload) -> Switch (table MAC) -> VLAN (segmentation)
```

**Sources** :  
- Notes "texte-collé.txt".  
- PDF "Ethernet.pdf".  
Aucune info complémentaire externe n'a été nécessaire, mais pour plus sur EtherType, voir [liste EtherType Wikipedia](https://en.wikipedia.org/wiki/EtherType).