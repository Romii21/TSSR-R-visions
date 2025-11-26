# Synthèse du cours sur IPv4

Cette synthèse est basée sur le cours fourni, en utilisant les termes techniques originaux comme **IETF**, **RFC**, **adresse IP**, **paquet**, **fragmentation**, etc. Elle est structurée en chapitres pour une meilleure organisation, avec un accent sur les points clés (mis en évidence en **gras**). J'ai ajouté des schémas graphiques simples via des images recherchées pour illustrer les concepts, et des explications ASCII pour la simplicité. Des informations utiles complémentaires proviennent des notes et du PDF, avec des ajouts mineurs pour clarifier (comme des exemples pratiques).

## Sommaire

- [Introduction](#introduction)
- [Les adresses](#les-adresses)
- [Configurer son réseau](#configurer-son-réseau)
- [Le paquet](#le-paquet)
- [Protocoles connexes](#protocoles-connexes)
- [Résumé global](#résumé-global)

## Introduction

L'**Internet Protocol version 4 (IPv4)** est un **protocole d'interconnexion de réseaux physiques**, standardisé par l'**IETF** (Internet Engineering Task Force). C'est un groupe international ouvert à tous, chargé d'élaborer les protocoles d'Internet à partir de la **couche 3** du modèle OSI (couche réseau) et au-dessus. 

Les produits de l'IETF sont les **RFC (Request For Comments)**, des documents numérotés décrivant les techniques pour l'interconnexion de réseaux. Les RFC évoluent : 
- **Internet Draft** : brouillon pour discussion.
- **Proposed Standard** : stable avec au moins une implémentation.
- **Draft Standard** : finale avec deux implémentations.
- **Internet Standard (STD)** : standard de fait.
- Autres : Experimental, Informational, Historic, ou **Best Current Practice (BCP)**.

Exemple clé : RFC 2026 (BCP 9) décrit le processus de standardisation. IPv4 date de 1981 (RFC 791, STD 5), tandis qu'IPv6 est de 1995 (RFC 1883, puis 2460).

**Points importants** : IPv4 permet **Internet**, un réseau mondial accessible à tous. Il suppose des protocoles physiques sous-jacents comme **Ethernet** (niveau 1 et 2), qui gère l'envoi de trames entre interfaces d'un même réseau physique. Ethernet fournit des services comme l'adressage physique (MAC), MTU, etc.

Dans IPv4, un **nœud** est un équipement supportant IP (comme un ordinateur ou routeur). Les services rendus par IPv4 incluent :
- **Routage** des paquets via des passerelles.
- **Adresse IP** par interface ou ensemble.
- **Paquet** : entête IP + charge utile.
- Gestion des erreurs (somme de contrôle), fragmentation pour liens à petit MTU.
- Configurations IP distinctes sur une même interface physique.

Pour communiquer, les configurations doivent être compatibles. IPv4 est **non orienté connexion** et **non fiable** (pas de garantie de livraison).

**Schéma simple (ASCII) du rôle d'IPv4 dans les couches** :
```
Couche 4+ (TCP/UDP) → Paquet IP (Couche 3) → Trame Ethernet (Couche 2) → Physique (Couche 1)
```

## Les adresses

Les **adresses IPv4** sont des identifiants logiques sur 32 bits, permettant d'identifier plusieurs adresses par machine (potentiellement plusieurs interfaces). Un nœud peut avoir plusieurs adresses logiques.

À l'origine, les adresses étaient classées :
- **Classe A** : premiers bits 0, 8 bits de partie réseau (ex. : grands réseaux).
- **Classe B** : premiers bits 10, 16 bits de partie réseau.
- **Classe C** : premiers bits 110, 24 bits de partie réseau.

Exemple : Adresse 192.168.0.254 (binaire : 11000000 10101000 00000000 11111110) → Id réseau : 11000000 10101000 00000000, Id nœud : 11111110.

**Notation décimal pointée (RFC 1166)** : Chaque octet en décimal, séparé par points (ex. : 192.168.0.254).

Problèmes des classes : Trop rigides (classes A trop grandes, B trop peu nombreuses). Solution : **CIDR (Classless Inter-Domain Routing, RFC 4632)**, notation /n où n est la longueur du préfixe réseau.

Exemple : 192.0.2.85/24 → Préfixe réseau : 192.0.2.x, Id nœud : 85.
- Adresse du réseau : 192.0.2.0 (tous bits nœud à 0).
- Adresse de diffusion (broadcast) : 192.0.2.255 (tous bits nœud à 1).

**Masque de sous-réseau** : Bits à 1 pour le réseau, 0 pour le nœud (ex. : 255.255.255.0 pour /24). Calcul : AND bit à bit entre adresse et masque donne l'adresse réseau.

Exemples de calcul :
- Adresse 192.0.2.85, Masque 255.255.255.0 → Réseau 192.0.2.0.
- Adresse 172.16.8.75, Masque 255.255.248.0 (/21) → Réseau 172.16.8.0.

Nombre d'hôtes : 2^(32-n) - 2 (enlève réseau et broadcast). Ex. : /21 → 2046 hôtes.

**Adresses spéciales** :
- Loopback : 127.0.0.1/8.
- Privées (RFC 1918) : 10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16.
- Autres : 0.0.0.0/32 (inconnu), 255.255.255.255/32 (broadcast local).

**Allocation** : Gérée par **IANA**, déléguée à **RIR (Registres Internet Régionaux)**, un par continent (ex. : RIPE NCC pour Europe/Moyen-Orient). RIR allouent à FAI/entreprises.

**Schéma graphique** :




**Info utile** : Les adresses privées nécessitent **NAT** pour accéder à Internet (voir section suivante).

## Configurer son réseau

Pour configurer un réseau IPv4, choisir des plages privées pour les machines internes, en fonction du nombre d'hôtes (actuel et futur). Pour l'accès Internet, utiliser **NAT (Network Address Translation)**, qui traduit les adresses privées en publique.

**Sous Linux** :
- Fichier /etc/network/interfaces (ou /etc/network/interfaces.d pour complexes).
- Commande **ip** : 
  - `ip -4 address show dev enp0s3` (voir config IPv4).
  - Objets : link (Ethernet), address (IP), neighbour (ARP).
  - Man pages : man ip-address, etc.

**Sous Windows** :
- Commandes PowerShell : Get-NetAdapter, New-NetIPAddress.
- Ex. : `New-NetIPAddress -InterfaceIndex 2 -IPAddress 192.168.0.24 -PrefixLength 24`.

Configurations persistantes.

**Points importants** : Utiliser NAT pour clients accédant à Internet. Choisir plages en fonction des besoins.

**Schéma graphique** :




**Info utile** : NAT conserve les adresses IPv4 limitées (4 milliards totales).

## Le paquet

Le **paquet IPv4** est formé d'un **entête** (variable, 20-60 octets) + charge utile. IP découpe les données de la couche supérieure (ex. : TCP) pour transmission au lien (ex. : Ethernet).

**Structure de l'entête** (sur 32 bits par ligne) :
- **Version** (4 bits) : 4.
- **IHL** (4 bits) : Longueur entête en mots de 32 bits (5-15).
- **ToS/DSCP** (8 bits) : Type of Service pour QoS (RFC 2474).
- **Total length** (16 bits) : Taille paquet (max 65535 octets).
- **Identification** (16 bits) : ID pour fragments.
- **Flags** (3 bits) : Don't Fragment (DF), More Fragments (MF).
- **Fragment offset** (13 bits) : Position du fragment.
- **TTL** (8 bits) : Time To Live, décrémenté par routeur (max 255).
- **Protocol** (8 bits) : Protocole encapsulé (ex. : 1=ICMP, 6=TCP, 17=UDP).
- **Header checksum** (16 bits) : Somme de contrôle de l'entête.
- **Source address** (32 bits).
- **Destination address** (32 bits).
- **Options** (variable) + Padding pour multiple de 32 bits.

**Fragmentation** : Si paquet > MTU du lien, découpage si DF=0. Réassemblage seulement au destinataire. Identification commun, offset indique position.

Ex. : Paquet trop grand → Découpe, copie entête (ajuste length, offset, MF=1 sauf dernier).

**Schéma graphique** :








**Points importants** : Entête assure routage, fragmentation gère MTU variés. Checksum recalculé à chaque hop.

## Protocoles connexes

**ICMP (Internet Control Message Protocol, protocol 1)** : Pour erreurs et diagnostics. Paquet : Type (8 bits, ex. : 0=Echo Reply, 3=Destination Unreachable, 8=Echo Request), Code (8 bits), Checksum (16 bits), Spécifique.

**ARP (Address Resolution Protocol)** : Lien entre IP (couche 3) et lien (couche 2, ex. : Ethernet). Résout adresse IP en MAC.

Paquet ARP (dans Ethernet, Ethertype 0x806) :
- Link type (ex. : 1=Ethernet).
- Protocol (ex. : 0x800=IPv4).
- Operation (1=Request, 2=Reply).
- Adresses source/destination (lien et protocole).

Fonctionnement : Requête broadcast "Qui a cette IP ?", réponse unicast avec MAC.

Problèmes sécurité : ARP spoofing possible.

**Schéma graphique** :




**Points importants** : ICMP pour erreurs (ex. : Time Exceeded pour traceroute), ARP essentiel pour liens comme Ethernet.

## Résumé global

IPv4 interconnecte réseaux via adresses, paquets et protocoles connexes. **Points clés** : Adresses CIDR pour flexibilité, paquet avec gestion fragmentation, configuration via outils OS, ICMP/ARP pour support.

**Schéma ASCII résumé** :
```
Nœud IP → Adresse (ex. 192.168.0.1/24) → Paquet (Entête + Données) → Fragmentation si besoin → ARP pour MAC → Envoi via Ethernet → Routage → Destinataire (Réassemblage)
NAT pour privé/public.
```

**Info utile** : Transition vers IPv6 due à épuisement adresses IPv4, mais IPv4 reste dominant.

## Sources
- Document PDF "Internet Protocol version 4.pdf" (principal).
- Notes du fichier "texte-collé.txt" (définitions de nœud, RIR, NAT).
- Images issues de recherches web (liens dans les métadonnées des images).