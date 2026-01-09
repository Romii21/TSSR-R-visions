

## 📋 Table des matières

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

## 🎯 Introduction à la vérification OSPF {#introduction}

La vérification et le dépannage OSPF sont des compétences essentielles pour tout administrateur réseau. OSPF étant un protocole à état de liens complexe, il nécessite une compréhension approfondie de ses mécanismes internes pour identifier et résoudre les problèmes.

> [!info] Pourquoi la vérification est cruciale
> 
> - OSPF peut sembler fonctionner alors que la topologie n'est pas optimale
> - Les adjacences peuvent être instables sans symptômes évidents
> - Les problèmes de base de données LSDB peuvent causer des boucles de routage
> - La convergence peut être lente sans que ce soit immédiatement visible

### Approche de vérification en trois niveaux

1. **Niveau Adjacence** : Vérifier que les voisins OSPF sont correctement formés
2. **Niveau Interface** : S'assurer que les interfaces OSPF sont configurées correctement
3. **Niveau Base de données** : Confirmer que la LSDB est cohérente et complète

---

## 👥 Show ip ospf neighbor {#show-ip-ospf-neighbor}

Cette commande est votre **premier réflexe** lors du dépannage OSPF. Elle affiche l'état des relations de voisinage OSPF.

### Syntaxe et utilisation

```bash
Router# show ip ospf neighbor
Router# show ip ospf neighbor detail
Router# show ip ospf neighbor [interface-type interface-number]
```

### Interprétation de la sortie

```bash
Router# show ip ospf neighbor

Neighbor ID     Pri   State           Dead Time   Address         Interface
192.168.3.1       1   FULL/DR         00:00:37    10.1.1.1        GigabitEthernet0/0
192.168.3.2       1   FULL/BDR        00:00:39    10.1.1.2        GigabitEthernet0/0
192.168.4.1       0   FULL/  -        00:00:35    10.2.2.1        Serial0/0/0
```

### Colonnes expliquées en détail

|Colonne|Description|Valeurs possibles|Signification|
|---|---|---|---|
|**Neighbor ID**|Router ID du voisin OSPF|Adresse IP format|Identifiant unique du routeur voisin|
|**Pri**|Priorité OSPF|0-255|Utilisé pour l'élection DR/BDR (0 = ne peut pas être DR/BDR)|
|**State**|État de l'adjacence|DOWN, INIT, 2WAY, EXSTART, EXCHANGE, LOADING, FULL|État actuel de la relation de voisinage|
|**Dead Time**|Temps avant expiration|Secondes (countdown)|Temps restant avant de déclarer le voisin mort|
|**Address**|Adresse IP du voisin|Adresse IP|Adresse IP de l'interface voisine|
|**Interface**|Interface locale|Nom d'interface|Interface locale connectée au voisin|

> [!example] Exemples d'états normaux
> 
> - **FULL/DR** : Adjacence complète avec le Designated Router
> - **FULL/BDR** : Adjacence complète avec le Backup Designated Router
> - **FULL/DROTHER** : Adjacence complète avec un routeur non-DR/BDR
> - **FULL/ -** : Adjacence complète sur lien point-à-point (pas de DR/BDR)
> - **2WAY/DROTHER** : Communication bidirectionnelle établie (normal pour routeurs non-DR/BDR entre eux)

### Commande avec option detail

```bash
Router# show ip ospf neighbor detail
```

Cette variante fournit des informations supplémentaires :

- **Options OSPF** négociées (E-bit pour zones non-stub, etc.)
- **Dead timer** exact et intervalles hello
- **État de synchronisation** de la base de données
- **Nombre de retransmissions** de LSA
- **Informations d'authentification** (si configurée)

```bash
Neighbor 192.168.3.1, interface address 10.1.1.1
    In the area 0 via interface GigabitEthernet0/0
    Neighbor priority is 1, State is FULL, 6 state changes
    DR is 10.1.1.1 BDR is 10.1.1.2
    Options is 0x52
    LLS Options is 0x1 (LR)
    Dead timer due in 00:00:37
    Neighbor is up for 01:23:45
    Index 1/1, retransmission queue length 0, number of retransmission 2
    First 0x0(0)/0x0(0) Next 0x0(0)/0x0(0)
    Last retransmission scan length is 1, maximum is 1
    Last retransmission scan time is 0 msec, maximum is 0 msec
```

> [!warning] Signaux d'alerte
> 
> - **État bloqué** : Un voisin reste en INIT, EXSTART ou EXCHANGE
> - **Dead Time qui réinitialise constamment** : Instabilité de lien
> - **Neighbor ID 0.0.0.0** : Problème de configuration du Router ID
> - **Priorité inattendue** : Peut causer des élections DR/BDR incorrectes
> - **Nombre élevé de state changes** : Instabilité chronique (flapping)

### Que vérifier avec cette commande

✅ **Checklist de vérification** :

- [ ] Tous les voisins attendus sont présents
- [ ] L'état est FULL (ou 2WAY si approprié)
- [ ] Les Dead Timers ne sont pas anormalement courts
- [ ] Les Router IDs sont uniques
- [ ] Le DR/BDR est élu correctement sur les réseaux multi-accès

> [!tip] Astuce de dépannage Si un voisin n'apparaît pas du tout, vérifiez d'abord la connectivité IP de base (ping) avant de chercher un problème OSPF spécifique.

---

## 🔌 Show ip ospf interface {#show-ip-ospf-interface}

Cette commande affiche les détails de configuration OSPF pour chaque interface. Elle est essentielle pour vérifier les paramètres OSPF au niveau interface.

### Syntaxe

```bash
Router# show ip ospf interface
Router# show ip ospf interface brief
Router# show ip ospf interface [interface-type interface-number]
```

### Sortie détaillée

```bash
Router# show ip ospf interface GigabitEthernet0/0

GigabitEthernet0/0 is up, line protocol is up
  Internet Address 10.1.1.3/24, Area 0, Attached via Network Statement
  Process ID 1, Router ID 192.168.3.3, Network Type BROADCAST, Cost: 1
  Topology-MTID    Cost    Disabled    Shutdown      Topology Name
        0           1         no          no            Base
  Enabled by interface config, including secondary ip addresses
  Transmit Delay is 1 sec, State BDR, Priority 1
  Designated Router (ID) 192.168.3.1, Interface address 10.1.1.1
  Backup Designated router (ID) 192.168.3.3, Interface address 10.1.1.3
  Timer intervals configured, Hello 10, Dead 40, Wait 40, Retransmit 5
    oob-resync timeout 40
    Hello due in 00:00:08
  Supports Link-local Signaling (LLS)
  Cisco NSF helper support enabled
  IETF NSF helper support enabled
  Index 1/1, flood queue length 0
  Next 0x0(0)/0x0(0)
  Last flood scan length is 1, maximum is 2
  Last flood scan time is 0 msec, maximum is 0 msec
  Neighbor Count is 2, Adjacent neighbor count is 2
    Adjacent with neighbor 192.168.3.1  (Designated Router)
    Adjacent with neighbor 192.168.3.2
  Suppress hello for 0 neighbor(s)
```

### Informations clés à analyser

#### 1. Configuration de base

|Paramètre|Description|Importance|
|---|---|---|
|**Internet Address**|Adresse IP et masque|Doit correspondre au plan d'adressage|
|**Area**|Zone OSPF|Doit être cohérent avec les voisins|
|**Process ID**|ID du processus OSPF local|Peut différer entre routeurs|
|**Router ID**|Identifiant du routeur|Doit être unique dans l'AS|
|**Network Type**|Type de réseau OSPF|Doit correspondre au média physique|
|**Cost**|Coût de l'interface|Impacte le calcul SPF|

#### 2. État de l'interface

```bash
State BDR, Priority 1
Designated Router (ID) 192.168.3.1, Interface address 10.1.1.1
Backup Designated router (ID) 192.168.3.3, Interface address 10.1.1.3
```

- **State** : Rôle de l'interface (DR, BDR, DROTHER, ou P2P)
- **Priority** : Valeur de priorité pour l'élection DR/BDR
- **DR/BDR** : Identité du DR et BDR actuels

#### 3. Timers OSPF

```bash
Timer intervals configured, Hello 10, Dead 40, Wait 40, Retransmit 5
```

|Timer|Par défaut|Rôle|Impact si mal configuré|
|---|---|---|---|
|**Hello**|10s (broadcast/P2P), 30s (NBMA)|Fréquence des hello packets|Adjacences ne se forment pas|
|**Dead**|40s (broadcast/P2P), 120s (NBMA)|Délai avant de déclarer voisin mort|Convergence lente ou faux positifs|
|**Wait**|= Dead Timer|Temps d'attente avant élection DR/BDR|DR/BDR sous-optimaux|
|**Retransmit**|5s|Intervalle de retransmission LSA|Performance dégradée|

