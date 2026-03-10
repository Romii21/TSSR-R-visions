# FICHE 1 — Fondamentaux Réseau
### CP4 : Exploiter un réseau IP | Formation TSSR

---

## 1. Les types de réseaux

| Type | Portée | Exemple concret |
|------|--------|-----------------|
| **PAN** | Personnelle (~10 m) | Bluetooth, synchronisation smartphone |
| **LAN** | Bâtiment / site | Réseau Wi-Fi d'une entreprise, réseau d'école |
| **MAN** | Ville | Réseau municipal, campus universitaire |
| **WAN** | Pays / continent | Liaisons inter-sites d'une entreprise |
| **Internet (GAN)** | Mondial | L'Internet public |

> **LAN ≠ adresses privées.** Le LAN désigne une portée géographique, pas un type d'adresse. Un WAN peut aussi utiliser des adresses privées sur des liaisons inter-sites.

---

## 2. Le modèle OSI — Les 7 couches

| N° | Couche | Unité | Rôle | Équipement |
|----|--------|-------|------|------------|
| 7 | **Application** | Données | Interface avec l'utilisateur (HTTP, DNS, FTP…) | Serveur, PC |
| 6 | **Présentation** | Données | Chiffrement, compression, encodage | — |
| 5 | **Session** | Données | Ouverture/fermeture de sessions | — |
| 4 | **Transport** | Segment | Fiabilité, ports, contrôle de flux (TCP/UDP) | — |
| 3 | **Réseau** | Paquet | Adressage IP, routage | Routeur |
| 2 | **Liaison** | Trame | Adressage MAC, accès au média | Switch |
| 1 | **Physique** | Bits | Transmission électrique/optique | Hub, câble |

> **Moyen mnémotechnique (de bas en haut)** : **P**our **L**e **R**éseau **T**out **S**e **P**asse **A**insi

---

## 3. Les équipements et leurs couches

### Hub (Couche 1 — Physique)
- Répéteur multiport : répète le signal sur **tous** les ports
- Pas d'intelligence — crée des **collisions**
- Pratiquement **obsolète** aujourd'hui

