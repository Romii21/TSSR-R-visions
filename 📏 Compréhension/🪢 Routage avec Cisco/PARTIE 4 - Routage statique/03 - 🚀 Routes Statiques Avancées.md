

## 📚 Table des matières

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

## Introduction

Les routes statiques avancées permettent d'optimiser la gestion du routage et d'implémenter des scénarios complexes comme la redondance, l'agrégation ou le filtrage de trafic. Ces techniques sont essentielles pour construire des infrastructures réseau robustes et efficaces.

> [!info] Contexte Cette partie traite de trois techniques avancées de routage statique qui étendent les fonctionnalités de base pour répondre à des besoins spécifiques d'architecture réseau.

---

## Routes statiques flottantes (backup)

### 📖 Concept

Une **route flottante** (floating static route) est une route statique configurée avec une **distance administrative supérieure** à celle de la route principale. Elle reste inactive dans la table de routage tant que la route principale est disponible, et ne s'active qu'en cas de défaillance de cette dernière.

> [!tip] Utilité Les routes flottantes créent un mécanisme de **backup automatique** sans nécessiter de protocole de routage dynamique. C'est idéal pour assurer la redondance dans des topologies simples.

### 🎯 Pourquoi utiliser des routes flottantes ?

- **Redondance** : Assurer la continuité de service en cas de panne du lien principal
- **Économie de bande passante** : Le lien de secours n'est utilisé qu'en cas de nécessité
- **Simplicité** : Pas besoin de protocole de routage dynamique pour gérer le basculement
- **Contrôle des coûts** : Utiliser un lien backup moins cher (ex: 4G, satellite)

### 🔧 Distance administrative

|Type de route|Distance administrative par défaut|
|---|---|
|Interface directement connectée|0|
|Route statique|1|
|EIGRP|90|
|OSPF|110|
|RIP|120|
|Route flottante (personnalisée)|5-254 (selon config)|

> [!warning] Point clé Pour qu'une route soit flottante, sa distance administrative doit être **supérieure** à celle de la route principale. La valeur standard pour une route flottante est souvent **5** ou plus.

### 📝 Syntaxe

```bash
Router(config)# ip route réseau masque {next-hop | interface} [distance-administrative]
```

**Paramètres** :

- `réseau` : Réseau de destination
- `masque` : Masque de sous-réseau
- `next-hop` : Adresse IP du routeur suivant
- `interface` : Interface de sortie
- `distance-administrative` : Valeur entre 1 et 255 (optionnel, 1 par défaut)

### 💡 Exemple pratique

Scénario : Vous avez deux liens vers Internet, un principal (FastEthernet) et un backup (Serial).

```bash
! Route principale (distance administrative = 1, par défaut)
Router(config)# ip route 0.0.0.0 0.0.0.0 203.0.113.1

! Route flottante de backup (distance administrative = 5)
Router(config)# ip route 0.0.0.0 0.0.0.0 198.51.100.1 5
```

**Vérification** :

```bash
Router# show ip route
[...]
S*    0.0.0.0/0 [1/0] via 203.0.113.1
```

La route avec distance 5 n'apparaît pas car la route principale est active.

Si le lien principal tombe :

```bash
Router# show ip route
[...]
S*    0.0.0.0/0 [5/0] via 198.51.100.1
```

La route flottante devient active automatiquement.

> [!example] Cas d'usage réel **Connexion Internet redondante** : Fibre optique principale (1 Gbps) + liaison 4G backup. La 4G ne s'active qu'en cas de panne de la fibre, économisant ainsi les données mobiles et les coûts associés.

### ⚠️ Pièges courants

1. **Oubli de la distance administrative** : Sans spécifier la distance, les deux routes ont la même priorité et le routeur peut faire du load balancing
2. **Distance trop faible** : Si la distance de backup est égale ou inférieure, elle ne restera pas inactive
3. **Problèmes de convergence** : Le basculement n'est pas instantané, il faut attendre que le routeur détecte la panne du lien principal

> [!warning] Détection de panne Le basculement ne se produit que si l'interface ou le next-hop devient **inaccessible**. Si le lien reste "up" mais ne fonctionne pas correctement, la route flottante ne s'activera pas. Utilisez des mécanismes comme IP SLA pour une détection plus fiable.

### ✅ Bonnes pratiques

