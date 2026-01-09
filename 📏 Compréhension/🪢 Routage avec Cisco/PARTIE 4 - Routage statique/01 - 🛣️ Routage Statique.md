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

## 🗺️ Table de routage

### Qu'est-ce que la table de routage ?

La table de routage est une base de données stockée dans la mémoire RAM du routeur qui contient toutes les informations nécessaires pour acheminer les paquets vers leur destination. C'est le "GPS" du routeur.

> [!info] Fonction principale Quand un paquet arrive sur le routeur, celui-ci consulte sa table de routage pour déterminer par quelle interface et vers quel prochain routeur (next-hop) envoyer ce paquet.

### Structure d'une entrée de routage

Chaque ligne de la table de routage contient plusieurs informations essentielles :

|Élément|Description|Exemple|
|---|---|---|
|**Code de route**|Type de route (C, S, R, O, etc.)|`S` pour statique|
|**Réseau de destination**|Réseau à atteindre avec son masque|`192.168.10.0/24`|
|**Distance administrative**|Fiabilité de la source|`1` pour route statique|
|**Métrique**|Coût de la route|`0` pour route statique|
|**Next-hop**|IP du prochain routeur|`10.0.0.2`|
|**Interface de sortie**|Interface physique à utiliser|`GigabitEthernet0/0`|

### Afficher la table de routage

```ba
Router# show ip route

# Affiche toute la table de routage avec tous les détails
```

> [!example] Exemple de sortie
> 
> ```
> Codes: C - connected, S - static, R - RIP, O - OSPF
>        * - candidate default
> 
> Gateway of last resort is not set
> 
> C    192.168.1.0/24 is directly connected, GigabitEthernet0/0
> S    192.168.10.0/24 [1/0] via 10.0.0.2
> S    0.0.0.0/0 [1/0] via 192.168.1.254
> ```

### Codes de routes importants

```bash
C - directly connected    # Interface active sur le routeur
L - local                 # Adresse IP de l'interface elle-même
S - static                # Route configurée manuellement
S* - static default       # Route par défaut statique
D - EIGRP                 # Protocole de routage dynamique (hors périmètre)
O - OSPF                  # Protocole de routage dynamique (hors périmètre)
R - RIP                   # Protocole de routage dynamique (hors périmètre)
```

> [!tip] Commandes utiles
> 
> ```bash
> show ip route static           # Affiche uniquement les routes statiques
> show ip route connected        # Affiche uniquement les routes connectées
> show ip route 192.168.10.5     # Affiche la route utilisée pour cette IP
> show ip route summary          # Résumé de la table de routage
> ```

### Processus de décision du routeur

1. **Réception du paquet** : Le routeur reçoit un paquet avec une IP de destination
2. **Consultation** : Il recherche l'entrée la plus spécifique dans sa table de routage
3. **Longest prefix match** : Il sélectionne la route avec le masque le plus long qui correspond
4. **Forwarding** : Il transmet le paquet via l'interface de sortie vers le next-hop
5. **Si aucune route** : Le paquet est abandonné (dropped)

> [!warning] Attention au longest prefix match Si plusieurs routes correspondent à une destination, le routeur choisit TOUJOURS celle avec le masque le plus long (la plus spécifique), peu importe la distance administrative.
> 
> Exemple : Pour atteindre `192.168.10.50`
> 
> - Route A : `192.168.0.0/16` → 65 536 adresses
> - Route B : `192.168.10.0/24` → 256 adresses ✅ CHOISIE
> 
> La route B est choisie car `/24` est plus spécifique que `/16`

---

## 📊 Métriques et distance administrative

### Distance administrative (AD)

La distance administrative est un **indice de confiance** qu'un routeur accorde à une source de routage. Plus la valeur est faible, plus la source est fiable.

> [!info] Pourquoi c'est important ? Un routeur peut apprendre une même route de plusieurs sources (statique, OSPF, RIP, etc.). La distance administrative permet de départager quelle source sera prioritaire.

#### Tableau des distances administratives par défaut

|Source de route|Distance Administrative|Fiabilité|
|---|---|---|
|**Interface directement connectée**|0|Maximum|
|**Route statique**|1|Très élevée|
|**Route statique flottante**|Variable (> 1)|Configurable|
|EIGRP (résumé)|5|Élevée|
|EIGRP (interne)|90|Élevée|
|OSPF|110|Moyenne|
|RIP|120|Faible|
|**Inconnue**|255|Nulle (route ignorée)|

