

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

## 🎯 Introduction au dépannage

Le dépannage réseau suit une méthodologie structurée qui s'appuie sur des commandes de diagnostic précises. Ces commandes permettent de vérifier la connectivité, identifier les chemins réseau, analyser l'état des interfaces et examiner la configuration du routage.

> [!info] Approche systématique Les commandes de diagnostic doivent être utilisées de manière progressive : d'abord vérifier la connectivité de base (ping), puis analyser le chemin (traceroute), ensuite l'état des interfaces (show ip interface brief), et enfin la table de routage (show ip route).

---

## 🔌 Ping (standard et étendu)

### Définition et utilité

La commande **ping** (Packet Internet Groper) utilise le protocole ICMP pour tester la connectivité réseau entre deux équipements. Elle envoie des paquets Echo Request et attend des réponses Echo Reply.

**Pourquoi l'utiliser ?**

- Vérifier la connectivité de base vers une destination
- Tester la résolution DNS
- Mesurer la latence réseau
- Identifier les pertes de paquets

**Quand l'utiliser ?**

- Première étape de dépannage de connectivité
- Vérification après une modification de configuration
- Test de disponibilité d'un équipement distant

---

### Ping standard (mode utilisateur)

Le ping standard est exécuté depuis le mode utilisateur ou privilégié et utilise l'adresse IP de l'interface de sortie comme source.

```bash
# Syntaxe de base
Router> ping [adresse_ip | nom_hôte]

# Exemples
Router> ping 192.168.1.1
Router> ping 8.8.8.8
Router> ping google.com
```

> [!example] Exemple d'output
> 
> ```
> Router> ping 192.168.1.1
> Type escape sequence to abort.
> Sending 5, 100-byte ICMP Echos to 192.168.1.1, timeout is 2 seconds:
> !!!!!
> Success rate is 100 percent (5/5), round-trip min/avg/max = 1/2/4 ms
> ```

**Interprétation des symboles :**

|Symbole|Signification|
|---|---|
|!|Echo reply reçu (succès)|
|.|Timeout (pas de réponse)|
|U|Destination unreachable|
|Q|Source quench (congestion)|
|M|Could not fragment|
|?|Type de paquet inconnu|

---

### Ping étendu (extended ping)

Le ping étendu offre des options avancées permettant de personnaliser les paramètres des paquets ICMP, notamment le choix de l'adresse source.

```bash
# Activation du ping étendu
Router# ping
Protocol [ip]: 
Target IP address: 10.0.0.1
Repeat count [5]: 10
Datagram size [100]: 1500
Timeout in seconds [2]: 5
Extended commands [n]: yes
Source address or interface: 192.168.1.1
Type of service [0]: 
Set DF bit in IP header? [no]: 
Validate reply data? [no]: 
Data pattern [0xABCD]: 
Loose, Strict, Record, Timestamp, Verbose[none]: 
Sweep range of sizes [n]: 
```

**Paramètres clés du ping étendu :**

- **Source address** : Spécifie l'adresse IP source du ping (crucial pour tester la connectivité depuis une interface spécifique)
- **Repeat count** : Nombre de paquets à envoyer (défaut : 5)
- **Datagram size** : Taille du paquet ICMP (défaut : 100 octets)
- **Timeout** : Délai d'attente avant timeout (défaut : 2 secondes)
- **DF bit** : Active le bit "Don't Fragment" pour tester la MTU
- **Data pattern** : Motif de données personnalisé pour le diagnostic

> [!tip] Astuce pratique Utilisez le ping étendu avec l'option "Source address" pour vérifier que les routes de retour sont correctement configurées. Si un ping standard réussit mais qu'un ping étendu depuis une interface spécifique échoue, le problème est probablement lié au routage de retour.

> [!warning] Attention Le ping étendu n'est disponible qu'en mode privilégié (enable). De plus, certains équipements bloquent les paquets ICMP par politique de sécurité, ce qui peut donner de faux négatifs.

**Exemple de test de MTU avec DF bit :**

```bash
Router# ping
Protocol [ip]: 
Target IP address: 10.0.0.1
Repeat count [5]: 
Datagram size [100]: 1500
Timeout in seconds [2]: 
Extended commands [n]: yes
Source address or interface: 
Set DF bit in IP header? [no]: yes
```

---

### Cas d'usage avancés

**Test de connectivité bidirectionnelle :**

```bash
# Depuis le routeur A vers B
RouterA# ping 10.0.0.2 source 10.0.0.1

# Depuis le routeur B vers A
RouterB# ping 10.0.0.1 source 10.0.0.2
```

**Test de MTU Path Discovery :**

```bash
# Test avec différentes tailles de paquets
Router# ping 10.0.0.1 size 1500 df-bit
Router# ping 10.0.0.1 size 1400 df-bit
Router# ping 10.0.0.1 size 1300 df-bit
```

> [!info] Bonnes pratiques
> 
> - Utilisez toujours le ping étendu pour les diagnostics avancés
> - Testez avec au moins 100 paquets pour identifier les pertes intermittentes
> - Vérifiez la connectivité dans les deux sens (bidirectionnelle)
> - Testez depuis différentes interfaces sources pour valider le routage

---

## 🗺️ Traceroute

### Définition et utilité

La commande **traceroute** (ou **tracert** sous Windows) identifie le chemin complet que prennent les paquets pour atteindre une destination en listant tous les routeurs intermédiaires (sauts/hops).

**Pourquoi l'utiliser ?**

- Identifier où se situe un problème de connectivité dans le réseau
- Visualiser le chemin pris par les paquets
- Détecter les boucles de routage
- Mesurer la latence à chaque saut

**Quand l'utiliser ?**

- Lorsque le ping échoue, pour localiser précisément où la communication est interrompue
- Pour vérifier que le trafic suit le chemin attendu
- Pour diagnostiquer des problèmes de performance (latence élevée)

---

### Fonctionnement technique

Traceroute utilise des paquets UDP (sous Cisco) ou ICMP avec des valeurs TTL (Time To Live) incrémentales :

1. Premier paquet avec TTL=1 → le premier routeur répond "Time Exceeded"
2. Deuxième paquet avec TTL=2 → le deuxième routeur répond "Time Exceeded"
3. Et ainsi de suite jusqu'à atteindre la destination

> [!info] Protocoles utilisés
> 
> - **Cisco IOS** : Utilise des paquets UDP vers des ports aléatoires (33434-33534)
> - **Windows** : Utilise des paquets ICMP Echo Request
> - **Linux** : Utilise généralement UDP comme Cisco

---

### Syntaxe et utilisation

```bash
# Syntaxe de base
Router> traceroute [adresse_ip | nom_hôte]

# Exemples
Router> traceroute 8.8.8.8
Router> traceroute www.google.com

# Traceroute depuis une interface spécifique (mode privilégié)
Router# traceroute 10.0.0.1 source GigabitEthernet0/0
```

> [!example] Exemple d'output
> 
> ```
> Router> traceroute 8.8.8.8
> Type escape sequence to abort.
> Tracing the route to 8.8.8.8
> 
>   1 192.168.1.1 4 msec 4 msec 4 msec
>   2 10.0.0.1 12 msec 8 msec 12 msec
>   3 172.16.0.1 20 msec 16 msec 20 msec
>   4 8.8.8.8 28 msec 24 msec 28 msec
> ```