- Utilisez une distance de **5 à 10** pour les routes flottantes standard
- Documentez clairement quelle route est le backup dans vos configurations
- Testez régulièrement le basculement pour vérifier que le mécanisme fonctionne
- Surveillez les liens avec des outils de monitoring
- Considérez l'utilisation d'IP SLA pour une détection proactive des pannes

---

## Routes statiques résumées

### 📖 Concept

Une **route résumée** (summary route ou supernet) agrège plusieurs réseaux contigus en une seule entrée de routage. Au lieu de créer une route pour chaque sous-réseau, on crée une route unique qui couvre l'ensemble de l'espace d'adressage.

> [!tip] Principe C'est l'inverse du subnetting : on regroupe plusieurs réseaux en un seul bloc d'adresses plus large en utilisant un masque plus court (moins spécifique).

### 🎯 Pourquoi utiliser des routes résumées ?

- **Optimisation de la table de routage** : Moins d'entrées = recherches plus rapides
- **Économie de mémoire** : Particulièrement important sur les routeurs avec ressources limitées
- **Réduction de la bande passante** : Moins d'annonces de routes à propager (dans un contexte de routage dynamique)
- **Simplification de la configuration** : Une ligne au lieu de plusieurs
- **Isolation des changements** : Les modifications dans les sous-réseaux n'affectent pas les routeurs distants

### 📐 Calcul de la route résumée

Pour résumer plusieurs réseaux, il faut :

1. **Identifier le préfixe commun** en binaire
2. **Trouver le premier bit qui diffère** entre les réseaux
3. **Créer un masque** qui inclut tous les réseaux

> [!example] Exemple de calcul
> 
> Vous avez ces quatre réseaux à résumer :
> 
> - 172.16.0.0/24
> - 172.16.1.0/24
> - 172.16.2.0/24
> - 172.16.3.0/24
> 
> **En binaire** :
> 
> ```
> 172.16.0.0 = 10101100.00010000.00000000.00000000
> 172.16.1.0 = 10101100.00010000.00000001.00000000
> 172.16.2.0 = 10101100.00010000.00000010.00000000
> 172.16.3.0 = 10101100.00010000.00000011.00000000
>                                    ^^^^^^ <- Différence ici
> ```
> 
> Les 22 premiers bits sont identiques. La route résumée est donc : **172.16.0.0/22** qui couvre de 172.16.0.0 à 172.16.3.255

### 📝 Syntaxe

```bash
Router(config)# ip route réseau-résumé masque-résumé {next-hop | interface}
```

### 💡 Exemple pratique

**Scénario** : Un routeur R1 doit atteindre les réseaux 192.168.8.0/24, 192.168.9.0/24, 192.168.10.0/24 et 192.168.11.0/24 via le next-hop 10.1.1.2.

**Méthode classique** (4 routes) :

```bash
Router(config)# ip route 192.168.8.0 255.255.255.0 10.1.1.2
Router(config)# ip route 192.168.9.0 255.255.255.0 10.1.1.2
Router(config)# ip route 192.168.10.0 255.255.255.0 10.1.1.2
Router(config)# ip route 192.168.11.0 255.255.255.0 10.1.1.2
```

**Avec route résumée** (1 route) :

```bash
! 192.168.8.0 à 192.168.11.0 = /22
Router(config)# ip route 192.168.8.0 255.255.252.0 10.1.1.2
```

**Vérification** :

```bash
Router# show ip route
[...]
S    192.168.8.0/22 [1/0] via 10.1.1.2
```

Une seule ligne remplace les quatre routes individuelles.

### 📊 Tableau des résumés courants

|Réseaux à résumer|Route résumée|Notation CIDR|
|---|---|---|
|192.168.0.0/24 à 192.168.1.0/24|192.168.0.0 255.255.254.0|/23|
|192.168.0.0/24 à 192.168.3.0/24|192.168.0.0 255.255.252.0|/22|
|192.168.0.0/24 à 192.168.7.0/24|192.168.0.0 255.255.248.0|/21|
|10.0.0.0/24 à 10.0.255.0/24|10.0.0.0 255.255.0.0|/16|

### ⚠️ Pièges courants

1. **Résumé trop large** : Inclure des réseaux qui ne vous appartiennent pas peut causer du blackholing
2. **Réseaux non contigus** : On ne peut pas résumer 192.168.0.0/24 et 192.168.5.0/24 sans inclure les réseaux intermédiaires
3. **Oubli de vérification** : Toujours vérifier que tous les sous-réseaux sont bien couverts
4. **Conflit avec des routes plus spécifiques** : Une route plus spécifique sera toujours préférée