> [!tip] Routes statiques flottantes En modifiant la distance administrative d'une route statique (en la mettant > 1), on peut créer une **route de secours** qui ne sera utilisée que si la route principale tombe. C'est très utile pour la redondance !

#### Exemple de distance administrative

```bash
Router# show ip route

S    192.168.10.0/24 [1/0] via 10.0.0.2
#                    ↑  ↑
#                    │  └─ Métrique (0 pour statique)
#                    └─ Distance administrative (1 pour statique)
```

### Métriques

La métrique représente le **coût** d'une route. Contrairement à la distance administrative qui compare des sources différentes, la métrique compare plusieurs chemins issus de la **même source de routage**.

> [!info] Différence clé
> 
> - **Distance administrative** : Compare des SOURCES différentes (statique vs OSPF)
> - **Métrique** : Compare des CHEMINS différents d'une même source

#### Métriques pour les routes statiques

Pour les routes statiques, la métrique est **toujours 0** car il n'y a pas de calcul de coût automatique. C'est l'administrateur qui décide manuellement quelle route utiliser.

```bash
Router(config)# ip route 192.168.10.0 255.255.255.0 10.0.0.2
# Métrique = 0 automatiquement
```

> [!example] Comparaison AD vs Métrique **Scénario** : Deux routes vers `192.168.10.0/24`
> 
> ```
> Route 1 (Statique) : AD=1, Métrique=0, via 10.0.0.2
> Route 2 (OSPF)     : AD=110, Métrique=50, via 10.0.0.3
> ```
> 
> **Résultat** : La route statique (AD=1) est choisie même si OSPF a une métrique plus élevée. La distance administrative est évaluée EN PREMIER.

#### Types de métriques selon les protocoles

Bien que nous nous concentrions sur le routage statique, voici les métriques utilisées par différents protocoles (pour information) :

|Protocole|Métrique basée sur|Valeur par défaut|
|---|---|---|
|**Statique**|Aucune (0)|0|
|RIP|Nombre de sauts (hops)|Compte des routeurs|
|OSPF|Bande passante|100 000 000 / bande passante|
|EIGRP|Bande passante + délai|Calcul composite complexe|

> [!warning] Load balancing Si plusieurs routes statiques ont la **même destination**, la **même distance administrative** ET la **même métrique**, le routeur peut faire du **load balancing** (répartition de charge) entre ces routes. Par défaut, Cisco utilise jusqu'à 4 chemins équivalents.

---

## 🎯 Next-hop vs interface de sortie

Lors de la configuration d'une route statique, vous avez deux options principales pour indiquer où envoyer le trafic :

1. **Next-hop** : Spécifier l'adresse IP du prochain routeur
2. **Interface de sortie** : Spécifier l'interface physique du routeur

### Syntaxe Next-hop

```bash
Router(config)# ip route [réseau-destination] [masque] [adresse-ip-next-hop]

# Exemple concret
Router(config)# ip route 192.168.10.0 255.255.255.0 10.0.0.2
```

> [!info] Fonctionnement Le routeur utilise l'adresse IP `10.0.0.2` comme prochaine étape. Il doit d'abord effectuer une **recherche récursive** pour trouver quelle interface utiliser pour atteindre cette IP.

#### Avantages du next-hop

- ✅ **Flexibilité** : Fonctionne sur tous les types de réseaux
- ✅ **Multi-access** : Idéal pour Ethernet où plusieurs routeurs peuvent exister
- ✅ **Clarté** : On sait exactement vers quel routeur le trafic est envoyé

#### Inconvénients du next-hop

- ❌ **Recherche supplémentaire** : Le routeur doit faire une recherche récursive (légère surcharge)
- ❌ **Dépendance** : Le next-hop doit être joignable, sinon la route est invalide

### Syntaxe Interface de sortie

```bash
Router(config)# ip route [réseau-destination] [masque] [interface-sortie]

# Exemple concret
Router(config)# ip route 192.168.10.0 255.255.255.0 GigabitEthernet0/1
```

> [!info] Fonctionnement Le routeur envoie directement les paquets via l'interface spécifiée, sans recherche supplémentaire. Le paquet est considéré comme étant sur un réseau directement connecté.

#### Avantages de l'interface de sortie

- ✅ **Performance** : Pas de recherche récursive
- ✅ **Liens point-à-point** : Idéal pour les liaisons série où il n'y a qu'un seul voisin