> [!warning] Incompatibilité de timers Les timers Hello et Dead **DOIVENT** être identiques entre voisins sur le même segment, sinon l'adjacence ne se formera pas.

#### 4. Compteurs de voisins

```bash
Neighbor Count is 2, Adjacent neighbor count is 2
  Adjacent with neighbor 192.168.3.1  (Designated Router)
  Adjacent with neighbor 192.168.3.2
```

- **Neighbor Count** : Nombre total de voisins découverts
- **Adjacent neighbor count** : Nombre de voisins avec lesquels une adjacence complète est établie

> [!info] Normal vs Anormal Sur un réseau broadcast :
> 
> - **Normal** : Les DROTHER ont 2 adjacences (avec DR et BDR)
> - **Normal** : Le DR a des adjacences avec tous les routeurs
> - **Anormal** : Neighbor Count > Adjacent count de manière inattendue

### Version brief

```bash
Router# show ip ospf interface brief

Interface    PID   Area            IP Address/Mask    Cost  State Nbrs F/C
Gi0/0        1     0               10.1.1.3/24        1     BDR   2/2
Gi0/1        1     0               10.3.3.1/24        1     DR    1/1
Se0/0/0      1     0               10.2.2.2/30        64    P2P   1/1
```

|Colonne|Signification|
|---|---|
|**Interface**|Nom de l'interface|
|**PID**|Process ID OSPF|
|**Area**|Zone OSPF|
|**IP Address/Mask**|Configuration IP|
|**Cost**|Coût OSPF de l'interface|
|**State**|État OSPF (DR/BDR/DROTHER/P2P)|
|**Nbrs F/C**|Voisins Full / Total Count|

> [!tip] Utilisation pratique La version `brief` est idéale pour un aperçu rapide de toutes les interfaces OSPF et identifier rapidement les problèmes (ex: Nbrs 0/2 indique un problème).

### Problèmes courants détectables

❌ **Interface passive non intentionnelle**

```bash
GigabitEthernet0/2 is up, line protocol is up
  Internet Address 10.4.4.1/24, Area 0, Attached via Network Statement
  Process ID 1, Router ID 192.168.3.3, Network Type BROADCAST, Cost: 1
  Transmit Delay is 1 sec, State WAITING, Priority 1
  No Hellos (Passive interface)
```

❌ **Mauvais type de réseau**

```bash
Serial0/0/0 is up, line protocol is up
  Internet Address 10.2.2.2/30, Area 0
  Network Type NON_BROADCAST  ← Devrait être POINT_TO_POINT
```

❌ **Coût incohérent**

```bash
GigabitEthernet0/0 is up, line protocol is up
  Cost: 100  ← Anormalement élevé pour un lien Gigabit
```

> [!tip] Astuce d'optimisation Utilisez `show ip ospf interface brief | include Down` pour identifier rapidement les interfaces OSPF down.

---

## 📊 Show ip ospf database {#show-ip-ospf-database}

La base de données LSDB (Link-State Database) est le cœur d'OSPF. Cette commande permet d'examiner tous les LSA (Link-State Advertisement) connus par le routeur.

### Syntaxe

```bash
Router# show ip ospf database
Router# show ip ospf database [lsa-type]
Router# show ip ospf database [lsa-type] [link-state-id]
Router# show ip ospf database [lsa-type] [advertising-router router-id]
Router# show ip ospf database database-summary
```

### Types de LSA

|Type|Nom|Généré par|Portée|Rôle|
|---|---|---|---|---|
|**Type 1**|Router LSA|Tous les routeurs|Intra-zone|Décrit les liens du routeur|
|**Type 2**|Network LSA|DR uniquement|Intra-zone|Décrit le réseau multi-accès|
|**Type 3**|Summary LSA|ABR|Inter-zones|Résume les routes entre zones|
|**Type 4**|ASBR Summary|ABR|Inter-zones|Localise l'ASBR|
|**Type 5**|External LSA|ASBR|Tout l'AS|Routes externes redistribuées|
|**Type 7**|NSSA External|ASBR en NSSA|Zone NSSA|Routes externes en zone NSSA|

### Vue d'ensemble de la base de données

```bash
Router# show ip ospf database

            OSPF Router with ID (192.168.3.3) (Process ID 1)

                Router Link States (Area 0)

Link ID         ADV Router      Age         Seq#       Checksum Link count
192.168.3.1     192.168.3.1     245         0x80000004 0x00B2C8 3
192.168.3.2     192.168.3.2     312         0x80000003 0x00A4D1 2
192.168.3.3     192.168.3.3     187         0x80000005 0x009ED3 4

                Net Link States (Area 0)

Link ID         ADV Router      Age         Seq#       Checksum
10.1.1.1        192.168.3.1     245         0x80000002 0x007AB2

                Summary Net Link States (Area 0)

Link ID         ADV Router      Age         Seq#       Checksum
10.5.5.0        192.168.1.1     892         0x80000001 0x00D4E3
172.16.0.0      192.168.1.1     892         0x80000001 0x00A2F1

                Type-5 AS External Link States

Link ID         ADV Router      Age         Seq#       Checksum Tag
0.0.0.0         192.168.4.1     1523        0x80000001 0x00E8A2 1
```

### Colonnes expliquées

|Colonne|Description|Utilité|
|---|---|---|
|**Link ID**|Identifiant du LSA|Type 1: Router ID<br>Type 2: IP du DR<br>Type 3/5: Réseau annoncé|
|**ADV Router**|Router ID de l'émetteur|Identifie qui a créé le LSA|
|**Age**|Âge en secondes|MaxAge = 3600s (1h), ensuite flush|
|**Seq#**|Numéro de séquence|Plus récent = numéro plus élevé|
|**Checksum**|Somme de contrôle|Détection de corruption|
|**Link count**|Nombre de liens (Type 1)|Connectivité du routeur|
|**Tag**|Tag de route (Type 5)|Marquage de routes externes|

> [!info] Importance du numéro de séquence
> 
> - Commence à 0x80000001
> - Incrémente à chaque mise à jour
> - MaxSeq = 0x7FFFFFFF
> - Permet de déterminer quelle version du LSA est la plus récente

### Examen détaillé d'un Router LSA (Type 1)

```bash
Router# show ip ospf database router 192.168.3.1

            OSPF Router with ID (192.168.3.3) (Process ID 1)

                Router Link States (Area 0)

  LS age: 287
  Options: (No TOS-capability, DC)
  LS Type: Router Links
  Link State ID: 192.168.3.1
  Advertising Router: 192.168.3.1
  LS Seq Number: 80000004
  Checksum: 0xB2C8
  Length: 60
  Number of Links: 3

    Link connected to: a Transit Network
     (Link ID) Designated Router address: 10.1.1.1
     (Link Data) Router Interface address: 10.1.1.1
      Number of MTID metrics: 0
       TOS 0 Metrics: 1

    Link connected to: a Stub Network
     (Link ID) Network/subnet number: 192.168.3.1
     (Link Data) Network Mask: 255.255.255.255
      Number of MTID metrics: 0
       TOS 0 Metrics: 1

    Link connected to: another Router (point-to-point)
     (Link ID) Neighboring Router ID: 192.168.4.1
     (Link Data) Router Interface address: 10.2.2.1
      Number of MTID metrics: 0
       TOS 0 Metrics: 64
```

#### Informations détaillées

**Options** :

- **E-bit** : Peut recevoir des LSA externes (désactivé dans stub areas)
- **DC-bit** : Demand Circuit supporté
- **O-bit** : Opaque LSA supporté

**Types de liens** :

1. **Transit Network** : Connexion à un réseau multi-accès (ethernet)
2. **Stub Network** : Réseau sans autres routeurs OSPF
3. **Point-to-Point** : Liaison directe à un autre routeur
4. **Virtual Link** : Lien logique à travers une zone

> [!example] Interprétation pratique Ce Router LSA montre que le routeur 192.168.3.1 :
> 
> - Est connecté à un réseau transit via 10.1.1.1 (coût 1)
> - A une loopback 192.168.3.1/32
> - Est lié en point-à-point au routeur 192.168.4.1 (coût 64)

### Examen d'un Network LSA (Type 2)

```bash
Router# show ip ospf database network 10.1.1.1

            OSPF Router with ID (192.168.3.3) (Process ID 1)

                Net Link States (Area 0)

  LS age: 312
  Options: (No TOS-capability, DC)
  LS Type: Network Links
  Link State ID: 10.1.1.1 (address of Designated Router)
  Advertising Router: 192.168.3.1
  LS Seq Number: 80000002
  Checksum: 0x7AB2
  Length: 32
  Network Mask: /24
        Attached Router: 192.168.3.1
        Attached Router: 192.168.3.2
        Attached Router: 192.168.3.3
```

