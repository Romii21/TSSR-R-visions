

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

## 🎯 Introduction {#introduction}

La vérification et le dépannage du routage statique sont des compétences essentielles pour tout administrateur réseau. Cette phase permet de s'assurer que les routes configurées fonctionnent correctement et d'identifier rapidement les problèmes de connectivité.

> [!info] Pourquoi la vérification est cruciale
> 
> - Le routage statique ne s'adapte pas automatiquement aux changements réseau
> - Une erreur de configuration peut isoler des portions entières du réseau
> - La vérification proactive évite les incidents en production
> - Les outils de diagnostic permettent d'identifier précisément la source d'un problème

---

## 📊 Analyse de la table de routage (show ip route) {#analyse-table-routage}

### Présentation de la commande

La commande `show ip route` affiche la table de routage complète du routeur. C'est la première commande à utiliser pour vérifier le routage.

```bash
Router# show ip route
```

> [!tip] Bonnes pratiques Utilisez toujours `show ip route` en mode privilégié pour voir l'intégralité de la table de routage.

### Structure de la table de routage

#### Codes de routes

```bash
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
```

|Code|Type de route|Description|
|---|---|---|
|**C**|Connected|Réseau directement connecté|
|**L**|Local|Adresse IP de l'interface locale|
|**S**|Static|Route statique configurée manuellement|
|**S***|Static Default|Route statique par défaut|

#### Exemple de sortie complète

```bash
Router# show ip route

Gateway of last resort is 192.168.1.1 to network 0.0.0.0

S*    0.0.0.0/0 [1/0] via 192.168.1.1
      10.0.0.0/8 is variably subnetted, 4 subnets, 2 masks
C        10.1.1.0/24 is directly connected, GigabitEthernet0/0
L        10.1.1.1/32 is directly connected, GigabitEthernet0/0
S        10.2.2.0/24 [1/0] via 10.1.1.2
S        10.3.3.0/24 [1/0] via 10.1.1.3
      192.168.1.0/24 is variably subnetted, 2 subnets, 2 masks
C        192.168.1.0/24 is directly connected, GigabitEthernet0/1
L        192.168.1.1/32 is directly connected, GigabitEthernet0/1
```

### Décryptage d'une ligne de route statique

```bash
S        10.2.2.0/24 [1/0] via 10.1.1.2
```

|Élément|Signification|
|---|---|
|**S**|Route statique|
|**10.2.2.0/24**|Réseau de destination|
|**[1/0]**|[Distance administrative / Métrique]|
|**via 10.1.1.2**|Adresse IP du saut suivant|

> [!info] Distance administrative
> 
> - Pour les routes statiques : 1 par défaut
> - Pour les routes statiques flottantes : peut être modifiée (exemple : 5, 10, etc.)
> - Plus la valeur est faible, plus la route est prioritaire

### Commandes dérivées pour filtrer l'affichage

#### Afficher uniquement les routes statiques

```bash
Router# show ip route static
```

#### Afficher une route spécifique

```bash
Router# show ip route 10.2.2.0
```

Sortie :

```bash
Routing entry for 10.2.2.0/24
  Known via "static", distance 1, metric 0
  Routing Descriptor Blocks:
  * 10.1.1.2
      Route metric is 0, traffic share count is 1
```

#### Afficher uniquement les routes connectées

```bash
Router# show ip route connected
```

> [!example] Cas pratique d'analyse Vous configurez une route statique vers 172.16.0.0/16 via 10.1.1.254 mais les paquets n'arrivent pas :
> 
> 1. Vérifiez que la route apparaît dans `show ip route`
> 2. Vérifiez que l'interface de sortie est UP/UP
> 3. Vérifiez que le next-hop (10.1.1.254) est joignable

### Signaux d'alerte dans la table de routage

> [!warning] Routes manquantes
> 
> - Si une route statique configurée n'apparaît pas, vérifiez :
>     - Que l'interface de sortie est active (up/up)
>     - Que le next-hop est dans un réseau directement connecté
>     - Qu'il n'y a pas d'erreur de syntaxe dans la configuration

