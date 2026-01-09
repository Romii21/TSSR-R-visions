

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

## 🎯 Introduction au routage statique

Le **routage statique** consiste à configurer manuellement les routes dans la table de routage d'un routeur. Contrairement aux protocoles de routage dynamique (RIP, OSPF, EIGRP), les routes statiques ne s'adaptent pas automatiquement aux changements de topologie.

> [!info] Pourquoi utiliser le routage statique ?
> 
> - **Prévisibilité** : Contrôle total sur le chemin des paquets
> - **Sécurité** : Pas d'échange d'informations de routage
> - **Faible consommation** : Aucune overhead CPU ou bande passante
> - **Réseaux petits/simples** : Idéal pour les topologies stables
> - **Routes de secours** : Comme backup pour les routes dynamiques

> [!warning] Limitations
> 
> - Pas de redondance automatique en cas de panne
> - Configuration manuelle fastidieuse sur de grandes topologies
> - Maintenance complexe lors de changements réseau

---

## 📍 Route statique standard

### Principe

Une route statique indique au routeur comment atteindre un réseau distant en spécifiant soit l'adresse du prochain routeur (next-hop), soit l'interface de sortie à utiliser.

### Syntaxe générale

```bash
Router(config)# ip route <réseau_destination> <masque> {<next-hop> | <interface_sortie>} [distance_administrative]
```

### Paramètres détaillés

|Paramètre|Description|Exemple|
|---|---|---|
|`réseau_destination`|Adresse réseau à atteindre|`192.168.10.0`|
|`masque`|Masque sous-réseau en notation décimale|`255.255.255.0`|
|`next-hop`|Adresse IP du routeur suivant|`10.0.0.2`|
|`interface_sortie`|Interface par laquelle sortir|`GigabitEthernet0/0`|
|`distance_administrative`|Priorité de la route (1-255, optionnel)|`10`|

> [!tip] Distance administrative par défaut Les routes statiques ont une distance administrative de **1** par défaut (très prioritaire). Une distance de **0** est réservée aux interfaces directement connectées.

---

## 🔗 Route statique avec next-hop

### Description

Cette méthode spécifie l'**adresse IP du routeur suivant** qui doit recevoir le paquet. Le routeur local effectue une recherche récursive dans sa table de routage pour déterminer l'interface de sortie.

### Syntaxe

```bash
Router(config)# ip route <réseau_destination> <masque> <next-hop_IP>
```

### Exemple pratique

**Topologie :**

- R1 connecté à R2 via 10.0.0.0/30
- R2 connecté au réseau 192.168.20.0/24

**Configuration sur R1 :**

```bash
R1(config)# ip route 192.168.20.0 255.255.255.0 10.0.0.2
```

> [!example] Explication
> 
> - **Réseau cible** : 192.168.20.0/24
> - **Next-hop** : 10.0.0.2 (adresse IP de R2)
> - R1 consulte sa table pour savoir comment atteindre 10.0.0.2
> - R1 découvre que 10.0.0.2 est accessible via GigabitEthernet0/0
> - Les paquets sont envoyés vers cette interface

### Vérification

```bash
R1# show ip route static

S    192.168.20.0/24 [1/0] via 10.0.0.2
```

> [!info] Recherche récursive Avec une route next-hop, le routeur effectue **deux recherches** :
> 
> 1. Trouver la route vers le réseau destination (192.168.20.0)
> 2. Trouver comment atteindre le next-hop (10.0.0.2)
> 
> Cela consomme légèrement plus de ressources CPU mais offre plus de flexibilité.

### Avantages et inconvénients

**✅ Avantages :**

- Fonctionne sur tous types de liens (Ethernet, série, etc.)
- Plus flexible si plusieurs chemins existent vers le next-hop
- Recommandé pour les réseaux multi-accès (Ethernet)

**❌ Inconvénients :**

