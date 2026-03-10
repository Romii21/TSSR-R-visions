# FICHE 6 — IPv6 et Diagnostic Réseau
### CP4 : Exploiter un réseau IP | Formation TSSR

---

## PARTIE A — IPv6

## 1. Pourquoi IPv6 ?

IPv4 est épuisé : ~4,3 milliards d'adresses insuffisantes face à la croissance d'Internet.

IPv6 a été conçu pour **remplacer IPv4** et résoudre ses limitations structurelles.

---

## 2. Structure d'une adresse IPv6

- Codée sur **128 bits**
- Notation **hexadécimale** en 8 groupes de 4 chiffres hex séparés par `:`
- Nombre d'adresses : **2¹²⁸** ≈ 340 undécillions (quasi illimité)

```
2001:0db8:0000:0000:0000:0000:0000:0001
```

### Les 2 règles de simplification

**Règle 1 — Supprimer les zéros de tête dans chaque groupe**
```
0db8 → db8
0000 → 0
```

**Règle 2 — Remplacer une séquence consécutive de groupes nuls par `::`**
```
2001:0db8:0000:0000:0000:0000:0000:0001
     ↓ Règle 1
2001:db8:0:0:0:0:0:1
     ↓ Règle 2
2001:db8::1
```

> ⚠️ `::` ne peut apparaître **qu'une seule fois** dans une adresse.

### Exemples de simplification

| Adresse complète | Adresse simplifiée |
|-----------------|-------------------|
| `FE80:0000:0000:0000:0202:B3FF:FE1E:8329` | `FE80::202:B3FF:FE1E:8329` |
| `2001:0DB8:0000:0000:0008:0800:200C:417A` | `2001:DB8::8:800:200C:417A` |
| `0000:0000:0000:0000:0000:0000:0000:0001` | `::1` (loopback) |
| `0000:0000:0000:0000:0000:0000:0000:0000` | `::` (adresse indéfinie) |

---

## 3. Types d'adresses IPv6

| Type | Préfixe | Description | Équivalent IPv4 |
|------|---------|-------------|-----------------|
| **Link-local** | `FE80::/10` | Auto-configurée, valide sur le **lien local uniquement** (non routable) | — |
| **Global Unicast** | `2000::/3` | Adresses **publiques**, routables sur Internet | Adresses publiques IPv4 |
| **Unique Local** | `FC00::/7` | Adresses **privées** (non routables sur Internet) | RFC 1918 (10.x, 172.x, 192.168.x) |
| **Multicast** | `FF00::/8` | Envoi vers **un groupe** de machines — remplace le broadcast | Broadcast IPv4 |
| **Loopback** | `::1/128` | La machine elle-même | 127.0.0.1 |
| **Indéfinie** | `::/128` | Adresse non configurée | 0.0.0.0 |

### Focus sur Link-local (`FE80::/10`)
- **Automatiquement configurée** sur chaque interface dès qu'IPv6 est activé
- Utilisée pour la communication entre voisins directs (NDP)
- **Non routable** : ne franchit pas un routeur
- Toujours présente même sans configuration manuelle

---

## 4. SLAAC — Autoconfiguration sans état

**SLAAC** (Stateless Address Autoconfiguration) = mécanisme permettant à une machine de **s'attribuer automatiquement une adresse IPv6 globale**, sans serveur DHCP.

### Fonctionnement

```
1. La machine génère une adresse link-local (FE80:: + EUI-64)
         ↓
2. Elle envoie un Router Solicitation (RS) en multicast
         ↓
3. Le routeur répond avec un Router Advertisement (RA)
   contenant : préfixe réseau + durée de vie
         ↓
4. La machine combine :
   [Préfixe annoncé par le routeur] + [ID d'interface EUI-64]
   → Adresse IPv6 globale unique
```

### Génération de l'identifiant EUI-64

L'ID d'interface (64 bits) est dérivé de l'**adresse MAC** (48 bits) :

```
MAC : AA:BB:CC:DD:EE:FF

Étape 1 : Insérer FF:FE au milieu
  AA:BB:CC:FF:FE:DD:EE:FF

Étape 2 : Inverser le 7ème bit (bit U/L) du premier octet
  AA = 10101010 → 10101000 = A8

Résultat EUI-64 : A8:BB:CC:FF:FE:DD:EE:FF
→ ID d'interface : A8BB:CCFF:FEDD:EEFF
```

---

## 5. NDP — Remplacement d'ARP en IPv6

**NDP** (Neighbor Discovery Protocol) remplace **ARP** en IPv6.

Il utilise des messages **ICMPv6** (type 135/136 pour NS/NA, 133/134 pour RS/RA).

### Messages NDP principaux