---

### Interprétation des résultats

**Symboles et leur signification :**

|Symbole|Signification|
|---|---|
|msec|Temps de réponse en millisecondes (succès)|
|*|Timeout (pas de réponse de ce saut)|
|!H|Host unreachable|
|!N|Network unreachable|
|!P|Protocol unreachable|
|!A|Administratively prohibited (ACL bloque)|

**Analyse des patterns :**

```bash
# Pattern 1 : Succès complet
1 192.168.1.1 4 msec 4 msec 4 msec
2 10.0.0.1 12 msec 8 msec 12 msec
3 8.8.8.8 28 msec 24 msec 28 msec
# → Chemin complet et fonctionnel

# Pattern 2 : Timeout intermédiaire
1 192.168.1.1 4 msec 4 msec 4 msec
2 * * *
3 172.16.0.1 20 msec 16 msec 20 msec
# → Le routeur du saut 2 ne répond pas aux ICMP Time Exceeded
# (courant, peut être normal si configuré ainsi)

# Pattern 3 : Échec à un point précis
1 192.168.1.1 4 msec 4 msec 4 msec
2 10.0.0.1 12 msec 8 msec 12 msec
3 * * *
4 * * *
# → Problème de routage ou de connectivité après le saut 2

# Pattern 4 : Boucle de routage
1 192.168.1.1 4 msec 4 msec 4 msec
2 10.0.0.1 12 msec 8 msec 12 msec
3 192.168.1.1 20 msec 16 msec 20 msec
4 10.0.0.1 28 msec 24 msec 28 msec
# → Les mêmes adresses se répètent, indiquant une boucle
```

---

### Options avancées

```bash
# Traceroute avec options étendues (mode privilégié)
Router# traceroute
Protocol [ip]: 
Target IP address: 8.8.8.8
Source address: 192.168.1.1
Numeric display [n]: 
Timeout in seconds [3]: 5
Probe count [3]: 5
Minimum Time to Live [1]: 
Maximum Time to Live [30]: 20
Port Number [33434]: 
Loose, Strict, Record, Timestamp, Verbose[none]: 
```

**Paramètres importants :**

- **Source address** : Adresse IP source pour le traceroute
- **Probe count** : Nombre de paquets envoyés par saut (défaut : 3)
- **Timeout** : Délai d'attente par saut (défaut : 3 secondes)
- **Max TTL** : TTL maximum (nombre max de sauts, défaut : 30)
- **Port Number** : Port UDP utilisé (défaut : 33434)

> [!tip] Diagnostic avancé Si tous les sauts au-delà d'un certain point montrent `* * *`, cela indique généralement que :
> 
> 1. Le routeur à ce saut bloque les paquets ICMP Time Exceeded
> 2. Il y a un firewall ou une ACL qui filtre le trafic
> 3. La destination est injoignable à partir de ce point

> [!warning] Limitations
> 
> - Certains routeurs ne répondent pas aux messages ICMP Time Exceeded par politique de sécurité
> - Le traceroute peut montrer des chemins différents pour chaque paquet si du load balancing est actif
> - Les paquets UDP utilisés peuvent être bloqués par des firewalls

---

### Comparaison avec le ping

|Aspect|Ping|Traceroute|
|---|---|---|
|**Objectif**|Tester la connectivité end-to-end|Identifier le chemin complet|
|**Protocole**|ICMP Echo Request/Reply|UDP + ICMP Time Exceeded|
|**Information**|Destination joignable ou non|Tous les sauts intermédiaires|
|**Usage**|Test rapide de connectivité|Localisation précise d'un problème|
|**Temps d'exécution**|Rapide (quelques secondes)|Plus lent (peut prendre 30+ secondes)|

> [!info] Complémentarité Utilisez d'abord `ping` pour vérifier si la destination est joignable, puis `traceroute` si le ping échoue pour identifier où se situe le problème dans le chemin réseau.

---

## 📊 Show ip interface brief

### Définition et utilité

La commande **show ip interface brief** affiche un résumé concis de l'état de toutes les interfaces réseau du routeur avec leurs adresses IP, statut administratif et statut opérationnel.

**Pourquoi l'utiliser ?**

- Vue d'ensemble rapide de toutes les interfaces
- Vérifier l'état up/down des interfaces
- Identifier rapidement les adresses IP configurées
- Détecter les interfaces désactivées administrativement

**Quand l'utiliser ?**

- Première commande à exécuter lors d'un dépannage d'interface
- Après avoir configuré une nouvelle interface
- Pour vérifier rapidement l'état général du routeur
- Avant d'approfondir avec des commandes plus détaillées

---

### Syntaxe et utilisation

```bash
# Syntaxe de base (mode utilisateur ou privilégié)
Router> show ip interface brief
Router# show ip interface brief

# Filtrer pour une interface spécifique
Router# show ip interface brief | include GigabitEthernet0/0

# Filtrer pour voir uniquement les interfaces up
Router# show ip interface brief | include up

# Filtrer pour voir les interfaces down
Router# show ip interface brief | include down
```

---

### Interprétation de l'output

> [!example] Exemple d'output typique
> 
> ```
> Interface              IP-Address      OK? Method Status                Protocol
> GigabitEthernet0/0     192.168.1.1     YES manual up                    up
> GigabitEthernet0/1     10.0.0.1        YES manual up                    up
> GigabitEthernet0/2     unassigned      YES unset  administratively down down
> Serial0/0/0            172.16.0.1      YES manual up                    up
> Serial0/0/1            unassigned      YES unset  down                  down
> Loopback0              1.1.1.1         YES manual up                    up
> ```

---

### Colonnes et leur signification

**1. Interface**

- Nom complet de l'interface (GigabitEthernet, Serial, Loopback, etc.)
- Identifie le type et le numéro de port physique ou logique

**2. IP-Address**

- Adresse IPv4 configurée sur l'interface
- `unassigned` : Aucune adresse IP configurée
- Affiche l'adresse même si l'interface est down

**3. OK?**

- `YES` : Configuration valide et cohérente
- `NO` : Problème de configuration (rare)
- Indique si l'IOS accepte la configuration

**4. Method**

- `manual` : Configurée manuellement via CLI
- `DHCP` : Adresse obtenue via DHCP
- `unset` : Aucune méthode d'attribution définie
- `NVRAM` : Configuration chargée depuis la configuration de démarrage
- `TFTP` : Configuration obtenue via TFTP

**5. Status** (État administratif - Layer 1)

- `up` : Interface activée administrativement (`no shutdown`)
- `administratively down` : Interface désactivée (`shutdown`)
- `down` : Interface active administrativement mais physiquement déconnectée
- Contrôlé par les commandes `shutdown` / `no shutdown`

**6. Protocol** (État opérationnel - Layer 2)

- `up` : Protocole de liaison fonctionnel (interface pleinement opérationnelle)
- `down` : Protocole de liaison non fonctionnel
- `up (spoofing)` : Interface Loopback ou tunnel (pas de lien physique réel)
- Dépend de la détection de porteuse et de la négociation de couche 2

---

### Combinaisons Status/Protocol et diagnostic