- Recherche récursive = légère overhead CPU
- Peut créer des routes invalides si le next-hop devient injoignable

---

## 🔌 Route statique avec interface de sortie

### Description

Cette méthode spécifie directement l'**interface de sortie** par laquelle les paquets doivent être envoyés. Pas de recherche récursive nécessaire.

### Syntaxe

```bash
Router(config)# ip route <réseau_destination> <masque> <interface_sortie>
```

### Exemple pratique

**Configuration sur R1 :**

```bash
R1(config)# ip route 192.168.20.0 255.255.255.0 GigabitEthernet0/0
```

> [!example] Explication
> 
> - **Réseau cible** : 192.168.20.0/24
> - **Interface** : GigabitEthernet0/0
> - R1 envoie directement les paquets sur cette interface
> - Pas de recherche supplémentaire dans la table de routage

### Vérification

```bash
R1# show ip route static

S    192.168.20.0/24 is directly connected, GigabitEthernet0/0
```

> [!warning] Attention aux réseaux multi-accès ! Sur les réseaux Ethernet (multi-accès), cette méthode peut créer des problèmes car le routeur ne sait pas quelle adresse MAC utiliser pour encapsuler le paquet. Cette méthode est **idéale pour les liens point-à-point** (Serial, PPP).

### Configuration recommandée : Interface + Next-hop

Pour les réseaux Ethernet, combinez les deux méthodes :

```bash
R1(config)# ip route 192.168.20.0 255.255.255.0 GigabitEthernet0/0 10.0.0.2
```

Cette approche :

- Évite la recherche récursive (meilleure performance)
- Spécifie l'adresse MAC à utiliser (next-hop)
- Offre le meilleur des deux mondes

### Avantages et inconvénients

**✅ Avantages :**

- Pas de recherche récursive (plus rapide)
- Parfait pour les liens point-à-point (Serial, PPP, tunnel)
- Route directement marquée comme "directly connected"

**❌ Inconvénients :**

- Peut créer des problèmes sur les réseaux multi-accès (Ethernet)
- Moins flexible en cas de multiples chemins

---

## 🌍 Route par défaut (default route)

### Principe

La **route par défaut** (gateway of last resort) est utilisée lorsque le routeur ne trouve aucune correspondance spécifique dans sa table de routage. Elle correspond au réseau 0.0.0.0/0 (tous les réseaux).

> [!info] Analogie C'est comme un panneau "Toutes directions" sur une autoroute. Si vous ne trouvez pas d'indication spécifique pour votre destination, vous suivez cette route.

### Syntaxe

```bash
Router(config)# ip route 0.0.0.0 0.0.0.0 {<next-hop> | <interface_sortie>}
```

### Cas d'usage typiques

1. **Routeur de bordure (stub router)** : Routeur avec une seule sortie vers Internet
2. **Simplification** : Éviter de configurer des centaines de routes spécifiques
3. **Réseau d'extrémité** : Site distant avec un seul lien vers le réseau principal

### Exemple : Routeur de bordure

**Topologie :**

- R1 (routeur interne) connecté à R_ISP (routeur FAI)
- R_ISP connecté à Internet

**Configuration sur R1 :**

```bash
# Méthode 1 : Next-hop
R1(config)# ip route 0.0.0.0 0.0.0.0 203.0.113.1

# Méthode 2 : Interface de sortie
R1(config)# ip route 0.0.0.0 0.0.0.0 Serial0/0/0

# Méthode 3 : Les deux (recommandée)
R1(config)# ip route 0.0.0.0 0.0.0.0 Serial0/0/0 203.0.113.1
```

### Vérification

```bash
R1# show ip route

Gateway of last resort is 203.0.113.1 to network 0.0.0.0

S*   0.0.0.0/0 [1/0] via 203.0.113.1
```

> [!tip] L'astérisque (*) Le symbole `S*` indique qu'il s'agit de la route candidate pour devenir la "Gateway of last resort".