### Switch / Commutateur (Couche 2 — Liaison)
- Commute les trames en fonction des **adresses MAC**
- Maintient une **table CAM** (table d'adresses MAC)
- Limite les domaines de collision (un domaine par port)
- Relie les machines d'un **même réseau**

### Routeur (Couche 3 — Réseau)
- Route les paquets en fonction des **adresses IP**
- Maintient une **table de routage**
- Relie plusieurs **réseaux différents** entre eux
- Sépare les domaines de broadcast

> **Rappel important :** Le switch utilise les **adresses MAC** (pas IP). La **table ARP** est gérée par les hôtes (PC, routeurs), pas par le switch. La table du switch s'appelle **table CAM**.

---

## 4. L'adresse MAC

### Définition
L'adresse MAC (Media Access Control) est l'**identifiant physique unique** d'une interface réseau.

### Caractéristiques
- Codée sur **6 octets = 48 bits**
- Représentation hexadécimale : `AA:BB:CC:DD:EE:FF` ou `AA-BB-CC-DD-EE-FF`
- Gravée par le fabricant (mais modifiable logiciellement)

### Structure
```
[ OUI - 3 octets ]  :  [ NIC - 3 octets ]
   AA : BB : CC           DD : EE : FF
   ↑
   Identifiant du fabricant (Organizationally Unique Identifier)
```

### Adresses MAC spéciales
| Adresse | Rôle |
|---------|------|
| `FF:FF:FF:FF:FF:FF` | **Broadcast** — envoyé à toutes les machines du segment |
| `01:00:5E:xx:xx:xx` | **Multicast IPv4** |
| `33:33:xx:xx:xx:xx` | **Multicast IPv6** |

---

## 5. Le protocole ARP

### Rôle
ARP (Address Resolution Protocol) permet de **résoudre une adresse IP en adresse MAC** au sein d'un même réseau local.

### Quand est-il déclenché ?
Lorsqu'une machine veut communiquer avec une IP du même réseau et qu'elle ne connaît **pas encore la MAC correspondante**.

### Fonctionnement
```
Machine A (192.168.1.10) veut joindre Machine B (192.168.1.20)

Étape 1 — ARP Request (Broadcast)
  Machine A → FF:FF:FF:FF:FF:FF
  "Qui a l'IP 192.168.1.20 ? Répondez-moi à AA:BB:CC:DD:EE:FF"

Étape 2 — ARP Reply (Unicast)
  Machine B → Machine A
  "C'est moi ! Mon adresse MAC est 11:22:33:44:55:66"

Étape 3 — Mise en cache
  Machine A stocke l'association IP↔MAC dans sa table ARP
```

### La table ARP
- Stockée en mémoire sur chaque hôte
- Consulter sous Linux : `ip neigh` ou `arp -n`
- Consulter sous Windows : `arp -a`
- Entrées avec durée de vie limitée (TTL)

---

## 6. L'encapsulation — Principe fondamental

### Définition
L'encapsulation consiste à **envelopper les données** d'une couche dans l'unité de la couche inférieure en ajoutant un en-tête (header).

### Schéma descendant (émission)

```
COUCHE 7 — Application
  [ DONNÉES HTTP ]

COUCHE 4 — Transport
  [ EN-TÊTE TCP | DONNÉES HTTP ]
    ↑ Ajout port source, port destination, numéro de séquence

COUCHE 3 — Réseau
  [ EN-TÊTE IP | EN-TÊTE TCP | DONNÉES HTTP ]
    ↑ Ajout IP source, IP destination, TTL

COUCHE 2 — Liaison
  [ EN-TÊTE ETH | EN-TÊTE IP | EN-TÊTE TCP | DONNÉES | FCS ]
    ↑ Ajout MAC source, MAC destination + FCS (contrôle d'erreur)

COUCHE 1 — Physique
  101010110011...  (bits transmis sur le câble)
```

### Désencapsulation (réception)
À chaque couche remontante, l'en-tête correspondant est **retiré et interprété** par la couche concernée.

### Noms des unités par couche
| Couche | Unité (PDU) |
|--------|-------------|
| Transport | **Segment** (TCP) / **Datagramme** (UDP) |
| Réseau | **Paquet** |
| Liaison | **Trame** |
| Physique | **Bits** |

---

## 7. Protocoles applicatifs courants

| Protocole | Port | Transport | Rôle |
|-----------|------|-----------|------|
| **HTTP** | 80 | TCP | Navigation web non chiffrée |
| **HTTPS** | 443 | TCP | Navigation web chiffrée (TLS) |
| **FTP** | 21 (ctrl) / 20 (data) | TCP | Transfert de fichiers |
| **SSH** | 22 | TCP | Connexion distante sécurisée |
| **Telnet** | 23 | TCP | Connexion distante non sécurisée (obsolète) |
| **SMTP** | 25 | TCP | Envoi d'e-mails |
| **DNS** | 53 | UDP (+ TCP pour transferts de zone) | Résolution de noms |
| **DHCP** | 67 (serveur) / 68 (client) | **UDP** | Attribution automatique d'IP |
| **NTP** | 123 | UDP | Synchronisation de l'heure |
| **RDP** | 3389 | TCP | Bureau à distance Windows |
| **SNMP** | 161 | UDP | Supervision réseau |

> ⚠️ **DHCP utilise UDP**, pas TCP. C'est une erreur très fréquente à l'examen.

---

## 8. Points clés à retenir ✅

- **LAN** = portée géographique locale, **pas** un type d'adresse
- **Switch** = couche 2, table **CAM** (adresses MAC), **pas** table ARP
- **Routeur** = couche 3, table de **routage** (adresses IP)
- **ARP** = résout une IP en MAC dans un même réseau
- **Encapsulation** = ajout d'en-têtes couche par couche à l'émission
- **DHCP** = UDP/67-68 (jamais TCP !)
- **DNS** = UDP/53 en priorité, TCP/53 pour les gros transferts

---

*Fiche 1/6 — CP4 Exploiter un réseau IP*
