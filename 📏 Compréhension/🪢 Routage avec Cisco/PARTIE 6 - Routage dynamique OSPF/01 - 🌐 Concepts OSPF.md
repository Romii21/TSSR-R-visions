

## 📑 Table des matières

```table-of-contents
title: 
style: nestedList # TOC style (nestedList|nestedOrderedList|inlineFirstLevel)
minLevel: 2 # Include headings from the specified level
maxLevel: 2 # Include headings up to the specified level
include: 
exclude: 
includeLinks: true # Make headings clickable
hideWhenEmpty: false # Hide TOC if no headings are found
debugInConsole: false # Print debug info in Obsidian console
```

---

## 🎯 Introduction à OSPF

**OSPF** (Open Shortest Path First) est un protocole de routage dynamique à **état de lien** (link-state) standardisé par l'IETF (RFC 2328 pour OSPFv2, RFC 5340 pour OSPFv3). C'est le protocole IGP (Interior Gateway Protocol) le plus utilisé dans les réseaux d'entreprise.

> [!info] Pourquoi OSPF ?
> 
> - **Convergence rapide** : Détection et réaction quasi-instantanée aux changements de topologie
> - **Sans boucle** : Utilisation de l'algorithme SPF (Dijkstra) garantissant des routes sans boucle
> - **Scalabilité** : Architecture hiérarchique avec zones permettant de supporter de grands réseaux
> - **Métrique intelligente** : Basée sur le coût (bande passante), pas sur le nombre de sauts
> - **Standard ouvert** : Compatible multi-constructeurs

### Différences avec RIP

|Caractéristique|OSPF|RIP|
|---|---|---|
|Type|État de lien|Vecteur de distance|
|Métrique|Coût (bande passante)|Nombre de sauts|
|Limite|Aucune|15 sauts max|
|Convergence|Rapide (secondes)|Lente (minutes)|
|Mises à jour|Événementielles|Périodiques (30s)|
|Hiérarchie|Zones|Non|

---

## 🔗 Protocole à état de lien

### Principe fondamental

Contrairement aux protocoles à vecteur de distance (comme RIP) qui échangent des tables de routage, OSPF échange des **informations sur l'état des liens** du réseau.

> [!example] Analogie **Vecteur de distance** : Vous demandez à votre voisin "Comment aller à Paris ?" et il vous dit "Prends la route à gauche, c'est à 200km"
> 
> **État de lien** : Chaque routeur possède une carte complète du réseau et calcule lui-même le meilleur chemin

### Processus de fonctionnement OSPF

#### 1️⃣ Découverte des voisins

Les routeurs OSPF envoient des paquets **Hello** sur leurs interfaces pour découvrir les voisins OSPF.

```bash
# Paramètres des paquets Hello
- Intervalle Hello : 10 secondes (broadcast/point-to-point)
- Intervalle Dead : 40 secondes (4 × Hello)
- Router ID
- Area ID
- Masque réseau
- Authentification (optionnelle)
```

> [!warning] Conditions d'adjacence Pour devenir voisins, deux routeurs doivent avoir :
> 
> - Même **Area ID**
> - Même **subnet/masque**
> - Même **intervalle Hello/Dead**
> - Même **authentification**
> - Même **stub area flag**

#### 2️⃣ Élection DR/BDR (sur réseaux broadcast)

Sur les réseaux multi-accès (Ethernet), un **DR** (Designated Router) et un **BDR** (Backup DR) sont élus pour optimiser les échanges.

#### 3️⃣ Construction de la base de données d'état de lien (LSDB)

Les routeurs échangent des **LSA** (Link State Advertisements) pour construire une base de données topologique identique sur tous les routeurs de la zone.

```bash
# Types de LSA principaux
Type 1 : Router LSA (topologie interne à la zone)
Type 2 : Network LSA (réseaux multi-accès)
Type 3 : Summary LSA (routes inter-zones)
Type 5 : External LSA (routes externes à OSPF)
```

#### 4️⃣ Calcul SPF (Shortest Path First)

Chaque routeur exécute l'**algorithme de Dijkstra** sur la LSDB pour calculer le chemin le plus court vers chaque destination.

```bash
# Résultat : Un arbre SPF avec le routeur comme racine
Router(config-router)# show ip ospf database
```

#### 5️⃣ Installation des routes

Les meilleures routes sont installées dans la table de routage avec une **distance administrative de 110**.

### Métrique de coût OSPF

La métrique OSPF est le **coût**, calculé ainsi :

```
Coût = 100 000 000 / Bande passante (bps)
```