|Status|Protocol|Signification|Cause probable|Action à prendre|
|---|---|---|---|---|
|up|up|✅ Interface fonctionnelle|Aucune|Aucune|
|administratively down|down|Interface désactivée|Commande `shutdown` active|`no shutdown`|
|down|down|Problème physique|Câble déconnecté, port switch down, interface défaillante|Vérifier câblage, port distant|
|up|down|Problème couche 2|Problème d'encapsulation, horloge manquante (serial), mismatch duplex|Vérifier encapsulation, clock rate, duplex/speed|
|up|up (spoofing)|Interface logique|Loopback ou tunnel|Normal pour ces types|

> [!warning] Combinaison critique : up / down Quand Status = up mais Protocol = down, l'interface est activée administrativement mais la couche liaison de données ne fonctionne pas. C'est souvent dû à :
> 
> - **Serial** : Absence de `clock rate` côté DCE
> - **Ethernet** : Mismatch duplex/speed avec le switch
> - **Encapsulation** : Incompatibilité d'encapsulation entre les deux extrémités

---

### Cas d'usage pratiques

**Scénario 1 : Vérification rapide après configuration**

```bash
Router# configure terminal
Router(config)# interface GigabitEthernet0/0
Router(config-if)# ip address 192.168.1.1 255.255.255.0
Router(config-if)# no shutdown
Router(config-if)# exit
Router(config)# exit

# Vérification immédiate
Router# show ip interface brief
Interface              IP-Address      OK? Method Status                Protocol
GigabitEthernet0/0     192.168.1.1     YES manual up                    up
```

**Scénario 2 : Identifier les interfaces inactives**

```bash
Router# show ip interface brief | include administratively down
GigabitEthernet0/2     unassigned      YES unset  administratively down down
GigabitEthernet0/3     unassigned      YES unset  administratively down down
```

**Scénario 3 : Lister uniquement les interfaces opérationnelles**

```bash
Router# show ip interface brief | include up.*up
GigabitEthernet0/0     192.168.1.1     YES manual up                    up
Serial0/0/0            172.16.0.1      YES manual up                    up
```

---

### Commandes complémentaires

Pour approfondir le diagnostic d'une interface spécifique après avoir identifié un problème avec `show ip interface brief` :

```bash
# Détails complets d'une interface (sera vu dans une autre partie)
Router# show interfaces GigabitEthernet0/0

# Statistiques des erreurs
Router# show interfaces GigabitEthernet0/0 | include error

# Configuration de l'interface
Router# show running-config interface GigabitEthernet0/0
```

> [!tip] Workflow de dépannage optimal
> 
> 1. **show ip interface brief** → Vue d'ensemble et identification du problème
> 2. **show interfaces [nom]** → Détails sur l'interface problématique
> 3. **show running-config interface [nom]** → Vérification de la configuration
> 4. **show controllers [nom]** → Diagnostic au niveau matériel si nécessaire

> [!info] Astuce de mémorisation **Status** = Administratif (ce que VOUS voulez)  
> **Protocol** = Opérationnel (ce que le matériel PEUT faire)
> 
> Pour qu'une interface fonctionne, les deux doivent être "up".

---

## 🛣️ Show ip route

### Définition et utilité

La commande **show ip route** affiche la table de routage IPv4 du routeur, contenant toutes les routes connues pour atteindre les réseaux distants, qu'elles soient directement connectées, statiques ou apprises via des protocoles de routage dynamique.

**Pourquoi l'utiliser ?**

- Vérifier que le routeur connaît un chemin vers une destination
- Identifier quelle route sera utilisée pour un paquet donné
- Valider que les routes statiques ou dynamiques sont correctement installées
- Diagnostiquer les problèmes de routage asymétrique
- Comprendre les décisions de routage basées sur la métrique

**Quand l'utiliser ?**

- Lorsque la connectivité échoue après avoir vérifié l'état des interfaces
- Après avoir configuré des routes statiques ou un protocole de routage
- Pour vérifier qu'une route annoncée par un protocole dynamique est bien reçue
- Pour diagnostiquer des problèmes de trafic qui ne suit pas le chemin attendu

---

### Syntaxe et utilisation

```bash
# Afficher toute la table de routage
Router> show ip route
Router# show ip route

# Afficher uniquement une route spécifique
Router# show ip route 192.168.10.0

# Afficher les routes d'un protocole spécifique
Router# show ip route connected
Router# show ip route static
Router# show ip route ospf
Router# show ip route eigrp
Router# show ip route rip

# Afficher un résumé de la table de routage
Router# show ip route summary
```

---

### Structure de la table de routage

> [!example] Exemple de table de routage complète
> 
> ```
> Router# show ip route
> Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
>        D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
>        N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
>        E1 - OSPF external type 1, E2 - OSPF external type 2
>        i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
>        ia - IS-IS inter area, * - candidate default, U - per-user static route
>        o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
>        a - application route
>        + - replicated route, % - next hop override, p - overrides from PfR
> 
> Gateway of last resort is 192.168.1.254 to network 0.0.0.0
> 
> S*    0.0.0.0/0 [1/0] via 192.168.1.254
>       10.0.0.0/8 is variably subnetted, 3 subnets, 2 masks
> C        10.0.0.0/30 is directly connected, Serial0/0/0
> L        10.0.0.1/32 is directly connected, Serial0/0/0
> S        10.1.0.0/16 [1/0] via 10.0.0.2
>       172.16.0.0/16 is variably subnetted, 2 subnets, 2 masks
> O        172.16.1.0/24 [110/65] via 10.0.0.2, 00:15:23, Serial0/0/0
> O        172.16.2.0/24 [110/129] via 10.0.0.2, 00:12:45, Serial0/0/0
>       192.168.1.0/24 is variably subnetted, 2 subnets, 2 masks
> C        192.168.1.0/24 is directly connected, GigabitEthernet0/0
> L        192.168.1.1/32 is directly connected, GigabitEthernet0/0
> ```

---

### Codes de source de route

Les codes indiquent comment une route a été apprise :

|Code|Signification|Description|
|---|---|---|
|**C**|Connected|Réseau directement connecté à une interface active|
|**L**|Local|Adresse IP d'interface locale (host route /32)|
|**S**|Static|Route statique configurée manuellement|
|**S***|Default Static|Route par défaut statique (0.0.0.0/0)|
|**D**|EIGRP|Route apprise via EIGRP|
|**EX**|EIGRP External|Route externe redistribuée dans EIGRP|
|**O**|OSPF|Route OSPF intra-area|
|**IA**|OSPF Inter Area|Route OSPF inter-area|
|**E1**|OSPF External Type 1|Route externe OSPF de type 1|
|**E2**|OSPF External Type 2|Route externe OSPF de type 2 (défaut)|
|**R**|RIP|Route apprise via RIP|
|**B**|BGP|Route apprise via BGP|

> [!info] Routes locales (L) Depuis IOS 15, Cisco affiche automatiquement les routes locales (/32) pour chaque adresse IP d'interface. Ces routes représentent l'adresse exacte de l'interface et sont utilisées pour le trafic destiné au routeur lui-même.

---

### Anatomie d'une entrée de route

Prenons cet exemple détaillé :

```
O    172.16.1.0/24 [110/65] via 10.0.0.2, 00:15:23, Serial0/0/0
```

**Décomposition :**