> [!warning] Routes en double
> 
> - Plusieurs routes vers le même réseau peuvent indiquer :
>     - Une redondance voulue (load balancing)
>     - Une erreur de configuration
>     - Un conflit entre routage statique et dynamique

---

## 🛤️ Commande traceroute {#commande-traceroute}

### Principe de fonctionnement

Traceroute permet de visualiser le chemin complet qu'emprunte un paquet du routeur source vers la destination. Elle envoie des paquets UDP (par défaut) avec des valeurs TTL (Time To Live) croissantes.

> [!info] Mécanisme TTL
> 
> 1. Premier paquet envoyé avec TTL=1 → le premier routeur répond avec ICMP Time Exceeded
> 2. Deuxième paquet avec TTL=2 → le deuxième routeur répond
> 3. Et ainsi de suite jusqu'à la destination finale

### Syntaxe de base

```bash
Router# traceroute [adresse_IP_ou_nom_domaine]
```

#### Exemple de sortie réussie

```bash
Router# traceroute 10.3.3.10

Type escape sequence to abort.
Tracing the route to 10.3.3.10
VRF info: (vrf in name/id, vrf out name/id)
  1 10.1.1.2 12 msec 8 msec 8 msec
  2 10.2.2.5 16 msec 12 msec 12 msec
  3 10.3.3.10 20 msec 16 msec 20 msec
```

|Colonne|Information|
|---|---|
|**Numéro**|Numéro du saut (hop)|
|**Adresse IP**|IP du routeur intermédiaire|
|**Temps 1, 2, 3**|Temps de réponse en millisecondes (3 sondes par défaut)|

#### Interprétation des résultats

```bash
Router# traceroute 192.168.50.1

Type escape sequence to abort.
Tracing the route to 192.168.50.1
VRF info: (vrf in name/id, vrf out name/id)
  1 10.1.1.2 8 msec 4 msec 8 msec
  2 10.2.2.5 12 msec 8 msec 12 msec
  3  *  *  * 
  4  *  *  * 
  5  *  *  *
```

> [!warning] Signification des astérisques Les `* * *` indiquent que :
> 
> - Le routeur intermédiaire ne répond pas aux paquets ICMP
> - Il y a un timeout (pas de réponse dans le délai imparti)
> - Un firewall bloque les paquets ICMP Time Exceeded
> - La route est incomplète ou incorrecte

### Options avancées de traceroute

#### Spécifier l'interface source

```bash
Router# traceroute 10.3.3.10 source GigabitEthernet0/0
```

> [!tip] Utilité Permet de tester le routage depuis une interface spécifique, utile sur un routeur multi-interfaces.

#### Modifier le protocole (ICMP au lieu d'UDP)

```bash
Router# traceroute 10.3.3.10 probe 1
```

#### Mode interactif pour options avancées

```bash
Router# traceroute
Protocol [ip]: 
Target IP address: 10.3.3.10
Source address: 
Numeric display [n]: 
Timeout in seconds [3]: 
Probe count [3]: 
Minimum Time to Live [1]: 
Maximum Time to Live [30]: 
Port Number [33434]: 
Loose, Strict, Record, Timestamp, Verbose [none]: 
```

> [!example] Cas d'usage Si un traceroute montre que les paquets atteignent un routeur intermédiaire mais pas la destination :
> 
> - Le problème se situe après le dernier routeur qui répond
> - Vérifiez la route de retour (routage asymétrique possible)
> - Vérifiez les ACLs sur le routeur suivant

### Différence entre traceroute et tracert

|Commande|Système|Protocole par défaut|
|---|---|---|
|**traceroute**|Cisco IOS|UDP|
|**tracert**|Windows|ICMP|

> [!warning] Attention aux firewalls De nombreux firewalls bloquent le trafic UDP utilisé par traceroute, ce qui peut donner des résultats incomplets même si la connectivité existe.

---

## 🎯 Commande ping étendu {#commande-ping-etendu}

### Principe et différence avec ping simple

Le ping étendu (extended ping) offre des options avancées que le ping simple ne propose pas, notamment le choix de l'interface source et la taille des paquets.

