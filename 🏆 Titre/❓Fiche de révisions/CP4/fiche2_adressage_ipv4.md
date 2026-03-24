# FICHE 2 — Adressage IPv4
### CP4 : Exploiter un réseau IP | Formation TSSR

---

## 1. Structure d'une adresse IPv4

- Codée sur **32 bits** = 4 octets
- Notation décimale pointée : `192.168.1.10`
- Nombre total d'adresses théoriques : **2³² = 4 294 967 296** (~4,3 milliards)

### Structure binaire
```
192      .  168      .  1        .  10
11000000    10101000    00000001    00001010
```

### Partie réseau et partie hôte
```
[ PARTIE RÉSEAU (bits à 1 dans le masque) | PARTIE HÔTE (bits à 0) ]

Exemple /24 :
192.168.1    .    10
└── Réseau ──┘   └── Hôte ──┘
```

---

## 2. Le masque de sous-réseau et la notation CIDR

### Notation CIDR
**CIDR** (Classless Inter-Domain Routing) = notation `/n` indiquant le nombre de bits dédiés à la partie **réseau**.

```
192.168.1.0/24
             ↑
             24 bits pour le réseau → 8 bits restants pour les hôtes
```

### Tableau des masques courants à mémoriser

| CIDR | Masque décimal | Bits hôtes | Nb d'hôtes utilisables | Usage typique |
|------|---------------|------------|------------------------|---------------|
| /8 | 255.0.0.0 | 24 | 16 777 214 | Classe A (grands réseaux) |
| /12 | 255.240.0.0 | 20 | 1 048 574 | RFC 1918 plage B |
| /16 | 255.255.0.0 | 16 | 65 534 | Classe B |
| /20 | 255.255.240.0 | 12 | 4 094 | Sous-réseaux moyens |
| /21 | 255.255.248.0 | 11 | 2 046 | — |
| /22 | 255.255.252.0 | 10 | 1 022 | — |
| /23 | 255.255.254.0 | 9 | 510 | — |
| /24 | 255.255.255.0 | 8 | 254 | Réseau LAN standard |
| /25 | 255.255.255.128 | 7 | 126 | — |
| /26 | 255.255.255.192 | 6 | 62 | — |
| /27 | 255.255.255.224 | 5 | 30 | — |
| /28 | 255.255.255.240 | 4 | 14 | — |
| /29 | 255.255.255.248 | 3 | 6 | — |
| /30 | 255.255.255.252 | 2 | 2 | **Lien point-à-point** |

> **Formule :** Nb hôtes = **2ⁿ - 2** (on enlève adresse réseau et broadcast, où n = bits hôtes)

---

## 3. Les adresses spéciales d'un réseau

Pour tout réseau X.Y.Z.0/24 :

| Adresse | Rôle |
|---------|------|
| **X.Y.Z.0** | Adresse **réseau** (tous les bits hôtes à 0) — non attribuable |
| **X.Y.Z.1 → X.Y.Z.254** | Adresses **hôtes** utilisables |
| **X.Y.Z.255** | Adresse de **broadcast** (tous les bits hôtes à 1) — non attribuable |

---

## 4. Méthode de calcul d'un sous-réseau

### Exemple avec /24 (simple)
```
Réseau    : 192.168.10.0/24
Masque    : 255.255.255.0
Réseau    : 192.168.10.0
Broadcast : 192.168.10.255
Hôtes     : 192.168.10.1 → 192.168.10.254  (254 hôtes)
```

### Exemple avec /20 (intermédiaire)
```
Réseau : 172.16.0.0/20
Masque : 255.255.240.0

Conversion du 3ème octet du masque :
  240 = 11110000  → 4 bits réseau, 4 bits hôtes dans cet octet

Bits hôtes total : 12 bits (8 du 4ème octet + 4 du 3ème)
Nb hôtes : 2¹² - 2 = 4094

Broadcast → mettre tous les bits hôtes à 1 :
  3ème octet : 00000000 OR 00001111 = 00001111 = 15
  4ème octet : 11111111 = 255

Réseau    : 172.16.0.0
Masque    : 255.255.240.0
Broadcast : 172.16.15.255    ← pas 172.16.255.255 !
Hôtes     : 172.16.0.1       → 172.16.15.254
```

### Exemple avec /22 (calcul demandé en exam)
```
Adresse : 10.0.5.42/22
Masque : 255.255.252.0

3ème octet du masque : 252 = 11111100

Convertir le 3ème octet de l'adresse : 5 = 00000101
Appliquer le masque AND : 00000101 AND 11111100 = 00000100 = 4

Réseau    : 10.0.4.0/22
Broadcast : 10.0.7.255
Hôtes     : 10.0.4.1 → 10.0.7.254  (1022 hôtes)
```

### Méthode rapide pour les masques non standard

**Règle :** Le dernier octet "intéressant" du masque donne un "bloc" de sous-réseau.

