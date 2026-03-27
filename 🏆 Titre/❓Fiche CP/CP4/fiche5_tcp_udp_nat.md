# FICHE 5 — TCP, UDP et NAT
### CP4 : Exploiter un réseau IP | Formation TSSR

---

## PARTIE A — TCP et UDP

## 1. Comparaison TCP / UDP

| Critère | **TCP** | **UDP** |
|---------|---------|---------|
| Connexion | **Orienté connexion** (Three-Way Handshake) | **Sans connexion** |
| Fiabilité | Accusé de réception, retransmission si perte | Aucune garantie de livraison |
| Ordre | Garantit l'ordre des paquets | Pas de garantie d'ordre |
| Contrôle de flux | Oui (fenêtre TCP) | Non |
| Vitesse | Plus lent (overhead) | **Plus rapide** |
| Cas d'usage | HTTP, HTTPS, SSH, FTP, SMTP | DNS, DHCP, streaming, VoIP, jeux en ligne |

---

## 2. Les ports réseau

Un **port** est un numéro (16 bits) identifiant un service ou une application sur une machine.

```
Connexion TCP vers un serveur web :
  [IP source:PORT_EPHÉMÈRE] → [IP destination:PORT_SERVEUR]
  ex: 192.168.1.10:52341  →  93.184.216.34:443
```

### Les 3 plages de ports

| Plage | Nom | Description | Exemples |
|-------|-----|-------------|---------|
| **0 – 1023** | Well-known ports | Ports réservés aux services standards | HTTP/80, HTTPS/443, SSH/22, DNS/53 |
| **1024 – 49151** | Registered ports | Ports d'applications enregistrées | RDP/3389, MySQL/3306, PostgreSQL/5432 |
| **49152 – 65535** | Dynamic / Ephemeral | Ports éphémères attribués dynamiquement par l'OS aux clients | Port source des connexions sortantes |

---

## 3. Le Three-Way Handshake TCP

Mécanisme établissant une connexion TCP avant tout échange de données.

```
CLIENT                                    SERVEUR
  │                                          │
  │  1. SYN  (seq=100)                       │
  │─────────────────────────────────────────>│
  │  "Je veux me connecter, mon seq = 100"   │
  │                                          │
  │  2. SYN-ACK  (seq=200, ack=101)          │
  │<─────────────────────────────────────────│
  │  "OK, mon seq = 200, j'attends ton 101"  │
  │                                          │
  │  3. ACK  (ack=201)                       │
  │─────────────────────────────────────────>│
  │  "Reçu, j'attends ton 201"               │
  │         ↓                                │
  │    CONNEXION ÉTABLIE                     │
```

**Pourquoi ce mécanisme ?** Il garantit que les deux côtés sont prêts à communiquer ET synchronise les numéros de séquence initiaux.

**Flags TCP impliqués :** SYN, ACK (et SYN+ACK combinés)

---

## 4. La fermeture TCP — Four-Way Handshake

La fermeture d'une connexion TCP est **symétrique** : chaque côté ferme sa direction indépendamment.

```
CLIENT                                    SERVEUR
  │                                          │
  │  1. FIN                                  │
  │─────────────────────────────────────────>│
  │  "Je n'ai plus de données à envoyer"     │
  │                                          │
  │  2. ACK                                  │
  │<─────────────────────────────────────────│
  │  "Reçu, mais j'ai peut-être encore des données"
  │                                          │
  │  3. FIN                                  │
  │<─────────────────────────────────────────│
  │  "Moi aussi j'ai fini"                   │
  │                                          │
  │  4. ACK                                  │
  │─────────────────────────────────────────>│
  │        CONNEXION FERMÉE                  │
```

> **RST (Reset)** : fermeture **brutale** d'une connexion TCP en cas d'erreur ou de rejet.

---

## 5. La fenêtre TCP (TCP Window Size)

### Définition
La fenêtre TCP définit le **nombre d'octets qu'un émetteur peut envoyer avant de devoir attendre un accusé de réception**.

### Problème résolu
Sans fenêtre, l'émetteur enverrait des données en continu → le récepteur lent serait **submergé**.

### Fonctionnement
```
Fenêtre de taille 3 (exemple simplifié) :

Émetteur envoie 3 paquets :  [1][2][3]
Attente ACK...
                              ACK 4 reçu → peut envoyer 3 paquets de plus
Émetteur envoie :            [4][5][6]
...
```

- Si le récepteur est **lent**, il **réduit la fenêtre annoncée** → l'émetteur ralentit
- Si le réseau est **congestionné** → mécanisme de **slow start** (fenêtre réduite puis augmentée progressivement)

---

## 6. Récapitulatif des flags TCP importants

| Flag | Nom | Usage |
|------|-----|-------|
| **SYN** | Synchronize | Initie une connexion |
| **ACK** | Acknowledge | Accuse réception d'un paquet |
| **FIN** | Finish | Fermeture propre d'une direction |
| **RST** | Reset | Fermeture brutale / rejet |
| **PSH** | Push | Données à traiter immédiatement |
| **URG** | Urgent | Données urgentes |

---

## PARTIE B — NAT

## 7. Pourquoi le NAT ?