### Exemple avancé : Route par défaut flottante

Une **route flottante** est une route de backup avec une distance administrative plus élevée :

```bash
# Route principale (AD = 1, par défaut)
R1(config)# ip route 0.0.0.0 0.0.0.0 Serial0/0/0 203.0.113.1

# Route de backup (AD = 10)
R1(config)# ip route 0.0.0.0 0.0.0.0 Serial0/0/1 198.51.100.1 10
```

> [!example] Fonctionnement
> 
> - En temps normal, seule la première route est active
> - Si Serial0/0/0 tombe, la route principale disparaît
> - La route de backup (AD=10) devient active automatiquement
> - Retour automatique quand Serial0/0/0 revient

---

## ⚖️ Comparaison des méthodes

### Tableau récapitulatif

|Critère|Next-hop|Interface|Interface + Next-hop|
|---|---|---|---|
|**Recherche récursive**|Oui|Non|Non|
|**Performance**|Bonne|Meilleure|Meilleure|
|**Liens point-à-point**|✅|✅|✅|
|**Réseaux multi-accès**|✅|⚠️|✅|
|**Simplicité**|✅|✅|❌|
|**Recommandation**|Ethernet|Serial/PPP|Ethernet (optimal)|

### Quel type choisir ?

```bash
# Pour un réseau Ethernet (multi-accès)
ip route 192.168.10.0 255.255.255.0 10.0.0.2

# Pour un lien série point-à-point
ip route 192.168.10.0 255.255.255.0 Serial0/0/0

# Pour un réseau Ethernet (performance optimale)
ip route 192.168.10.0 255.255.255.0 GigabitEthernet0/0 10.0.0.2

# Pour une route par défaut (stub router)
ip route 0.0.0.0 0.0.0.0 Serial0/0/0 203.0.113.1
```

---

## ⚠️ Pièges courants

### 1. Masque de sous-réseau incorrect

```bash
# ❌ ERREUR : Masque inversé (notation OSPF)
R1(config)# ip route 192.168.10.0 0.0.0.255 10.0.0.2

# ✅ CORRECT : Notation décimale standard
R1(config)# ip route 192.168.10.0 255.255.255.0 10.0.0.2
```

### 2. Next-hop non joignable

```bash
# ❌ PROBLÈME : 10.0.0.2 n'est pas dans un réseau connecté
R1(config)# ip route 192.168.10.0 255.255.255.0 10.0.0.2

# Vérification : La route n'apparaît pas comme active
R1# show ip route
# (Aucune route vers 192.168.10.0)
```

> [!warning] Vérifiez toujours ! Utilisez `show ip route` pour confirmer que la route est bien installée dans la table de routage.

### 3. Interface de sortie sur Ethernet sans next-hop

```bash
# ⚠️ PROBLÉMATIQUE : Sur Ethernet multi-accès
R1(config)# ip route 192.168.10.0 255.255.255.0 GigabitEthernet0/0

# Le routeur ne sait pas quelle adresse MAC utiliser !
# Cela peut causer des broadcasts ARP excessifs
```

### 4. Routes conflictuelles

```bash
# Si deux routes vers le même réseau existent :
R1(config)# ip route 192.168.10.0 255.255.255.0 10.0.0.2
R1(config)# ip route 192.168.10.0 255.255.255.0 10.0.0.3

# Les deux sont actives (load-balancing)
# Sauf si une distance administrative différente est spécifiée
```

### 5. Oubli de la route de retour

```bash
# Sur R1
R1(config)# ip route 192.168.20.0 255.255.255.0 10.0.0.2

# ❌ OUBLI : Pas de route de retour sur R2 !
# Le trafic va dans un sens mais pas dans l'autre
```

> [!tip] Règle d'or **Le routage est bidirectionnel** : configurez toujours les routes dans les deux sens !

---

## ✅ Bonnes pratiques

