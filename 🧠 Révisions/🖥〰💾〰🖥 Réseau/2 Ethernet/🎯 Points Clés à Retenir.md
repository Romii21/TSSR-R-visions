## Résumé Ethernet - Points Essentiels

**Qu'est-ce qu'Ethernet ?** Norme IEEE 802.3 pour réseaux locaux filaires (LAN), opérant aux couches 1 (physique) et 2 (liaison de données) du modèle OSI.

**Câblage**

- **RJ45** (paire torsadée) : Standard actuel, CAT 5e (1 Gb/s) à CAT 8 (40 Gb/s)
- **Fibre optique** : Longues distances et très hauts débits

**Adresse MAC**

- 48 bits (6 octets) en hexadécimal, unique par port réseau
- Identifie physiquement chaque interface sur le réseau

**Trame Ethernet** Structure : Préambule → MAC destination → MAC source → **EtherType** → Données (46-1500 octets) → CRC

- EtherType indique le protocole encapsulé (ex: 0x0800 = IPv4)
- Taille totale : 64 à 1518 octets

**Switch**

- Équipement couche 2, apprend les adresses MAC par port
- Crée des canaux directs, évite les collisions

**VLAN**

- Segmente logiquement un réseau physique
- Isole les domaines de broadcast, améliore sécurité/performance

**À retenir** : Ethernet = réseau local, Internet = réseau global. Le switch utilise les MAC pour diriger intelligemment le trafic.