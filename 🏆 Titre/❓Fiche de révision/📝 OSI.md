# 📚 Fiche de révision — Modèle OSI

---

## 🔑 C'est quoi ?

Le modèle **OSI** (Open Systems Interconnection) est un cadre de référence en **7 couches** décrivant comment les données transitent d'une application à une autre à travers un réseau. Il standardise la communication entre systèmes hétérogènes.

> 💡 Mnémotechnique (bas → haut) : **"Please Do Not Throw Sausage Pizza Away"**

---

## 🗂️ Les 7 couches

| # | Couche | Rôle | Unité | Exemples de protocoles | Ports clés |
|---|---|---|---|---|---|
| 7 | **Application** | Interface avec l'utilisateur | Données | HTTP, HTTPS, FTP, SMTP, DNS, DHCP, SSH, SNMP | 80, 443, 21, 25, 53, 67/68, 22, 161 |
| 6 | **Présentation** | Encodage, chiffrement, compression | Données | SSL/TLS, JPEG, ASCII, MIME | — |
| 5 | **Session** | Ouverture/fermeture de sessions | Données | NetBIOS, RPC, PPTP | 139, 445 |
| 4 | **Transport** | Segmentation, fiabilité, contrôle de flux | Segments / Datagrammes | TCP, UDP | Tous les ports applicatifs |
| 3 | **Réseau** | Adressage logique, routage | Paquets | IP, ICMP, ARP, OSPF, BGP, RIP | — |
| 2 | **Liaison** | Adressage physique (MAC), accès au médium | Trames | Ethernet, Wi-Fi (802.11), PPP, VLAN (802.1Q) | — |
| 1 | **Physique** | Transmission des bits sur le médium | Bits | RJ45, Fibre optique, DSL, Bluetooth | — |

---

## 🔌 Ports essentiels à connaître

### Protocoles courants

| Port | Protocole | Couche | Description |
|---|---|---|---|
| **20/21** | FTP | 7 | Transfert de fichiers (20 = data, 21 = contrôle) |
| **22** | SSH | 7 | Accès distant sécurisé |
| **23** | Telnet | 7 | Accès distant non sécurisé |
| **25** | SMTP | 7 | Envoi de mails |
| **53** | DNS | 7 | Résolution de noms (UDP par défaut, TCP si > 512 bytes) |
| **67/68** | DHCP | 7 | Attribution IP (67 = serveur, 68 = client) |
| **80** | HTTP | 7 | Web non chiffré |
| **110** | POP3 | 7 | Réception de mails |
| **143** | IMAP | 7 | Réception de mails (synchronisé) |
| **161/162** | SNMP | 7 | Supervision réseau (161 = requête, 162 = trap) |
| **443** | HTTPS | 7 | Web chiffré (TLS) |
| **445** | SMB | 5/7 | Partage de fichiers Windows |
| **3389** | RDP | 7 | Bureau à distance Windows |

---

## ⚙️ TCP vs UDP (Couche 4)

| | TCP | UDP |
|---|---|---|
| **Connexion** | Orienté connexion (3-way handshake) | Sans connexion |
| **Fiabilité** | ✅ Accusés de réception, retransmission | ❌ Aucune garantie |
| **Ordre** | ✅ Garanti | ❌ Non garanti |
| **Vitesse** | Plus lent | Plus rapide |
| **Usage** | HTTP, FTP, SSH, SMTP | DNS, DHCP, streaming, VoIP |

> 🤝 **3-way handshake TCP** : `SYN → SYN-ACK → ACK`

---

## 🌐 Couche 3 — Protocoles réseau clés

| Protocole | Rôle |
|---|---|
| **IP** | Adressage et routage des paquets |
| **ICMP** | Messages d'erreur et diagnostics (`ping`, `traceroute`) |
| **ARP** | Résolution IP → adresse MAC |
| **OSPF** | Protocole de routage dynamique interne (IGP) |
| **BGP** | Protocole de routage inter-AS (Internet) |
| **RIP** | Protocole de routage dynamique simple (distance vector) |

---

## 🔗 Couche 2 — Points clés

- Adressage par **adresse MAC** (48 bits, ex: `AA:BB:CC:DD:EE:FF`)
- **Switch** = équipement de couche 2
- **VLAN (802.1Q)** : segmentation logique du réseau
- **STP (Spanning Tree Protocol)** : évite les boucles réseau

---

## 📦 Encapsulation des données

```
Application  → Données
Transport    → Segment (TCP) / Datagramme (UDP)  [ajout port src/dst]
Réseau       → Paquet                             [ajout IP src/dst]
Liaison      → Trame                              [ajout MAC src/dst]
Physique     → Bits
```
> À la réception, le processus inverse s'appelle la **désencapsulation**.

---

## 🧠 Équipements par couche

| Couche | Équipement |
|---|---|
| 1 — Physique | Hub, câble, répéteur |
| 2 — Liaison | Switch, pont (bridge) |
| 3 — Réseau | Routeur, pare-feu (L3) |
| 4 à 7 | Pare-feu applicatif, proxy, load balancer |

---

## 📝 À retenir absolument

1. Le modèle OSI est une **référence théorique** ; en pratique on utilise le modèle **TCP/IP** (4 couches)
2. Les données sont **encapsulées** couche par couche à l'envoi, **désencapsulées** à la réception
3. **TCP = fiable** (connexion établie) | **UDP = rapide** (pas de garantie)
4. **DNS (53)** utilise UDP par défaut, TCP si la réponse dépasse 512 octets
5. Retenir les ports : **22** (SSH), **80** (HTTP), **443** (HTTPS), **53** (DNS), **25** (SMTP), **3389** (RDP)