> [!info] Rôle du Network LSA Créé uniquement par le DR sur les réseaux multi-accès pour :
> 
> - Indiquer le masque du réseau
> - Lister tous les routeurs attachés au segment
> - Éviter les LSA redondants de chaque routeur

### Examen d'un Summary LSA (Type 3)

```bash
Router# show ip ospf database summary 10.5.5.0

            OSPF Router with ID (192.168.3.3) (Process ID 1)

                Summary Net Link States (Area 0)

  LS age: 978
  Options: (No TOS-capability, DC, Upward)
  LS Type: Summary Links(Network)
  Link State ID: 10.5.5.0 (summary Network Number)
  Advertising Router: 192.168.1.1
  LS Seq Number: 80000001
  Checksum: 0xD4E3
  Length: 28
  Network Mask: /24
        MTID: 0         Metric: 10
```

> [!warning] Importance des Summary LSA
> 
> - Générés par les ABR pour annoncer des routes entre zones
> - Le coût indiqué est le coût depuis l'ABR jusqu'au réseau destination
> - Permettent l'agrégation de routes (summarization)

### Examen d'un External LSA (Type 5)

```bash
Router# show ip ospf database external 0.0.0.0

            OSPF Router with ID (192.168.3.3) (Process ID 1)

                Type-5 AS External Link States

  LS age: 1634
  Options: (No TOS-capability, DC, Upward)
  LS Type: AS External Link
  Link State ID: 0.0.0.0 (External Network Number)
  Advertising Router: 192.168.4.1
  LS Seq Number: 80000001
  Checksum: 0xE8A2
  Length: 36
  Network Mask: /0
        Metric Type: 2 (Larger than any link state path)
        MTID: 0
        Metric: 1
        Forward Address: 0.0.0.0
        External Route Tag: 1
```

#### Métrique Type 1 vs Type 2

|Aspect|Type 1 (E1)|Type 2 (E2)|
|---|---|---|
|**Calcul**|Coût externe + coût interne|Coût externe uniquement|
|**Défaut**|Non|**Oui** (Type 2)|
|**Usage**|Routes avec coûts comparables|Routes de dernier recours|
|**Préférence**|E1 préféré à E2|E2 si même métrique externe|

> [!tip] Choix du type de métrique
> 
> - **Type 2** : Par défaut, pour les routes Internet ou routes de backup
> - **Type 1** : Pour des routes où le coût interne compte (ex: plusieurs ASBR)

### Commande database-summary

```bash
Router# show ip ospf database database-summary

            OSPF Router with ID (192.168.3.3) (Process ID 1)

Area 0 database summary
  LSA Type      Count    Delete   Maxage
  Router        3        0        0
  Network       1        0        0
  Summary Net   2        0        0
  Summary ASBR  0        0        0
  Type-7 Ext    0        0        0
    Prefixes redistributed in Type-7  0
  Opaque Link   0        0        0
  Opaque Area   0        0        0
  Subtotal      6        0        0

Process 1 database summary
  LSA Type      Count    Delete   Maxage
  Router        3        0        0
  Network       1        0        0
  Summary Net   2        0        0
  Summary ASBR  0        0        0
  Type-5 Ext    1        0        0
    Prefixes redistributed in Type-5  1
  Opaque AS     0        0        0
  Total         7        0        0
```

> [!info] Utilité du summary
> 
> - Donne un aperçu rapide de la taille de la LSDB
> - **Delete** : LSA en cours de suppression
> - **Maxage** : LSA ayant atteint l'âge maximum (prêts à être flushés)
> - Utile pour détecter une base de données anormalement volumineuse

### Problèmes courants détectables

❌ **LSA différents entre routeurs**

```bash
# Sur routeur A
Router-A# show ip ospf database | include 10.3.3.0
# Aucune sortie

# Sur routeur B
Router-B# show ip ospf database | include 10.3.3.0
10.3.3.0        192.168.5.1     234         0x80000002 0x00A1B3
```

→ Indique un problème de synchronisation ou de partition de zone

❌ **LSA avec âge MaxAge persistant**

```bash
Link ID         ADV Router      Age         Seq#       Checksum
10.7.7.0        192.168.6.1     3600        0x80000012 0x00C2D4
```

→ Le LSA ne peut pas être supprimé, possible partition

❌ **Nombre excessif de LSA Type 5**

```bash
Type-5 Ext    15847        0        0
```

→ Trop de routes redistribuées, envisager la summarization

> [!tip] Commande de comparaison Pour comparer rapidement les LSDB entre deux routeurs :
> 
> ```bash
> show ip ospf database database-summary
> ```
> 
> Les compteurs doivent être identiques dans la même zone.

---

## 🔄 États OSPF et adjacences {#états-ospf-et-adjacences}

La formation d'une adjacence OSPF passe par plusieurs états. Comprendre ces états est crucial pour le dépannage.

### Machine à états OSPF

```
DOWN → INIT → 2WAY → EXSTART → EXCHANGE → LOADING → FULL
```

### États détaillés

#### 1. DOWN

**Description** : Aucun Hello reçu du voisin

**Caractéristiques** :

- État initial
- Aucun paquet Hello reçu récemment
- Le voisin peut être dans la table de configuration mais pas actif

**Causes courantes** :

- Interface down physiquement
- Problème de connectivité L2
- Configuration OSPF absente sur le voisin
- ACL bloquant les paquets OSPF (224.0.0.5 ou 224.0.0.6)

> [!warning] Diagnostic DOWN Si l'état reste DOWN :
> 
> 1. Vérifier `show ip interface brief` (interface up?)
> 2. Tester la connectivité (ping)
> 3. Vérifier `show ip ospf interface` (OSPF activé?)
> 4. Capturer les paquets (`debug ip ospf hello`)

---

#### 2. INIT

**Description** : Hello reçu du voisin, mais le voisin ne liste pas encore notre Router ID

**Caractéristiques** :

- Communication unidirectionnelle établie
- Notre routeur voit le voisin
- Le voisin ne nous voit pas encore (ou pas correctement)

**Causes courantes** :

- **Mismatch de subnet** : Routeurs sur des subnets différents
- **Mismatch de masque** : Masques différents sur le même segment
- **ACL asymétrique** : Paquets bloqués dans un sens seulement
- **Problème matériel** : Carte réseau, câble, port switch défaillant en TX ou RX

> [!example] Exemple de diagnostic INIT
> 
> ```bash
> Router-A# show ip ospf neighbor
> Neighbor ID     Pri   State           Dead Time   Address         Interface
> 192.168.3.2       1   INIT/DROTHER    00:00:35    10.1.1.2        Gi0/0
> 
> # Vérifier les adresses IP
> Router-A# show ip interface brief | include Gi0/0
> GigabitEthernet0/0     10.1.1.1        YES manual up                    up
> 
> # Sur le voisin
> Router-B# show ip interface brief | include Gi0/0
> GigabitEthernet0/0     10.1.2.2        YES manual up                    up
>                        ^^^^^^^^ ← Problème : subnet différent !
> ```

---

#### 3. 2WAY

**Description** : Communication bidirectionnelle établie

**Caractéristiques** :

- Les deux routeurs se voient mutuellement
- Chaque routeur voit son propre Router ID dans le Hello du voisin
- **Sur réseaux multi-accès** : État normal pour les routeurs DROTHER entre eux
- **Sur liens point-à-point** : État transitoire vers FULL

**Décision DR/BDR** :

- L'élection DR/BDR se produit pendant cet état
- Basée sur la priorité (plus haute gagne) puis Router ID (plus élevé gagne)
- DR et BDR passent ensuite à EXSTART
- Les DROTHER restent en 2WAY entre eux (normal !)

> [!info] 2WAY est-il un problème ? **Non sur réseau broadcast** entre DROTHER :
> 
> - Les DROTHER n'établissent pas d'adjacence complète entre eux
> - Ils communiquent via le DR/BDR
> - 2WAY/DROTHER est un état **normal et stable**
> 
> **Oui sur lien point-à-point** :
> 
> - Doit progresser vers FULL
> - Si bloqué en 2WAY, problème de paramètres OSPF

**Causes de blocage en 2WAY** (sur P2P) :

- **Mismatch de zone** : Routeurs dans des zones différentes
- **Mismatch d'authentification** : Types ou clés différents
- **Mismatch de stub flag** : Un routeur en stub, l'autre non
- **MTU mismatch** : Tailles MTU différentes (bloque en EXSTART souvent)

---

#### 4. EXSTART

**Description** : Début de l'échange de base de données, négociation du master/slave

**Caractéristiques** :

- Négociation du rôle master/slave
- Le **master** a le Router ID le plus élevé
- Le master contrôle le processus d'échange (incrémente le numéro de séquence)
- Échange de DBD (Database Description) packets vides

**Processus** :

