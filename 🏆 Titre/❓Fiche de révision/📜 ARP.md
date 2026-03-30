# 📚 Fiche de révision — Protocole ARP & Table ARP

---

## 🔑 C'est quoi ARP ?

**ARP** (Address Resolution Protocol) est un protocole de **couche 2/3** qui permet de **résoudre une adresse IP en adresse MAC**.

> Quand une machine veut communiquer avec une autre sur le même réseau local, elle connaît son IP mais pas son MAC. ARP sert à faire le lien.

| Propriété | Valeur |
|---|---|
| **Couche OSI** | 2 (Liaison) / 3 (Réseau) |
| **EtherType** | `0x0806` |
| **Protocole transport** | Aucun (directement dans la trame Ethernet) |
| **Portée** | Réseau local uniquement (non routable) |

---

## ⚙️ Fonctionnement — Étape par étape

```
Machine A (192.168.1.1)          Machine B (192.168.1.2)
       │                                    │
       │  1. "Qui a l'IP 192.168.1.2 ?"    │
       │──── ARP Request (broadcast) ──────▶│ (et tous les autres)
       │     Dest MAC : FF:FF:FF:FF:FF:FF   │
       │                                    │
       │  2. "C'est moi ! Mon MAC = BB:BB"  │
       │◀─── ARP Reply (unicast) ───────────│
       │     Dest MAC : AA:AA (Machine A)   │
       │                                    │
       │  3. Machine A met à jour sa table  │
       │     192.168.1.2 → BB:BB:BB:BB:BB:BB│
```

### Les 2 types de messages ARP

| Message | Type | Destination MAC | Rôle |
|---|---|---|---|
| **ARP Request** | Broadcast | `FF:FF:FF:FF:FF:FF` | "Qui a cette IP ?" |
| **ARP Reply** | Unicast | MAC de l'émetteur | "C'est moi, voici mon MAC" |

---

## 🧩 Structure d'un paquet ARP

```
┌────────────┬────────────┬───────┬───────┬────────┬───────────────────────────────┐
│ HW Type    │ Proto Type │HW len │Pr len │  Op   │ MAC src │ IP src │MAC dst│IP dst│
│  2 octets  │  2 octets  │  1 o  │  1 o  │  2 o  │  6 o    │  4 o   │  6 o  │  4 o │
└────────────┴────────────┴───────┴───────┴────────┴───────────────────────────────┘
```

| Champ | Valeur courante | Signification |
|---|---|---|
| **HW Type** | `0x0001` | Ethernet |
| **Proto Type** | `0x0800` | IPv4 |
| **HW len** | `6` | Longueur MAC (6 octets) |
| **Proto len** | `4` | Longueur IP (4 octets) |
| **Opcode** | `1` = Request / `2` = Reply | Type de message |

---

## 🗂️ La Table ARP (ARP Cache)

### C'est quoi ?
La table ARP est un **cache local** stockant les correspondances `IP ↔ MAC` récemment apprises, pour éviter de refaire une requête ARP à chaque communication.

### Afficher la table ARP

```bash
# Windows
arp -a                        # Affiche toutes les entrées
arp -a -N 192.168.1.1         # Filtre sur une interface

# Linux / macOS
arp -n                        # Sans résolution DNS
ip neigh                      # Commande moderne (iproute2)
ip neigh show                 # Équivalent
```

### Exemple de table ARP

```
Interface : 192.168.1.10
  Adresse Internet    Adresse physique    Type
  192.168.1.1         AA:BB:CC:DD:EE:01   dynamique
  192.168.1.2         AA:BB:CC:DD:EE:02   dynamique
  192.168.1.255       FF:FF:FF:FF:FF:FF   statique
  224.0.0.22          01:00:5E:00:00:16   statique
```

### Types d'entrées

