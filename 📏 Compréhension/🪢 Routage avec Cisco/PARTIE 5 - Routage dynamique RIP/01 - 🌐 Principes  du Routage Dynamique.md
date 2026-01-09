

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

## 🎯 Introduction au routage dynamique

Le **routage dynamique** permet aux routeurs d'échanger automatiquement des informations de routage et d'adapter leurs tables de routage en fonction des changements topologiques du réseau. Contrairement au routage statique où chaque route doit être configurée manuellement, le routage dynamique offre une solution évolutive et auto-adaptative.

> [!info] Pourquoi utiliser le routage dynamique ?
> 
> - **Scalabilité** : Gestion automatique de grands réseaux
> - **Résilience** : Adaptation automatique aux pannes
> - **Maintenance réduite** : Moins d'interventions manuelles
> - **Convergence** : Recalcul automatique des routes optimales

### Fonctionnement général

Les protocoles de routage dynamique permettent aux routeurs de :

1. **Découvrir** les réseaux distants automatiquement
2. **Échanger** des informations de routage avec les routeurs voisins
3. **Calculer** les meilleures routes selon des métriques spécifiques
4. **Mettre à jour** la table de routage dynamiquement
5. **Maintenir** les informations à jour via des messages périodiques ou événementiels

> [!warning] Compromis à considérer Le routage dynamique consomme :
> 
> - De la **bande passante** (échange de messages)
> - De la **mémoire** (stockage des informations de routage)
> - Du **CPU** (calculs d'algorithmes de routage)

---

## 🔄 Protocoles à vecteur de distance vs état de lien

Les protocoles de routage dynamique se divisent en deux grandes familles selon leur approche algorithmique.

### 📍 Protocoles à vecteur de distance (Distance Vector)

#### Principe de fonctionnement

Les protocoles à vecteur de distance fonctionnent selon le principe **"apprendre par ouï-dire"**. Chaque routeur :

- Connaît uniquement ses **voisins directs**
- Partage sa **table de routage complète** avec ses voisins
- Calcule les routes en se basant sur les informations reçues
- Prend des décisions basées sur la **distance** et la **direction**

> [!example] Analogie C'est comme demander son chemin : "Pour aller à Paris, prends cette direction et tu es à 3 villes de distance". Vous ne connaissez pas le chemin complet, juste la prochaine étape.

#### Algorithme de Bellman-Ford

Les protocoles à vecteur de distance utilisent l'**algorithme de Bellman-Ford** :

```
Pour chaque destination :
    Distance = Distance_vers_voisin + Distance_annoncée_par_voisin
    Si Distance < Distance_actuelle :
        Mettre à jour la route
```

#### Caractéristiques principales

|Aspect|Description|
|---|---|
|**Vision du réseau**|Limitée aux voisins directs|
|**Informations échangées**|Table de routage complète|
|**Fréquence de mise à jour**|Périodique (ex: toutes les 30s)|
|**Métrique**|Nombre de sauts, bande passante, etc.|
|**Convergence**|Lente (plusieurs cycles d'échange)|

#### Avantages

- ✅ **Simplicité** de configuration et de compréhension
- ✅ **Faible consommation CPU** (calculs simples)
- ✅ **Adapté aux petits réseaux**
- ✅ **Implémentation légère**

#### Inconvénients

- ❌ **Convergence lente** (problème du "count-to-infinity")
- ❌ **Boucles de routage** possibles
- ❌ **Scalabilité limitée**
- ❌ **Connaissance partielle** du réseau

> [!warning] Problème du "Count-to-Infinity" Lorsqu'un réseau devient inaccessible, l'information de la panne se propage lentement. Les routeurs peuvent s'échanger des informations obsolètes, incrémentant progressivement la métrique jusqu'à l'infini (ou une valeur maximale).

#### Exemples de protocoles

- **RIP** (Routing Information Protocol)
- **RIPv2** (version améliorée avec VLSM)
- **IGRP** (Interior Gateway Routing Protocol - Cisco, obsolète)

---

### 🗺️ Protocoles à état de lien (Link State)

#### Principe de fonctionnement

Les protocoles à état de lien fonctionnent selon le principe **"voir pour croire"**. Chaque routeur :

- Découvre ses **voisins immédiats**
- Échange des informations sur l'**état de ses liens** (pas toute la table)
- Construit une **carte complète du réseau** (topologie)
- Calcule les meilleures routes avec l'**algorithme SPF** (Shortest Path First)

> [!example] Analogie C'est comme avoir une carte routière complète. Vous voyez tout le réseau et calculez vous-même le meilleur chemin avec un GPS.

#### Algorithme de Dijkstra (SPF)

Les protocoles à état de lien utilisent l'**algorithme de Dijkstra** :

```
1. Créer une carte topologique complète du réseau
2. Se placer comme racine de l'arbre SPF
3. Calculer le chemin le plus court vers chaque destination
4. Construire la table de routage
```

#### Caractéristiques principales

|Aspect|Description|
|---|---|
|**Vision du réseau**|Complète (carte topologique)|
|**Informations échangées**|État des liens uniquement (LSA)|
|**Fréquence de mise à jour**|Événementielle (changement détecté)|
|**Métrique**|Coût calculé (bande passante, délai)|
|**Convergence**|Rapide (recalcul immédiat)|

#### Avantages

- ✅ **Convergence rapide** (mises à jour événementielles)
- ✅ **Pas de boucles de routage**
- ✅ **Scalabilité élevée** (design hiérarchique possible)
- ✅ **Connaissance complète** du réseau
- ✅ **Calcul précis** des meilleures routes

#### Inconvénients

- ❌ **Complexité** de configuration
- ❌ **Consommation CPU élevée** (calculs SPF)
- ❌ **Consommation mémoire** (base de données topologique)
- ❌ **Temps d'initialisation** plus long

> [!tip] Quand utiliser l'état de lien ?
> 
> - Réseaux de moyenne à grande taille
> - Besoin de convergence rapide
> - Topologie complexe avec redondance
> - Environnement critique nécessitant stabilité

#### Exemples de protocoles

- **OSPF** (Open Shortest Path First)
- **IS-IS** (Intermediate System to Intermediate System)
- **EIGRP** (Enhanced Interior Gateway Routing Protocol - hybride Cisco)

---

### 🔄 Tableau comparatif

|Critère|Vecteur de distance|État de lien|
|---|---|---|
|**Algorithme**|Bellman-Ford|Dijkstra (SPF)|
|**Vision réseau**|Voisins directs|Topologie complète|
|**Échange d'infos**|Table de routage|État des liens (LSA)|
|**Fréquence MAJ**|Périodique|Événementielle|
|**Convergence**|Lente|Rapide|
|**CPU/Mémoire**|Faible|Élevé|
|**Scalabilité**|Limitée|Excellente|
|**Boucles**|Possibles|Impossible|
|**Complexité**|Simple|Complexe|

---

## 🏢 Protocoles IGP vs EGP

Les protocoles de routage se classent également selon leur **domaine d'application** : à l'intérieur d'un système autonome ou entre systèmes autonomes.

### 🔵 IGP - Interior Gateway Protocol

#### Définition

Les **IGP** (Interior Gateway Protocol) sont des protocoles de routage conçus pour fonctionner **à l'intérieur d'un système autonome** (AS - Autonomous System).

> [!info] Qu'est-ce qu'un Système Autonome (AS) ? Un AS est un ensemble de réseaux IP et de routeurs sous le contrôle d'une **seule entité administrative** (entreprise, FAI, organisation) qui présente une politique de routage commune vers Internet.

#### Caractéristiques

- **Portée** : Interne à une organisation
- **Objectif** : Optimiser les routes internes
- **Confiance** : Tous les routeurs sont sous contrôle administratif commun
- **Métriques** : Adaptées aux besoins internes (bande passante, délai, fiabilité)
- **Convergence** : Priorité à la rapidité

#### Exemples d'IGP

|Protocole|Type|Métrique|Usage|
|---|---|---|---|
|**RIP**|Vecteur de distance|Nombre de sauts|Petits réseaux|
|**RIPv2**|Vecteur de distance|Nombre de sauts|Petits réseaux avec VLSM|
|**OSPF**|État de lien|Coût (bande passante)|Réseaux moyens/grands|
|**IS-IS**|État de lien|Coût configurable|Grands réseaux, FAI|
|**EIGRP**|Hybride (Cisco)|Composite|Réseaux Cisco|

> [!example] Cas d'usage IGP Une entreprise avec plusieurs sites interconnectés utilise OSPF pour gérer le routage entre ses différents bureaux, data centers et agences.

---

### 🔴 EGP - Exterior Gateway Protocol

#### Définition

Les **EGP** (Exterior Gateway Protocol) sont des protocoles de routage conçus pour échanger des informations de routage **entre différents systèmes autonomes**.

#### Caractéristiques

- **Portée** : Entre organisations (Internet)
- **Objectif** : Connecter différents AS
- **Confiance** : Environnement non fiable (politiques différentes)
- **Métriques** : Basées sur des politiques (préférences administratives)
- **Convergence** : Stabilité prioritaire sur rapidité

#### BGP - Border Gateway Protocol

**BGP** (Border Gateway Protocol) est le protocole EGP standard d'Internet (actuellement BGPv4).

> [!info] BGP - Le protocole d'Internet BGP est le protocole qui fait fonctionner Internet en permettant aux différents FAI, entreprises et organisations de s'échanger des routes. Il gère plus de 900 000 routes sur Internet.

**Particularités de BGP** :

- Protocole de **Path Vector** (évolution du vecteur de distance)
- Basé sur des **politiques** (policy-based routing)
- Supporte des **attributs complexes** (AS-PATH, Local Preference, MED, etc.)
- Conçu pour la **scalabilité Internet**
- Prévention des boucles via l'AS-PATH

```cisco
! Exemple simplifié de configuration BGP (aperçu)
router bgp 65001
 neighbor 203.0.113.1 remote-as 65002
 network 192.168.1.0 mask 255.255.255.0
```

> [!warning] Complexité de BGP BGP est un protocole extrêmement complexe généralement utilisé uniquement par :
> 
> - Les **FAI** (Fournisseurs d'Accès Internet)
> - Les **grandes entreprises** multi-sites
> - Les **data centers** interconnectés
> - Les organisations ayant leur **propre AS**

---

### 🔄 Comparaison IGP vs EGP

|Critère|IGP|EGP (BGP)|
|---|---|---|
|**Domaine**|Intra-AS (interne)|Inter-AS (externe)|
|**Confiance**|Élevée|Faible|
|**Priorité**|Performance/rapidité|Politique/stabilité|
|**Métrique**|Technique (BP, délai)|Politique (préférences)|
|**Scalabilité**|Milliers de routes|Centaines de milliers|
|**Convergence**|Rapide|Lente mais stable|
|**Exemples**|RIP, OSPF, EIGRP|BGP|

> [!tip] Analogie
> 
> - **IGP** = Routes à l'intérieur de votre ville (vous contrôlez tout)
> - **EGP** = Routes entre différentes villes/pays (accords nécessaires)

---

## 📊 Classes de protocoles de routage

Les protocoles de routage peuvent également être classés selon leur support des **classes d'adresses** et du **VLSM/CIDR**.

### 🔵 Protocoles Classful (à classes)

#### Définition

Les protocoles **classful** ne transmettent **pas le masque de sous-réseau** dans leurs mises à jour de routage. Ils supposent l'utilisation des masques par défaut des classes A, B, et C.

#### Caractéristiques

- ❌ **Pas de VLSM** (Variable Length Subnet Mask)
- ❌ **Pas de CIDR** (Classless Inter-Domain Routing)
- ❌ **Résumé automatique** aux frontières de classe
- ❌ **Gaspillage d'adresses** potentiel

#### Fonctionnement

```
Classe A : /8  (255.0.0.0)
Classe B : /16 (255.255.0.0)
Classe C : /24 (255.255.255.0)

Mise à jour RIPv1 : "Réseau 192.168.1.0 existe"
(Le masque /24 est supposé automatiquement)
```

> [!warning] Limitations des protocoles classful
> 
> - Impossible d'avoir des sous-réseaux de tailles différentes
> - Résumé automatique peut causer des problèmes
> - Inadapté aux architectures modernes
> - Gaspillage d'espace d'adressage

#### Exemples

- **RIPv1** (Routing Information Protocol version 1)
- **IGRP** (Interior Gateway Routing Protocol - Cisco, obsolète)

---

### 🔴 Protocoles Classless (sans classe)

#### Définition

Les protocoles **classless** transmettent le **masque de sous-réseau** avec chaque réseau dans leurs mises à jour. Ils supportent le VLSM et le CIDR.

#### Caractéristiques

- ✅ **Support VLSM** (sous-réseaux de tailles variables)
- ✅ **Support CIDR** (notation /prefix)
- ✅ **Résumé manuel** contrôlé
- ✅ **Utilisation efficace** des adresses IP

#### Fonctionnement

```
Mise à jour RIPv2 : "Réseau 192.168.1.0/26 existe"
Mise à jour RIPv2 : "Réseau 192.168.1.64/27 existe"
Mise à jour RIPv2 : "Réseau 192.168.1.96/28 existe"

(Chaque réseau peut avoir un masque différent)
```

> [!tip] Avantages du classless
> 
> - **Flexibilité** : Adaptation aux besoins réels
> - **Économie** : Pas de gaspillage d'adresses
> - **Agrégation** : Résumé de routes contrôlé
> - **Modernité** : Adapté à Internet actuel

#### Exemples

- **RIPv2** (évolution de RIPv1)
- **OSPF** (Open Shortest Path First)
- **EIGRP** (Enhanced IGRP - Cisco)
- **IS-IS**
- **BGP**

---

### 🔄 Tableau comparatif Classful vs Classless

|Aspect|Classful|Classless|
|---|---|---|
|**Masque transmis**|❌ Non|✅ Oui|
|**VLSM**|❌ Non supporté|✅ Supporté|
|**CIDR**|❌ Non supporté|✅ Supporté|
|**Résumé**|Automatique|Manuel/contrôlé|
|**Gaspillage IP**|Possible|Minimal|
|**Flexibilité**|Faible|Élevée|
|**Exemples**|RIPv1, IGRP|RIPv2, OSPF, EIGRP|

---

### 📋 Classification globale des protocoles

Voici une vue d'ensemble de la classification complète :

```
Protocoles de routage
│
├─ IGP (Interior Gateway Protocol)
│  │
│  ├─ Vecteur de distance
│  │  ├─ Classful : RIPv1, IGRP
│  │  └─ Classless : RIPv2
│  │
│  ├─ État de lien
│  │  └─ Classless : OSPF, IS-IS
│  │
│  └─ Hybride
│     └─ Classless : EIGRP
│
└─ EGP (Exterior Gateway Protocol)
   └─ Path Vector
      └─ Classless : BGP
```

> [!info] Choix d'un protocole Le choix dépend de :
> 
> - **Taille du réseau** (petit : RIP, moyen/grand : OSPF)
> - **Topologie** (simple : RIP, complexe : OSPF)
> - **Équipements** (multi-vendeurs : OSPF/RIP, Cisco : EIGRP)
> - **Compétences** (débutants : RIP, experts : OSPF/BGP)
> - **Besoins** (convergence rapide : OSPF, simplicité : RIP)

---

## 🎯 Synthèse des concepts clés

> [!tip] Points essentiels à retenir
> 
> **Vecteur de distance** : Simple, lent, vision locale (RIP) **État de lien** : Complexe, rapide, vision globale (OSPF)
> 
> **IGP** : Routage interne à l'organisation (RIP, OSPF, EIGRP) **EGP** : Routage entre organisations, Internet (BGP)
> 
> **Classful** : Pas de masque transmis, obsolète (RIPv1) **Classless** : Masque transmis, moderne, VLSM (RIPv2, OSPF)

### Critères de sélection

|Besoin|Protocole recommandé|
|---|---|
|Petit réseau simple|RIPv2|
|Réseau moyen/complexe|OSPF|
|Réseau Cisco uniquement|EIGRP|
|Interconnexion FAI|BGP|
|Convergence rapide|OSPF, EIGRP|
|Facilité de gestion|RIPv2|

---

> [!warning] Prérequis pour la suite Ces concepts fondamentaux sont essentiels pour comprendre la configuration de RIP qui sera abordée dans les parties suivantes. Assurez-vous de bien maîtriser les différences entre vecteur de distance et état de lien, ainsi que les notions de classful/classless.