|Interface|Bande passante|Coût par défaut|
|---|---|---|
|FastEthernet|100 Mbps|1|
|Ethernet|10 Mbps|10|
|T1|1.544 Mbps|64|
|64 kbps|64 kbps|1562|

> [!tip] Astuce Pour les interfaces Gigabit et supérieures, le coût par défaut reste 1. Utilisez la commande `auto-cost reference-bandwidth` pour ajuster :
> 
> ```bash
> Router(config-router)# auto-cost reference-bandwidth 10000
> # Référence = 10 Gbps → GigaEthernet aura coût de 10
> ```

---

## 🗺️ Zones OSPF

### Concept de zones

OSPF utilise une architecture **hiérarchique à deux niveaux** basée sur les zones pour améliorer la scalabilité et réduire la charge CPU/mémoire.

```
┌─────────────────────────────────────────┐
│          Area 0 (Backbone)              │
│   ┌────┐      ┌────┐      ┌────┐      │
│   │ R1 │──────│ R2 │──────│ R3 │      │
│   └────┘      └────┘      └────┘      │
│      │           │           │          │
└──────┼───────────┼───────────┼──────────┘
       │           │           │
  ┌────▼───┐  ┌───▼────┐  ┌──▼─────┐
  │ Area 1 │  │ Area 2 │  │ Area 3 │
  └────────┘  └────────┘  └────────┘
```

### Area 0 - Backbone obligatoire

> [!warning] Règle fondamentale **Toutes les zones OSPF doivent être connectées à l'Area 0** (backbone). C'est une exigence architecturale d'OSPF.

**Pourquoi Area 0 ?**

- Centre du réseau OSPF
- Distribue les informations de routage entre zones
- Empêche les boucles de routage inter-zones

### Avantages du découpage en zones

1. **Réduction de la LSDB** : Chaque zone a sa propre LSDB, limitée à la topologie de la zone
2. **Isolation des changements** : Un changement dans Area 1 ne déclenche pas de recalcul SPF dans Area 2
3. **Réduction du trafic** : Moins de LSA échangés
4. **Meilleure stabilité** : Problèmes isolés par zone

### Types de zones spéciales

|Type de zone|Description|Bloque|
|---|---|---|
|**Standard**|Zone normale OSPF|Rien|
|**Stub**|Pas de routes externes (Type 5)|LSA Type 5|
|**Totally Stub**|Pas de routes externes ni inter-zones|LSA Type 3, 4, 5|
|**NSSA**|Stub avec import limité d'externes|LSA Type 5 (remplacé par Type 7)|

> [!info] Quand utiliser une zone Stub ?
> 
> - Zone en périphérie sans connexion externe
> - Routeurs avec ressources limitées (mémoire/CPU)
> - Simplification de la table de routage

---

## 🆔 Router ID

### Définition

Le **Router ID** est un identifiant unique **au format d'une adresse IP** (32 bits) qui identifie de manière univoque chaque routeur OSPF dans le domaine de routage.

> [!warning] Attention Le Router ID **n'est PAS une adresse IP routable**, c'est juste un identifiant utilisant le format IPv4.

### Sélection automatique du Router ID

Si non configuré manuellement, le Router ID est choisi selon cet ordre de priorité :

```
1. Router ID configuré manuellement (recommandé)
   ↓
2. Adresse IP la plus élevée parmi les interfaces loopback UP
   ↓
3. Adresse IP la plus élevée parmi les interfaces physiques UP
```

> [!example] Exemple de sélection
> 
> ```bash
> # Routeur avec interfaces :
> GigabitEthernet0/0 : 192.168.1.1
> GigabitEthernet0/1 : 10.0.0.1
> Loopback0 : 1.1.1.1
> 
> # Router ID automatique = 1.1.1.1 (loopback la plus élevée)
> ```

### Configuration manuelle (recommandée)

```bash
Router(config)# router ospf 1
Router(config-router)# router-id 1.1.1.1
```

> [!tip] Bonne pratique Utilisez toujours une interface **Loopback** pour le Router ID ou configurez-le manuellement :
> 
> - Les loopbacks ne tombent jamais (toujours UP)
> - Stabilité du Router ID
> - Convention : `x.x.x.x` où x = numéro du routeur

### Modification du Router ID

Le Router ID ne peut être changé qu'après un redémarrage du processus OSPF :

```bash
Router(config-router)# router-id 2.2.2.2
% OSPF: Reload or use "clear ip ospf process" command, for this to take effect

Router# clear ip ospf process
Reset ALL OSPF processes? [no]: yes
```

### Vérification