1. **O** = Code source (OSPF)
2. **172.16.1.0/24** = Réseau de destination et masque (notation CIDR)
3. **[110/65]** = Distance administrative [110] et métrique [65]
4. **via 10.0.0.2** = Adresse IP du prochain saut (next-hop)
5. **00:15:23** = Durée depuis laquelle la route est dans la table (uptime)
6. **Serial0/0/0** = Interface de sortie pour atteindre cette destination

---

### Distance administrative (AD)

La **distance administrative** est utilisée pour déterminer quelle route installer dans la table de routage quand plusieurs sources proposent le même réseau.

**Plus la valeur est faible, plus la source est fiable.**

|Source de route|Distance Administrative|
|---|---|
|Interface connectée|0|
|Route statique|1|
|Route statique flottante|Configurable (généralement 5-254)|
|eBGP|20|
|EIGRP interne|90|
|IGRP|100|
|OSPF|110|
|IS-IS|115|
|RIP|120|
|EIGRP externe|170|
|iBGP|200|
|Route inconnu|255 (non installée)|

> [!tip] Utilisation pratique de l'AD Si le routeur reçoit une route vers 10.0.0.0/8 via :
> 
> - OSPF avec AD=110
> - RIP avec AD=120
> - Route statique avec AD=1
> 
> Il choisira la **route statique** car AD=1 est la plus faible. Pour forcer l'utilisation d'une route dynamique, configurez la route statique avec une AD plus élevée (route flottante de backup).

---

### Métrique

La **métrique** est utilisée quand plusieurs routes vers la même destination proviennent de la **même source** (même AD). C'est le coût calculé par le protocole de routage.

**Plus la métrique est faible, meilleur est le chemin.**

|Protocole|Métrique basée sur|
|---|---|
|**OSPF**|Coût (Cost) = 10^8 / Bandwidth|
|**EIGRP**|Composite (Bandwidth + Delay par défaut)|
|**RIP**|Hop count (nombre de sauts)|
|**BGP**|Attributs multiples (AS Path, Local Pref, etc.)|

> [!example] Exemple de sélection de route
> 
> ```
> O    172.16.1.0/24 [110/65] via 10.0.0.2, Serial0/0/0
> O    172.16.1.0/24 [110/129] via 10.0.0.6, Serial0/0/1
> ```
> 
> Ici, les deux routes ont la même AD (110), donc la route avec la métrique la plus faible (65) sera préférée.

---

### Gateway of last resort

La **Gateway of last resort** est la route par défaut (0.0.0.0/0) utilisée quand aucune route plus spécifique n'existe pour une destination.

```
Gateway of last resort is 192.168.1.254 to network 0.0.0.0

S*    0.0.0.0/0 [1/0] via 192.168.1.254
```

**Interprétation :**

- Le routeur utilisera 192.168.1.254 comme next-hop pour tout trafic vers des réseaux inconnus
- L'astérisque (*) dans `S*` indique que c'est une route candidate par défaut
- Si aucune gateway of last resort n'est configurée, le message affichera "is not set"

> [!warning] Absence de route par défaut Si "Gateway of last resort is not set" apparaît et qu'aucune route spécifique n'existe pour une destination, les paquets seront droppés avec un message ICMP "Destination Unreachable".

---

### Longest Prefix Match (Correspondance du préfixe le plus long)

Le routeur utilise toujours la route la **plus spécifique** (masque le plus long) correspondant à la destination.

**Ordre de sélection :**

1. **Longest prefix match** (route la plus spécifique)
2. **Distance administrative** (si plusieurs sources pour le même préfixe)
3. **Métrique** (si même source et même préfixe)

> [!example] Exemple de longest prefix match
> 
> ```
> S    192.168.1.0/24 [1/0] via 10.0.0.1
> S    192.168.1.128/25 [1/0] via 10.0.0.2
> S    192.168.1.200/32 [1/0] via 10.0.0.3
> ```
> 
> Pour un paquet destiné à **192.168.1.200** :
> 
> - 192.168.1.0/24 correspond ✓
> - 192.168.1.128/25 correspond ✓
> - 192.168.1.200/32 correspond ✓ **(route choisie - /32 est le plus spécifique)**
> 
> Pour un paquet destiné à **192.168.1.150** :
> 
> - 192.168.1.0/24 correspond ✓
> - 192.168.1.128/25 correspond ✓ **(route choisie - /25 est plus spécifique que /24)**
> - 192.168.1.200/32 ne correspond pas ✗

---

### Variable Length Subnet Mask (VLSM)

Quand un réseau utilise différents masques de sous-réseau, l'IOS indique "variably subnetted" :

```
      10.0.0.0/8 is variably subnetted, 3 subnets, 2 masks
C        10.0.0.0/30 is directly connected, Serial0/0/0
L        10.0.0.1/32 is directly connected, Serial0/0/0
S        10.1.0.0/16 [1/0] via 10.0.0.2
```

**Signification :**

- Le réseau majeur 10.0.0.0/8 contient 3 sous-réseaux
- Ces sous-réseaux utilisent 2 masques différents (/30, /32, /16)
- Ceci est normal et attendu dans les réseaux modernes utilisant VLSM

---

### Commandes de filtrage et d'analyse

```bash
# Chercher une route spécifique
Router# show ip route 192.168.10.0

# Afficher uniquement les routes statiques
Router# show ip route static

# Afficher uniquement les routes OSPF
Router# show ip route ospf

# Compter le nombre total de routes
Router# show ip route summary

# Filtrer les routes d'un sous-réseau
Router# show ip route | include 172.16

# Afficher la route qui sera utilisée pour une IP spécifique
Router# show ip route 192.168.10.50
```

> [!tip] Commande de diagnostic critique
> 
> ```bash
> Router# show ip route [adresse_IP]
> ```
> 
> Cette commande montre exactement quelle route sera utilisée pour atteindre une adresse IP spécifique, ce qui est crucial pour diagnostiquer des problèmes de routage.

---

### Cas d'usage pratiques

**Scénario 1 : Vérifier si une route existe**

```bash
Router# show ip route 172.16.1.0
% Network not in table
```

→ Le routeur n'a aucune route vers ce réseau, les paquets seront droppés.

**Scénario 2 : Routes multiples vers le même réseau**

```bash
Router# show ip route 10.0.0.0
Routing entry for 10.0.0.0/8, 4 known subnets
  Attached (2 connections)
  Variably subnetted with 3 masks
  Redistributing via ospf 1

C    10.0.0.0/30 is directly connected, Serial0/0/0
L    10.0.0.1/32 is directly connected, Serial0/0/0
O    10.1.0.0/16 [110/65] via 10.0.0.2, Serial0/0/0
S    10.2.0.0/16 [1/0] via 10.0.0.2
```

**Scénario 3 : Load balancing (Equal Cost Multi-Path)**

```bash
Router# show ip route
O    172.16.1.0/24 [110/65] via 10.0.0.2, Serial0/0/0
                   [110/65] via 10.0.0.6, Serial0/0/1
```

→ Deux chemins avec la même AD et métrique : le trafic sera réparti entre les deux interfaces.

---

### Dépannage avec show ip route