#### Ping simple (limité)

```bash
Router# ping 10.2.2.1

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.2.2.1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/2/4 ms
```

|Symbole|Signification|
|---|---|
|**!**|Réponse ICMP Echo Reply reçue (succès)|
|**.**|Timeout (pas de réponse)|
|**U**|Destination unreachable (ICMP)|
|**Q**|Source quench (congestion)|
|**M**|Could not fragment|
|**?**|Type de paquet inconnu|
|**&**|Temps de vie du paquet expiré|

### Activation du mode étendu

```bash
Router# ping
Protocol [ip]: 
Target IP address: 10.3.3.10
Repeat count [5]: 
Datagram size [100]: 
Timeout in seconds [2]: 
Extended commands [n]: yes
Source address or interface: 10.1.1.1
Type of service [0]: 
Set DF bit in IP header? [no]: 
Validate reply data? [no]: 
Data pattern [0xABCD]: 
Loose, Strict, Record, Timestamp, Verbose [none]: 
Sweep range of sizes [n]: 
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.3.3.10, timeout is 2 seconds:
Packet sent with a source address of 10.1.1.1
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/2/4 ms
```

### Options principales du ping étendu

#### 1. Choisir l'adresse/interface source

```bash
Source address or interface: GigabitEthernet0/1
```

> [!info] Pourquoi c'est important Par défaut, le routeur utilise l'IP de l'interface de sortie comme source. Spécifier une autre interface permet de :
> 
> - Tester le routage depuis un segment spécifique
> - Simuler du trafic provenant d'un réseau particulier
> - Vérifier les routes de retour (reverse path)

#### 2. Modifier le nombre de paquets

```bash
Repeat count [5]: 100
```

> [!tip] Usage Augmenter le nombre de paquets pour des tests de stabilité ou pour détecter des pertes intermittentes.

#### 3. Modifier la taille des paquets

```bash
Datagram size [100]: 1500
```

> [!example] Cas d'usage
> 
> - Tester la MTU (Maximum Transmission Unit) du chemin réseau
> - Détecter des problèmes de fragmentation
> - Valeur typique pour Ethernet : 1500 octets

#### 4. Activer le bit DF (Don't Fragment)

```bash
Set DF bit in IP header? [no]: yes
```

> [!info] Utilité du bit DF
> 
> - Empêche la fragmentation des paquets
> - Permet de découvrir la MTU du chemin (Path MTU Discovery)
> - Si un routeur doit fragmenter mais que DF est activé, il renvoie ICMP "Fragmentation Needed"

#### 5. Sweep de tailles de paquets

```bash
Sweep range of sizes [n]: yes
Sweep min size [36]: 100
Sweep max size [18024]: 1500
Sweep interval [1]: 100
```

> [!tip] Test automatisé Envoie automatiquement des pings avec des tailles croissantes pour identifier la MTU maximale supportée.

### Exemples pratiques de ping étendu

#### Test de connectivité avec interface source spécifique

```bash
Router# ping 172.16.1.1 source Loopback0
```

> [!example] Scénario Vous voulez vérifier si le réseau distant peut joindre votre interface Loopback0 (souvent utilisée pour les connexions de gestion).

#### Test de MTU avec DF activé

```bash
Router# ping 10.5.5.5 size 1500 df-bit
```

Si le paquet est trop grand pour passer :

```bash
Sending 5, 1500-byte ICMP Echos to 10.5.5.5, timeout is 2 seconds:
.....
Success rate is 0 percent (0/5)
```

> [!warning] Interprétation Un taux de succès de 0% avec le bit DF peut indiquer :
> 
> - La MTU d'un lien intermédiaire est inférieure à 1500
> - Un routeur doit fragmenter mais ne peut pas (bit DF activé)
> - Solution : réduire la taille du paquet ou ajuster la MTU

### Synthèse des commandes ping

|Commande|Usage|
|---|---|
|`ping [IP]`|Ping simple depuis l'interface de sortie|
|`ping [IP] source [interface]`|Spécifier l'interface source|
|`ping [IP] repeat [count]`|Nombre de paquets à envoyer|
|`ping [IP] size [bytes]`|Taille des paquets ICMP|
|`ping [IP] df-bit`|Activer le bit Don't Fragment|
|`ping` (mode interactif)|Accès à toutes les options avancées|