| Type | Description | Durée de vie |
|---|---|---|
| **Dynamique** | Apprise automatiquement via ARP | ~2 min (Windows) / ~20 min (Linux) |
| **Statique** | Configurée manuellement | Permanente (jusqu'au redémarrage) |

---

## 🛠️ Gestion manuelle de la table ARP

```bash
# Windows
arp -a                           # Voir la table
arp -d 192.168.1.1               # Supprimer une entrée
arp -s 192.168.1.1 AA-BB-CC-DD-EE-FF  # Ajouter une entrée statique

# Linux
ip neigh show                    # Voir la table
ip neigh del 192.168.1.1 dev eth0             # Supprimer
ip neigh add 192.168.1.1 lladdr AA:BB:CC:DD:EE:FF dev eth0 nud permanent  # Ajouter statique
ip neigh flush all               # Vider le cache ARP
```

---

## 🌐 ARP et la passerelle (Gateway)

Quand la destination est **hors du réseau local**, la machine envoie le paquet à sa **passerelle par défaut** :

```
Machine A → veut joindre 8.8.8.8 (hors réseau)
  │
  ├─ ARP Request : "Qui a l'IP de ma gateway (192.168.1.254) ?"
  ├─ ARP Reply   : "C'est moi, MAC = GG:GG:GG:GG:GG:GG"
  └─ Envoi du paquet au routeur (MAC gateway, IP 8.8.8.8)
```

> ⚠️ **ARP ne traverse pas les routeurs** — il reste confiné au réseau local (LAN/VLAN).

---

## 🔄 Cas particuliers

### Gratuitous ARP (ARP Gratuit)
Une machine envoie un **ARP Request avec sa propre IP** comme cible.

**Usages :**
- Annoncer sa présence sur le réseau
- Mettre à jour les tables ARP des autres (ex: après un changement de MAC)
- Utilisé par les systèmes de **haute disponibilité** (failover, VIP)

```
ARP Request : "Qui a 192.168.1.10 ?" envoyé par 192.168.1.10 lui-même
```

### Proxy ARP
Un **routeur répond aux ARP Requests** à la place d'une autre machine (transparence de routage).

---

## ⚠️ Sécurité — ARP Spoofing / ARP Poisoning

### Comment ça marche ?
ARP est **sans authentification** → un attaquant peut envoyer de faux ARP Reply pour corrompre les tables des autres machines.

```
Attaquant envoie à Victime :
  "L'IP de la Gateway (192.168.1.1) → c'est MON MAC"

Attaquant envoie à la Gateway :
  "L'IP de la Victime (192.168.1.50) → c'est MON MAC"

Résultat : tout le trafic passe par l'attaquant → Man-in-the-Middle (MitM)
```

### Contremesures

| Mesure | Description |
|---|---|
| **ARP statique** | Fixer manuellement les entrées critiques (gateway) |
| **Dynamic ARP Inspection (DAI)** | Fonctionnalité switch — valide les ARP via une table DHCP snooping |
| **DHCP Snooping** | Pré-requis à DAI, enregistre les attributions DHCP |
| **VLAN** | Limiter la portée des broadcasts ARP |
| **Surveillance** | Détecter des ARP Reply non sollicités (Wireshark, IDS) |

---

## 🦈 Analyser ARP avec Wireshark

```bash
# Filtres d'affichage
arp                                  # Tout le trafic ARP
arp.opcode == 1                      # ARP Requests uniquement
arp.opcode == 2                      # ARP Replies uniquement
arp.src.proto_ipv4 == 192.168.1.1    # ARP depuis cette IP
arp.dst.proto_ipv4 == 192.168.1.1    # ARP vers cette IP

# Détecter un ARP Spoofing
arp.duplicate-address-detected       # Filtre intégré Wireshark
arp.opcode == 2 && arp.src.hw_mac != eth.src  # Incohérence MAC
```

> 💡 **Signe d'ARP Spoofing** : plusieurs ARP Reply pour la même IP avec des MAC différents, ou des Gratuitous ARP inattendus.

---

## 📝 À retenir absolument

1. ARP résout **IP → MAC**, indispensable pour toute communication sur un LAN
2. **ARP Request** = broadcast (`FF:FF:FF:FF:FF:FF`) | **ARP Reply** = unicast
3. La **table ARP** est un cache local avec entrées **dynamiques** (temporaires) et **statiques** (manuelles)
4. ARP est **non routable** — il reste dans le même réseau local / VLAN
5. Pour joindre une IP externe, la machine fait un ARP sur sa **gateway**, pas sur la destination
6. ARP est **sans authentification** → vulnérable au **ARP Spoofing** (attaque MitM)
7. **DAI + DHCP Snooping** = protection efficace côté infrastructure switch