> [!info] Workflow de dépannage routage
> 
> 1. **Vérifier la connectivité** : `ping` échoue
> 2. **Vérifier les interfaces** : `show ip interface brief` → interfaces up/up
> 3. **Vérifier la table de routage** : `show ip route [destination]`
>     - Route existe ? → Vérifier next-hop et interface de sortie
>     - Route n'existe pas ? → Configurer route statique ou protocole de routage
>     - Mauvaise route préférée ? → Vérifier AD et métrique
> 4. **Vérifier le routage de retour** sur le routeur distant

**Questions clés à se poser :**

- ✅ Le routeur a-t-il une route vers la destination ?
- ✅ La route pointe-t-elle vers le bon next-hop ?
- ✅ L'interface de sortie est-elle up/up ?
- ✅ Le routeur distant a-t-il une route de retour ?

> [!warning] Pièges courants
> 
> - **Oublier la route de retour** : Le trafic aller fonctionne mais pas le retour
> - **AD trop élevée sur route statique** : Une route dynamique moins optimale est préférée
> - **Masque incorrect** : La route existe mais ne correspond pas à cause du masque
> - **Next-hop injoignable** : La route existe mais le next-hop lui-même n'est pas accessible

---

## 👥 Show cdp neighbors

### Définition et utilité

Le protocole **CDP (Cisco Discovery Protocol)** est un protocole propriétaire Cisco de couche 2 qui permet aux équipements Cisco directement connectés de se découvrir mutuellement et d'échanger des informations de base.

**Pourquoi l'utiliser ?**

- Découvrir la topologie réseau sans documentation
- Identifier les équipements Cisco directement connectés
- Vérifier les connexions physiques et logiques
- Obtenir des informations sur les modèles, interfaces, adresses IP des voisins
- Diagnostiquer les problèmes de câblage

**Quand l'utiliser ?**

- Lors de la découverte initiale d'un réseau
- Pour vérifier qu'un câble est bien connecté au bon port
- Identifier un équipement inconnu dans le rack
- Valider la topologie physique
- Dépannage de connectivité au niveau liaison de données

---

### Caractéristiques de CDP

**Propriétés du protocole :**

- **Couche 2** : Fonctionne indépendamment du protocole de couche 3 (IP, IPX, etc.)
- **Propriétaire Cisco** : Fonctionne uniquement entre équipements Cisco
- **Multicast** : Utilise l'adresse MAC 01:00:0C:CC:CC:CC
- **Périodique** : Messages envoyés toutes les 60 secondes par défaut
- **Holdtime** : Les informations sont conservées pendant 180 secondes par défaut
- **Activé par défaut** : Sur tous les équipements Cisco (routeurs, switches)

> [!info] Alternative LLDP Le protocole **LLDP (Link Layer Discovery Protocol)** est l'équivalent standard (IEEE 802.1AB) de CDP, supporté par de nombreux fabricants. Sur Cisco, LLDP doit être activé manuellement et peut coexister avec CDP.

---

### Syntaxe et utilisation

```bash
# Vue basique des voisins (mode utilisateur ou privilégié)
Router> show cdp neighbors
Router# show cdp neighbors

# Vue détaillée de tous les voisins
Router# show cdp neighbors detail

# Informations sur un voisin spécifique
Router# show cdp entry [nom_équipement]
Router# show cdp entry *

# Informations sur les interfaces locales utilisant CDP
Router# show cdp interface

# Statistiques CDP globales
Router# show cdp traffic

# Vérifier si CDP est activé globalement
Router# show cdp
```

---

### Output de show cdp neighbors

> [!example] Exemple d'output basique
> 
> ```
> Router# show cdp neighbors
> Capability Codes: R - Router, T - Trans Bridge, B - Source Route Bridge
>                   S - Switch, H - Host, I - IGMP, r - Repeater, P - Phone,
>                   D - Remote, C - CVTA, M - Two-port Mac Relay
> 
> Device ID        Local Intrfce     Holdtme    Capability  Platform  Port ID
> Switch1          Gig 0/0           165        S I         WS-C2960  Fas 0/1
> Router2          Ser 0/0/0         142        R           ISR4321   Ser 0/0/0
> Router3          Gig 0/1           178        R           ISR4331   Gig 0/0
> ```

**Colonnes expliquées :**

1. **Device ID** : Nom d'hôte (hostname) de l'équipement voisin
2. **Local Interface** : Interface locale connectée au voisin
3. **Holdtime** : Temps restant (en secondes) avant expiration des informations
4. **Capability** : Type d'équipement (R=Router, S=Switch, etc.)
5. **Platform** : Modèle matériel de l'équipement voisin
6. **Port ID** : Interface distante connectée à notre interface locale

---

### Capability Codes

|Code|Signification|Description|
|---|---|---|
|**R**|Router|Équipement de routage (couche 3)|
|**S**|Switch|Commutateur (couche 2)|
|**H**|Host|Équipement terminal|
|**I**|IGMP|Supporte IGMP|
|**r**|Repeater|Répéteur|
|**P**|Phone|Téléphone IP Cisco|
|**B**|Source Route Bridge|Pont avec routage source|
|**T**|Trans Bridge|Pont transparent|
|**D**|Remote|Appareil distant (RSPAN)|
|**C**|CVTA|Cisco VoIP Telephony Adapter|

