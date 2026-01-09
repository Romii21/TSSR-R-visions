# Architecture des Ordinateurs - Synthèse Complète

## 📋 Sommaire

1. [Qu'est-ce qu'un ordinateur ?](https://claude.ai/chat/cea50fe0-b3d2-499d-bc2d-de7b59efd6bb#quest-ce-quun-ordinateur)
2. [Unités et codage de l'information](https://claude.ai/chat/cea50fe0-b3d2-499d-bc2d-de7b59efd6bb#unit%C3%A9s-et-codage-de-linformation)
3. [Les composants principaux](https://claude.ai/chat/cea50fe0-b3d2-499d-bc2d-de7b59efd6bb#les-composants-principaux)
    - [Le CPU](https://claude.ai/chat/cea50fe0-b3d2-499d-bc2d-de7b59efd6bb#le-cpu)
    - [La RAM](https://claude.ai/chat/cea50fe0-b3d2-499d-bc2d-de7b59efd6bb#la-ram)
    - [La carte mère](https://claude.ai/chat/cea50fe0-b3d2-499d-bc2d-de7b59efd6bb#la-carte-m%C3%A8re)
    - [Le boîtier](https://claude.ai/chat/cea50fe0-b3d2-499d-bc2d-de7b59efd6bb#le-bo%C3%AEtier)
4. [Le stockage](https://claude.ai/chat/cea50fe0-b3d2-499d-bc2d-de7b59efd6bb#le-stockage)
    - [Types de disques](https://claude.ai/chat/cea50fe0-b3d2-499d-bc2d-de7b59efd6bb#types-de-disques)
    - [Partitionnement](https://claude.ai/chat/cea50fe0-b3d2-499d-bc2d-de7b59efd6bb#partitionnement)
5. [Les périphériques I/O](https://claude.ai/chat/cea50fe0-b3d2-499d-bc2d-de7b59efd6bb#les-p%C3%A9riph%C3%A9riques-io)
6. [Résumé visuel](https://claude.ai/chat/cea50fe0-b3d2-499d-bc2d-de7b59efd6bb#r%C3%A9sum%C3%A9-visuel)

---

## Qu'est-ce qu'un ordinateur ?

Un ordinateur est une **machine électronique, numérique et programmable** capable d'effectuer des opérations arithmétiques de base automatiquement. C'est l'outil de calcul universel par excellence !

### Architecture conceptuelle : Von Neumann

```
┌─────────────────────────────────────┐
│           ORDINATEUR                │
├─────────────────────────────────────┤
│                                     │
│  ┌──────────┐      ┌──────────┐     │
│  │ ENTRÉE   │      │ SORTIE   │     │
│  └────┬─────┘      └─────┬────┘     │
│       │                  │          │
│  ┌────▼─────────────────▼────┐      │
│  │   UNITÉ DE CONTRÔLE       │      │
│  │   (Séquence les opérations)      │
│  └────┬──────────────────┬───┘      │
│       │                  │          │
│  ┌────▼──┐           ┌───▼───┐      │
│  │  UAL  │           │MÉMOIRE│      │
│  │ (Calcul)          │       │      │
│  └───────┘           └───────┘      │
│                                     │
└─────────────────────────────────────┘
```

**Les 4 éléments clés :**

- **UAL (Unité Arithmétique et Logique)** : Effectue les calculs
- **Unité de contrôle** : Dirige le flux des opérations
- **Mémoire** : Stocke données et programmes
- **Entrées/Sorties (I/O)** : Communique avec l'extérieur

---

## Unités et codage de l'information

### Le BIT : L'unité élémentaire

Le **BIT** (Binary Digit) est l'unité de base du calcul informatique.

- **Une seule valeur possible : 0 ou 1**
- Représente deux états distincts (allumé/éteint, vrai/faux, etc.)

### L'OCTET : L'unité de mesure

Un **OCTET** = **8 bits** regroupés ensemble

```
1 octet : [1][0][1][0][0][1][0][1]
           ↑ ↑ ↑ ↑ ↑ ↑ ↑ ↑
           8 positions binaires
```

**Nombre de valeurs possibles :**

- 2 possibilités × 2 possibilités × ... (8 fois)
- = 2⁸ = **256 valeurs différentes** (de 0 à 255)

### Multiples de l'octet

```
1 Kio (kibioctet)    = 1 024 octets
1 Mio (mébioctet)    = 1 024 Kio
1 Gio (gibioctet)    = 1 024 Mio
1 Tio (tébioctet)    = 1 024 Gio
```

---

## Les composants principaux

### Le CPU

Le **CPU (Central Processing Unit)** est le **cerveau de l'ordinateur**. C'est un circuit intégré qui regroupe :

```
┌─────────────────────────────┐
│         CPU                 │
├─────────────────────────────┤
│ • UAL (Unité Arithmétique)  │
│ • Unité de contrôle         │
│ • Horloge (fréquence)       │
│ • Registres (mémoire rapide)│
│ • Mémoires caches (L1, L2, L3)│
└─────────────────────────────┘
```

**Caractéristiques principales :**

|Caractéristique|Exemple PC|Exemple Raspberry Pi 4|
|---|---|---|
|**Nombre de cœurs**|1 à 16+|4|
|**Taille des registres**|64 bits|64 bits|
|**Fréquence**|1 à 5 GHz|1,5 GHz|
|**Jeu d'instructions**|x86-64|ARMv8|
|**Cache L1**|10aine kio/cœur|80 kio/cœur|
|**Cache L2**|100aine kio/cœur|1 Mio/cœur|

### La RAM

La **RAM (Random Access Memory)** est la **mémoire vive** de l'ordinateur.

**Caractéristiques :**

- ✅ **Accès direct** : On peut accéder à n'importe quelle donnée instantanément
- ⚠️ **Temporaire** : Se vide complètement à l'extinction du PC
- 🔄 **En rotation constante** : Les données s'y échangent en continu
- 📊 **Stockage par octets** : Chaque adresse mémoire correspond à 1 octet (8 bits)

**Important :** La RAM n'est PAS une mémoire de masse. Elle est destinée au stockage temporaire des données et programmes en cours d'utilisation.

### Hiérarchie mémoire : Vitesse vs Capacité

```
PLUS RAPIDE
PLUS COÛTEUX
PLUS PETIT
     ↓
┌─────────────────┬──────────────┐
│  REGISTRES      │ ns (nanosecondes) │
├─────────────────┼──────────────┤
│  Cache L1       │ kio          │
├─────────────────┼──────────────┤
│  Cache L2       │ 100aine kio  │
├─────────────────┼──────────────┤
│  Cache L3       │ 10aine Mio   │
├─────────────────┼──────────────┤
│  RAM            │ 10aine Gio   │
├─────────────────┼──────────────┤
│  SSD            │ ms (millisecondes) │
├─────────────────┼──────────────┤
│  HDD            │ 10aine ms    │
├─────────────────┼──────────────┤
│  Stockage loint │ s (secondes)  │
└─────────────────┴──────────────┘
     ↑
PLUS LENT
MOINS COÛTEUX
PLUS GRAND
```

### La carte mère

La **carte mère** est la **plaque de base** qui relie tous les composants.

**Éléments clés :**

- **Chipset** : Gère la communication entre les composants
- **Support processeur** : Accueille le CPU
- **Connecteurs mémoire** : Pour la RAM
- **Connecteurs PCI Express** : Pour les cartes additionnelles (carte graphique, etc.)
- **Connecteurs stockage** : M.2, NVMe, SATA
- **Périphériques I/O intégrés** : Carte réseau, son, WiFi, etc.

**Format :** ATX, microATX, mini-ITX (différentes tailles)

### Firmware de la carte mère

Le **firmware** est un petit programme embarqué dans une **mémoire morte (ROM)** qui démarre avant le système d'exploitation.

```
BIOS (Basic Input Output System)
│
├─ Détecte les périphériques
├─ Configuration générale
└─ Choisit le périphérique de démarrage
           ↓
        UEFI (Unified Extensible Firmware Interface)
        │
        ├─ Fonctionnalités avancées
        └─ Meilleure gestion du démarrage
```

**UEFI** est le successeur moderne du BIOS avec plus de fonctionnalités.

### Le boîtier

Le **boîtier** est l'enveloppe physique qui contient et protège tous les composants.

**Éléments importants :**

- **Format** : Desktop, tour, rackable, etc.
- **Alimentation (PSU)** : Fournit l'électricité nécessaire (puissance + rendement)
- **Refroidissement** : Circulation d'air pour éviter la surchauffe

---

## Le stockage

### Types de disques

#### 1. HDD (Hard Disk Drive) - Disque dur mécanique

```
┌──────────────────────┐
│     PLATEAU          │
│  (disque qui tourne) │
│                      │
│  Tête de lecture → O │  ← Secteur
│  (bouger + lire)     │
└──────────────────────┘
```

**Caractéristiques :**

- 🔄 **Mécanique** : Plateau qui tourne (~7200 tr/min en entreprise)
- 📊 **Lecture séquentielle** : Plus rapide en accès continu
- 💪 **Robustesse** : Peut tourner 24h/24 en entreprise
- ⚠️ **Risques** : Panne mécanique (choc, usure)
- **En entreprise** : Souvent doublés ou triplés pour éviter la perte de données

#### 2. SSD (Solid State Drive) - Disque électronique

```
┌─────────────────────┐
│  Mémoire FLASH      │
│  (pas de pièces     │
│   mobiles)          │
└─────────────────────┘
```

**Caractéristiques :**

- ⚡ **Électronique** : Pas de pièces mobiles
- 🚀 **Plus rapide** : Accès ultra-rapide
- 🔋 **Fiable** : Moins de pannes mécaniques
- ⚠️ **Limite d'écriture** : La mémoire flash s'use avec le temps (mais ça prend des années)

### Taille des secteurs

Les données sont organisées en **secteurs** sur le disque.

```
Disque = Suite de secteurs
Chaque secteur = Suite binaire
```

**Calcul important :**

- Adresses 32 bits + secteurs de 512 octets = **2 Tio maximum**
- Adresses 64 bits + secteurs de 512 octets = **8 Zio (zébioctets) maximum** 🤯

**Taille habituelle** : **512 octets par secteur**

### Partitionnement

Le **partitionnement** divise le disque en plusieurs sections (_partitions_) gérables indépendamment.

**Pourquoi partitionner ?**

- Organiser les données
- Avoir plusieurs systèmes d'exploitation
- Protéger les données importantes
- Améliorer la performance

#### MBR (Master Boot Record) - Système historique

```
┌─────────────────────────────────┐
│  Secteur 1 : Zone d'amorçage    │
├─────────────────────────────────┤
│  Table de partitions            │
│  → 4 partitions primaires max   │
│  → 1 peut être partition étendue│
│     └─ Contient partitions      │
│        logiques (secondaires)   │
└─────────────────────────────────┘
```

**Limitations :**

- Partitions **jusqu'à 2 Tio maximum**
- Complexe à gérer au-delà de 4 partitions

#### GPT (GUID Partition Table) - Système moderne

```
┌──────────────────────────────┐
│ Secteur 1 : MBR protecteur   │
├──────────────────────────────┤
│ Secteur 2 : En-tête GPT      │
├──────────────────────────────┤
│ Secteurs 3-34 : Table (128   │
│ partitions max, identifiées  │
│ par GUID)                    │
└──────────────────────────────┘
```

**Avantages :**

- ✅ Partitions **jusqu'à 8 Zio maximum** (énorme !)
- ✅ **128 partitions** au lieu de 4
- ✅ Identification unique (GUID) pour chaque partition
- ✅ Plus robuste et moderne

---

## Les périphériques I/O

Les périphériques d'entrée/sortie permettent à l'ordinateur de communiquer avec le monde extérieur.

### Périphériques d'entrée 🔌

Ils apportent des informations À la machine :

- **Clavier** : Saisie de texte
- **Souris** : Pointage
- **Écran tactile** : Saisie + affichage
- **Webcam** : Capture vidéo
- **Microphone** : Capture audio
- **Scanner** : Numérisation de documents
- **Carte réseau** : Réception de données

### Périphériques de sortie 📤

Ils envoient des informations DE la machine :

- **Écran** : Affichage visuel
- **Haut-parleurs** : Son
- **Casque audio** : Son privatif
- **Imprimante** : Impression papier
- **Projecteur** : Affichage grand format
- **Carte son** : Traitement audio
- **Carte réseau** : Envoi de données

---

## Résumé visuel

### Schéma complet d'un ordinateur

```
╔════════════════════════════════════════════════════════════════╗
║                    ORDINATEUR COMPLET                         ║
╠════════════════════════════════════════════════════════════════╣
║                                                                ║
║  ENTRÉES          ┌─────────────────────────────────────────┐ ║
║  • Clavier   ────→│         BOÎTIER (Case)               │ ║
║  • Souris    ────→│                                      │ ║
║  • Webcam    ────→│  ┌─────────────────────────────────┐ │ ║
║  • Micro     ────→│  │    CARTE MÈRE                  │ │ ║
║  • Réseau    ────→│  │  ┌──────┐  ┌──────┐ ┌──────┐ │ │ ║
║                   │  │  │ CPU  │  │ RAM  │ │BIOS  │ │ │ ║
║                   │  │  └──────┘  └──────┘ └──────┘ │ │ ║
║                   │  │  ┌──────────────────────────┐ │ │ ║
║                   │  │  │  Connecteurs (PCI, SATA) │ │ │ ║
║                   │  │  └──────────────────────────┘ │ │ ║
║                   │  └─────────────────────────────────┘ │ ║
║                   │  ┌──────────────────────────────────┐ │ ║
║                   │  │  STOCKAGE                       │ │ ║
║                   │  │  • SSD (rapide)                 │ │ ║
║                   │  │  • HDD (grande capacité)        │ │ ║
║                   │  └──────────────────────────────────┘ │ ║
║                   │  ┌──────────────────────────────────┐ │ ║
║                   │  │  ALIMENTATION (PSU)             │ │ ║
║                   │  └──────────────────────────────────┘ │ ║
║                   └─────────────────────────────────────────┘ ║
║                                                                ║
║  SORTIES          ┌─────────────────────────────────────────┐ ║
║  • Écran    ←────→│         REFROIDISSEMENT                │ ║
║  • HP       ←────→│    (Ventilateurs, circulation)          │ ║
║  • Casque   ←────→└─────────────────────────────────────────┘ ║
║  • Imprim.  ←────                                             ║
║  • Réseau   ←────                                             ║
║                                                                ║
╚════════════════════════════════════════════════════════════════╝
```

### Points clés à retenir

```
📌 BINAIRE
   └─ 1 bit = 0 ou 1
      8 bits (1 octet) = 256 valeurs possibles

🧠 CPU
   └─ Cerveau de l'ordinateur
      Exécute les instructions

💾 RAM
   └─ Mémoire temporaire ultra-rapide
      Se vide à l'extinction

📀 STOCKAGE
   └─ HDD : Grand volume, mécanique
      SSD : Rapide, électronique

🔌 I/O
   └─ Communication avec l'extérieur
      Entrées (clavier, souris...)
      Sorties (écran, imprimante...)
```

---

## Sources et notes

- **Document principal** : Architecture des ordinateurs - Notions de base
- **Notes personnelles** : Compléments pratiques
- **Concepts fondamentaux** : Basés sur l'architecture de Von Neumann et la théorie informatique classique

---

**Dernier point important :** Tous ces composants travaillent en harmonie pour transformer l'électricité en calculs utiles. C'est magique ! ✨# Architecture des Ordinateurs - Synthèse Complète

## 📋 Sommaire

1. [Qu'est-ce qu'un ordinateur ?](https://claude.ai/chat/cea50fe0-b3d2-499d-bc2d-de7b59efd6bb#quest-ce-quun-ordinateur)
2. [Unités et codage de l'information](https://claude.ai/chat/cea50fe0-b3d2-499d-bc2d-de7b59efd6bb#unit%C3%A9s-et-codage-de-linformation)
3. [Les composants principaux](https://claude.ai/chat/cea50fe0-b3d2-499d-bc2d-de7b59efd6bb#les-composants-principaux)
    - [Le CPU](https://claude.ai/chat/cea50fe0-b3d2-499d-bc2d-de7b59efd6bb#le-cpu)
    - [La RAM](https://claude.ai/chat/cea50fe0-b3d2-499d-bc2d-de7b59efd6bb#la-ram)
    - [La carte mère](https://claude.ai/chat/cea50fe0-b3d2-499d-bc2d-de7b59efd6bb#la-carte-m%C3%A8re)
    - [Le boîtier](https://claude.ai/chat/cea50fe0-b3d2-499d-bc2d-de7b59efd6bb#le-bo%C3%AEtier)
4. [Le stockage](https://claude.ai/chat/cea50fe0-b3d2-499d-bc2d-de7b59efd6bb#le-stockage)
    - [Types de disques](https://claude.ai/chat/cea50fe0-b3d2-499d-bc2d-de7b59efd6bb#types-de-disques)
    - [Partitionnement](https://claude.ai/chat/cea50fe0-b3d2-499d-bc2d-de7b59efd6bb#partitionnement)
5. [Les périphériques I/O](https://claude.ai/chat/cea50fe0-b3d2-499d-bc2d-de7b59efd6bb#les-p%C3%A9riph%C3%A9riques-io)
6. [Résumé visuel](https://claude.ai/chat/cea50fe0-b3d2-499d-bc2d-de7b59efd6bb#r%C3%A9sum%C3%A9-visuel)

---

## Qu'est-ce qu'un ordinateur ?

Un ordinateur est une **machine électronique, numérique et programmable** capable d'effectuer des opérations arithmétiques de base automatiquement. C'est l'outil de calcul universel par excellence !

### Architecture conceptuelle : Von Neumann

```
┌─────────────────────────────────────┐
│           ORDINATEUR                │
├─────────────────────────────────────┤
│                                     │
│  ┌──────────┐      ┌──────────┐   │
│  │ ENTRÉE   │      │ SORTIE   │   │
│  └────┬─────┘      └─────┬────┘   │
│       │                   │        │
│  ┌────▼─────────────────▼────┐   │
│  │   UNITÉ DE CONTRÔLE       │   │
│  │   (Séquence les opérations)   │
│  └────┬──────────────────┬────┘   │
│       │                  │        │
│  ┌────▼──┐          ┌───▼───┐   │
│  │  UAL  │          │MÉMOIRE│   │
│  │ (Calcul)         │       │   │
│  └───────┘          └───────┘   │
│                                     │
└─────────────────────────────────────┘
```

**Les 4 éléments clés :**

- **UAL (Unité Arithmétique et Logique)** : Effectue les calculs
- **Unité de contrôle** : Dirige le flux des opérations
- **Mémoire** : Stocke données et programmes
- **Entrées/Sorties (I/O)** : Communique avec l'extérieur

---

## Unités et codage de l'information

### Le BIT : L'unité élémentaire

Le **BIT** (Binary Digit) est l'unité de base du calcul informatique.

- **Une seule valeur possible : 0 ou 1**
- Représente deux états distincts (allumé/éteint, vrai/faux, etc.)

### L'OCTET : L'unité de mesure

Un **OCTET** = **8 bits** regroupés ensemble

```
1 octet : [1][0][1][0][0][1][0][1]
           ↑ ↑ ↑ ↑ ↑ ↑ ↑ ↑
           8 positions binaires
```

**Nombre de valeurs possibles :**

- 2 possibilités × 2 possibilités × ... (8 fois)
- = 2⁸ = **256 valeurs différentes** (de 0 à 255)

### Multiples de l'octet

```
1 Kio (kibioctet)    = 1 024 octets
1 Mio (mébioctet)    = 1 024 Kio
1 Gio (gibioctet)    = 1 024 Mio
1 Tio (tébioctet)    = 1 024 Gio
```

---

## Les composants principaux

### Le CPU

Le **CPU (Central Processing Unit)** est le **cerveau de l'ordinateur**. C'est un circuit intégré qui regroupe :

```
┌─────────────────────────────┐
│         CPU                 │
├─────────────────────────────┤
│ • UAL (Unité Arithmétique)  │
│ • Unité de contrôle         │
│ • Horloge (fréquence)       │
│ • Registres (mémoire rapide)│
│ • Mémoires caches (L1, L2, L3)│
└─────────────────────────────┘
```

**Caractéristiques principales :**

|Caractéristique|Exemple PC|Exemple Raspberry Pi 4|
|---|---|---|
|**Nombre de cœurs**|1 à 16+|4|
|**Taille des registres**|64 bits|64 bits|
|**Fréquence**|1 à 5 GHz|1,5 GHz|
|**Jeu d'instructions**|x86-64|ARMv8|
|**Cache L1**|10aine kio/cœur|80 kio/cœur|
|**Cache L2**|100aine kio/cœur|1 Mio/cœur|

### La RAM

La **RAM (Random Access Memory)** est la **mémoire vive** de l'ordinateur.

**Caractéristiques :**

- ✅ **Accès direct** : On peut accéder à n'importe quelle donnée instantanément
- ⚠️ **Temporaire** : Se vide complètement à l'extinction du PC
- 🔄 **En rotation constante** : Les données s'y échangent en continu
- 📊 **Stockage par octets** : Chaque adresse mémoire correspond à 1 octet (8 bits)

**Important :** La RAM n'est PAS une mémoire de masse. Elle est destinée au stockage temporaire des données et programmes en cours d'utilisation.

### Hiérarchie mémoire : Vitesse vs Capacité

```
PLUS RAPIDE
PLUS COÛTEUX
PLUS PETIT
     ↓
┌─────────────────┬──────────────┐
│  REGISTRES      │ ns (nanosecondes) │
├─────────────────┼──────────────┤
│  Cache L1       │ kio          │
├─────────────────┼──────────────┤
│  Cache L2       │ 100aine kio  │
├─────────────────┼──────────────┤
│  Cache L3       │ 10aine Mio   │
├─────────────────┼──────────────┤
│  RAM            │ 10aine Gio   │
├─────────────────┼──────────────┤
│  SSD            │ ms (millisecondes) │
├─────────────────┼──────────────┤
│  HDD            │ 10aine ms    │
├─────────────────┼──────────────┤
│  Stockage loint │ s (secondes)  │
└─────────────────┴──────────────┘
     ↑
PLUS LENT
MOINS COÛTEUX
PLUS GRAND
```

### La carte mère

La **carte mère** est la **plaque de base** qui relie tous les composants.

**Éléments clés :**

- **Chipset** : Gère la communication entre les composants
- **Support processeur** : Accueille le CPU
- **Connecteurs mémoire** : Pour la RAM
- **Connecteurs PCI Express** : Pour les cartes additionnelles (carte graphique, etc.)
- **Connecteurs stockage** : M.2, NVMe, SATA
- **Périphériques I/O intégrés** : Carte réseau, son, WiFi, etc.

**Format :** ATX, microATX, mini-ITX (différentes tailles)

### Firmware de la carte mère

Le **firmware** est un petit programme embarqué dans une **mémoire morte (ROM)** qui démarre avant le système d'exploitation.

```
BIOS (Basic Input Output System)
│
├─ Détecte les périphériques
├─ Configuration générale
└─ Choisit le périphérique de démarrage
           ↓
        UEFI (Unified Extensible Firmware Interface)
        │
        ├─ Fonctionnalités avancées
        └─ Meilleure gestion du démarrage
```

**UEFI** est le successeur moderne du BIOS avec plus de fonctionnalités.

### Le boîtier

Le **boîtier** est l'enveloppe physique qui contient et protège tous les composants.

**Éléments importants :**

- **Format** : Desktop, tour, rackable, etc.
- **Alimentation (PSU)** : Fournit l'électricité nécessaire (puissance + rendement)
- **Refroidissement** : Circulation d'air pour éviter la surchauffe

---

## Le stockage

### Types de disques

#### 1. HDD (Hard Disk Drive) - Disque dur mécanique

```
┌──────────────────────┐
│     PLATEAU          │
│  (disque qui tourne) │
│                      │
│  Tête de lecture → O │  ← Secteur
│  (bouger + lire)     │
└──────────────────────┘
```

**Caractéristiques :**

- 🔄 **Mécanique** : Plateau qui tourne (~7200 tr/min en entreprise)
- 📊 **Lecture séquentielle** : Plus rapide en accès continu
- 💪 **Robustesse** : Peut tourner 24h/24 en entreprise
- ⚠️ **Risques** : Panne mécanique (choc, usure)
- **En entreprise** : Souvent doublés ou triplés pour éviter la perte de données

#### 2. SSD (Solid State Drive) - Disque électronique

```
┌─────────────────────┐
│  Mémoire FLASH      │
│  (pas de pièces     │
│   mobiles)          │
└─────────────────────┘
```

**Caractéristiques :**

- ⚡ **Électronique** : Pas de pièces mobiles
- 🚀 **Plus rapide** : Accès ultra-rapide
- 🔋 **Fiable** : Moins de pannes mécaniques
- ⚠️ **Limite d'écriture** : La mémoire flash s'use avec le temps (mais ça prend des années)

### Taille des secteurs

Les données sont organisées en **secteurs** sur le disque.

```
Disque = Suite de secteurs
Chaque secteur = Suite binaire
```

**Calcul important :**

- Adresses 32 bits + secteurs de 512 octets = **2 Tio maximum**
- Adresses 64 bits + secteurs de 512 octets = **8 Zio (zébioctets) maximum** 🤯

**Taille habituelle** : **512 octets par secteur**

### Partitionnement

Le **partitionnement** divise le disque en plusieurs sections (_partitions_) gérables indépendamment.

**Pourquoi partitionner ?**

- Organiser les données
- Avoir plusieurs systèmes d'exploitation
- Protéger les données importantes
- Améliorer la performance

#### MBR (Master Boot Record) - Système historique

```
┌─────────────────────────────────┐
│  Secteur 1 : Zone d'amorçage    │
├─────────────────────────────────┤
│  Table de partitions            │
│  → 4 partitions primaires max   │
│  → 1 peut être partition étendue│
│     └─ Contient partitions      │
│        logiques (secondaires)   │
└─────────────────────────────────┘
```

**Limitations :**

- Partitions **jusqu'à 2 Tio maximum**
- Complexe à gérer au-delà de 4 partitions

#### GPT (GUID Partition Table) - Système moderne

```
┌──────────────────────────────┐
│ Secteur 1 : MBR protecteur   │
├──────────────────────────────┤
│ Secteur 2 : En-tête GPT      │
├──────────────────────────────┤
│ Secteurs 3-34 : Table (128   │
│ partitions max, identifiées  │
│ par GUID)                    │
└──────────────────────────────┘
```

**Avantages :**

- ✅ Partitions **jusqu'à 8 Zio maximum** (énorme !)
- ✅ **128 partitions** au lieu de 4
- ✅ Identification unique (GUID) pour chaque partition
- ✅ Plus robuste et moderne

---

## Les périphériques I/O

Les périphériques d'entrée/sortie permettent à l'ordinateur de communiquer avec le monde extérieur.

### Périphériques d'entrée 🔌

Ils apportent des informations À la machine :

- **Clavier** : Saisie de texte
- **Souris** : Pointage
- **Écran tactile** : Saisie + affichage
- **Webcam** : Capture vidéo
- **Microphone** : Capture audio
- **Scanner** : Numérisation de documents
- **Carte réseau** : Réception de données

### Périphériques de sortie 📤

Ils envoient des informations DE la machine :

- **Écran** : Affichage visuel
- **Haut-parleurs** : Son
- **Casque audio** : Son privatif
- **Imprimante** : Impression papier
- **Projecteur** : Affichage grand format
- **Carte son** : Traitement audio
- **Carte réseau** : Envoi de données

---

## Résumé visuel

### Schéma complet d'un ordinateur

```
╔════════════════════════════════════════════════════════════════╗
║                    ORDINATEUR COMPLET                         ║
╠════════════════════════════════════════════════════════════════╣
║                                                                ║
║  ENTRÉES          ┌─────────────────────────────────────────┐ ║
║  • Clavier   ────→│         BOÎTIER (Case)               │ ║
║  • Souris    ────→│                                      │ ║
║  • Webcam    ────→│  ┌─────────────────────────────────┐ │ ║
║  • Micro     ────→│  │    CARTE MÈRE                  │ │ ║
║  • Réseau    ────→│  │  ┌──────┐  ┌──────┐ ┌──────┐ │ │ ║
║                   │  │  │ CPU  │  │ RAM  │ │BIOS  │ │ │ ║
║                   │  │  └──────┘  └──────┘ └──────┘ │ │ ║
║                   │  │  ┌──────────────────────────┐ │ │ ║
║                   │  │  │  Connecteurs (PCI, SATA) │ │ │ ║
║                   │  │  └──────────────────────────┘ │ │ ║
║                   │  └─────────────────────────────────┘ │ ║
║                   │  ┌──────────────────────────────────┐ │ ║
║                   │  │  STOCKAGE                       │ │ ║
║                   │  │  • SSD (rapide)                 │ │ ║
║                   │  │  • HDD (grande capacité)        │ │ ║
║                   │  └──────────────────────────────────┘ │ ║
║                   │  ┌──────────────────────────────────┐ │ ║
║                   │  │  ALIMENTATION (PSU)             │ │ ║
║                   │  └──────────────────────────────────┘ │ ║
║                   └─────────────────────────────────────────┘ ║
║                                                                ║
║  SORTIES          ┌─────────────────────────────────────────┐ ║
║  • Écran    ←────→│         REFROIDISSEMENT                │ ║
║  • HP       ←────→│    (Ventilateurs, circulation)          │ ║
║  • Casque   ←────→└─────────────────────────────────────────┘ ║
║  • Imprim.  ←────                                             ║
║  • Réseau   ←────                                             ║
║                                                                ║
╚════════════════════════════════════════════════════════════════╝
```

### Points clés à retenir

```
📌 BINAIRE
   └─ 1 bit = 0 ou 1
      8 bits (1 octet) = 256 valeurs possibles

🧠 CPU
   └─ Cerveau de l'ordinateur
      Exécute les instructions

💾 RAM
   └─ Mémoire temporaire ultra-rapide
      Se vide à l'extinction

📀 STOCKAGE
   └─ HDD : Grand volume, mécanique
      SSD : Rapide, électronique

🔌 I/O
   └─ Communication avec l'extérieur
      Entrées (clavier, souris...)
      Sorties (écran, imprimante...)
```

---

## Sources et notes

- **Document principal** : Architecture des ordinateurs - Notions de base
- **Notes personnelles** : Compléments pratiques
- **Concepts fondamentaux** : Basés sur l'architecture de Von Neumann et la théorie informatique classique

---

**Dernier point important :** Tous ces composants travaillent en harmonie pour transformer l'électricité en calculs utiles. C'est magique ! ✨