1. Les deux routeurs s'envoient des DBD avec le bit I (Initial), M (More), MS (Master/Slave)
2. Le routeur avec le Router ID le plus élevé devient master
3. Le slave accepte le numéro de séquence du master

> [!warning] Problème courant : MTU mismatch Le problème le plus fréquent causant un blocage en EXSTART est une **différence de MTU**.
> 
> **Symptômes** :
> 
> - État bloqué en EXSTART ou EXCHANGE
> - Messages répétés : "Mismatch MTU" dans les logs
> - Retransmissions constantes de DBD packets
> 
> **Solution** :
> 
> ```bash
> # Vérifier le MTU
> Router# show interface GigabitEthernet0/0 | include MTU
> MTU 1500 bytes
> 
> # Option 1 : Ajuster le MTU physique
> Router(config-if)# mtu 1500
> 
> # Option 2 : Ignorer le MTU mismatch (temporaire)
> Router(config-if)# ip ospf mtu-ignore
> ```

**Autres causes de blocage** :

- Paquets DBD corrompus ou perdus
- Problème de séquençage
- Surcharge CPU du routeur

---

#### 5. EXCHANGE

**Description** : Échange des Database Description packets

**Caractéristiques** :

- Échange de résumés de la LSDB (headers de LSA uniquement)
- Le master envoie DBD, le slave acknowledge
- Les routeurs comparent les DBD reçus avec leur propre LSDB
- Identification des LSA manquants ou obsolètes

**Contenu des DBD packets** :

- Headers de tous les LSA dans la LSDB
- Information : Type, Link State ID, Advertising Router, Sequence Number

**Ce qui se passe** :

1. Master envoie DBD avec bit M (More) si d'autres paquets suivent
2. Slave répond avec son propre DBD
3. Chaque routeur note les LSA qu'il ne possède pas ou qui sont plus récents
4. Préparation de la liste de requêtes (Link State Request)

> [!tip] Durée normale de EXCHANGE
> 
> - Réseau simple : quelques secondes
> - Grande LSDB : jusqu'à 30 secondes
> - Si bloqué > 1 minute : problème

**Causes de blocage prolongé** :

- Base de données très volumineuse
- Bande passante limitée
- Perte de paquets
- Problème de fragmentation

---

#### 6. LOADING

**Description** : Téléchargement des LSA manquants ou obsolètes

**Caractéristiques** :

- Envoi de Link State Request (LSR) pour demander les LSA nécessaires
- Réception de Link State Update (LSU) contenant les LSA demandés
- Envoi de Link State Acknowledgment (LSAck) pour confirmer la réception
- Peut y avoir plusieurs échanges LSR/LSU/LSAck

**Processus détaillé** :

```
Routeur A → LSR → Routeur B  (Demande LSA X, Y, Z)
Routeur B → LSU → Routeur A  (Envoie LSA X, Y, Z)
Routeur A → LSAck → Routeur B (Confirme réception)
```

> [!info] État transitoire LOADING est généralement très bref (< 5 secondes) sauf si :
> 
> - Nombreux LSA à télécharger
> - Lien lent ou congestionné
> - Problème de fiabilité du lien

**Causes de blocage en LOADING** :

- **Perte de paquets** : LSU perdus, nécessitant retransmission
- **LSA corrompu** : Checksum invalide
- **Boucle de requêtes** : Routeur demande continuellement un LSA qui ne peut être fourni
- **Problème de mémoire** : Routeur ne peut stocker les LSA reçus

**Diagnostic LOADING prolongé** :

```bash
Router# show ip ospf neighbor detail
  # Vérifier "retransmission queue length"
  # Si > 0 et constant : problème de réception/acknowledge
  
Router# debug ip ospf adj
  # Observer les échanges LSR/LSU/LSAck
```

---

#### 7. FULL

**Description** : Adjacence complète établie

**Caractéristiques** :

- **Base de données synchronisée** entre les deux voisins
- Les routeurs ont des LSDB identiques pour leur zone commune
- Échange de LSU uniquement lors de changements topologiques
- État stable et opérationnel

**Maintien de l'état FULL** :

- Hello packets échangés régulièrement (keepalive)
- Si pas de Hello reçu pendant Dead Timer : retour à DOWN
- LSU envoyé immédiatement en cas de changement
- Retransmission des LSU non-acknowledgés toutes les 5s (par défaut)

> [!tip] État FULL = adjacence fonctionnelle C'est l'état normal et souhaité pour tous les liens point-à-point et pour les liens entre DR/BDR et DROTHER sur réseaux multi-accès.

---

### Tableau récapitulatif des états

|État|Description courte|Durée normale|Action du routeur|
|---|---|---|---|
|**DOWN**|Pas de Hello reçu|N/A|Envoie des Hello|
|**INIT**|Hello reçu, unidirectionnel|< 2 seconds|Continue d'envoyer Hello avec voisins listés|
|**2WAY**|Bidirectionnel établi|Stable (DROTHER) ou < 1s (P2P)|Élection DR/BDR, décide de former adjacence|
|**EXSTART**|Négociation master/slave|< 5 seconds|Échange DBD vides, négocie séquence|
|**EXCHANGE**|Échange résumés LSDB|5-30 seconds|Envoie headers LSA, note LSA manquants|
|**LOADING**|Téléchargement LSA|< 5 seconds|Envoie LSR, reçoit LSU, envoie LSAck|
|**FULL**|Adjacence complète|Stable|Maintient adjacence, route le trafic|

### Diagramme de transition d'états

```
                    START
                      ↓
    ┌─────────────► DOWN ◄─────────────┐
    │                 ↓                  │
    │              Hello reçu           │
    │                 ↓                  │
    │               INIT                │
    │                 ↓                  │
    │         Bidirectionnel OK         │
    │                 ↓                  │
    │               2WAY ────────┐      │
    │                 ↓           │      │
    │        Décision d'adjacence│      │
    │                 ↓           │      │
    │        ┌─── Pas besoin     │      │
    │        │     d'adjacence    │      │
    │        │         │          │      │
    │        │         └──► Reste 2WAY  │
    │        │              (DROTHER)    │
    │        │                           │
    │        │     Besoin                │
    │        │    d'adjacence            │
    │        │         ↓                 │
    │        └───► EXSTART               │
    │                 ↓                  │
    │          Master/Slave OK          │
    │                 ↓                  │
    │             EXCHANGE               │
    │                 ↓                  │
    │          Échange terminé          │
    │                 ↓                  │
    │              LOADING               │
    │                 ↓                  │
    │          LSA complets            │
    │                 ↓                  │
    │               FULL                │
    │                 │                  │
    │          Dead Timer expire        │
    │          ou changement             │
    └─────────────────┴──────────────────┘
```

### Conditions de formation d'adjacence

Une adjacence FULL n'est formée que si :

1. **Sur lien point-à-point** : Toujours
2. **Sur réseau multi-accès** :
    - Entre DR et tous les routeurs
    - Entre BDR et tous les routeurs
    - **Pas** entre DROTHER (restent en 2WAY)
3. **Sur lien virtuel** : Toujours

### Problèmes communs par état

|État bloqué|Cause probable|Vérification|
|---|---|---|
|**DOWN**|Connectivité L1/L2|`ping`, `show interface`|
|**INIT**|Subnet/masque différent|`show ip interface`|
|**2WAY** (sur P2P)|Zone/auth/stub mismatch|`show ip ospf interface`|
|**EXSTART**|MTU mismatch|`show interface \| include MTU`|
|**EXCHANGE**|Grande LSDB, perte paquets|`show ip ospf database database-summary`|
|**LOADING**|LSA corrompu, perte|`debug ip ospf adj`|

> [!warning] Flapping d'adjacence Si l'état oscille entre DOWN et FULL rapidement :
> 
> - **Instabilité physique** : Câble, interface, switch
> - **Timers trop courts** : Environnement instable
> - **Surcharge CPU** : Hello packets non traités à temps
> - **Duplex mismatch** : Half/Full duplex incompatible

---

## 🐛 Debug ip ospf {#debug-ip-ospf}

Les commandes debug permettent d'observer le fonctionnement d'OSPF en temps réel. **ATTENTION** : Debug consomme des ressources CPU.

### Précautions d'usage

> [!warning] DANGER : Impact sur la production
> 
> - Les commandes debug peuvent **saturer la CPU** sur un routeur en production
> - Toujours utiliser avec des ACL pour limiter le scope
> - Préférer les commandes show quand possible
> - Désactiver immédiatement après usage avec `undebug all`

**Bonnes pratiques** :

```bash
# Activer logging avec timestamp
Router# service timestamps debug datetime msec

# Limiter les debug à une interface spécifique
Router# access-list 100 permit ip host 10.1.1.1 any
Router# debug ip ospf events 100

# Monitorer l'usage CPU
Router# show processes cpu | include CPU

# Désactiver TOUT debug
Router# undebug all
Router# no debug all
```