```bash
Router# show ip protocols
*** IP Routing is NSF aware ***

Routing Protocol is "ospf 1"
  Outgoing update filter list for all interfaces is not set
  Incoming update filter list for all interfaces is not set
  Router ID 1.1.1.1    ← Router ID actif
  Number of areas in this router is 1. 1 normal 0 stub 0 nssa
```

---

## 👥 Types de routeurs OSPF

Sur les réseaux **multi-accès** (comme Ethernet), OSPF optimise les échanges en élisant des routeurs spéciaux.

### DR - Designated Router

Le **DR** est le routeur central de communication sur un segment multi-accès.

**Rôles :**

- Reçoit les LSA de tous les routeurs du segment
- Génère le Network LSA (Type 2) pour le segment
- Redistribue les LSA à tous les routeurs du segment
- Réduit le nombre d'adjacences nécessaires

### BDR - Backup Designated Router

Le **BDR** est le routeur de secours qui prend automatiquement le relais si le DR tombe.

**Rôles :**

- Écoute tous les échanges
- Maintient une adjacence avec le DR et tous les routeurs
- Devient DR instantanément si le DR tombe
- N'assume pas de rôle actif tant que le DR fonctionne

### DRother - Routeurs ordinaires

Tous les autres routeurs du segment sont des **DRother**.

**Rôles :**

- Forment une adjacence UNIQUEMENT avec DR et BDR
- Envoient leurs LSA à l'adresse multicast 224.0.0.6 (AllDRouters)
- Reçoivent les LSA via l'adresse 224.0.0.5 (AllSPFRouters)

### Schéma des adjacences

```
Sans DR/BDR (5 routeurs) :          Avec DR/BDR :
10 adjacences nécessaires !          4 adjacences seulement

    R1 ─── R2                           R1      R2
    │ ╲   ╱ │                            │      │
    │  ╲ ╱  │                            │      │
    │   ╳   │                          ┌─┴──────┴─┐
    │  ╱ ╲  │                          │    DR    │
    │ ╱   ╲ │                          └─┬──────┬─┘
    R3 ─── R4                            │      │
       ╲   ╱                             │      │
        ╲ ╱                             R3      R4
         R5                            (BDR)  (DRother)
```

### Processus d'élection DR/BDR

L'élection se fait selon ces critères (dans l'ordre) :

```
1. Priorité OSPF la plus élevée (0-255, défaut = 1)
   ↓
2. Router ID le plus élevé
```

> [!warning] L'élection est NON préemptive Une fois élu, le DR reste en place même si un routeur avec une priorité supérieure arrive. Le DR ne change que s'il tombe ou si le processus OSPF redémarre.

### Configuration de la priorité

```bash
# Par défaut, priorité = 1
Router(config)# interface GigabitEthernet0/0
Router(config-if)# ip ospf priority 100

# Priorité 0 = Ne jamais devenir DR/BDR
Router(config-if)# ip ospf priority 0
```

> [!tip] Stratégie de priorité
> 
> - **Routeurs puissants centraux** : Priorité 100-255
> - **Routeurs normaux** : Priorité par défaut (1)
> - **Routeurs périphériques/faibles** : Priorité 0

### Vérification

```bash
Router# show ip ospf neighbor

Neighbor ID     Pri   State           Dead Time   Address         Interface
2.2.2.2         1     FULL/DR         00:00:38    192.168.1.2     Gi0/0
3.3.3.3         1     FULL/BDR        00:00:32    192.168.1.3     Gi0/0
4.4.4.4         1     FULL/DROTHER    00:00:35    192.168.1.4     Gi0/0
```

**États d'adjacence :**

- **FULL** : Adjacence complète, LSDB synchronisée
- **2-WAY** : Voisins détectés (état normal pour DRother entre eux)

---

## 🔌 Types de réseaux OSPF

OSPF s'adapte au type de média réseau et optimise son comportement en conséquence.

### 1. Broadcast (Multi-accès)

**Exemples :** Ethernet, Fast Ethernet, Gigabit Ethernet

**Caractéristiques :**

- Plusieurs routeurs sur le même segment
- Élection DR/BDR automatique
- Utilisation du multicast (224.0.0.5 et 224.0.0.6)
- Timers : Hello 10s, Dead 40s

```bash
Router(config-if)# ip ospf network broadcast
```

> [!info] Comportement par défaut C'est le type par défaut sur les interfaces Ethernet.

**Adjacences :**

- DR/BDR : Adjacences FULL avec tous
- DRother : Adjacences FULL uniquement avec DR/BDR, 2-WAY entre eux

### 2. Point-to-Point

**Exemples :** Liaison série, PPP, HDLC, sous-interfaces point-to-point

**Caractéristiques :**