> [!warning] Principe de correspondance la plus longue Le routeur choisit toujours la route avec le **préfixe le plus long** (masque le plus spécifique). Si vous avez à la fois 192.168.8.0/22 et 192.168.9.0/24, le trafic vers 192.168.9.x utilisera la /24, pas la /22.

### ✅ Bonnes pratiques

- **Planifiez votre plan d'adressage** : Utilisez des blocs contigus pour faciliter la résumé
- **Documentez vos résumés** : Indiquez quels réseaux sont couverts
- **Vérifiez la couverture** : Assurez-vous qu'aucun réseau nécessaire n'est oublié
- **Évitez le sur-résumé** : Ne résumez pas si cela inclut des réseaux non désirés
- **Utilisez des outils** : Des calculateurs de sous-réseaux peuvent vous aider

### 🔧 Astuce pratique

Pour vérifier rapidement si une adresse est couverte par votre résumé :

```bash
Router# show ip route 192.168.9.15
Routing entry for 192.168.8.0/22
  Known via "static", distance 1, metric 0
  Routing Descriptor Blocks:
  * 10.1.1.2
```

---

## Routes statiques vers Null0 (black hole)

### 📖 Concept

Une **route vers Null0** (aussi appelée black hole route) dirige le trafic vers l'interface virtuelle **Null0**, qui rejette silencieusement tous les paquets sans générer de messages ICMP. C'est l'équivalent réseau d'un trou noir : tout ce qui y entre disparaît.

> [!info] Interface Null0 Null0 est une interface virtuelle présente sur tous les routeurs Cisco. Elle est toujours "up/up" mais ne tranmet jamais de paquets. C'est un mécanisme plus efficace qu'une ACL pour filtrer du trafic.

### 🎯 Pourquoi utiliser des routes vers Null0 ?

- **Protection contre les boucles de routage** : Prévenir le routing loop lors de l'utilisation de routes résumées
- **Filtrage de trafic indésirable** : Bloquer des réseaux malveillants ou inutilisés
- **Sécurité** : Créer des "sinkholes" pour isoler du trafic suspect
- **Optimisation** : Réduire le trafic inutile sans utiliser d'ACL
- **Prévention de l'utilisation de la route par défaut** : Éviter que des paquets non désirés ne sortent par la route 0.0.0.0/0

### 📝 Syntaxe

```bash
Router(config)# ip route réseau masque Null0
```

### 💡 Exemple pratique : Prévention des boucles avec résumé

**Scénario** : R1 annonce le résumé 172.16.0.0/22 à R2, mais n'a que les réseaux 172.16.0.0/24 et 172.16.1.0/24 configurés localement.

**Problème** : Si un paquet arrive pour 172.16.2.50 (réseau inclus dans le résumé mais non existant), R1 utiliserait sa route par défaut et renverrait le paquet vers R2, créant une boucle.

**Solution** :

```bash
! Route résumée vers R2
R1(config)# ip route 172.16.0.0 255.255.252.0 Serial0/0/0

! Black hole pour empêcher les boucles
R1(config)# ip route 172.16.0.0 255.255.252.0 Null0
```

> [!warning] Attention à la distance administrative Les deux routes ont la même distance administrative (1). Le routeur garde les deux mais utilisera **d'abord** les routes plus spécifiques (172.16.0.0/24, 172.16.1.0/24) et enverra vers Null0 uniquement ce qui ne matche pas ces routes plus spécifiques.

**Comportement** :

- Trafic vers 172.16.0.x ou 172.16.1.x → Acheminé normalement
- Trafic vers 172.16.2.x ou 172.16.3.x → Envoyé vers Null0 (rejeté)

### 💡 Exemple pratique : Bloquer un réseau malveillant

**Scénario** : Vous détectez des attaques provenant du réseau 203.0.113.0/24.

```bash
! Bloquer tout le trafic vers ce réseau
Router(config)# ip route 203.0.113.0 255.255.255.0 Null0
```

Tous les paquets destinés à 203.0.113.0/24 seront rejetés silencieusement, sans réponse ICMP.

### 💡 Exemple pratique : Prévenir l'utilisation de la route par défaut

**Scénario** : Vous voulez vous assurer que le trafic vers certains réseaux privés ne sorte jamais par Internet.