### Principales commandes debug

#### 1. debug ip ospf adj

**Utilité** : Observer la formation et la rupture des adjacences

```bash
Router# debug ip ospf adj
OSPF adjacency events debugging is on
```

**Sortie typique** :

```
*Dec 21 14:23:15.234: OSPF-1 ADJ   Gi0/0: Rcv hello from 192.168.3.2 area 0
*Dec 21 14:23:15.235: OSPF-1 ADJ   Gi0/0: 2 Way Communication to 192.168.3.2, state 2WAY
*Dec 21 14:23:15.236: OSPF-1 ADJ   Gi0/0: Backup seen Event before WAIT timer
*Dec 21 14:23:15.237: OSPF-1 ADJ   Gi0/0: DR/BDR election
*Dec 21 14:23:15.238: OSPF-1 ADJ   Gi0/0: Elect BDR 192.168.3.3
*Dec 21 14:23:15.239: OSPF-1 ADJ   Gi0/0: Elect DR 192.168.3.1
*Dec 21 14:23:15.240: OSPF-1 ADJ   Gi0/0: DR: 192.168.3.1 (Id) 10.1.1.1 (Interface)
*Dec 21 14:23:15.241: OSPF-1 ADJ   Gi0/0: BDR: 192.168.3.3 (Id) 10.1.1.3 (Interface)
*Dec 21 14:23:15.245: OSPF-1 ADJ   Gi0/0: Send DBD to 192.168.3.1 seq 0x1A2B length 32 opt 0x52 flag 0x7 state EXSTART
*Dec 21 14:23:15.267: OSPF-1 ADJ   Gi0/0: Rcv DBD from 192.168.3.1 seq 0x2C3D opt 0x52 flag 0x7 length 32 state EXSTART
*Dec 21 14:23:15.268: OSPF-1 ADJ   Gi0/0: First DBD and we are not SLAVE
*Dec 21 14:23:15.278: OSPF-1 ADJ   Gi0/0: Rcv DBD from 192.168.3.1 seq 0x1A2B opt 0x52 flag 0x2 length 172 state EXSTART
*Dec 21 14:23:15.279: OSPF-1 ADJ   Gi0/0: NBR Negotiation Done. We are the SLAVE
*Dec 21 14:23:15.280: OSPF-1 ADJ   Gi0/0: Nbr 192.168.3.1 state EXSTART -> EXCHANGE
```

**À rechercher** :

- ✅ Progression normale : 2WAY → EXSTART → EXCHANGE → LOADING → FULL
- ❌ Retour à DOWN ou INIT répété
- ❌ Blocage dans un état intermédiaire
- ❌ Messages d'erreur (mismatch, bad packets, etc.)

---

#### 2. debug ip ospf hello

**Utilité** : Observer les paquets Hello (envoi et réception)

```bash
Router# debug ip ospf hello
OSPF hello events debugging is on
```

**Sortie typique** :

```
*Dec 21 14:25:10.123: OSPF-1 HELLO Gi0/0: Send hello to 224.0.0.5 area 0 from 10.1.1.3
*Dec 21 14:25:12.456: OSPF-1 HELLO Gi0/0: Rcv hello from 192.168.3.1 area 0 10.1.1.1
*Dec 21 14:25:12.457: OSPF-1 HELLO Gi0/0: Mismatched hello parameters from 10.1.1.1
*Dec 21 14:25:12.458: OSPF-1 HELLO Gi0/0: Dead timer mismatch 40 vs 120
```

**Problèmes détectables** :

- **Hello envoyé mais pas reçu** : Problème unidirectionnel
- **Mismatch de paramètres** :
    - Dead timer différent
    - Hello timer différent
    - Network mask différent
    - Zone différente
    - Authentification différente
    - Stub flag différent

> [!example] Diagnostic de mismatch
> 
> ```
> OSPF-1 HELLO Gi0/0: Mismatched hello parameters from 10.1.1.2
> OSPF-1 HELLO Gi0/0: Hello interval mismatch 10 vs 30
> ```
> 
> → Le voisin 10.1.1.2 utilise un hello interval de 30s au lieu de 10s

---

#### 3. debug ip ospf packet

**Utilité** : Observer TOUS les types de paquets OSPF

```bash
Router# debug ip ospf packet
OSPF packet debugging is on
```

> [!warning] Très verbeux Cette commande génère **beaucoup** de sortie. À utiliser avec prudence en production.

**Types de paquets** :

1. **Hello** : Découverte et maintien de voisinage
2. **DBD** (Database Description) : Résumés de LSDB
3. **LSR** (Link State Request) : Demande de LSA
4. **LSU** (Link State Update) : Envoi de LSA
5. **LSAck** (Link State Acknowledgment) : Confirmation de réception

**Sortie exemple** :

```
*Dec 21 14:30:05.123: OSPF-1 PAK  Gi0/0: OUT Hello, Area 0, length 48, to 224.0.0.5
*Dec 21 14:30:05.145: OSPF-1 PAK  Gi0/0: IN  Hello, Area 0, length 48, from 10.1.1.1
*Dec 21 14:30:05.167: OSPF-1 PAK  Gi0/0: OUT DBD, Area 0, length 32, seq 0x1234, to 10.1.1.1
*Dec 21 14:30:05.189: OSPF-1 PAK  Gi0/0: IN  DBD, Area 0, length 172, seq 0x1234, from 10.1.1.1
*Dec 21 14:30:05.201: OSPF-1 PAK  Gi0/0: OUT LSReq, Area 0, length 36, to 10.1.1.1
*Dec 21 14:30:05.223: OSPF-1 PAK  Gi0/0: IN  LSUpd, Area 0, length 124, from 10.1.1.1
*Dec 21 14:30:05.245: OSPF-1 PAK  Gi0/0: OUT LSAck, Area 0, length 44, to 10.1.1.1
```

---

#### 4. debug ip ospf events

**Utilité** : Observer les événements OSPF majeurs

```bash
Router# debug ip ospf events
OSPF events debugging is on
```

**Événements trackés** :

- Calcul SPF (Shortest Path First)
- Changements de routes
- Élections DR/BDR
- Changements d'adjacence
- Mises à jour de LSDB

**Sortie exemple** :

```
*Dec 21 14:35:00.123: OSPF-1 EVENT Gi0/0: Interface going Down
*Dec 21 14:35:00.145: OSPF-1 EVENT: Schedule SPF in area 0
*Dec 21 14:35:00.167: OSPF-1 EVENT: Run SPF now
*Dec 21 14:35:00.189: OSPF-1 EVENT: SPF complete. New routing table
*Dec 21 14:35:00.201: OSPF-1 EVENT: Route 10.3.3.0/24 deleted
*Dec 21 14:35:15.456: OSPF-1 EVENT Gi0/0: Interface going Up
*Dec 21 14:35:15.478: OSPF-1 EVENT Gi0/0: DR election
```

> [!info] Utilité pour la performance Cette commande permet de voir la fréquence des calculs SPF, qui peuvent être coûteux en CPU sur des topologies complexes.

---

#### 5. debug ip ospf lsa-generation

**Utilité** : Observer la génération et le traitement des LSA

```bash
Router# debug ip ospf lsa-generation
OSPF LSA generation debugging is on
```

**Sortie exemple** :

```
*Dec 21 14:40:00.123: OSPF-1 LSA: Generate Router LSA 192.168.3.3, mask /24, type 1, age 0, seq 0x80000005
*Dec 21 14:40:00.145: OSPF-1 LSA: Flooding LSA 1/192.168.3.3 on Gi0/0
*Dec 21 14:40:00.167: OSPF-1 LSA: Rcv Type 1, LSID 192.168.3.1, Adv Rtr 192.168.3.1, age 12, seq 0x80000006
*Dec 21 14:40:00.189: OSPF-1 LSA: Installing LSA 1/192.168.3.1 in LSDB
```

**Utile pour** :

- Vérifier qu'un changement génère bien un LSA
- Observer le flooding des LSA dans le réseau
- Identifier les LSA généré excessivement (instabilité)

---

### Commandes debug avancées

#### debug condition

Permet de limiter le debug à un voisin spécifique :

```bash
Router# debug condition interface GigabitEthernet0/0
Router# debug condition peer 10.1.1.1

Router# debug ip ospf adj
# Debug s'applique uniquement à la condition

Router# undebug condition all
```

> [!tip] Réduire l'impact CPU Toujours utiliser `debug condition` en production pour limiter le scope du debug.

---

### Tableau récapitulatif des debug