- Seulement 2 routeurs sur le lien
- **PAS d'élection DR/BDR** (inutile)
- Adjacence directe entre les deux routeurs
- Utilisation du multicast 224.0.0.5
- Timers : Hello 10s, Dead 40s

```bash
Router(config-if)# ip ospf network point-to-point
```

> [!tip] Quand utiliser ?
> 
> - Liaisons série directes
> - Tunnels GRE/IPsec
> - Sous-interfaces point-to-point Frame Relay/ATM
> - Pour éviter l'élection DR/BDR sur des VLANs avec seulement 2 routeurs

**Avantages :**

- Convergence plus rapide (pas d'attente d'élection)
- Configuration plus simple
- Moins de overhead

### 3. Point-to-Multipoint

**Exemples :** Frame Relay/ATM sans réseau maillé complet

**Caractéristiques :**

- Un lien physique vers plusieurs destinations
- **PAS d'élection DR/BDR**
- Adjacences entre tous les routeurs
- Hello en unicast vers chaque voisin
- Timers : Hello 30s, Dead 120s

```bash
Router(config-if)# ip ospf network point-to-multipoint
```

### 4. Non-Broadcast (NBMA)

**Exemples :** Frame Relay, X.25, ATM (maillage complet)

**Caractéristiques :**

- Multi-accès mais sans capacité broadcast
- Élection DR/BDR nécessaire
- Configuration manuelle des voisins (pas de multicast)
- Timers : Hello 30s, Dead 120s

```bash
Router(config-if)# ip ospf network non-broadcast
Router(config-router)# neighbor 10.1.1.2
```

> [!warning] Configuration complexe NBMA nécessite la configuration manuelle de tous les voisins et une topologie en étoile (hub-and-spoke) pour que le DR soit au centre.

### 5. Point-to-Multipoint Non-Broadcast

Combinaison de point-to-multipoint et NBMA.

```bash
Router(config-if)# ip ospf network point-to-multipoint non-broadcast
Router(config-router)# neighbor 10.1.1.2
```

### Tableau récapitulatif

|Type réseau|DR/BDR|Hello/Dead|Découverte auto|Utilisation|
|---|---|---|---|---|
|**Broadcast**|Oui|10s/40s|Oui (multicast)|Ethernet|
|**Point-to-Point**|Non|10s/40s|Oui (multicast)|Série, Tunnels|
|**Point-to-Multipoint**|Non|30s/120s|Oui (unicast)|Frame Relay partiel|
|**NBMA**|Oui|30s/120s|Non (manuel)|Frame Relay complet|
|**P2MP Non-Broadcast**|Non|30s/120s|Non (manuel)|Rare|

### Vérification du type de réseau

```bash
Router# show ip ospf interface GigabitEthernet0/0

GigabitEthernet0/0 is up, line protocol is up
  Internet Address 192.168.1.1/24, Area 0
  Process ID 1, Router ID 1.1.1.1, Network Type BROADCAST, Cost: 1
  Transmit Delay is 1 sec, State DR, Priority 1
  Designated Router (ID) 1.1.1.1, Interface address 192.168.1.1
  Backup Designated router (ID) 2.2.2.2, Interface address 192.168.1.2
  Timer intervals configured, Hello 10, Dead 40, Wait 40, Retransmit 5
```

> [!tip] Modification du type de réseau Changer le type de réseau sur une liaison point-to-point en `point-to-point` évite l'élection DR/BDR inutile et accélère la convergence :
> 
> ```bash
> Router(config)# interface GigabitEthernet0/0
> Router(config-if)# ip ospf network point-to-point
> ```

---

## 🎯 Points clés à retenir

> [!tip] Résumé OSPF
> 
> 1. **État de lien** : Chaque routeur connaît la topologie complète de sa zone
> 2. **Area 0** : Backbone obligatoire, toutes les zones doivent s'y connecter
> 3. **Router ID** : Identifiant unique, configurez-le manuellement ou via loopback
> 4. **DR/BDR** : Optimisation sur réseaux broadcast, élection basée sur priorité
> 5. **Types de réseau** : Adaptez le comportement OSPF au média physique

> [!warning] Pièges courants
> 
> - ❌ Ne pas configurer Area 0 (backbone)
> - ❌ Oublier que l'élection DR/BDR est non-préemptive
> - ❌ Laisser le Router ID se sélectionner automatiquement
> - ❌ Ne pas ajuster le type de réseau sur liaisons point-to-point
> - ❌ Utiliser des timers Hello/Dead différents entre voisins

---

_Cours réalisé pour la configuration de routeurs Cisco - Partie : Routage dynamique OSPF_