| Message | Type ICMPv6 | Équivalent IPv4 | Rôle |
|---------|------------|-----------------|------|
| **Neighbor Solicitation (NS)** | 135 | ARP Request | "Qui a cette adresse IPv6 ?" |
| **Neighbor Advertisement (NA)** | 136 | ARP Reply | "C'est moi, voici ma MAC" |
| **Router Solicitation (RS)** | 133 | — | "Y a-t-il un routeur sur ce lien ?" |
| **Router Advertisement (RA)** | 134 | — | "Je suis un routeur, voici le préfixe" |

### DAD — Détection d'adresses dupliquées

Avant d'utiliser une adresse IPv6, la machine envoie un **NS** vers sa propre adresse pour vérifier qu'elle n'est pas déjà utilisée. Si personne ne répond → adresse disponible.

---

## 6. Avantages d'IPv6 vs IPv4

| Avantage | Description |
|----------|-------------|
| **Espace d'adressage** | 2¹²⁸ adresses → quasi illimité |
| **Suppression du NAT** | Chaque machine peut avoir une adresse publique unique |
| **Autoconfiguration** | SLAAC — sans serveur DHCP obligatoire |
| **En-tête simplifié** | Moins de champs → traitement plus rapide par les routeurs |
| **Sécurité native** | IPsec intégré dans les spécifications de base |
| **Multicast** | Remplace le broadcast → plus efficace |
| **Mobilité** | Mobile IPv6 — maintien de connexion en changeant de réseau |
| **QoS améliorée** | Champ Flow Label dans l'en-tête |

---

## 7. Structure du paquet IPv6

```
┌─────────────────────────────────────────────────────────┐
│  Version (4)  │  Traffic Class (8)  │  Flow Label (20)  │
├────────────────────────────┬─────────────┬──────────────┤
│  Payload Length (16)       │ Next Header │   Hop Limit  │
├─────────────────────────────────────────────────────────┤
│              Adresse Source (128 bits)                   │
├─────────────────────────────────────────────────────────┤
│            Adresse Destination (128 bits)                │
└─────────────────────────────────────────────────────────┘
```

- **No fragmentation in transit** : seul l'émetteur peut fragmenter
- **Hop Limit** : équivalent du TTL IPv4
- **Next Header** : indique le protocole de la couche suivante (TCP=6, UDP=17, ICMPv6=58)

---

## PARTIE B — Diagnostic Réseau

## 8. Méthodologie de diagnostic — Règle d'or

> **Toujours diagnostiquer couche par couche, du bas vers le haut.**

```
COUCHE 1 — Physique  →  Le câble est branché ? Le voyant clignote ?
COUCHE 2 — Liaison   →  L'interface est active ? Adresse MAC visible ?
COUCHE 3 — Réseau    →  L'IP est correcte ? La passerelle répond ?
COUCHE 4 — Transport →  Le port est ouvert ? Le service tourne ?
COUCHE 7 — Application → Le service répond bien ?
```

---

## 9. Scénario : Poste ne pouvant pas accéder à Internet

```
ÉTAPE 1 — Tester la pile réseau locale
  ping 127.0.0.1
  ✅ OK → pile TCP/IP fonctionnelle
  ❌ KO → réinstaller/réparer les pilotes réseau

ÉTAPE 2 — Tester la carte réseau et l'IP locale
  ping [adresse_IP_du_poste]
  ✅ OK → interface réseau fonctionnelle
  ❌ KO → problème de carte réseau ou de configuration

ÉTAPE 3 — Tester l'accès à la passerelle
  ping [adresse_gateway]  ex: ping 192.168.1.1
  ✅ OK → lien local opérationnel
  ❌ KO → problème réseau local (câble, switch, VLAN) ou gateway mal configurée

ÉTAPE 4 — Tester la connexion Internet brute (par IP)
  ping 8.8.8.8
  ✅ OK → connexion Internet fonctionnelle
  ❌ KO → problème FAI ou routeur/pare-feu

ÉTAPE 5 — Tester la résolution DNS
  ping www.google.com   ou   nslookup www.google.com
  ✅ OK → DNS fonctionne, le problème vient de l'application
  ❌ KO → problème DNS (serveur DNS inaccessible ou mal configuré)
```

---

## 10. Scénario : Accès local OK mais navigation web lente

| Cause possible | Investigation |
|---------------|---------------|
| **DNS lent** | `nslookup www.google.com` — mesurer le temps de réponse. Tester avec DNS alternatif (8.8.8.8) |
| **DNS primaire inaccessible** | `ping 192.168.1.1` (DNS primaire) — s'il ne répond pas, le client utilise le secondaire après timeout |
| **Lien WAN saturé** | Tester la bande passante (speedtest) — vérifier si tous les utilisateurs sont affectés |
| **Proxy configuré** | Vérifier les paramètres proxy dans le navigateur ou dans les paramètres système |
| **Problème serveur distant** | Tester avec d'autres sites — si un seul site est lent, le problème est côté serveur |