> [!tip] Interprétation des Capability Un équipement peut avoir plusieurs capabilities. Par exemple, "R S I" indique un routeur qui peut aussi faire du switching et supporte IGMP (typique d'un routeur multicouche).

---

### Show cdp neighbors detail

Cette version détaillée fournit beaucoup plus d'informations sur chaque voisin :

> [!example] Exemple d'output détaillé
> 
> ```
> Router# show cdp neighbors detail
> 
> -------------------------
> Device ID: Switch1
> Entry address(es): 
>   IP address: 192.168.1.2
> Platform: cisco WS-C2960-24TT-L,  Capabilities: Switch IGMP 
> Interface: GigabitEthernet0/0,  Port ID (outgoing port): FastEthernet0/1
> Holdtime : 142 sec
> 
> Version :
> Cisco IOS Software, C2960 Software (C2960-LANBASEK9-M), Version 15.0(2)SE11
> 
> advertisement version: 2
> Protocol Hello:  OUI=0x00000C, Protocol ID=0x0112; payload len=27
> VTP Management Domain: 'MYDOMAIN'
> Native VLAN: 1
> Duplex: full
> Management address(es): 
>   IP address: 192.168.1.2
> 
> -------------------------
> Device ID: Router2
> Entry address(es): 
>   IP address: 10.0.0.2
> Platform: Cisco ISR4321/K9,  Capabilities: Router
> Interface: Serial0/0/0,  Port ID (outgoing port): Serial0/0/0
> Holdtime : 178 sec
> 
> Version :
> Cisco IOS Software, ISR4300 Software (X86_64_LINUX_IOSD-UNIVERSALK9-M), Version 16.9.4
> 
> advertisement version: 2
> Duplex: full
> Management address(es): 
>   IP address: 10.0.0.2
> ```

**Informations supplémentaires fournies :**

- **IP address** : Adresses IP de l'équipement distant
- **Version** : Version d'IOS détaillée
- **VTP Domain** : Domaine VTP (pour les switches)
- **Native VLAN** : VLAN natif sur les liens trunk
- **Duplex** : Mode duplex de l'interface (full/half)
- **Management address** : Adresses de management

---

### Configuration de CDP

```bash
# Activer CDP globalement (activé par défaut)
Router(config)# cdp run

# Désactiver CDP globalement
Router(config)# no cdp run

# Désactiver CDP sur une interface spécifique
Router(config)# interface GigabitEthernet0/0
Router(config-if)# no cdp enable

# Activer CDP sur une interface
Router(config-if)# cdp enable

# Modifier le timer CDP (défaut : 60 secondes)
Router(config)# cdp timer 30

# Modifier le holdtime (défaut : 180 secondes)
Router(config)# cdp holdtime 120

# Vérifier la configuration CDP
Router# show cdp
Global CDP information:
    Sending CDP packets every 60 seconds
    Sending a holdtime value of 180 seconds
    Sending CDPv2 advertisements is enabled
```

---

### Cas d'usage pratiques

**Scénario 1 : Identifier un équipement inconnu**

```bash
Router# show cdp neighbors
Device ID        Local Intrfce     Holdtme    Capability  Platform  Port ID
UnknownDevice    Gig 0/1           142        R S         ISR4331   Gig 0/0

Router# show cdp neighbors detail | begin UnknownDevice
Device ID: UnknownDevice
Entry address(es): 
  IP address: 172.16.50.1
Platform: Cisco ISR4331/K9,  Capabilities: Router Switch
Interface: GigabitEthernet0/1,  Port ID (outgoing port): GigabitEthernet0/0
```

→ On peut maintenant se connecter en SSH à 172.16.50.1 pour identifier l'équipement

**Scénario 2 : Vérifier le câblage**

```bash
# On s'attend à ce que Gig0/0 soit connecté au port Gig0/1 du Switch1
Router# show cdp neighbors
Device ID        Local Intrfce     Holdtme    Capability  Platform  Port ID
Switch1          Gig 0/0           165        S I         WS-C2960  Fas 0/24
```

→ Le câble est connecté à Fas0/24 au lieu de Gig0/1 !

**Scénario 3 : Pas de voisin détecté**

```bash
Router# show cdp neighbors
Router#
```

**Causes possibles :**

1. CDP désactivé sur l'interface ou globalement
2. Câble débranché ou défectueux
3. Équipement distant éteint
4. Équipement distant non-Cisco (utiliser LLDP à la place)
5. Holdtime expiré (attendre 60 secondes pour nouvelle annonce)

---

### Sécurité et CDP

> [!warning] Risque de sécurité CDP révèle beaucoup d'informations sur votre infrastructure :
> 
> - Modèles d'équipements (vulnérabilités connues)
> - Versions d'IOS (exploits possibles)
> - Adresses IP de management
> - Topologie réseau
> 
> **Bonnes pratiques :**
> 
> - Désactiver CDP sur les interfaces exposées à des réseaux non fiables
> - Désactiver CDP sur les interfaces connectées à Internet
> - Désactiver CDP sur les ports d'accès utilisateurs (si non nécessaire)
> - Utiliser `no cdp enable` sur les interfaces vers l'extérieur

```bash
# Désactiver CDP sur l'interface WAN
Router(config)# interface GigabitEthernet0/2
Router(config-if)# description WAN Link to Internet
Router(config-if)# no cdp enable
```

---

### Comparaison CDP vs LLDP

|Aspect|CDP|LLDP|
|---|---|---|
|**Standard**|Propriétaire Cisco|IEEE 802.1AB (ouvert)|
|**Compatibilité**|Équipements Cisco uniquement|Multi-fabricants|
|**Par défaut**|Activé|Désactivé (sur Cisco)|
|**Informations**|Très détaillées (IP, VTP, etc.)|Standard (device ID, port, capabilities)|
|**Configuration**|`cdp run`|`lldp run`|

```bash
# Activer LLDP (pour environnements multi-fabricants)
Router(config)# lldp run

# Voir les voisins LLDP
Router# show lldp neighbors
Router# show lldp neighbors detail
```

---

## 🔌 Show controllers

### Définition et utilité

La commande **show controllers** affiche des informations de bas niveau sur les contrôleurs d'interface matériels, incluant des détails sur le câblage, l'état physique de la couche 1, et la configuration du matériel.

**Pourquoi l'utiliser ?**

- Déterminer si une interface série est DCE ou DTE
- Vérifier la présence et le type de câble physique
- Diagnostiquer les problèmes matériels au niveau des contrôleurs
- Voir les statistiques détaillées des erreurs de transmission
- Identifier les problèmes de signalisation électrique

**Quand l'utiliser ?**

- Lors de la configuration initiale d'interfaces série (pour savoir où mettre le clock rate)
- Quand une interface est "up/down" et que les autres commandes n'ont rien révélé
- Pour diagnostiquer des erreurs CRC ou des problèmes de couche physique
- Pour identifier le type de câble et de contrôleur installé

---

### Syntaxe et utilisation

```bash
# Voir les informations du contrôleur d'une interface spécifique
Router# show controllers [type] [numéro]

# Exemples
Router# show controllers serial 0/0/0
Router# show controllers serial 0/0/1
Router# show controllers GigabitEthernet 0/0
```

> [!warning] Mode privilégié requis La commande `show controllers` n'est disponible qu'en mode privilégié (enable), pas en mode utilisateur.

---

### Interfaces série : DCE vs DTE

L'utilisation la plus courante de `show controllers` est de déterminer si une interface série est **DCE** (Data Circuit-terminating Equipment) ou **DTE** (Data Terminal Equipment).

**Pourquoi c'est important ?**

- Seul le côté **DCE** doit fournir le signal d'horloge (clock rate)
- Si le clock rate n'est pas configuré côté DCE, l'interface sera "up/down"
- Dans les environnements de laboratoire (routeurs back-to-back), il faut identifier le DCE

> [!example] Output pour interface série DCE
> 
> ```
> Router# show controllers serial 0/0/0
> Interface Serial0/0/0
> Hardware is PowerQUICC MPC860
> DCE V.35, clock rate 64000
> idb at 0x81081AC4, driver data structure at 0x81084AC0
> SCC Registers:
> General [GSMR]=0x2:0x00000000, Protocol-specific [PSMR]=0x8
> Events [SCCE]=0x0000, Mask [SCCM]=0x0000, Status [SCCS]=0x00
> Transmit on Demand [TODR]=0x0, Data Sync [DSR]=0x7E7E
> ```
> 
> **Informations clés :**
> 
> - **DCE V.35** : L'interface est DCE avec câble V.35
> - **clock rate 64000** : Horloge configurée à 64 kbps

> [!example] Output pour interface série DTE
> 
> ```
> Router# show controllers serial 0/0/1
> Interface Serial0/0/1
> Hardware is PowerQUICC MPC860
> DTE V.35 TX and RX clocks detected
> idb at 0x8108254C, driver data structure at 0x810854A0
> ```
> 
> **Informations clés :**
> 
> - **DTE V.35** : L'interface est DTE
> - **TX and RX clocks detected** : L'horloge est reçue depuis le DCE distant

---

### Configuration du clock rate (DCE)

Une fois identifié que l'interface est DCE, il faut configurer le clock rate :

```bash
Router# configure terminal
Router(config)# interface serial 0/0/0
Router(config-if)# clock rate 64000
Router(config-if)# exit
Router(config)# exit

# Vérification
Router# show controllers serial 0/0/0 | include clock rate
DCE V.35, clock rate 64000
```

**Valeurs courantes de clock rate (en bits/s) :**

- 56000 (56 Kbps)
- 64000 (64 Kbps) - Standard T1/E1
- 128000 (128 Kbps)
- 1544000 (T1 - 1.544 Mbps)
- 2048000 (E1 - 2.048 Mbps)

> [!tip] Meilleures pratiques
> 
> - Dans un lab, choisissez une vitesse cohérente (généralement 64000 ou 128000)
> - En production, le clock rate est fourni par le CSU/DSU ou l'opérateur
> - Si vous voyez "DTE mode", ne configurez JAMAIS de clock rate (commande ignorée ou erreur)

---

### Types de câbles série

Les interfaces série peuvent utiliser différents types de câbles :

|Type de câble|Description|Usage|
|---|---|---|
|**V.35**|Standard pour liaisons WAN haute vitesse|Très courant en lab|
|**RS-232**|Standard série classique|Débits faibles (<64 Kbps)|
|**X.21**|Standard européen|Moins courant|
|**EIA/TIA-449**|Extension de RS-232|Débits moyens|

```bash
Router# show controllers serial 0/0/0 | include cable
DCE V.35, cable type : V.35 DCE cable
```

---

### Diagnostiquer les problèmes matériels

La commande `show controllers` révèle également des problèmes matériels comme :

**1. Absence de câble**

```bash
Router# show controllers serial 0/0/0
Interface Serial0/0/0
Hardware is PowerQUICC MPC860
No cable attached
```

→ Aucun câble n'est branché sur l'interface

**2. Erreurs de transmission**

```bash
Router# show controllers serial 0/0/0
...
0 input aborts, 0 CRC, 0 frame, 0 overrun
5 output errors, 0 collisions, 2 interface resets
0 output buffer failures, 0 output buffers swapped out
```

**Compteurs d'erreurs importants :**

- **CRC errors** : Erreurs de contrôle de redondance cyclique (problème de câble ou d'interférences)
- **Frame errors** : Trames mal formées (problème d'horloge ou de synchronisation)
- **Input/output errors** : Erreurs générales d'entrée/sortie
- **Interface resets** : Nombre de fois où l'interface a été réinitialisée
- **Aborts** : Transmissions abandonnées

> [!warning] Interprétation des erreurs
> 
> - **Quelques erreurs** : Normal sur des longues périodes
> - **Erreurs croissantes** : Problème actif (câble défectueux, interférences, problème matériel)
> - **CRC élevé** : Souvent un câble endommagé ou des interférences électromagnétiques
> - **Frame errors + up/down flapping** : Problème d'horloge (vérifier clock rate côté DCE)

---

### Show controllers pour interfaces Ethernet

Pour les interfaces Ethernet/GigabitEthernet, `show controllers` affiche des informations sur le PHY (Physical Layer) et le contrôleur Ethernet :

```bash
Router# show controllers GigabitEthernet 0/0
Interface GigabitEthernet0/0
Hardware is MV96340 Internal MAC
Embedded PHY (copper)
...
Link is UP
Full-duplex, 1000Mb/s, link type is auto
```

**Informations utiles :**

- **Type de PHY** : Copper (cuivre) ou Fiber (fibre optique)
- **Link state** : UP ou DOWN au niveau matériel
- **Speed/Duplex** : Vitesse négociée et mode duplex
- **Auto-negotiation** : Si activée ou manuelle

---

### Cas d'usage pratiques

**Scénario 1 : Interface série up/down**

```bash
Router# show ip interface brief
Interface              IP-Address      OK? Method Status                Protocol
Serial0/0/0            10.0.0.1        YES manual up                    down

Router# show controllers serial 0/0/0
Interface Serial0/0/0
Hardware is PowerQUICC MPC860
DCE V.35, no clock rate configured
```

→ **Solution** : Configurer le clock rate car l'interface est DCE

```bash
Router(config)# interface serial 0/0/0
Router(config-if)# clock rate 64000
Router(config-if)# exit

Router# show ip interface brief | include Serial0/0/0
Serial0/0/0            10.0.0.1        YES manual up                    up
```

**Scénario 2 : Identifier DCE pour lab back-to-back**

```bash
# Sur RouterA
RouterA# show controllers serial 0/0/0
DCE V.35, no clock rate configured

# Sur RouterB
RouterB# show controllers serial 0/0/0
DTE V.35 TX and RX clocks detected
```

→ RouterA est DCE, il faut y configurer le clock rate

**Scénario 3 : Erreurs CRC croissantes**

```bash
Router# show controllers serial 0/0/0 | include CRC
45 input aborts, 127 CRC, 12 frame, 0 overrun

# Attendre 5 minutes puis revérifier
Router# show controllers serial 0/0/0 | include CRC
45 input aborts, 198 CRC, 18 frame, 0 overrun
```

→ Les erreurs CRC augmentent : probable problème de câble à remplacer

---

### Commandes complémentaires

```bash
# Voir toutes les interfaces et leur statut (alternative plus simple)
Router# show interfaces status

# Voir les statistiques d'une interface série
Router# show interfaces serial 0/0/0

# Réinitialiser les compteurs d'erreurs
Router# clear counters serial 0/0/0

# Voir uniquement les erreurs
Router# show interfaces serial 0/0/0 | include error
```

> [!tip] Workflow de dépannage interface série
> 
> 1. **show ip interface brief** → Status/Protocol
> 2. Si "up/down" → **show controllers serial X** → Vérifier DCE/DTE et clock rate
> 3. Si erreurs → **show controllers serial X** → Analyser compteurs d'erreurs
> 4. **show interfaces serial X** → Détails complets et statistiques
> 5. Si problème persiste → Tester avec un autre câble ou port

---

## 🎓 Résumé des commandes de diagnostic

|Commande|Couche OSI|Objectif principal|Quand l'utiliser|
|---|---|---|---|
|**ping**|3 (Réseau)|Tester connectivité IP end-to-end|Premier test de connectivité|
|**traceroute**|3 (Réseau)|Identifier le chemin et localiser problèmes|Quand ping échoue|
|**show ip interface brief**|2-3 (Liaison-Réseau)|Vue d'ensemble rapide des interfaces|Toujours en premier lors du dépannage|
|**show ip route**|3 (Réseau)|Vérifier routes disponibles|Problème de routage|
|**show cdp neighbors**|2 (Liaison)|Découvrir topologie et voisins|Vérifier connexions physiques|
|**show controllers**|1 (Physique)|Diagnostiquer problèmes matériels|Interface up/down, erreurs physiques|

---

### 🔄 Méthodologie de dépannage complète

> [!info] Approche systématique (modèle OSI ascendant) **Étape 1 - Couche Physique (Layer 1)**
> 
> ```bash
> Router# show ip interface brief
> Router# show controllers [interface]
> ```
> 
> → Câble branché ? Interface up ? DCE configuré avec clock rate ?
> 
> **Étape 2 - Couche Liaison (Layer 2)**
> 
> ```bash
> Router# show cdp neighbors
> Router# show interfaces [interface]
> ```
> 
> → Voisin détecté ? Protocol up ? Erreurs de trame ?
> 
> **Étape 3 - Couche Réseau (Layer 3)**
> 
> ```bash
> Router# show ip interface brief
> Router# show ip route
> Router# ping [destination]
> ```
> 
> → IP configurée ? Route existe ? Connectivité ?
> 
> **Étape 4 - Localisation du problème**
> 
> ```bash
> Router# traceroute [destination]
> Router# ping [destination] source [interface]
> ```
> 
> → Où est le problème dans le chemin ? Routage bidirectionnel OK ?

---

### 💡 Combinaison efficace des commandes

**Exemple de dépannage réel :**

```bash
# Utilisateur : "Je ne peux pas accéder à 172.16.1.10"

# Étape 1 : Vérifier l'état des interfaces
Router# show ip interface brief
Interface              IP-Address      OK? Method Status                Protocol
GigabitEthernet0/0     192.168.1.1     YES manual up                    up
Serial0/0/0            10.0.0.1        YES manual up                    down  ← Problème !

# Étape 2 : Identifier la cause du "up/down"
Router# show controllers serial 0/0/0
DCE V.35, no clock rate configured  ← Cause identifiée !

# Étape 3 : Corriger le problème
Router(config)# interface serial 0/0/0
Router(config-if)# clock rate 64000
Router(config-if)# exit

# Étape 4 : Vérifier que l'interface est maintenant up/up
Router# show ip interface brief | include Serial0/0/0
Serial0/0/0            10.0.0.1        YES manual up                    up

# Étape 5 : Vérifier que la route existe
Router# show ip route 172.16.1.0
Routing entry for 172.16.1.0/24
  Known via "ospf 1", distance 110, metric 65
  Last update from 10.0.0.2 on Serial0/0/0

# Étape 6 : Tester la connectivité
Router# ping 172.16.1.10
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 172.16.1.10, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5)
```

---

### ⚠️ Pièges courants et solutions

> [!warning] Erreur #1 : Oublier la route de retour **Symptôme** : Ping étendu depuis interface A fonctionne, mais depuis interface B échoue
> 
> ```bash
> RouterA# ping 10.0.0.2 source 192.168.1.1
> !!!!!  ← Succès
> 
> RouterA# ping 10.0.0.2 source 172.16.1.1
> .....  ← Échec
> ```
> 
> **Cause** : Le routeur distant n'a pas de route de retour vers 172.16.1.0/24
> 
> **Solution** : Vérifier la table de routage sur RouterB
> 
> ```bash
> RouterB# show ip route 172.16.1.0
> % Network not in table  ← Pas de route !
> ```

> [!warning] Erreur #2 : Interface "up/down" sur liaison série **Symptôme** : Status = up, Protocol = down
> 
> **Causes possibles :**
> 
> 1. Clock rate manquant côté DCE
> 2. Encapsulation mismatch (HDLC vs PPP)
> 3. Problème d'authentification PPP
> 
> **Diagnostic :**
> 
> ```bash
> Router# show controllers serial 0/0/0  ← Vérifier DCE/DTE
> Router# show interfaces serial 0/0/0 | include Encapsulation
> ```

> [!warning] Erreur #3 : CDP ne montre aucun voisin alors que le câble est branché **Causes possibles :**
> 
> 4. CDP désactivé globalement ou sur l'interface
> 5. Équipement distant non-Cisco
> 6. Holdtime expiré (attendre 60 secondes)
> 
> **Vérification :**
> 
> ```bash
> Router# show cdp
> % CDP is not enabled  ← CDP désactivé !
> 
> Router# show cdp interface GigabitEthernet0/0
> % CDP is not enabled on GigabitEthernet0/0  ← CDP désactivé sur l'interface
> ```

---

### 📌 Checklist de dépannage rapide

**✅ Connectivité de base**

- [ ] `show ip interface brief` → Toutes les interfaces concernées sont up/up ?
- [ ] `ping [destination]` → Destination joignable ?
- [ ] `show ip route [destination]` → Route existe vers la destination ?

**✅ Problème de routage**

- [ ] `traceroute [destination]` → Où s'arrête le chemin ?
- [ ] `show ip route` → Routes correctes installées ?
- [ ] Routage de retour vérifié sur l'équipement distant ?

**✅ Problème physique/liaison**

- [ ] `show cdp neighbors` → Voisin détecté ?
- [ ] `show controllers [interface]` → Interface DCE avec clock rate ?
- [ ] Câble bien branché des deux côtés ?

**✅ Erreurs et performance**

- [ ] `show interfaces [interface]` → Erreurs CRC/frame croissantes ?
- [ ] `show controllers [interface]` → Compteurs d'erreurs matérielles ?
- [ ] Latence excessive ou perte de paquets ?

---

### 🎯 Points clés à retenir

> [!tip] Les 7 commandements du dépannage Cisco
> 
> 1. **Toujours commencer par show ip interface brief** pour avoir une vue d'ensemble
> 2. **Utiliser ping standard d'abord**, puis ping étendu pour tests avancés
> 3. **Traceroute localise** où le problème se situe dans le réseau
> 4. **Show ip route vérifie** que le routeur sait comment atteindre la destination
> 5. **Show cdp neighbors confirme** les connexions physiques et la topologie
> 6. **Show controllers diagnostique** les problèmes matériels de couche 1
> 7. **Ne jamais oublier** de vérifier le chemin de retour (routage bidirectionnel)

> [!info] Ordre logique des commandes
> 
> ```
> show ip interface brief
>         ↓
>    Interface down ?
>         ↓
> show controllers → Vérifier matériel/clock rate
>         ↓
>    Interface up/up ?
>         ↓
> show ip route → Route existe ?
>         ↓
>    Route existe ?
>         ↓
> ping → Connectivité OK ?
>         ↓
>    Ping échoue ?
>         ↓
> traceroute → Localiser le problème
> ```

---

## 📚 Tableau récapitulatif complet

|Situation|Commande à utiliser|Ce qu'elle révèle|
|---|---|---|
|Vue d'ensemble initiale|`show ip interface brief`|État de toutes les interfaces|
|Tester connectivité simple|`ping [IP]`|Destination joignable ou non|
|Tester depuis interface spécifique|`ping [IP] source [interface]`|Connectivité avec IP source spécifique|
|Localiser où ça bloque|`traceroute [IP]`|Chemin complet et point de défaillance|
|Vérifier existence d'une route|`show ip route [réseau]`|Route vers destination et next-hop|
|Identifier équipement connecté|`show cdp neighbors`|Voisins Cisco directement connectés|
|Détails sur un voisin|`show cdp neighbors detail`|IP, version IOS, capabilities|
|Interface série up/down|`show controllers serial X`|DCE/DTE, clock rate, type de câble|
|Erreurs matérielles|`show controllers [interface]`|CRC, frame errors, resets|
|Problème de câble série lab|`show controllers serial X`|Identifier qui est DCE pour clock rate|

---

**🎉 Fin du cours sur les commandes de diagnostic essentielles**

Ce document couvre l'ensemble des commandes fondamentales pour diagnostiquer les problèmes de connectivité et de routage sur les routeurs Cisco. Maîtriser ces commandes et leur utilisation méthodique est essentiel pour tout administrateur réseau travaillant avec des équipements Cisco.