|Commande|Utilité|Verbosité|Usage recommandé|
|---|---|---|---|
|`debug ip ospf adj`|Formation adjacence|Moyenne|Problèmes de voisinage|
|`debug ip ospf hello`|Paquets Hello|Élevée|Mismatch de paramètres|
|`debug ip ospf packet`|Tous les paquets|**Très élevée**|Problèmes de communication|
|`debug ip ospf events`|Événements majeurs|Faible|Vue d'ensemble, SPF|
|`debug ip ospf lsa-generation`|Génération LSA|Moyenne|Problèmes de LSDB|

---

## 🔧 Méthodologie de dépannage {#méthodologie-de-dépannage}

### Approche systématique en 5 étapes

#### Étape 1 : Vérifier la connectivité de base

```bash
# 1. État physique des interfaces
Router# show ip interface brief

# 2. Connectivité IP
Router# ping [neighbor-ip]

# 3. Routage IP activé
Router# show ip protocols
```

> [!tip] 80% des problèmes OSPF sont des problèmes L1/L2/L3 basiques Ne perdez pas de temps sur OSPF si la connectivité IP de base n'est pas établie.

---

#### Étape 2 : Vérifier les adjacences

```bash
# 1. État des voisins
Router# show ip ospf neighbor

# Questions à se poser :
# - Tous les voisins attendus sont-ils présents ?
# - Quel est leur état ? (devrait être FULL ou 2WAY si approprié)
# - Les Dead Timers sont-ils stables ?
```

**Décision** :

- ✅ Voisins en FULL → Passer à l'étape 3
- ❌ Voisins absents ou état incorrect → Approfondir les adjacences

---

#### Étape 3 : Vérifier la configuration des interfaces

```bash
# 1. Configuration OSPF des interfaces
Router# show ip ospf interface brief

# 2. Détails d'une interface spécifique
Router# show ip ospf interface [interface]

# Vérifier :
# - Zone correcte
# - Timers (Hello/Dead) cohérents
# - Network type approprié
# - Coût configuré ou par défaut
# - Pas d'interface passive non intentionnelle
```

---

#### Étape 4 : Vérifier la base de données LSDB

```bash
# 1. Vue d'ensemble
Router# show ip ospf database

# 2. Comparer avec les voisins
Router# show ip ospf database database-summary

# 3. Examiner des LSA spécifiques si nécessaire
Router# show ip ospf database router [router-id]
Router# show ip ospf database network [link-id]

# Vérifier :
# - LSDB similaire entre voisins dans la même zone
# - Pas de LSA à MaxAge bloqués
# - Pas de nombre excessif de LSA
```

---

#### Étape 5 : Vérifier les routes dans la table de routage

```bash
# 1. Routes OSPF installées
Router# show ip route ospf

# 2. Détails d'une route spécifique
Router# show ip route [network]

# Vérifier :
# - Routes attendues présentes
# - Métrique correcte
# - Next-hop approprié
# - Pas de routes en doublon
```

---

### Arbres de décision pour problèmes courants

#### Problème : Pas d'adjacence formée

```
Voisin n'apparaît pas dans "show ip ospf neighbor"
│
├─ Vérifier connectivité IP (ping)
│  └─ Échec → Problème L1/L2/L3 (pas OSPF)
│
├─ Vérifier "show ip ospf interface"
│  ├─ Interface pas listée → OSPF pas activé sur l'interface
│  └─ Interface passive → Retirer "passive-interface"
│
├─ Vérifier configuration réseau/zone
│  ├─ Subnet différent → Corriger adressage IP
│  └─ Zone différente → Aligner les zones
│
└─ Utiliser "debug ip ospf hello"
   ├─ Hello envoyé mais pas reçu → Problème firewall/ACL
   ├─ Hello reçu avec mismatch → Aligner paramètres
   └─ Pas de Hello du tout → Vérifier config du voisin
```

---

#### Problème : Adjacence bloquée (pas FULL)

```
Voisin en INIT, 2WAY, EXSTART, EXCHANGE ou LOADING
│
├─ État INIT
│  └─ Vérifier subnet/masque identiques
│
├─ État 2WAY (sur lien P2P)
│  ├─ Vérifier même zone
│  ├─ Vérifier authentification identique
│  └─ Vérifier pas de mismatch stub/not-so-stubby
│
├─ État EXSTART
│  ├─ Vérifier MTU identique avec "show interface"
│  ├─ Option temporaire: "ip ospf mtu-ignore"
│  └─ Utiliser "debug ip ospf adj"
│
├─ État EXCHANGE
│  ├─ Vérifier taille LSDB normale
│  └─ Vérifier pas de perte de paquets
│
└─ État LOADING
   ├─ Vérifier "show ip ospf neighbor detail"
   ├─ Retransmission queue > 0 → Problème acknowledgement
   └─ Utiliser "debug ip ospf adj"
```

---

#### Problème : Routes OSPF manquantes

```
Route attendue absente de "show ip route ospf"
│
├─ Vérifier que le réseau est annoncé
│  └─ "show ip ospf database" → LSA présent ?
│     ├─ Non → Problème d'annonce sur le routeur source
│     └─ Oui → Continuer
│
├─ Vérifier route pas filtrée
│  ├─ "show ip prefix-list" / "show route-map"
│  └─ "show ip distribute-list"
│
├─ Vérifier pas de route plus spécifique
│  └─ Route déjà installée par autre protocole ?
│
└─ Vérifier calcul SPF
   ├─ Utiliser "debug ip ospf spf"
   └─ Coût de la route peut-être sous-optimal
```

---

### Checklist complète de vérification OSPF

#### Configuration de base

- [ ] `router ospf [process-id]` configuré
- [ ] `network` statements corrects
- [ ] Router ID configuré (automatique ou manuel)
- [ ] Pas de doublon de Router ID dans l'AS

#### Interfaces

- [ ] Interfaces physiquement up
- [ ] Adresses IP correctes et dans le bon subnet
- [ ] OSPF activé sur les bonnes interfaces
- [ ] Zone OSPF cohérente avec les voisins
- [ ] Timers (Hello/Dead) identiques aux voisins
- [ ] MTU identique aux voisins
- [ ] Network type approprié au média
- [ ] Pas d'interface passive non intentionnelle
- [ ] Authentification (si utilisée) identique

#### Adjacences

- [ ] Tous les voisins attendus présents
- [ ] État FULL (ou 2WAY si DROTHER)
- [ ] Dead Timer stable (pas de reset constant)
- [ ] DR/BDR élus correctement (si multi-accès)
- [ ] Pas de flapping d'adjacence

#### Base de données LSDB

- [ ] LSDB cohérente entre voisins d'une même zone
- [ ] Pas de LSA à MaxAge bloqués
- [ ] Nombre de LSA raisonnable
- [ ] Pas de LSA corrompus (checksum)
- [ ] Numéros de séquence progressent normalement

#### Routage

- [ ] Routes OSPF attendues dans la table de routage
- [ ] Coûts des routes appropriés
- [ ] Next-hop corrects
- [ ] Pas de boucles de routage
- [ ] Pas de routes flappantes

---

### Scénarios de dépannage pratiques

#### Scénario 1 : Adjacence ne se forme pas

**Symptômes** :

```bash
Router# show ip ospf neighbor
# Aucune sortie ou voisin en INIT
```

**Diagnostic étape par étape** :

```bash
# Étape 1 : Vérifier interface up
Router# show ip interface brief | include Gi0/0
GigabitEthernet0/0     10.1.1.3        YES manual up                    up

# Étape 2 : Tester connectivité
Router# ping 10.1.1.1
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.1.1.1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5)

# Étape 3 : Vérifier OSPF activé sur interface
Router# show ip ospf interface Gi0/0
GigabitEthernet0/0 is up, line protocol is up
  Internet Address 10.1.1.3/24, Area 0, Attached via Network Statement
  
# Étape 4 : Vérifier réception de Hello
Router# debug ip ospf hello
*Dec 21 15:00:10: OSPF-1 HELLO Gi0/0: Rcv hello from 192.168.3.1 area 0 10.1.1.1
*Dec 21 15:00:10: OSPF-1 HELLO Gi0/0: Mismatched hello parameters from 10.1.1.1
*Dec 21 15:00:10: OSPF-1 HELLO Gi0/0: Dead interval mismatch 40 vs 120

# Problème identifié : Dead interval différent !
```

**Solution** :

```bash
Router(config)# interface GigabitEthernet0/0
Router(config-if)# ip ospf dead-interval 40
# OU sur le voisin :
# ip ospf dead-interval 120
```

---

#### Scénario 2 : Adjacence bloquée en EXSTART

**Symptômes** :

```bash
Router# show ip ospf neighbor
Neighbor ID     Pri   State           Dead Time   Address         Interface
192.168.3.1       1   EXSTART/DR      00:00:35    10.1.1.1        Gi0/0
# État reste en EXSTART, ne progresse jamais
```

**Diagnostic** :

