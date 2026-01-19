## Définition

Le **découpage réseau** (ou **subnetting**) consiste à diviser un réseau IP en plusieurs sous-réseaux plus petits. Cette technique permet d'optimiser l'utilisation des adresses IP et d'améliorer la gestion et la sécurité du réseau.

## Objectifs du découpage

|Objectif|Description|
|---|---|
|**Optimisation**|Utiliser efficacement l'espace d'adressage disponible|
|**Organisation**|Séparer logiquement les services (production, administration, invités)|
|**Sécurité**|Isoler les segments de réseau et contrôler les flux|
|**Performance**|Réduire les domaines de broadcast et améliorer le trafic|

## Principe de base

Un réseau IP se compose de deux parties :

- **Préfixe réseau** : identifie le réseau
- **Identifiant d'hôte** : identifie les machines sur ce réseau

Le découpage consiste à **emprunter des bits** à la partie hôte pour créer de nouveaux sous-réseaux.

## Notation CIDR (Classless Inter-Domain Routing)

Format : `adresse_IP/longueur_préfixe`

**Exemple** : `192.168.1.0/24`

- `/24` signifie que les 24 premiers bits identifient le réseau
- Les 8 bits restants (32-24) identifient les hôtes

## Calcul des sous-réseaux

### Formules essentielles

|Élément|Formule|Description|
|---|---|---|
|**Nombre de sous-réseaux**|2^n|n = bits empruntés|
|**Nombre d'hôtes par sous-réseau**|2^h - 2|h = bits restants pour les hôtes|
|**Masque de sous-réseau**|Bits à 1 pour le réseau, 0 pour les hôtes|Exemple : /26 = 255.255.255.192|

### Pourquoi soustraire 2 pour les hôtes ?

Deux adresses sont réservées dans chaque sous-réseau :

- **Adresse réseau** : tous les bits hôte à 0
- **Adresse broadcast** : tous les bits hôte à 1

## Exemple pratique

**Situation** : Vous avez le réseau `192.168.1.0/24` et devez créer 4 sous-réseaux.

**Étape 1** : Calculer les bits nécessaires

- 4 sous-réseaux nécessitent 2 bits (2^2 = 4)
- Nouveau préfixe : /24 + 2 = **/26**

**Étape 2** : Calculer le nombre d'hôtes

- Bits restants : 32 - 26 = 6 bits
- Hôtes disponibles : 2^6 - 2 = **62 hôtes** par sous-réseau

**Étape 3** : Déterminer les sous-réseaux

|Sous-réseau|Adresse réseau|Première IP utilisable|Dernière IP utilisable|Broadcast|
|---|---|---|---|---|
|1|192.168.1.0/26|192.168.1.1|192.168.1.62|192.168.1.63|
|2|192.168.1.64/26|192.168.1.65|192.168.1.126|192.168.1.127|
|3|192.168.1.128/26|192.168.1.129|192.168.1.190|192.168.1.191|
|4|192.168.1.192/26|192.168.1.193|192.168.1.254|192.168.1.255|

## Masques de sous-réseau courants

|CIDR|Masque décimal|Hôtes disponibles|Usage typique|
|---|---|---|---|
|/30|255.255.255.252|2|Liaison point-à-point|
|/29|255.255.255.248|6|Très petit réseau|
|/27|255.255.255.224|30|Petit département|
|/26|255.255.255.192|62|Département moyen|
|/25|255.255.255.128|126|Grand département|
|/24|255.255.255.0|254|Réseau standard|

## VLSM (Variable Length Subnet Mask)

**Définition** : Technique permettant d'utiliser des masques de sous-réseau de **longueurs différentes** dans un même réseau.

**Avantage** : Optimisation maximale de l'espace d'adressage en adaptant la taille de chaque sous-réseau aux besoins réels.

**Exemple** :

- Service A (100 hôtes) → /25 (126 hôtes)
- Service B (50 hôtes) → /26 (62 hôtes)
- Liaison routeur (2 hôtes) → /30 (2 hôtes)

## Points de vigilance

⚠️ **Attention** :

- Toujours planifier les besoins futurs (croissance du réseau)
- Documenter le plan d'adressage
- Éviter le gaspillage d'adresses IP
- Vérifier la compatibilité des équipements avec CIDR/VLSM

## Outils pratiques

**Calcul manuel** : Convertir en binaire pour comprendre **Calculateurs en ligne** : Pour vérifier rapidement (ipcalc, subnet calculator) **Commandes système** :

- Linux : `ipcalc 192.168.1.0/26`
- Windows : Calculer manuellement ou utiliser des outils tiers