---

## 11. Analyse d'une configuration IP (exemple d'examen)

```
Adresse IP      : 192.168.1.45
Masque          : 255.255.255.0
Passerelle      : 192.168.1.254
DNS primaire    : 192.168.1.1
DNS secondaire  : 8.8.8.8
```

**Check-list de vérification :**

| Vérification | Résultat |
|-------------|---------|
| IP dans le réseau ? | 192.168.1.45 est dans 192.168.1.0/24 ✅ |
| Passerelle dans le réseau ? | 192.168.1.254 est dans 192.168.1.0/24 ✅ |
| DNS primaire accessible ? | `ping 192.168.1.1` |
| DNS secondaire public ? | 8.8.8.8 = DNS Google ✅ |
| Adresse APIPA ? | Non (169.254.x.x serait un signe de DHCP KO) |

---

## 12. Commandes de diagnostic à connaître

### Connectivité
```bash
ping 8.8.8.8              # Test de joignabilité
ping -c 4 8.8.8.8         # Linux : 4 pings
ping -n 4 8.8.8.8         # Windows : 4 pings
ping -t 8.8.8.8           # Windows : ping continu
```

### Routage
```bash
# Linux
ip route                  # Table de routage
traceroute 8.8.8.8        # Chemin réseau
ip route get 8.8.8.8      # Par quelle route ce paquet ?

# Windows
route print               # Table de routage
tracert 8.8.8.8           # Chemin réseau
```

### DNS
```bash
nslookup www.google.com            # Résolution DNS (Linux/Windows)
dig www.google.com                 # Résolution DNS (Linux)
dig +short www.google.com          # Résultat simplifié
dig @1.1.1.1 www.google.com        # Via serveur Cloudflare
```

### Configuration réseau
```bash
# Linux
ip addr                    # Adresses IP et interfaces
ip addr show eth0          # Interface spécifique
ip link                    # État des interfaces

# Windows
ipconfig                   # Config réseau de base
ipconfig /all              # Config complète (avec MAC, DHCP...)
ipconfig /release          # Libère le bail DHCP
ipconfig /renew            # Demande un nouveau bail DHCP
ipconfig /flushdns         # Vide le cache DNS
```

### ARP
```bash
# Linux
ip neigh                   # Table ARP
arp -n                     # Table ARP (ancienne commande)

# Windows
arp -a                     # Table ARP
```

---

## 13. Tableau de synthèse — Erreurs fréquentes et leur cause

| Symptôme | Cause probable | Commande de vérification |
|----------|---------------|--------------------------|
| `ping 127.0.0.1` KO | Pile TCP/IP défaillante | Réinstaller les pilotes |
| `ping gateway` KO | Problème réseau local | Vérifier câble, VLAN, IP |
| `ping 8.8.8.8` KO mais gateway OK | Problème FAI / routeur / pare-feu | Contacter FAI / vérifier routeur |
| `ping 8.8.8.8` OK mais `ping google.com` KO | **Problème DNS** | `nslookup google.com` |
| IP en 169.254.x.x | **DHCP ne répond pas** | Vérifier serveur DHCP, `ipconfig /renew` |
| Navigation lente | DNS lent, lien saturé, proxy | `nslookup` + speedtest |

---

## 14. Points clés à retenir ✅

**IPv6**
- IPv6 = **128 bits** (vs 32 pour IPv4)
- Simplification : supprimer zéros de tête + `::` pour les groupes nuls consécutifs
- **FE80::/10** = link-local (auto-configurée, non routable)
- **2000::/3** = global unicast (adresses publiques)
- **FF00::/8** = multicast (remplace broadcast)
- **SLAAC** = autoconfiguration sans DHCP (RS → RA → préfixe + EUI-64)
- **NDP** remplace ARP : NS/NA (résolution d'adresse), RS/RA (découverte de routeur)
- **::1** = loopback IPv6 (équivalent de 127.0.0.1)

**Diagnostic**
- Toujours diagnostiquer **couche par couche, du bas vers le haut**
- Ordre : `ping 127.0.0.1` → `ping IP locale` → `ping gateway` → `ping 8.8.8.8` → `ping google.com`
- Si ping IP OK mais ping nom KO → **problème DNS**
- IP en **169.254.x.x** → DHCP ne répond pas (APIPA)
- `traceroute/tracert` → identifier **où** la panne se produit dans le chemin réseau

---

*Fiche 6/6 — CP4 Exploiter un réseau IP*