```bash
# Vérifier MTU
Router# show interface Gi0/0 | include MTU
  MTU 1500 bytes, BW 1000000 Kbit/sec

# Sur le voisin
RouterB# show interface Gi0/0 | include MTU
  MTU 1400 bytes, BW 1000000 Kbit/sec
  # MTU différent ! C'est le problème
  
# Observer les logs
Router# debug ip ospf adj
*Dec 21 15:05:15: OSPF-1 ADJ Gi0/0: Nbr 192.168.3.1: Packet size 1500 exceeds MTU 1400
```

**Solution 1** : Aligner les MTU

```bash
RouterB(config)# interface Gi0/0
RouterB(config-if)# mtu 1500
```

**Solution 2** : Ignorer le MTU (temporaire)

```bash
Router(config)# interface Gi0/0
Router(config-if)# ip ospf mtu-ignore
```

> [!warning] ip ospf mtu-ignore C'est un workaround, pas une solution permanente. Les paquets OSPF pourraient être fragmentés, ce qui dégrade les performances.

---

#### Scénario 3 : Routes OSPF manquantes

**Symptômes** :

```bash
Router# show ip route ospf
# Route 10.5.5.0/24 absente alors qu'elle devrait être présente
```

**Diagnostic** :

```bash
# Étape 1 : Vérifier présence dans LSDB
Router# show ip ospf database | include 10.5.5.0
# Aucune sortie → LSA absent de la LSDB

# Étape 2 : Vérifier sur routeur source
RouterSource# show ip ospf database router self-originate

# Si LSA présent sur source mais pas sur notre routeur :
# → Problème de propagation de LSA

# Étape 3 : Vérifier adjacences complètes
Router# show ip ospf neighbor
# Tous les voisins doivent être en FULL

# Étape 4 : Vérifier pas de filtrage
Router# show ip prefix-list
Router# show route-map
Router# show run | section distribute-list
```

**Causes possibles** :

1. **Réseau pas annoncé** sur routeur source
2. **Filtrage** avec distribute-list, prefix-list ou route-map
3. **ABR filter** avec `area X filter-list`
4. **Zone stub** empêchant LSA Type 5
5. **Partition** : pas de chemin OSPF vers la source

---

#### Scénario 4 : Flapping d'adjacence

**Symptômes** :

```bash
Router# show ip ospf neighbor
Neighbor ID     Pri   State           Dead Time   Address         Interface
192.168.3.1       1   FULL/DR         00:00:38    10.1.1.1        Gi0/0

# 30 secondes plus tard...
Router# show ip ospf neighbor
# Voisin absent

# Puis réapparaît...
```

**Diagnostic** :

```bash
# Vérifier nombre de changements d'état
Router# show ip ospf neighbor detail
Neighbor 192.168.3.1, interface address 10.1.1.1
    State is FULL, 247 state changes  ← PROBLÈME !
    # Plus de 10 state changes = instabilité

# Observer les transitions
Router# debug ip ospf adj
*Dec 21 15:10:05: OSPF-1 ADJ Gi0/0: Nbr 192.168.3.1 Down: Dead timer expired
*Dec 21 15:10:15: OSPF-1 ADJ Gi0/0: Nbr 192.168.3.1 2 Way Communication
*Dec 21 15:10:35: OSPF-1 ADJ Gi0/0: Nbr 192.168.3.1 Down: Dead timer expired
# Cycle répété

# Vérifier connectivité avec ping continu
Router# ping 10.1.1.1 repeat 1000
!!!!!!!!!!!!!!.!!!!!!!!!!!.!!!!!!!!!!!!!!!.!!!!!
# Points (.) indiquent des pertes de paquets
```

**Causes courantes** :

1. **Problème physique** : Câble défectueux, port switch instable
2. **Duplex mismatch** : Un côté full-duplex, l'autre half-duplex
3. **Congestion** : Paquets Hello perdus sous charge
4. **Surcharge CPU** : Hello packets non traités à temps
5. **Spanning Tree** : Reconvergence STP fréquente

**Solutions** :

```bash
# Si environnement instable : augmenter timers
Router(config)# interface Gi0/0
Router(config-if)# ip ospf hello-interval 30
Router(config-if)# ip ospf dead-interval 120

# Vérifier duplex
Router# show interface Gi0/0 | include duplex
Full-duplex, 1000Mb/s
# Doit être identique des deux côtés

# Forcer duplex si auto-négociation échoue
Router(config-if)# duplex full
Router(config-if)# speed 1000
```

---

#### Scénario 5 : Convergence lente

**Symptômes** :

- Temps de convergence > 10 secondes après un changement
- Utilisateurs signalent des coupures lors de changements réseau

**Diagnostic** :

```bash
# Vérifier fréquence calculs SPF
Router# show ip ospf | begin SPF
 SPF schedule delay 5000 msecs, Hold time between two SPFs 10000 msecs
 Number of external LSA 156. Checksum Sum 0x005A2B3C
 Number of DCbitless external and opaque AS LSA 0
 Number of DoNotAge external and opaque AS LSA 0
 Number of areas in this router is 1. 1 normal 0 stub 0 nssa
 Number of areas transit capable is 0
 External flood list length 0
 SPF algorithm last executed 00:00:23.456 ago
 SPF algorithm executed 347 times  ← Trop fréquent si réseau stable !
 
# Observer calculs SPF en temps réel
Router# debug ip ospf spf
*Dec 21 15:20:00: OSPF-1 SPFSCHED: Scheduling SPF calculation
*Dec 21 15:20:05: OSPF-1 SPF: Begin SPF calculation
*Dec 21 15:20:05: OSPF-1 SPF: Complete SPF calculation in 45ms
# 45ms est normal, mais >500ms indique un problème
```

**Causes et solutions** :

1. **Trop de LSA** (LSDB volumineuse)

```bash
Router# show ip ospf database database-summary
Type-5 Ext    15847    ← Beaucoup trop !

# Solution : Summarization sur ASBR
RouterASBR(config-router)# summary-address 10.0.0.0 255.0.0.0
```

2. **SPF triggé trop fréquemment**

```bash
# Ajuster timers SPF pour éviter calculs excessifs
Router(config-router)# timers throttle spf 5000 10000 20000
# 5s initial delay, 10s hold time, 20s max wait
```

3. **BFD non utilisé**

```bash
# Activer BFD pour détection rapide de pannes
Router(config)# interface Gi0/0
Router(config-if)# ip ospf bfd
# BFD détecte pannes en <1s vs 40s avec Dead Timer
```

---

### Outils et commandes de monitoring continu

#### Monitoring des adjacences

```bash
# Script de surveillance (à exécuter périodiquement)
Router# show ip ospf neighbor | include FULL
# Si moins de voisins attendus : alerte

# Vérifier stabilité
Router# show ip ospf neighbor detail | include state changes
# Si augmentation constante : instabilité
```

#### Monitoring de la LSDB

```bash
# Surveiller taille LSDB
Router# show ip ospf database database-summary
# Une augmentation soudaine peut indiquer un problème

# Vérifier LSA suspects
Router# show ip ospf database | include 3600
# LSA à MaxAge (3600s) devraient être flushés
```

#### Monitoring des routes

```bash
# Nombre de routes OSPF
Router# show ip route summary | include ospf
ospf      150 subnets, 150 masks
# Suivre l'évolution dans le temps

# Routes flappantes
Router# show ip route | include variably
# Présence de "variably subnetted" peut indiquer summarization
```

#### Monitoring des performances

```bash
# Fréquence SPF
Router# show ip ospf | include SPF
 SPF algorithm executed 347 times
# Noter et comparer périodiquement

# Usage mémoire OSPF
Router# show processes memory | include OSPF
125  5276    4088    4088    4088   10452     0  OSPF-1 Router
```

---

### Commandes de maintenance préventive

#### Avant un changement réseau

```bash
# 1. Documenter état actuel
Router# show ip ospf neighbor > neighbor-before.txt
Router# show ip route ospf > routes-before.txt
Router# show ip ospf database database-summary > lsdb-before.txt

# 2. Vérifier santé du réseau
Router# show ip ospf interface brief | include Down
# Ne devrait rien retourner

# 3. Vérifier pas d'instabilité
Router# show ip ospf neighbor detail | include state changes
# Pas de valeurs élevées
```

#### Après un changement réseau

```bash
# 1. Vérifier adjacences rétablies
Router# show ip ospf neighbor
# Tous les voisins en FULL

# 2. Vérifier routes présentes
Router# show ip route ospf
# Comparer avec routes-before.txt

# 3. Vérifier convergence
Router# show ip ospf | include SPF
# SPF devrait avoir été exécuté une fois

# 4. Tester connectivité
Router# ping [destinations critiques]
```

---

### Pièges et erreurs courantes

#### Piège 1 : Oublier le masque wildcard

❌ **Incorrect** :

```bash
Router(config-router)# network 10.1.1.0 255.255.255.0 area 0
```

✅ **Correct** :