```
Masque 255.255.252.0 → bloc de 4 dans le 3ème octet
  Réseaux : .0.0, .4.0, .8.0, .12.0...
  L'adresse .5.42 tombe dans le bloc .4.0 → réseau 10.0.4.0

Masque 255.255.240.0 → bloc de 16 dans le 3ème octet
  Réseaux : .0.0, .16.0, .32.0...
  L'adresse .5.42 tombe dans le bloc .0.0 → réseau 172.16.0.0
```

---

## 5. Les adresses RFC 1918 (adresses privées)

Ces plages sont **non routables sur Internet**. Elles nécessitent du NAT pour accéder à Internet.

| Plage                             | CIDR               | Masque      | Nb d'adresses | Usage                        |
| --------------------------------- | ------------------ | ----------- | ------------- | ---------------------------- |
| `10.0.0.0` – `10.255.255.255`     | **10.0.0.0/8**     | 255.0.0.0   | ~16 millions  | Grands réseaux d'entreprise  |
| `172.16.0.0` – `172.31.255.255`   | **172.16.0.0/12**  | 255.240.0.0 | ~1 million    | Réseaux moyens               |
| `192.168.0.0` – `192.168.255.255` | **192.168.0.0/16** | 255.255.0.0 | ~65 000       | Petits réseaux, box internet |

> ⚠️ **Erreur fréquente :** La plage B est `/12` et **non `/20`** ! 172.16.0.0/**12** couvre 172.16.x.x jusqu'à 172.31.x.x.

---

## 6. Adresses IPv4 spéciales

| Adresse | Rôle |
|---------|------|
| `127.0.0.1` (loopback) | La machine elle-même — teste la pile réseau locale |
| `127.0.0.0/8` | Plage loopback complète |
| `0.0.0.0` | Adresse indéfinie / route par défaut |
| `255.255.255.255` | Broadcast local (limité au segment) |
| `169.254.x.x/16` | APIPA — adresse auto-assignée quand le DHCP ne répond pas |

> **Tester la pile réseau :** `ping 127.0.0.1` — si ça répond, TCP/IP fonctionne sur la machine.

---

## 7. Le VLSM — Variable Length Subnet Masking

### Définition
Le VLSM permet de **découper un espace d'adressage en sous-réseaux de tailles différentes**, chacun avec son propre masque adapté à ses besoins réels.

### Pourquoi l'utiliser ?
Sans VLSM, on attribue le même masque à tous les sous-réseaux → **gaspillage d'adresses**.

Exemple : si on a besoin d'un réseau à 500 hôtes ET d'un lien WAN (2 hôtes), sans VLSM on prendrait deux /23 (510 hôtes chacun) → 508 adresses gaspillées sur le lien WAN.

### Méthode VLSM — Règle d'or
> **Toujours commencer par le plus grand besoin et descendre au plus petit.**

### Exemple complet : Réseau 10.10.0.0/16

Besoins : Site A (500 hôtes), Site B (250 hôtes), Site C (100 hôtes), Lien WAN (2 équipements)

| Site | Hôtes requis | Masque choisi | Justification | Réseau | Broadcast | Hôtes dispo |
|------|-------------|---------------|---------------|--------|-----------|-------------|
| **Site A** | 500 | **/23** | 2⁹-2 = 510 ≥ 500 | 10.10.0.0/23 | 10.10.1.255 | 510 |
| **Site B** | 250 | **/24** | 2⁸-2 = 254 ≥ 250 | 10.10.2.0/24 | 10.10.2.255 | 254 |
| **Site C** | 100 | **/25** | 2⁷-2 = 126 ≥ 100 | 10.10.3.0/25 | 10.10.3.127 | 126 |
| **Lien WAN** | 2 | **/30** | 2²-2 = 2 ≥ 2 | 10.10.3.128/30 | 10.10.3.131 | 2 |

> **Conseil :** Toujours choisir la plus petite puissance de 2 qui satisfait le besoin (n bits hôtes tel que 2ⁿ - 2 ≥ nb hôtes requis).

---

## 8. Points clés à retenir ✅

- IPv4 = **32 bits** → **2³²** = ~4,3 milliards d'adresses
- Formule hôtes : **2ⁿ - 2** (n = bits hôtes)
- RFC 1918 : **10.0.0.0/8** | **172.16.0.0/12** | **192.168.0.0/16**
- **172.16.0.0/12** ← pas /20 ! Couvre jusqu'à 172.31.255.255
- Pour calculer le broadcast : mettre **tous les bits hôtes à 1**
- VLSM = toujours **du plus grand besoin au plus petit**
- `/30` = 2 hôtes → idéal pour les **liaisons point-à-point** (WAN, inter-routeurs)
- APIPA `169.254.x.x` = signe que le **DHCP n'a pas répondu**

---

*Fiche 2/6 — CP4 Exploiter un réseau IP*