---

## 🔧 Debug et troubleshooting {#debug-troubleshooting}

### Principe des commandes debug

Les commandes `debug` sont des outils puissants qui affichent en temps réel les événements internes du routeur. Elles sont essentielles pour diagnostiquer des problèmes complexes.

> [!warning] ATTENTION - Impact sur les performances Les commandes debug génèrent une charge CPU importante et peuvent ralentir voire bloquer le routeur en production. Utilisez-les avec précaution !

### Configuration préalable recommandée

#### Activer l'horodatage

```bash
Router# configure terminal
Router(config)# service timestamps debug datetime msec
Router(config)# service timestamps log datetime msec
Router(config)# end
```

> [!tip] Pourquoi l'horodatage Permet de corréler les événements debug avec d'autres logs et de mesurer les délais entre événements.

#### Limiter les messages au terminal

```bash
Router# terminal monitor
```

> [!info] terminal monitor Active l'affichage des messages debug et log sur la session Telnet/SSH en cours (par défaut, ils n'apparaissent que sur la console).

### Commandes debug essentielles pour le routage

#### 1. debug ip routing

Affiche en temps réel les ajouts et suppressions de routes dans la table de routage.

```bash
Router# debug ip routing
IP routing debugging is on
```

Sortie exemple :

```bash
*Mar  1 12:15:23.456: RT: add 10.5.5.0/24 via 10.1.1.2, static metric [1/0]
*Mar  1 12:15:45.789: RT: delete route to 10.5.5.0 via 10.1.1.2
*Mar  1 12:15:45.790: RT: no routes to 10.5.5.0
```

> [!example] Utilité
> 
> - Observer quand une route est installée ou supprimée
> - Vérifier si une route flottante prend le relais
> - Détecter des oscillations (flapping) de routes

#### 2. debug ip packet

Affiche les paquets IP traités par le routeur (dangereux en production).

```bash
Router# debug ip packet [access-list-number]
```

> [!warning] CRITIQUE Cette commande est extrêmement gourmande en CPU. Utilisez TOUJOURS avec une ACL pour filtrer les paquets à debugger.

Exemple avec ACL :

```bash
Router# configure terminal
Router(config)# access-list 100 permit ip host 10.1.1.5 host 10.3.3.10
Router(config)# exit
Router# debug ip packet 100
```

Sortie :

```bash
*Mar  1 12:20:10.123: IP: s=10.1.1.5 (GigabitEthernet0/0), d=10.3.3.10, len 100, forward
*Mar  1 12:20:10.456: IP: s=10.3.3.10 (GigabitEthernet0/1), d=10.1.1.5, len 100, forward
```

#### 3. debug ip icmp

Affiche les messages ICMP envoyés et reçus (ping, unreachable, etc.).

```bash
Router# debug ip icmp
ICMP packet debugging is on
```

Sortie :

```bash
*Mar  1 12:25:30.123: ICMP: echo reply sent, src 10.1.1.1, dst 10.1.1.5
*Mar  1 12:25:35.456: ICMP: dst (10.5.5.1) host unreachable sent to 10.1.1.5
```

> [!info] Interprétation
> 
> - `echo reply sent` : réponse à un ping
> - `host unreachable` : le routeur n'a pas de route vers la destination
> - `redirect` : le routeur informe qu'un meilleur chemin existe

### Commandes de désactivation des debugs

#### Désactiver un debug spécifique

```bash
Router# no debug ip routing
```

#### Désactiver TOUS les debugs

```bash
Router# undebug all
```

Ou la forme abrégée :

```bash
Router# u all
```

> [!tip] Réflexe de sécurité Toujours exécuter `u all` dès que vous avez terminé votre diagnostic pour éviter une surcharge prolongée du CPU.

### Vérifier les debugs actifs

```bash
Router# show debugging
```

Sortie :