**Problème :** IPv4 ne dispose que de ~4,3 milliards d'adresses → **épuisement** des adresses publiques.

**Solution :** Le **NAT (Network Address Translation)** permet à de nombreuses machines avec des **adresses privées** de partager une ou quelques **adresses IP publiques**.

- Défini dans la **RFC 3022**
- Mis en place sur un **routeur** ou un **pare-feu**
- Permet aussi de **masquer l'architecture interne** du réseau

---

## 8. Les 3 types de NAT

### PAT / NAPT — Port Address Translation

Le type de NAT le **plus utilisé** (celui de votre box Internet).

**Principe :** Plusieurs IP privées → **1 seule IP publique**, distinguées par les **ports TCP/UDP**.

```
RÉSEAU INTERNE           ROUTEUR NAT              INTERNET
192.168.1.10:52341  →   203.0.113.1:52341  →   93.184.216.34:443
192.168.1.20:48201  →   203.0.113.1:48201  →   142.250.74.46:80
192.168.1.30:61045  →   203.0.113.1:61045  →   93.184.216.34:443
```

La **table NAT** associe chaque session (IP interne + port) à un port public unique.

**Cas d'usage :** Accès Internet d'un réseau d'entreprise ou domestique.

### DNAT — Destination NAT (Port Forwarding)

**Principe :** Redirige le **trafic entrant** vers un serveur interne.

```
INTERNET → 203.0.113.1:80  →  192.168.1.100:80  (serveur web interne)
INTERNET → 203.0.113.1:443 →  192.168.1.100:443
```

**Cas d'usage :** Publier un serveur web, un serveur mail ou un accès VPN derrière un routeur NAT.

> ⚠️ **DNAT ≠ DMZ**. La DMZ est une **architecture réseau** de sécurité (zone isolée). Le DNAT est la **technique de translation** qui rend un serveur accessible. En pratique, les deux sont souvent combinés.

### NAT Statique 1:1

**Principe :** Une IP privée ↔ **une IP publique dédiée**, dans les deux sens.

```
IP privée 192.168.1.50  ↔  IP publique 203.0.113.10 (fixe)
```

**Cas d'usage :** Serveur devant être joignable avec une IP publique fixe (ex : serveur FTP d'une entreprise).

---

## 9. Comparatif des 3 types de NAT

| Type | Sens | Mode | Différenciation | Usage principal |
|------|------|------|-----------------|-----------------|
| **PAT/NAPT** | Source (sortant) | Dynamique | IP + **port** | Accès Internet clients |
| **DNAT** | Destination (entrant) | Statique | IP (+ port) | Publication de services |
| **NAT 1:1** | Source + Destination | Statique | IP dédiée | Serveurs avec IP publique fixe |

---

## 10. Fonctionnement détaillé PAT — Scénario complet

```
PC (192.168.1.10) accède à www.google.com

ALLER :
  Source     : 192.168.1.10:52341
  Destination: 142.250.x.x:443
        ↓ Routeur NAT traduit
  Source     : 203.0.113.1:52341    ← IP publique du routeur
  Destination: 142.250.x.x:443

RETOUR :
  Source     : 142.250.x.x:443
  Destination: 203.0.113.1:52341
        ↓ Routeur consulte la table NAT
        ↓ 203.0.113.1:52341 → 192.168.1.10:52341
  Source     : 142.250.x.x:443
  Destination: 192.168.1.10:52341   ← rendu au bon PC
```

---

## 11. Avantages et inconvénients du NAT

| Avantages | Inconvénients |
|-----------|--------------|
| Économise les adresses IPv4 | Casse le principe de **bout en bout** d'IP |
| Cache l'architecture interne | Incompatible avec certains protocoles (FTP actif, SIP…) |
| Couche de sécurité supplémentaire | Complexifie le diagnostic réseau |
| Pas besoin d'IPv6 immédiatement | Ajoute de la latence (traduction à chaque paquet) |

> **IPv6 et NAT :** En IPv6, le NAT n'est **plus nécessaire** (espace d'adressage quasi illimité). Chaque machine peut avoir une adresse publique unique.

---

## 12. Points clés à retenir ✅

**TCP/UDP**
- TCP = **orienté connexion**, fiable, ordonné
- UDP = **sans connexion**, rapide, sans garantie
- **Three-Way Handshake** : SYN → SYN-ACK → ACK
- **Fermeture TCP** : FIN → ACK → FIN → ACK (Four-Way)
- Ports **0–1023** = well-known | **1024–49151** = registered | **49152–65535** = éphémères
- Fenêtre TCP = contrôle de flux (évite de submerger le récepteur)

**NAT**
- NAT créé pour pallier la **pénurie d'IPv4**
- **PAT/NAPT** = le plus courant, 1 IP publique + différenciation par ports
- **DNAT** = rend un service interne accessible depuis Internet (port forwarding)
- **NAT 1:1** = 1 IP privée ↔ 1 IP publique fixe
- DNAT ≠ DMZ (la DMZ est une architecture, pas une technique NAT)
- En **IPv6** → NAT **non nécessaire**

---

*Fiche 5/6 — CP4 Exploiter un réseau IP*