```bash
Router(config-router)# network 10.1.1.0 0.0.0.255 area 0
# Wildcard mask, pas subnet mask !
```

---

#### Piège 2 : Router ID dupliqué

**Symptôme** : Comportement OSPF erratique, adjacences instables

```bash
# Vérifier Router ID
Router# show ip protocols | include Router ID
  Router ID 192.168.1.1

# Si dupliqué avec un autre routeur :
Router(config)# router ospf 1
Router(config-router)# router-id 192.168.1.2
Router(config-router)# end
Router# clear ip ospf process
# OSPF doit redémarrer pour appliquer le nouveau Router ID
```

> [!warning] Impact du Router ID dupliqué Deux routeurs avec le même Router ID causent :
> 
> - Élections DR/BDR incorrectes
> - LSA ignorés ou remplacés incorrectement
> - Boucles de routage potentielles

---

#### Piège 3 : Interface passive mal configurée

**Symptôme** : Pas de voisin formé, mais interface dans OSPF

```bash
Router# show ip ospf interface Gi0/0
GigabitEthernet0/0 is up, line protocol is up
  Internet Address 10.1.1.3/24, Area 0
  No Hellos (Passive interface)  ← Problème !

# Retirer le passive
Router(config)# router ospf 1
Router(config-router)# no passive-interface GigabitEthernet0/0
```

> [!tip] Bonne pratique : passive-interface default
> 
> ```bash
> Router(config-router)# passive-interface default
> Router(config-router)# no passive-interface Gi0/0
> Router(config-router)# no passive-interface Gi0/1
> # Ainsi, seules les interfaces explicitement activées envoient Hello
> ```

---

#### Piège 4 : Oublier le coût sur liens lents

**Problème** : Trafic prend un chemin sous-optimal

```bash
# Lien serial 64 kbps a un coût par défaut très élevé
Router# show ip ospf interface Serial0/0/0
  Cost: 1562  ← Coût calculé automatiquement

# Mais un lien satellite 2 Mbps peut avoir un coût plus faible
# alors que la latence est horrible !

# Solution : Ajuster manuellement
Router(config)# interface Serial0/0/0
Router(config-if)# ip ospf cost 10000
# Forcer un coût élevé pour refléter la latence
```

---

#### Piège 5 : Mauvais type de réseau

**Problème** : Élections DR/BDR sur lien point-à-point

```bash
Router# show ip ospf interface Serial0/0/0
  Network Type BROADCAST  ← Incorrect pour lien serial !
  
# Conséquences :
# - Wait timer de 40s avant communication
# - Élection DR/BDR inutile
# - Overhead de packets

# Correction
Router(config)# interface Serial0/0/0
Router(config-if)# ip ospf network point-to-point
```

**Types de réseau et leurs usages** :

|Type|Usage|DR/BDR|Timers par défaut|
|---|---|---|---|
|**broadcast**|Ethernet, FastEthernet, GigabitEthernet|Oui|Hello 10s, Dead 40s|
|**point-to-point**|Serial, Frame Relay P2P|Non|Hello 10s, Dead 40s|
|**point-to-multipoint**|Frame Relay, ATM|Non|Hello 30s, Dead 120s|
|**non-broadcast**|Frame Relay, ATM full-mesh|Oui|Hello 30s, Dead 120s|

---

### Optimisations avancées

#### Optimisation 1 : BFD pour détection rapide de pannes

```bash
# Activer BFD globalement
Router(config)# router ospf 1
Router(config-router)# bfd all-interfaces

# Ou par interface
Router(config)# interface Gi0/0
Router(config-if)# ip ospf bfd

# Configurer timers BFD agressifs
Router(config)# interface Gi0/0
Router(config-if)# bfd interval 50 min_rx 50 multiplier 3
# Détection en 150ms (50ms × 3)
```

> [!info] Avantage de BFD
> 
> - Détection de panne en < 1 seconde (vs 40s avec Dead Timer)
> - Indépendant du protocole de routage
> - Faible overhead CPU (implémenté en hardware sur beaucoup d'équipements)

---

#### Optimisation 2 : Throttle SPF pour stabilité

```bash
# Éviter calculs SPF excessifs lors d'instabilité
Router(config)# router ospf 1
Router(config-router)# timers throttle spf 5000 10000 20000
# start-interval: 5s
# hold-interval: 10s (double à chaque SPF successif)
# max-wait: 20s (maximum entre deux SPF)
```

**Comportement** :

- 1er changement → SPF après 5s
- 2ème changement dans les 10s → SPF après 10s
- 3ème changement → SPF après 20s (max)
- Stabilité > 20s → reset à 5s

---

#### Optimisation 3 : LSA throttling

```bash
# Limiter génération de LSA lors de changements fréquents
Router(config)# router ospf 1
Router(config-router)# timers throttle lsa all 5000 10000 20000
# start-interval: 5s
# hold-interval: 10s
# max-wait: 20s
```

> [!tip] Quand utiliser LSA throttling Environnements avec :
> 
> - Nombreuses interfaces flappantes
> - Redistribution de routes instables
> - Grande quantité de LSA externes

---

#### Optimisation 4 : Stub areas pour réduire LSDB

```bash
# Configuration zone stub (pas de LSA Type 5)
Router(config)# router ospf 1
Router(config-router)# area 10 stub

# Totally stubby area (pas de Type 3 inter-area non plus)
RouterABR(config-router)# area 10 stub no-summary
# Seul l'ABR configure "no-summary"

# NSSA (permet redistribution dans la stub area)
Router(config-router)# area 10 nssa
```

**Réduction LSDB** :

|Type zone|LSA Type 1|LSA Type 2|LSA Type 3|LSA Type 5|
|---|---|---|---|---|
|**Normal**|✓|✓|✓|✓|
|**Stub**|✓|✓|✓|✗ (Route par défaut)|
|**Totally Stubby**|✓|✓|✗ (Route par défaut)|✗|
|**NSSA**|✓|✓|✓|✗ (Type 7 internes)|

---

### Résumé des commandes essentielles

#### Vérification rapide

|Objectif|Commande|
|---|---|
|Vue d'ensemble voisins|`show ip ospf neighbor`|
|Détails d'un voisin|`show ip ospf neighbor detail`|
|Interfaces OSPF|`show ip ospf interface brief`|
|Détails interface|`show ip ospf interface [interface]`|
|Vue LSDB|`show ip ospf database`|
|Résumé LSDB|`show ip ospf database database-summary`|
|Routes OSPF|`show ip route ospf`|
|Info générales OSPF|`show ip ospf`|

#### Debug ciblé

|Problème|Commande debug|
|---|---|
|Formation adjacence|`debug ip ospf adj`|
|Paquets Hello|`debug ip ospf hello`|
|Tous paquets|`debug ip ospf packet`|
|Événements majeurs|`debug ip ospf events`|
|Génération LSA|`debug ip ospf lsa-generation`|
|Calculs SPF|`debug ip ospf spf`|

> [!warning] N'oubliez jamais
> 
> ```bash
> Router# undebug all
> ```
> 
> Après chaque session de debug !

---

## 🎓 Points clés à retenir

### Hiérarchie de vérification

1. **Niveau physique** : Interfaces up, câbles OK
2. **Niveau IP** : Connectivité, adressage correct
3. **Niveau OSPF basique** : Adjacences, interfaces configurées
4. **Niveau OSPF avancé** : LSDB, routes, performance

### États OSPF normaux

- **FULL** : État normal sur P2P et entre DR/BDR et DROTHER
- **2WAY** : État normal entre DROTHER sur réseaux multi-accès
- Tout autre état est transitoire ou indique un problème

### Outils de diagnostic par priorité

1. **show ip ospf neighbor** : Première commande à utiliser
2. **show ip ospf interface** : Vérification de configuration
3. **show ip ospf database** : Analyse de la topologie
4. **debug** : En dernier recours, avec précaution

### Problèmes les plus fréquents

1. MTU mismatch (EXSTART)
2. Timers incohérents (INIT, pas d'adjacence)
3. Zone différente (2WAY sur P2P)
4. Interface passive non intentionnelle
5. Subnet/masque différent (INIT)

### Optimisations recommandées

- **BFD** pour détection rapide de pannes
- **Throttle SPF** pour stabilité lors d'instabilité
- **Stub areas** pour réduire LSDB
- **Summarization** pour limiter nombre de routes

---

> [!tip] Philosophie du dépannage OSPF
> 
> - **Méthodique** : Suivre l'approche en couches (L1 → L2 → L3 → OSPF)
> - **Progressif** : show commands avant debug
> - **Comparatif** : Vérifier des deux côtés de l'adjacence
> - **Documenté** : Capturer l'état avant/après changements
> - **Prudent** : Debug en production = danger

---

_Fin du cours - OSPF Vérification et Dépannage_ 🎯