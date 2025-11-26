# 📡 Synthèse : Principes des Réseaux Informatiques

---

## 📑 Sommaire

1. [Introduction aux Réseaux](#introduction-aux-reseaux)
2. [Les Protocoles Réseaux](https://claude.ai/chat/c439bf20-caab-40b3-9d31-73f23d68a801#2-les-protocoles-r%C3%A9seaux)
3. [Architecture en Couches et Encapsulation](https://claude.ai/chat/c439bf20-caab-40b3-9d31-73f23d68a801#3-architecture-en-couches-et-encapsulation)
4. [Le Modèle OSI](https://claude.ai/chat/c439bf20-caab-40b3-9d31-73f23d68a801#4-le-mod%C3%A8le-osi)
5. [Le Modèle TCP/IP](https://claude.ai/chat/c439bf20-caab-40b3-9d31-73f23d68a801#5-le-mod%C3%A8le-tcpip)
6. [Les Équipements Réseaux](https://claude.ai/chat/c439bf20-caab-40b3-9d31-73f23d68a801#6-les-%C3%A9quipements-r%C3%A9seaux)
7. [Topologies Réseaux](https://claude.ai/chat/c439bf20-caab-40b3-9d31-73f23d68a801#7-topologies-r%C3%A9seaux)
8. [Récapitulatif Visuel](https://claude.ai/chat/c439bf20-caab-40b3-9d31-73f23d68a801#8-r%C3%A9capitulatif-visuel)

---

## 1. Introduction aux Réseaux
<span id="introduction-aux-reseaux"></span>

### Qu'est-ce qu'un réseau informatique ?


Un **réseau informatique** est un ensemble d'éléments reliés les uns aux autres à travers lesquels circulent des informations.

```
┌─────────────────────────────────────┐
│      RÉSEAU INFORMATIQUE            │
├─────────────────────────────────────┤
│ 📶 Supports physiques (câbles)      │
│ 🔧 Équipements d'interconnexion     │
│ 📋 Protocoles réseaux               │
│ 💻 Hôtes (ordinateurs)              │
└─────────────────────────────────────┘
```

### Types de Réseaux (par échelle)

|Type|Échelle|Exemple|
|---|---|---|
|**PAN**|Personnelle|Bluetooth, synchronisation téléphone|
|**LAN**|Bâtiment|Wi-Fi d'une maison, réseau d'entreprise|
|**MAN**|Ville|Réseau municipal|
|**WAN**|Pays/Continent|Internet, réseau international|
|**GAN**|Mondial|Internet global|

---

## 2. Les Protocoles Réseaux

### Définition

Un **protocole réseau** est une norme de communication qui définit comment les équipements vont dialoguer sur un réseau informatique.

```
PROTOCOLE = Règles de communication
  ├─ Format des messages
  ├─ Scénario de communication
  └─ Interopérabilité entre équipements
```

### Rôles des Protocoles

✅ **Rendre des services** aux applications  
✅ **Standardiser** la communication (normes publiques)  
✅ **Assurer l'interopérabilité** entre différents équipements

### Organismes de Standardisation

|Organisme|Domaine|
|---|---|
|**IETF**|Suite TCP/IP, protocoles Internet (RFC)|
|**IEEE**|Protocoles réseau : Ethernet, Wi-Fi, VLAN|
|**UIT**|Normalisation internationale (télécom, codecs)|

### Protocoles Courants

|Protocole|Port|Rôle|
|---|---|---|
|**HTTP**|TCP 80|Communication web|
|**HTTPS**|TCP 443|Web sécurisé|
|**FTP**|TCP 21|Transfert de fichiers|
|**DNS**|UDP 53|Résolution de noms (exemple.com → IP)|
|**TCP**|—|Transport **fiable** avec accusé de réception|
|**UDP**|—|Transport **rapide** sans vérification|

### Port vs Protocole

```
PORT = Numéro d'identification (0-65535)
  └─ "Porte d'entrée" d'une application
  
PROTOCOLE = Règles de communication
  └─ "Comment dialoguer"

💡 Exemple : HTTP (protocole) utilise le port TCP 80 (porte)
```

---

## 3. Architecture en Couches et Encapsulation

### Le Problème de Complexité

Concevoir un seul protocole gérant tous les aspects des réseaux est **impossible**. La solution : découper en **sous-problèmes indépendants** = **architecture en couches**.

```
┌─────────────────────────────────────┐
│  Couche N+1 (service complexe)      │
│  ↓ Demande service plus simple ↓    │
│  Couche N (service basique)         │
│  ↓ Répétez jusqu'à la base ↓        │
│  Couche 1 (physique)                │
└─────────────────────────────────────┘
```

### PDU : Protocol Data Unit

Chaque protocole transmet 2 choses :

```
PDU = EN-TÊTE (header) + CHARGE UTILE (payload)
      ├─ Header : informations de gestion
      └─ Payload : données à transmettre
```

### Encapsulation

Chaque couche **ajoute son propre header** au payload reçu de la couche supérieure :

```
Couche 7 (Application)
│ Données = "Bonjour"
│ PDU = Header₇ + "Bonjour"
│
├→ Couche 6 (Présentation)
  │ PDU = Header₆ + (Header₇ + "Bonjour")
  │
  ├→ Couche 5 (Session)
    │ PDU = Header₅ + (Header₆ + Header₇ + "Bonjour")
    │
    └→ ... jusqu'à la couche physique
       PDU final = bits émis sur le câble
```

**À la réception, c'est l'inverse** : chaque couche enlève son header.

---

## 4. Le Modèle OSI

### Contexte Historique

```
1970s : 3 architectures incompatibles (Bull, DEC, IBM)
1978  : Modèle OSI présenté (7 couches)
1983  : TCP/IP adopté par ARPANET
1984  : OSI devient norme ISO 7498
```

### Les 7 Couches du Modèle OSI

```
┌────────────────────────────────────┐
│ 7 - APPLICATION                    │ } Couches
│ 6 - PRÉSENTATION                   │ } hautes
│ 5 - SESSION                        │
├────────────────────────────────────┤
│ 4 - TRANSPORT                      │ } Couches
├────────────────────────────────────┤ } moyennes
│ 3 - RÉSEAU                         │
│ 2 - LIAISON                        │ } Couches
│ 1 - PHYSIQUE                       │ } basses
└────────────────────────────────────┘
```

### Description Détaillée

#### 🔴 Couche 7 : Application

- **Rôle** : Interfaces avec l'utilisateur final
- **PDU** : Données applicatives
- **Exemples** : HTTP, FTP, DNS, SMTP
- **Contient** : Type et signification des informations échangées

#### 🟠 Couche 6 : Présentation

- **Rôle** : Formatage des données
- **PDU** : Données
- **Exemples** : Chiffrement, compression, encodage (UTF-8, etc.)

#### 🟡 Couche 5 : Session

- **Rôle** : Gestion du dialogue
- **PDU** : Données
- **Fonctions** : Ouverture/fermeture de connexion, reprise après coupure

#### 🟢 Couche 4 : Transport

- **Rôle** : **Transfert de bout en bout** des données
- **PDU** : Segment (TCP) ou Datagramme (UDP)
- **Fonctions** : Découpage des données, contrôle de flux, réordonnancement
- **Protocoles clés** :
    - **TCP** = Fiable (avec accusé de réception) → Courrier recommandé ✅
    - **UDP** = Rapide (sans vérification) → Streaming 🎬

#### 🔵 Couche 3 : Réseau

- **Rôle** : **Routage et interconnexion** de réseaux différents
- **PDU** : Paquet
- **Protocole clé** : IP (v4 et v6)
- **Outil** : Adresse IP unique pour identifier chaque machine

#### 🟣 Couche 2 : Liaison

- **Rôle** : Transmission de données sur le **même réseau physique**
- **PDU** : Trame
- **Fonctions** : Accès au média, détection d'erreurs (CRC)
- **Sous-couches** :
    - MAC : Adresse physique (MAC address)
    - LLC : Encapsulation, détection d'erreur

#### ⚫ Couche 1 : Physique

- **Rôle** : **Support physique** de transmission
- **PDU** : Bit
- **Contient** : Câbles, connecteurs, niveaux de tension, fréquences

### Critique du Modèle OSI

- ⚠️ **Modèle théorique** : rarement appliqué tel quel en pratique
- ⚠️ **Certaines couches peu utilisées** : Session (5) et Présentation (6) quasi inutilisées
- ⚠️ **Redondance** : Contrôle de flux/erreurs en couches 2 ET 4
- ✅ **Reste utile** : Grille d'analyse pour comprendre les protocoles

---

## 5. Le Modèle TCP/IP

### Contexte Historique

```
1974 : Cerf & Kahn publient travaux sur IP
1983 : TCP/IP adopté comme suite de protocoles d'ARPANET
      → Deviendra plus tard INTERNET
```

### Architecture TCP/IP (Simplifié)

```
┌─────────────────────────────────┐
│ 7 - APPLICATION                 │ HTTP, FTP, DNS, SMTP...
│ 5 - TLS (optionnel)             │ Chiffrement SSL/TLS
│ 4 - TRANSPORT                   │ TCP, UDP
│ 3 - INTERNET (IP)               │ IPv4, IPv6
│ 2 - LIAISON (MAC)               │ Ethernet, Wi-Fi
│ 1 - PHYSIQUE                    │ Câbles, connecteurs
└─────────────────────────────────┘
```

### Les 4 Couches Principales

#### 1️⃣ Couche Physique

- Câbles, connecteurs, voltages
- Supporte couche liaison

#### 2️⃣ Couche Liaison (MAC)

- Adressage MAC (adresse physique)
- Fonctionnement sur un **réseau physique local**
- Protocoles : Ethernet, Wi-Fi

#### 3️⃣ Couche Internet (IP)

- **Interconnexion de réseaux physiques différents**
- Chaque machine a une adresse IP unique
- Les paquets transitent via des **routeurs** (passerelles)
- Protocoles IETF : IPv4, IPv6

#### 4️⃣ Couche Transport

- **TCP** : Fiable, orienté connexion
    
    - Avec accusé de réception
    - Garantit l'ordre et l'intégrité
- **UDP** : Rapide, sans connexion
    
    - Pas d'accusé de réception
    - Idéal pour streaming, jeux, VoIP
- **TLS** : Chiffrage optionnel pour sécuriser les communications
    

#### 5️⃣ Couche Application

- Protocoles applicatifs (HTTP, FTP, DNS, SMTP...)
- Interagissent avec les services réseau

### Comparaison OSI ↔ TCP/IP

```
OSI (7 couches)          TCP/IP (4 couches)
7. Application    ──────→ Application
8. Présentation   ──────→ (fusionnées)
9. Session        ──────→ (fusionnées)
10. Transport      ──────→ Transport
11. Réseau         ──────→ Internet
12. Liaison        ──────→ Liaison
13. Physique       ──────→ Physique
```

---

## 6. Les Équipements Réseaux

### Relation entre Équipements et Couches OSI

```
Couche  │ Équipement           │ Fonction
────────┼──────────────────────┼─────────────────────
L1      │ Répéteur / Hub       │ Amplification signal
L2      │ Pont / Switch        │ Commutation (MAC)
L3      │ Routeur              │ Routage (IP)
L3-L7   │ Passerelle           │ Interconnexion réseaux
L3-L7   │ Pare-feu             │ Filtrage
```

### Détail des Équipements

#### 🔄 **Couche 1 : Répéteur**

- Amplifie le signal
- Augmente la portée du réseau
- Matériel passif

#### 🔀 **Couche 1 : Concentrateur / Hub**

- Recopie les données reçues sur **TOUS les ports**
- Concentre le trafic
- Matériel passif

#### 🌉 **Couche 2 : Pont (Bridge)**

- Relie deux **réseaux physiques différents**
- Lit les adresses MAC
- Matériel actif

#### 🎛️ **Couche 2 : Commutateur / Switch**

- **Multiports** → relie plusieurs segments
- Recopie les données vers le **port de destination** (adresse MAC)
- Supporte Full Duplex (bidirectionnel)
- Gère les VLANs
- Matériel actif
- ⭐ **ÉQUIPEMENT CLÉS DANS LES RÉSEAUX MODERNES**

```
MAC Address = Adresse physique de la carte réseau
             └─ Identifie l'équipement sur le réseau local
```

#### 🛣️ **Couche 3 : Routeur**

- Interconnecte **plusieurs réseaux**
- Achemine les paquets (adresse IP)
- Dispose d'une **table de routage** (détermine le chemin)
- Matériel actif

#### 🎚️ **Couche 3 : Switch L2/L3**

- Fonctionne comme un Switch L2 + Routeur L3
- Gère les VLANs
- Assure routage inter-VLAN
- Matériel actif

#### 🚪 **Couche 3-7 : Passerelle (Gateway)**

- Point d'entrée vers un **autre réseau (VLAN)**
- Interconnecte réseaux logiques
- Matériel actif

#### 🛡️ **Couche 3-7 : Pare-feu (Firewall)**

- Contrôle les communications entre réseaux
- Applique des **règles de filtrage**
- Protège contre les accès non autorisés
- Matériel actif

### Rôles selon la Position Réseau

```
         ⚡ CŒUR DE RÉSEAU (Core)
         Matériel très haut débit
              ↓
    ┌─────────────────────┐
    │   DISTRIBUTION      │
    │   (Aggregation)     │
    └─────────────────────┘
         ↓    ↓    ↓
    ┌────┐ ┌────┐ ┌────┐
    │ACC │ │ACC │ │ACC │  ACCÈS
    │    │ │    │ │    │  (connexion terminaux)
    └────┘ └────┘ └────┘
```

---

## 7. Topologies Réseaux

### Définition

Une **topologie de réseau** est l'architecture (physique ou logique) décrivant comment les équipements sont interconnectés.

### Types de Topologies

```
ÉTOILE (Star)
    ┌─────┬─────┬─────┐
    │     │     │     │
    ○─────●─────○     │
    │     │           │
    ○─────○───────────○
   (Switch/Hub central)

ANNEAU (Ring)
    ○───────○
    │       │
    ○       ○
    │       │
    ○───────○

BUS (Line)
    ○───○───○───○───○
    
MAILLÉE (Mesh)
    ○───○
    ├───┼───○
    ├───┤   │
    ○───○───○

HYBRIDE (Mixed)
    Combinaison des topologies précédentes
```

---

## 8. Récapitulatif Visuel

### Flux de Communication Simplifié

```
ÉTAPE 1 : CRÉATION (Couches hautes → couches basses)
─────────────────────────────────────────────────

Application (Couche 7)
│ Données : "Bonjour"
│
Transport (Couche 4 - TCP)
│ + Numéro de port source/destination
│ PDU = Segment + Données
│
Internet (Couche 3 - IP)
│ + Adresse IP source/destination
│ PDU = Paquet + Segment + Données
│
Liaison (Couche 2 - MAC)
│ + Adresse MAC source/destination
│ PDU = Trame + Paquet + Segment + Données
│
Physique (Couche 1)
│ Conversion en BITS
│ Transmission sur le câble
```

### Tableau de Synthèse

|Aspect|Détail|
|---|---|
|**OSI 7 couches**|Modèle théorique (grille d'analyse)|
|**TCP/IP**|Modèle pratique (Internet réel)|
|**Encapsulation**|Chaque couche ajoute son header|
|**Protocoles**|Normes standardisées (IETF, IEEE)|
|**Équipements L1**|Répéteur, Hub|
|**Équipements L2**|Switch, Pont|
|**Équipements L3**|Routeur, Switch L3|
|**Transport fiable**|TCP (avec accusé)|
|**Transport rapide**|UDP (sans vérification)|
|**Adressage L2**|MAC (réseau local)|
|**Adressage L3**|IP (Internet)|

### Schéma Global

```
┌─────────────────────────────────────────────┐
│          INTERNET / RÉSEAU                  │
├─────────────────────────────────────────────┤
│                                             │
│  ┌──────────  ROUTEUR ──────────┐           │
│  │         (Passerelle)         │           │
│  │                              │           │
│  ├──── SWITCH L3 ───────────────┤           │
│  │   (Routage inter-VLAN)       │           │
│  │                              │           │
│  ├──── SWITCH L2 ─┐             │           │
│  │   (Commutation)│             │           │
│  │                │             │           │
│  ├── PC1 ─── PC2 ─┴─ PC         │           │
│  │   (Terminaux)                │           │
│  │                              │           │
│  └──────────────────────────────┘           │
│                                             │
└─────────────────────────────────────────────┘
```

---

## 🎯 Points Clés à Retenir

✅ **Réseau** = Ensemble d'éléments reliés par protocoles et équipements

✅ **Architecture en couches** = Diviser la complexité en sous-problèmes

✅ **Encapsulation** = Chaque couche ajoute ses informations (header)

✅ **OSI** = 7 couches théoriques (grille d'analyse)

✅ **TCP/IP** = 4 couches pratiques (Internet réel)

✅ **TCP** = Fiable (courrier recommandé) ✉️

✅ **UDP** = Rapide (pas de vérification) 🎬

✅ **Switch** = Équipement clé, commutatrice L2

✅ **Routeur** = Interconnecte réseaux différents

✅ **Adresses MAC** = Locales (L2)

✅ **Adresses IP** = Globales (L3)

---

## 📚 Sources

- Document fourni : _Principe des réseaux.pdf_
- Notes personnelles du cours
- Normes ISO 7498 (Modèle OSI)
- IETF RFC (Protocoles Internet)

---

**Fin de la synthèse** | Bonne compréhension ! 🚀