#### Inconvénients de l'interface de sortie

- ❌ **Réseaux multi-access** : Sur Ethernet, le routeur ne sait pas quelle adresse MAC utiliser pour l'encapsulation
- ❌ **Broadcasts ARP** : Peut générer des requêtes ARP inutiles sur des réseaux Ethernet

> [!warning] Problème sur les réseaux Ethernet Sur un réseau Ethernet avec plusieurs hôtes, spécifier uniquement l'interface de sortie peut causer des problèmes. Le routeur ne connaît pas l'adresse MAC du next-hop spécifique et peut tenter des ARP pour chaque paquet.

### Syntaxe combinée (Recommandée pour Ethernet)

```bash
Router(config)# ip route [réseau-destination] [masque] [interface-sortie] [adresse-ip-next-hop]

# Exemple concret - MEILLEURE PRATIQUE
Router(config)# ip route 192.168.10.0 255.255.255.0 GigabitEthernet0/1 10.0.0.2
```

> [!tip] Meilleure pratique Sur les **réseaux Ethernet**, utilisez TOUJOURS la syntaxe combinée (interface + next-hop) pour éviter les problèmes d'encapsulation et améliorer les performances.
> 
> Sur les **liaisons point-à-point** (Serial, PPP), vous pouvez utiliser uniquement l'interface de sortie.

### Tableau comparatif

|Critère|Next-hop seul|Interface seule|Combiné|
|---|---|---|---|
|**Performance**|Moyenne (recherche récursive)|Rapide|Rapide|
|**Ethernet multi-access**|✅ Fonctionne bien|❌ Problèmes ARP|✅ Optimal|
|**Liaison point-à-point**|✅ Fonctionne|✅ Optimal|✅ Fonctionne|
|**Clarté**|✅ Claire|Moyenne|✅ Très claire|
|**Recommandation**|Acceptable|Éviter sur Ethernet|**Meilleure pratique**|

> [!example] Exemple pratique **Topologie** :
> 
> ```
> R1 [Gi0/0: 10.0.0.1/30] ------- [Gi0/0: 10.0.0.2/30] R2 ------- [Gi0/1: 192.168.10.1/24] LAN
> ```
> 
> **Sur R1, pour atteindre le LAN 192.168.10.0/24** :
> 
> ```bash
> # Option 1 : Next-hop uniquement (acceptable)
> R1(config)# ip route 192.168.10.0 255.255.255.0 10.0.0.2
> 
> # Option 2 : Interface uniquement (déconseillé sur Ethernet)
> R1(config)# ip route 192.168.10.0 255.255.255.0 GigabitEthernet0/0
> 
> # Option 3 : Combiné (RECOMMANDÉ)
> R1(config)# ip route 192.168.10.0 255.255.255.0 GigabitEthernet0/0 10.0.0.2
> ```

---

## 🔌 Routes directement connectées

### Qu'est-ce qu'une route directement connectée ?

Une route directement connectée est automatiquement ajoutée à la table de routage dès qu'une interface est configurée avec une adresse IP et activée (état `up/up`). Aucune configuration manuelle n'est nécessaire.

> [!info] Automatique et essentiel Ces routes sont la base du routage. Sans elles, le routeur ne pourrait même pas communiquer avec ses réseaux locaux. Elles ont une distance administrative de **0** (la plus fiable).

### Création automatique

```bash
Router(config)# interface GigabitEthernet0/0
Router(config-if)# ip address 192.168.1.1 255.255.255.0
Router(config-if)# no shutdown
Router(config-if)# exit

# La route est automatiquement ajoutée
```

### Vérification

```bash
Router# show ip route connected

C    192.168.1.0/24 is directly connected, GigabitEthernet0/0
L    192.168.1.1/32 is directly connected, GigabitEthernet0/0
```

> [!info] Deux types d'entrées
> 
> - **C (Connected)** : Représente tout le réseau auquel l'interface appartient
> - **L (Local)** : Représente l'adresse IP spécifique de l'interface (/32)

### Caractéristiques des routes connectées

|Caractéristique|Valeur|
|---|---|
|**Code**|C (connected)|
|**Distance administrative**|0|
|**Métrique**|0|
|**Next-hop**|Aucun (réseau local)|
|**Interface**|Interface physique configurée|

### Routes locales (/32)

À partir d'IOS 15.0, Cisco ajoute automatiquement une route locale (`L`) pour l'adresse IP de chaque interface :