```bash
Generic IP:
  IP routing debugging is on
  ICMP packet debugging is on
```

### Méthodologie de troubleshooting

#### Approche systématique

1. **Identifier le problème**
    
    - Quel est le symptôme exact ? (pas de ping, lenteur, perte de paquets)
    - Depuis quand le problème existe-t-il ?
2. **Vérifier la couche physique**
    
    ```bash
    Router# show ip interface brief
    ```
    
    - Toutes les interfaces sont-elles up/up ?
3. **Analyser la table de routage**
    
    ```bash
    Router# show ip route
    ```
    
    - La route vers la destination existe-t-elle ?
    - Quel est le next-hop ?
4. **Tester la connectivité de base**
    
    ```bash
    Router# ping [destination]
    Router# traceroute [destination]
    ```
    
5. **Vérifier la route de retour**
    
    - Le trafic peut arriver à destination mais ne pas revenir (routage asymétrique)
    - Utilisez ping étendu pour spécifier l'interface source
6. **Activer les debugs ciblés**
    
    - Seulement si les étapes précédentes n'ont pas identifié le problème
    - Toujours avec des ACLs pour limiter l'impact
7. **Consulter les logs**
    
    ```bash
    Router# show logging
    ```
    

#### Checklist des erreurs courantes

> [!warning] Erreurs fréquentes en routage statique
> 
> - ✓ Next-hop incorrect ou non joignable
> - ✓ Masque de sous-réseau erroné
> - ✓ Interface de sortie down
> - ✓ Route configurée sur le mauvais routeur
> - ✓ Route de retour manquante (asymétrie)
> - ✓ ACL bloquant le trafic
> - ✓ MTU incompatible sans fragmentation
> - ✓ Distance administrative incorrecte (route flottante qui ne s'active pas)

### Commandes de vérification complémentaires

#### Vérifier l'état des interfaces

```bash
Router# show ip interface brief

Interface              IP-Address      OK? Method Status                Protocol
GigabitEthernet0/0     10.1.1.1        YES manual up                    up
GigabitEthernet0/1     192.168.1.1     YES manual up                    up
GigabitEthernet0/2     unassigned      YES unset  administratively down down
```

#### Vérifier les statistiques d'interface

```bash
Router# show interfaces GigabitEthernet0/0
```

> [!tip] Indicateurs à surveiller
> 
> - Erreurs CRC (problème câblage/physique)
> - Input/output errors
> - Collisions excessives
> - Drops de paquets

#### Vérifier la configuration complète

```bash
Router# show running-config | section ip route
```

Affiche uniquement les lignes contenant "ip route" :

```bash
ip route 10.2.2.0 255.255.255.0 10.1.1.2
ip route 10.3.3.0 255.255.255.0 10.1.1.3
ip route 0.0.0.0 0.0.0.0 192.168.1.1
```

### Outils avancés de diagnostic

#### show ip cef (Cisco Express Forwarding)

```bash
Router# show ip cef 10.3.3.10
```

> [!info] CEF Montre comment les paquets sont réellement forwardés dans le plan de données (data plane), au-delà de la simple table de routage.

#### show processes cpu

```bash
Router# show processes cpu sorted
```

> [!warning] Surveillance CPU Si le CPU est saturé à cause des debugs, les paquets peuvent être droppés et fausser le diagnostic.

---

## 🎓 Résumé des bonnes pratiques

|Pratique|Recommandation|
|---|---|
|**Vérification initiale**|Toujours commencer par `show ip route`|
|**Tests de base**|Utiliser ping et traceroute avant les debugs|
|**Debugs**|Les activer uniquement en cas de besoin, avec ACL|
|**Performance**|Désactiver immédiatement les debugs après usage (`u all`)|
|**Documentation**|Noter les résultats des tests pour analyse ultérieure|
|**Route de retour**|Toujours vérifier le routage dans les deux sens|

> [!tip] Astuce professionnelle Créez des scripts ou des alias pour les séquences de commandes fréquentes. Par exemple, un alias pour désactiver tous les debugs et vérifier le CPU en une seule commande.

---

_Fin du cours - Vérification et dépannage du routage statique_