```bash
! Bloquer les RFC 1918 non utilisés
Router(config)# ip route 10.0.0.0 255.0.0.0 Null0
Router(config)# ip route 192.168.0.0 255.255.0.0 Null0

! Route par défaut vers Internet
Router(config)# ip route 0.0.0.0 0.0.0.0 203.0.113.1
```

Si un paquet destiné à 10.x.x.x ou 192.168.x.x arrive mais ne correspond à aucune route plus spécifique, il sera rejeté au lieu d'être envoyé vers Internet.

### 🔍 Vérification

```bash
Router# show ip route
[...]
S    172.16.0.0/22 is directly connected, Null0
S    203.0.113.0/24 is directly connected, Null0

Router# show interface Null0
Null0 is up, line protocol is up
  Hardware is Unknown
  MTU 1500 bytes, BW 10000000 Kbit/sec
  [...]
  5 minute input rate 0 bits/sec, 0 packets/sec
  5 minute output rate 0 bits/sec, 0 packets/sec
     0 packets input, 0 bytes, 0 no buffer
     0 packets output, 0 bytes, 0 underruns
```

### 📊 Comparaison : ACL vs Route Null0

|Critère|ACL|Route Null0|
|---|---|---|
|Performance|Examine chaque paquet|Décision au niveau routage (plus rapide)|
|Flexibilité|Très flexible (ports, protocoles)|Basé uniquement sur l'IP de destination|
|Complexité|Plus complexe à configurer|Simple, une ligne suffit|
|Visibilité|Compteurs ACL disponibles|Visible dans la table de routage|
|Messages ICMP|Peut générer des ICMP unreachable|Aucun message généré|

> [!tip] Quand utiliser l'un ou l'autre ?
> 
> - **Route Null0** : Pour bloquer rapidement des réseaux entiers, optimiser le routage
> - **ACL** : Pour un filtrage granulaire (ports, protocoles, bidirectionnel)

### ⚠️ Pièges courants

1. **Pas de message d'erreur** : Les paquets disparaissent silencieusement, ce qui peut compliquer le dépannage
2. **Ordre des routes** : Une route plus spécifique sera toujours préférée à une route Null0 plus générale
3. **Oubli de documentation** : Documentez pourquoi vous utilisez Null0, sinon d'autres administrateurs seront confus
4. **Suppression accidentelle** : Supprimer une route Null0 utilisée pour la prévention de boucles peut causer des problèmes graves

> [!warning] Dépannage difficile Comme Null0 ne génère pas de messages ICMP, les utilisateurs ne recevront pas de "destination unreachable". Les connexions sembleront simplement timeout. Utilisez `debug ip packet` avec précaution pour diagnostiquer.

### ✅ Bonnes pratiques

- **Utilisez systématiquement Null0 avec les routes résumées** pour prévenir les boucles
- **Documentez vos routes Null0** : Ajoutez des commentaires dans votre configuration
- **Surveillez les compteurs** : Vérifiez régulièrement combien de trafic est rejeté
- **Testez avant de déployer** : Assurez-vous de ne pas bloquer du trafic légitime
- **Combinez avec des ACL** si nécessaire pour un filtrage plus granulaire

### 🔧 Astuce avancée

Vous pouvez utiliser des route-maps avec Null0 pour un contrôle plus fin :

```bash
! Créer une route-map
Router(config)# route-map BLOCK-SUBNET permit 10
Router(config-route-map)# match ip address prefix-list MALICIOUS

! L'appliquer avec policy-routing
Router(config)# interface GigabitEthernet0/0
Router(config-if)# ip policy route-map BLOCK-SUBNET

! Définir la policy pour envoyer vers Null0
Router(config)# route-map BLOCK-SUBNET permit 10
Router(config-route-map)# set interface Null0
```

---

## 📝 Synthèse

|Technique|Objectif principal|Cas d'usage typique|
|---|---|---|
|**Routes flottantes**|Redondance et backup|Liens Internet redondants, failover automatique|
|**Routes résumées**|Optimisation de la table de routage|Grandes infrastructures, agrégation d'adresses|
|**Routes vers Null0**|Filtrage et prévention de boucles|Sécurité, black holes, protection des résumés|

> [!tip] Conseil final Ces trois techniques sont complémentaires et peuvent être combinées dans une même infrastructure pour créer un réseau robuste, optimisé et sécurisé. Maîtrisez-les pour devenir un expert en routage Cisco !