```bash
L    192.168.1.1/32 is directly connected, GigabitEthernet0/0
```

> [!tip] Utilité des routes locales Ces routes /32 permettent au routeur de traiter efficacement le trafic destiné à ses propres interfaces (management, protocoles de routage, etc.) sans avoir à vérifier si le paquet lui est destiné.

### Conditions pour qu'une route connectée existe

Une route connectée n'apparaît dans la table de routage que si **TOUTES** ces conditions sont remplies :

1. ✅ Une adresse IP est configurée sur l'interface
2. ✅ L'interface a un masque de sous-réseau valide
3. ✅ La commande `no shutdown` a été exécutée
4. ✅ L'interface est physiquement opérationnelle (câble connecté, état `up/up`)

> [!warning] Interface down = Route supprimée Si une interface passe à l'état `down`, sa route connectée est **immédiatement retirée** de la table de routage. Cela peut affecter d'autres routes statiques qui dépendent de cette interface.

### Impact sur le routage statique

Les routes directement connectées sont essentielles pour les routes statiques :

```bash
# Configuration d'une route statique
Router(config)# ip route 10.10.10.0 255.255.255.0 192.168.1.254

# Cette route ne fonctionnera QUE si :
# - Le next-hop 192.168.1.254 est joignable via une route connectée
# - OU l'interface de sortie spécifiée existe et est up
```

> [!example] Exemple de dépendance **Topologie** :
> 
> ```
> R1 [Gi0/0: 192.168.1.1/24] ------- [192.168.1.254] Internet
> ```
> 
> **Configuration** :
> 
> ```bash
> R1(config)# interface GigabitEthernet0/0
> R1(config-if)# ip address 192.168.1.1 255.255.255.0
> R1(config-if)# no shutdown
> 
> R1(config)# ip route 0.0.0.0 0.0.0.0 192.168.1.254
> ```
> 
> **Table de routage** :
> 
> ```
> C    192.168.1.0/24 is directly connected, GigabitEthernet0/0
> S*   0.0.0.0/0 [1/0] via 192.168.1.254
> ```
> 
> ⚠️ Si `Gi0/0` tombe, la route connectée disparaît ET la route statique devient invalide !

### Commandes de vérification

```bash
# Voir toutes les routes connectées
Router# show ip route connected

# Voir l'état des interfaces
Router# show ip interface brief

# Voir les détails d'une interface spécifique
Router# show interface GigabitEthernet0/0

# Voir le résumé de la table avec les connectées
Router# show ip route summary
```

> [!tip] Dépannage rapide Si une route attendue n'apparaît pas :
> 
> 1. Vérifiez l'état de l'interface : `show ip interface brief`
> 2. Vérifiez la configuration IP : `show running-config interface [nom]`
> 3. Vérifiez les logs : `show logging`
> 4. Vérifiez le câblage physique

---

## 🎯 Récapitulatif des concepts

### Points essentiels à retenir

> [!tip] Table de routage
> 
> - Base de données contenant toutes les routes connues
> - Consultée pour chaque paquet à router
> - Utilise le principe du **longest prefix match** (route la plus spécifique)
> - Commande principale : `show ip route`

> [!tip] Distance administrative
> 
> - Mesure de **confiance** dans une source de routage (0-255)
> - Plus elle est basse, plus la source est fiable
> - Comparée EN PREMIER avant la métrique
> - Routes connectées = 0, statiques = 1

> [!tip] Métrique
> 
> - Mesure du **coût** d'une route
> - Utilisée pour comparer des routes de la MÊME source
> - Toujours 0 pour les routes statiques
> - Permet le load balancing si égale

> [!tip] Next-hop vs Interface
> 
> - **Next-hop** : Flexible, fonctionne partout, léger overhead
> - **Interface** : Rapide, idéal pour point-à-point
> - **Combiné** : Meilleure pratique sur Ethernet
> - Toujours privilégier la syntaxe combinée sur réseaux multi-access

> [!tip] Routes connectées
> 
> - Créées automatiquement (AD=0)
> - Requièrent interface `up/up`
> - Fondamentales pour tout routage
> - Disparaissent si l'interface tombe

---

### Ordre de priorité dans la sélection de route

1. **Longest prefix match** (masque le plus long)
2. **Distance administrative** (source la plus fiable)
3. **Métrique** (chemin le moins coûteux)
4. **Load balancing** (si tout est égal)

Cette hiérarchie est cruciale pour comprendre comment un routeur choisit son chemin !