### 1. Documentation systématique

```bash
# Utilisez des commentaires
R1(config)# ip route 192.168.10.0 255.255.255.0 10.0.0.2
R1(config)# ! Route vers le réseau RH (R2-LAN)

# Ou créez une documentation externe
# Réseau : 192.168.10.0/24 -> RH
# Next-hop : 10.0.0.2 (R2)
# Interface locale : Gi0/0
```

### 2. Vérification après configuration

```bash
# Vérifier la table de routage
R1# show ip route static

# Tester la connectivité
R1# ping 192.168.10.1 source 192.168.1.1

# Tracer le chemin
R1# traceroute 192.168.10.1
```

### 3. Utilisation des routes flottantes

```bash
# Route principale (AD = 1)
R1(config)# ip route 0.0.0.0 0.0.0.0 10.0.0.2

# Route de backup (AD = 5)
R1(config)# ip route 0.0.0.0 0.0.0.0 10.0.0.6 5
```

> [!tip] Distances administratives courantes
> 
> - **0** : Interface directement connectée
> - **1** : Route statique (défaut)
> - **5-10** : Route flottante de backup
> - **90** : EIGRP interne
> - **110** : OSPF
> - **120** : RIP

### 4. Cohérence des méthodes

Choisissez une méthode et restez-y cohérent dans toute votre configuration :

```bash
# ✅ COHÉRENT : Toutes les routes utilisent next-hop
R1(config)# ip route 192.168.10.0 255.255.255.0 10.0.0.2
R1(config)# ip route 192.168.20.0 255.255.255.0 10.0.0.6
R1(config)# ip route 192.168.30.0 255.255.255.0 10.0.0.10
```

### 5. Sauvegarde de la configuration

```bash
# Sauvegarder après chaque modification importante
R1# copy running-config startup-config

# Ou version courte
R1# write memory
R1# wr
```

### 6. Résumé de routes (quand possible)

Au lieu de :

```bash
R1(config)# ip route 192.168.10.0 255.255.255.0 10.0.0.2
R1(config)# ip route 192.168.11.0 255.255.255.0 10.0.0.2
R1(config)# ip route 192.168.12.0 255.255.255.0 10.0.0.2
R1(config)# ip route 192.168.13.0 255.255.255.0 10.0.0.2
```

Utilisez une route résumée :

```bash
R1(config)# ip route 192.168.8.0 255.255.252.0 10.0.0.2
# Couvre 192.168.8.0 à 192.168.11.255
```

> [!tip] Calcul du résumé Utilisez un calculateur de sous-réseaux ou la méthode binaire pour déterminer le masque de résumé approprié.

---

## 🔧 Commandes utiles

### Affichage

```bash
# Table de routage complète
show ip route

# Seulement les routes statiques
show ip route static

# Détails d'une route spécifique
show ip route 192.168.10.0

# Configuration des routes statiques
show running-config | include ip route

# Statistiques d'interface
show ip interface brief
```

### Suppression

```bash
# Supprimer une route statique
R1(config)# no ip route 192.168.10.0 255.255.255.0 10.0.0.2

# Supprimer toutes les routes statiques
R1(config)# no ip route 0.0.0.0 0.0.0.0
```

### Dépannage

```bash
# Test de connectivité
ping <destination>
ping <destination> source <interface_source>

# Traçage du chemin
traceroute <destination>

# Vérification ARP (pour next-hop)
show ip arp

# État des interfaces
show ip interface brief
show interface <interface_name>
```

---

> [!success] Résumé Le routage statique offre un **contrôle précis** et une **sécurité accrue** au prix d'une configuration manuelle. Les trois méthodes principales (next-hop, interface de sortie, et les deux combinés) ont chacune leurs avantages selon le type de lien. La route par défaut simplifie la configuration des routeurs de bordure, tandis que les routes flottantes offrent une redondance basique sans protocole dynamique.