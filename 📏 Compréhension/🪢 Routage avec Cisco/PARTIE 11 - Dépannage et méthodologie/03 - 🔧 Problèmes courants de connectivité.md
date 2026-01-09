

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

## 🚨 Problèmes courants de connectivité

### Interface down/down vs up/down

#### 📊 Comprendre les états d'interface

Les interfaces réseau peuvent se trouver dans différents états qui indiquent le niveau de connectivité. Ces états sont essentiels pour diagnostiquer rapidement l'origine d'un problème.

> [!info] Les deux états d'une interface Chaque interface Cisco possède deux états :
> 
> - **État physique (Layer 1)** : État du câble et du matériel
> - **État protocolaire (Layer 2)** : État du protocole de liaison de données

#### 🔍 Les différents états possibles

|État|Signification|Causes principales|
|---|---|---|
|**up/up**|✅ Interface opérationnelle|Tout fonctionne correctement|
|**down/down**|❌ Problème physique|Câble débranché, interface shutdown, problème matériel|
|**up/down**|⚠️ Problème de protocole|Désaccord d'encapsulation, keepalive manquants, problème de configuration|
|**administratively down/down**|🔒 Interface désactivée|Commande `shutdown` active|

#### 🔧 Diagnostic et résolution

**État down/down :**

```bash
# Vérifier l'état de l'interface
Router# show interfaces gigabitEthernet 0/0

GigabitEthernet0/0 is down, line protocol is down
  Hardware is iGbE, address is xxxx.xxxx.xxxx
  MTU 1500 bytes, BW 1000000 Kbit/sec
```

> [!example] Causes et solutions pour down/down
> 
> **Causes possibles :**
> 
> - Câble physiquement déconnecté ou défectueux
> - Interface administrativement désactivée (`shutdown`)
> - Port switch désactivé côté distant
> - Problème matériel (carte réseau défectueuse)
> - Mauvais type de câble (droit vs croisé)
> 
> **Actions de dépannage :**
> 
> 1. Vérifier la connexion physique du câble
> 2. Tester avec un câble différent
> 3. Activer l'interface avec `no shutdown`
> 4. Vérifier les LEDs du port
> 5. Tester le port avec un autre équipement

```bash
# Solution typique pour down/down
Router(config)# interface gigabitEthernet 0/0
Router(config-if)# no shutdown
Router(config-if)# end

# Vérifier le changement d'état
Router# show interfaces gigabitEthernet 0/0 | include line protocol
GigabitEthernet0/0 is up, line protocol is up
```

**État up/down :**

```bash
Router# show interfaces serial 0/0/0

Serial0/0/0 is up, line protocol is down
  Hardware is GT96K Serial
  Internet address is 10.1.1.1/30
  MTU 1500 bytes, BW 1544 Kbit/sec
```

> [!example] Causes et solutions pour up/down
> 
> **Causes possibles :**
> 
> - Désaccord d'encapsulation (HDLC vs PPP)
> - Problème de clock rate sur liaison série
> - Keepalive non reçus
> - Mauvaise configuration du protocole Layer 2
> - Problème d'authentification (PPP/CHAP)
> 
> **Actions de dépannage :**
> 
> 1. Vérifier l'encapsulation des deux côtés
> 2. Vérifier le clock rate (interface DCE)
> 3. Vérifier les keepalive
> 4. Contrôler la configuration du protocole

```bash
# Vérifier l'encapsulation
Router# show interfaces serial 0/0/0 | include Encapsulation
  Encapsulation HDLC

# Changer l'encapsulation si nécessaire
Router(config)# interface serial 0/0/0
Router(config-if)# encapsulation ppp
Router(config-if)# end

# Vérifier le clock rate (pour interface DCE)
Router# show controllers serial 0/0/0
Interface Serial0/0/0
Hardware is GT96K
DCE V.35, clock rate 64000

# Configurer le clock rate si nécessaire
Router(config)# interface serial 0/0/0
Router(config-if)# clock rate 64000
```

> [!warning] Pièges courants
> 
> - Ne pas vérifier les deux extrémités de la liaison
> - Oublier le `no shutdown` après une modification
> - Confondre interface DCE et DTE sur liaisons série
> - Ne pas attendre suffisamment après une modification (keepalive timers)

> [!tip] Astuces de dépannage
> 
> - Utilisez `show interfaces status` pour une vue d'ensemble rapide
> - La commande `show ip interface brief` est votre meilleur ami pour un diagnostic rapide
> - Les logs système (`show logging`) peuvent révéler des changements d'état automatiques
> - Pour les liaisons série, toujours identifier quelle extrémité est DCE avec `show controllers`

---

### Problèmes d'adressage IP

#### 📊 Types de problèmes d'adressage

L'adressage IP incorrect est l'une des causes les plus fréquentes de problèmes de connectivité réseau. Une erreur à ce niveau empêche toute communication.

> [!info] Pourquoi l'adressage IP est critique L'adresse IP est l'identifiant unique de chaque interface dans un réseau. Une mauvaise configuration peut :
> 
> - Empêcher la communication entre équipements
> - Créer des conflits d'adresses
> - Causer des problèmes de routage
> - Isoler des segments du réseau

#### 🔍 Problèmes courants d'adressage

**1. Mauvais masque de sous-réseau**

```bash
# Configuration incorrecte
Router(config)# interface gigabitEthernet 0/0
Router(config-if)# ip address 192.168.1.1 255.255.0.0
# Devrait être 255.255.255.0 pour un réseau /24

# Vérification
Router# show ip interface brief
Interface              IP-Address      OK? Method Status                Protocol
GigabitEthernet0/0     192.168.1.1     YES manual up                    up

# Vérification détaillée du masque
Router# show ip interface gigabitEthernet 0/0 | include Internet address
  Internet address is 192.168.1.1/16
```

> [!example] Impact d'un mauvais masque
> 
> **Scénario :**
> 
> - Interface configurée avec 192.168.1.1/16 au lieu de /24
> - L'équipement considère tout le réseau 192.168.0.0/16 comme local
> - Les paquets destinés à 192.168.2.x ne seront pas routés mais cherchés localement via ARP
> 
> **Symptômes :**
> 
> - Impossible de communiquer avec certains sous-réseaux
> - Table ARP incomplète
> - Timeout sur certaines destinations

```bash
# Correction du masque
Router(config)# interface gigabitEthernet 0/0
Router(config-if)# ip address 192.168.1.1 255.255.255.0
Router(config-if)# end

# Vérification après correction
Router# show ip interface gigabitEthernet 0/0 | include Internet address
  Internet address is 192.168.1.1/24
```

**2. Adresses IP sur des sous-réseaux différents**

```bash
# Routeur A
RouterA(config)# interface gigabitEthernet 0/0
RouterA(config-if)# ip address 10.1.1.1 255.255.255.0

# Routeur B (ERREUR : sous-réseau différent)
RouterB(config)# interface gigabitEthernet 0/0
RouterB(config-if)# ip address 10.1.2.1 255.255.255.0
```

> [!warning] Conséquence d'un désaccord de sous-réseau
> 
> - Les équipements ne peuvent pas communiquer directement
> - Pas de résolution ARP possible
> - Les pings échouent même si la couche physique est opérationnelle
> - Chaque équipement pense que l'autre est sur un réseau distant

```bash
# Diagnostic du problème
RouterA# ping 10.1.2.1
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.1.2.1, timeout is 2 seconds:
.....
Success rate is 0 percent (0/5)

# Vérifier la table ARP
RouterA# show arp
Protocol  Address          Age (min)  Hardware Addr   Type   Interface
Internet  10.1.1.1                -   xxxx.xxxx.xxxx  ARPA   GigabitEthernet0/0
# Aucune entrée pour 10.1.2.1

# Solution : mettre les deux interfaces sur le même sous-réseau
RouterB(config)# interface gigabitEthernet 0/0
RouterB(config-if)# ip address 10.1.1.2 255.255.255.0
```

**3. Conflit d'adresses IP**

```bash
# Deux interfaces avec la même adresse
# Routeur A
RouterA(config-if)# ip address 192.168.10.1 255.255.255.0

# Routeur B (ERREUR : même adresse)
RouterB(config-if)# ip address 192.168.10.1 255.255.255.0
```

> [!example] Symptômes d'un conflit d'adresse
> 
> **Comportements observés :**
> 
> - Messages système de conflit d'adresse
> - Connectivité intermittente (fonctionne parfois, échoue parfois)
> - Problèmes dans la table ARP (adresse MAC qui change)
> - Réponses ping incohérentes
> 
> **Messages logs typiques :**
> 
> ```
> %IP-4-DUPADDR: Duplicate address 192.168.10.1 on GigabitEthernet0/0, 
> sourced by xxxx.xxxx.xxxx
> ```

```bash
# Vérifier les conflits via ARP
Router# show arp
Protocol  Address          Age (min)  Hardware Addr   Type   Interface
Internet  192.168.10.1            0   aabb.cc00.0100  ARPA   GigabitEthernet0/0
Internet  192.168.10.1            0   aabb.cc00.0200  ARPA   GigabitEthernet0/0
# Deux MAC différentes pour la même IP = CONFLIT

# Solution : changer l'adresse sur un des équipements
RouterB(config)# interface gigabitEthernet 0/0
RouterB(config-if)# ip address 192.168.10.2 255.255.255.0
```

**4. Adresse IP dans le mauvais réseau**

```bash
# Configuration erronée
Router(config)# interface gigabitEthernet 0/0
Router(config-if)# ip address 172.16.1.1 255.255.255.0
# Devrait être 192.168.1.1 selon le plan d'adressage

# Diagnostic : pas de connectivité avec le reste du réseau
Router# ping 192.168.1.254
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.1.254, timeout is 2 seconds:
.....
Success rate is 0 percent (0/5)
```

#### 🛠️ Méthodologie de dépannage d'adressage

```bash
# 1. Vérifier la configuration IP de base
Router# show ip interface brief
Interface              IP-Address      OK? Method Status                Protocol
GigabitEthernet0/0     192.168.1.1     YES manual up                    up
GigabitEthernet0/1     10.1.1.1        YES manual up                    up

# 2. Vérifier le détail d'une interface spécifique
Router# show ip interface gigabitEthernet 0/0
GigabitEthernet0/0 is up, line protocol is up
  Internet address is 192.168.1.1/24
  Broadcast address is 255.255.255.255
  Address determined by manual configuration
  MTU is 1500 bytes

# 3. Vérifier la table ARP pour les voisins directs
Router# show arp
Protocol  Address          Age (min)  Hardware Addr   Type   Interface
Internet  192.168.1.1             -   xxxx.xxxx.0001  ARPA   GigabitEthernet0/0
Internet  192.168.1.2             5   xxxx.xxxx.0002  ARPA   GigabitEthernet0/0

# 4. Tester la connectivité locale
Router# ping 192.168.1.2
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.1.2, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/2/4 ms

# 5. Vérifier la configuration running
Router# show running-config interface gigabitEthernet 0/0
Building configuration...

Current configuration : 120 bytes
!
interface GigabitEthernet0/0
 ip address 192.168.1.1 255.255.255.0
 duplex auto
 speed auto
end
```

> [!tip] Commandes essentielles pour le diagnostic d'adressage
> 
> **Vue d'ensemble rapide :**
> 
> - `show ip interface brief` : État et adresses de toutes les interfaces
> - `show interfaces` : Détails complets sur une interface
> 
> **Vérification détaillée :**
> 
> - `show ip interface [nom]` : Configuration IP détaillée
> - `show running-config interface [nom]` : Configuration exacte appliquée
> 
> **Diagnostic de connectivité locale :**
> 
> - `show arp` : Table de résolution d'adresses
> - `ping [ip]` : Test de connectivité basique
> - `traceroute [ip]` : Trace du chemin réseau

> [!warning] Erreurs fréquentes lors du dépannage
> 
> - Oublier de vérifier le masque de sous-réseau complet
> - Ne pas valider que les équipements sont sur le même sous-réseau
> - Confondre adresse réseau et adresse broadcast
> - Ne pas effacer la table ARP après correction (`clear arp`)
> - Oublier le `no shutdown` après modification d'adresse

---

### Problèmes de routage

#### 📊 Comprendre les problèmes de routage

Les problèmes de routage surviennent lorsque les paquets ne peuvent pas atteindre leur destination à cause d'informations de routage incorrectes ou manquantes. Contrairement aux problèmes d'adressage local, ces problèmes affectent la communication entre réseaux différents.

> [!info] Différence entre problème local et problème de routage
> 
> - **Problème local** : Impossible de communiquer avec un équipement du même sous-réseau
> - **Problème de routage** : Impossible de communiquer avec un équipement d'un autre sous-réseau
> 
> Le routage n'intervient que pour la communication entre réseaux différents.

#### 🔍 Types de problèmes de routage

**1. Route manquante dans la table de routage**

```bash
# Vérifier la table de routage
Router# show ip route
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area

Gateway of last resort is not set

      10.0.0.0/8 is variably subnetted, 2 subnets, 2 masks
C        10.1.1.0/24 is directly connected, GigabitEthernet0/0
L        10.1.1.1/32 is directly connected, GigabitEthernet0/0
```

> [!example] Symptômes d'une route manquante
> 
> **Scénario :**
> 
> - Tentative de ping vers 192.168.5.10
> - Pas de route vers 192.168.5.0/24 dans la table
> 
> **Résultat :**
> 
> ```bash
> Router# ping 192.168.5.10
> % Unrecognized host or address, or protocol not running.
> ```
> 
> **Diagnostic :**
> 
> ```bash
> Router# show ip route 192.168.5.10
> % Network not in table
> ```

```bash
# Solution 1 : Ajouter une route statique
Router(config)# ip route 192.168.5.0 255.255.255.0 10.1.1.2
# Destination: 192.168.5.0/24, via next-hop: 10.1.1.2

# Solution 2 : Ajouter une route par défaut
Router(config)# ip route 0.0.0.0 0.0.0.0 10.1.1.2
# Toutes les destinations inconnues via 10.1.1.2

# Vérification après ajout
Router# show ip route
Gateway of last resort is 10.1.1.2 to network 0.0.0.0

S*    0.0.0.0/0 [1/0] via 10.1.1.2
      10.0.0.0/8 is variably subnetted, 2 subnets, 2 masks
C        10.1.1.0/24 is directly connected, GigabitEthernet0/0
L        10.1.1.1/32 is directly connected, GigabitEthernet0/0
S        192.168.5.0/24 [1/0] via 10.1.1.2
```

**2. Route incorrecte (mauvais next-hop)**

```bash
# Configuration avec erreur
Router(config)# ip route 172.16.0.0 255.255.0.0 10.1.1.99
# Next-hop 10.1.1.99 n'existe pas ou n'est pas joignable

# Vérification de la route
Router# show ip route 172.16.0.0
Routing entry for 172.16.0.0/16
  Known via "static", distance 1, metric 0
  Routing Descriptor Blocks:
  * 10.1.1.99
      Route metric is 0, traffic share count is 1
```

> [!warning] Reconnaître un next-hop invalide
> 
> **Signes d'un problème :**
> 
> ```bash
> Router# ping 172.16.1.1
> Type escape sequence to abort.
> Sending 5, 100-byte ICMP Echos to 172.16.1.1, timeout is 2 seconds:
> .....
> Success rate is 0 percent (0/5)
> 
> # Mais le ping vers le next-hop échoue aussi
> Router# ping 10.1.1.99
> .....
> Success rate is 0 percent (0/5)
> ```
> 
> **Diagnostic :**
> 
> - Si le next-hop n'est pas joignable, la route est inutilisable
> - Vérifier la table ARP pour confirmer
> 
> ```bash
> Router# show arp | include 10.1.1.99
> # Aucune entrée = next-hop inaccessible
> ```

```bash
# Correction : utiliser le bon next-hop
Router(config)# no ip route 172.16.0.0 255.255.0.0 10.1.1.99
Router(config)# ip route 172.16.0.0 255.255.0.0 10.1.1.2

# Vérification après correction
Router# ping 10.1.1.2
!!!!!
Success rate is 100 percent (5/5)

Router# ping 172.16.1.1
!!!!!
Success rate is 100 percent (5/5)
```

**3. Problème de route récursive**

```bash
# Configuration problématique
Router(config)# ip route 192.168.10.0 255.255.255.0 172.16.1.1
# Mais 172.16.1.1 lui-même nécessite une route

# Diagnostic
Router# show ip route 192.168.10.0
% Network not in table

Router# show ip route 172.16.1.1
% Network not in table
# Le next-hop n'est pas dans la table = route invalide
```

> [!example] Comprendre la récursivité de routage
> 
> **Fonctionnement :**
> 
> 1. Le routeur cherche comment atteindre 192.168.10.0
> 2. Trouve qu'il faut passer par 172.16.1.1
> 3. Cherche comment atteindre 172.16.1.1
> 4. Si aucune route vers 172.16.1.1 n'existe → échec de résolution récursive
> 
> **Solution :** Ajouter d'abord la route vers le next-hop

```bash
# Correction : route vers le next-hop d'abord
Router(config)# ip route 172.16.1.0 255.255.255.0 10.1.1.2
Router(config)# ip route 192.168.10.0 255.255.255.0 172.16.1.1

# Vérification de la récursivité
Router# show ip route 192.168.10.0
Routing entry for 192.168.10.0/24
  Known via "static", distance 1, metric 0
  Routing Descriptor Blocks:
  * 172.16.1.1
      Route metric is 0, traffic share count is 1
      # Route valide car 172.16.1.1 est maintenant joignable via 10.1.1.2
```

**4. Distance administrative et préférence de route**

```bash
# Plusieurs routes vers la même destination
Router# show ip route 10.5.5.0
Routing entry for 10.5.5.0/24
  Known via "static", distance 1, metric 0
  Known via "rip", distance 120, metric 2
  Routing Descriptor Blocks:
  * 10.1.1.2
      Route metric is 0, traffic share count is 1
```

> [!info] Distance administrative La distance administrative détermine quelle route est préférée quand plusieurs sources fournissent des routes vers la même destination.
> 
> **Valeurs par défaut :**
> 
> - Route connectée : 0
> - Route statique : 1
> - EIGRP : 90
> - OSPF : 110
> - RIP : 120
> 
> **Plus la valeur est basse, plus la route est préférée.**

```bash
# Forcer une préférence différente avec une distance administrative custom
Router(config)# ip route 10.5.5.0 255.255.255.0 10.1.1.3 150
# Distance administrative de 150 (moins préférée qu'une route RIP)

# Vérification
Router# show ip route 10.5.5.0
Routing entry for 10.5.5.0/24
  Known via "rip", distance 120, metric 2
  Routing Descriptor Blocks:
  * 10.1.1.4
      Route metric is 2, traffic share count is 1
      # La route RIP (AD 120) est préférée à la statique (AD 150)
```

#### 🛠️ Méthodologie de dépannage de routage

```bash
# 1. Vérifier la connectivité de base
Router# ping 192.168.50.10

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.50.10, timeout is 2 seconds:
.....
Success rate is 0 percent (0/5)

# 2. Vérifier si une route existe
Router# show ip route 192.168.50.10
% Network not in table
# Aucune route = problème identifié

# 3. Vérifier toutes les routes disponibles
Router# show ip route
Gateway of last resort is not set

      10.0.0.0/8 is variably subnetted, 2 subnets, 2 masks
C        10.1.1.0/24 is directly connected, GigabitEthernet0/0
L        10.1.1.1/32 is directly connected, GigabitEthernet0/0

# 4. Vérifier les routes statiques configurées
Router# show ip route static
# Aucune sortie = pas de routes statiques

# 5. Vérifier les protocoles de routage actifs
Router# show ip protocols
# Permet de voir si RIP, OSPF, EIGRP sont configurés

# 6. Ajouter la route manquante
Router(config)# ip route 192.168.50.0 255.255.255.0 10.1.1.2

# 7. Vérifier que la route est installée
Router# show ip route | include 192.168.50
S        192.168.50.0/24 [1/0] via 10.1.1.2

# 8. Tester à nouveau la connectivité
Router# ping 192.168.50.10
!!!!!
Success rate is 100 percent (5/5)

# 9. Utiliser traceroute pour voir le chemin
Router# traceroute 192.168.50.10
Type escape sequence to abort.
Tracing the route to 192.168.50.10
  1 10.1.1.2 4 msec 4 msec 4 msec
  2 192.168.50.10 8 msec 8 msec 8 msec
```

> [!tip] Commandes essentielles pour le diagnostic de routage
> 
> **Analyse de la table de routage :**
> 
> - `show ip route` : Table de routage complète
> - `show ip route [destination]` : Route spécifique vers une destination
> - `show ip route static` : Uniquement les routes statiques
> - `show ip route connected` : Uniquement les réseaux directement connectés
> 
> **Tests de connectivité :**
> 
> - `ping [destination]` : Test de connectivité basique
> - `traceroute [destination]` : Affiche le chemin emprunté
> - `ping [destination] source [interface]` : Ping depuis une interface spécifique
> 
> **Diagnostic avancé :**
> 
> - `show ip protocols` : Protocoles de routage actifs
> - `show ip cef [destination]` : Vérifier le CEF (Cisco Express Forwarding)

> [!warning] Erreurs fréquentes en dépannage de routage
> 
> - Ne pas vérifier que le next-hop est joignable avant de l'utiliser
> - Oublier de configurer une route de retour (routage asymétrique)
> - Utiliser un masque incorrect dans la route statique
> - Configurer des routes contradictoires ou qui se chevauchent
> - Ne pas considérer la distance administrative lors de routes multiples
> - Oublier le `write memory` ou `copy running-config startup-config` après modification

---

### Problèmes d'encapsulation

#### 📊 Comprendre l'encapsulation

L'encapsulation est le processus d'ajout d'informations de protocole Layer 2 aux données. Sur les liaisons WAN (Wide Area Network), notamment les liaisons série, le type d'encapsulation doit être identique des deux côtés de la liaison pour que la communication fonctionne.

> [!info] Pourquoi l'encapsulation est importante
> 
> - L'encapsulation définit le format des trames au niveau Layer 2
> - Un désaccord d'encapsulation empêche les équipements de comprendre les trames reçues
> - Ce problème se manifeste par un état **up/down** de l'interface
> - Principalement rencontré sur les liaisons WAN série

#### 🔍 Types d'encapsulation courants

|Encapsulation|Description|Usage typique|
|---|---|---|
|**HDLC**|High-Level Data Link Control (propriétaire Cisco)|Liaison série point-à-point Cisco-Cisco|
|**PPP**|Point-to-Point Protocol (standard)|Liaison série multi-vendeur, support authentification|
|**Frame Relay**|Protocole WAN à commutation|Anciennes liaisons WAN (obsolète)|
|**Ethernet**|Standard LAN|Réseaux locaux|

#### 🔧 Diagnostic et résolution

**Symptôme : Interface up/down**

```bash
Router# show interfaces serial 0/0/0

Serial0/0/0 is up, line protocol is down
  Hardware is GT96K Serial
  Internet address is 10.1.1.1/30
  MTU 1500 bytes, BW 1544 Kbit/sec, DLY 20000 usec,
     reliability 255/255, txload 1/255, rxload 1/255
  Encapsulation HDLC, loopback not set
```

> [!example] Diagnostic du problème d'encapsulation
> 
> **Indicateurs d'un désaccord d'encapsulation :**
> 
> - Interface physiquement active (**up**)
> - Protocole inactif (**down**)
> - Pas de keepalive reçus
> - Interface connectée mais aucune communication possible

**Vérification de l'encapsulation des deux côtés :**

```bash
# Sur le Routeur A
RouterA# show interfaces serial 0/0/0 | include Encapsulation
  Encapsulation HDLC, loopback not set

# Sur le Routeur B
RouterB# show interfaces serial 0/0/0 | include Encapsulation
  Encapsulation PPP, LCP Closed
# PROBLÈME : HDLC d'un côté, PPP de l'autre

# Vérification détaillée de l'état
RouterA# show interfaces serial 0/0/0
Serial0/0/0 is up, line protocol is down
  Encapsulation HDLC
  Last input never, output never, output hang never
  Last clearing of "show interface" counters never
  Input queue: 0/75/0/0 (size/max/drops/flushes); Total output drops: 0
  # Remarquez : "Last input never" = aucune trame valide reçue

RouterB# show interfaces serial 0/0/0
Serial0/0/0 is up, line protocol is down
  Encapsulation PPP, LCP Closed
  # LCP Closed = PPP ne peut pas établir de connexion
```

**Solution : Uniformiser l'encapsulation**

```bash
# Option 1 : Changer le Routeur B vers HDLC (défaut Cisco)
RouterB(config)# interface serial 0/0/0
RouterB(config-if)# encapsulation hdlc
RouterB(config-if)# end

# Vérification immédiate
RouterB# show interfaces serial 0/0/0 | include line protocol
Serial0/0/0 is up, line protocol is up
# Interface maintenant opérationnelle !

# Option 2 : Changer le Routeur A vers PPP (standard multi-vendeur)
RouterA(config)# interface serial 0/0/0
RouterA(config-if)# encapsulation ppp
RouterA(config-if)# end

# Vérification
RouterA# show interfaces serial 0/0/0
Serial0/0/0 is up, line protocol is up
  Encapsulation PPP, LCP Open
  Open: IPCP, CDPCP
```

> [!info] HDLC vs PPP - Quand utiliser quoi ?
> 
> **HDLC (High-Level Data Link Control) :**
> 
> - Encapsulation par défaut sur les interfaces série Cisco
> - Propriétaire Cisco (version étendue du HDLC standard)
> - Léger et performant
> - **Utiliser quand :** Liaison point-à-point entre deux équipements Cisco
> 
> **PPP (Point-to-Point Protocol) :**
> 
> - Standard ouvert (RFC 1661)
> - Supporte l'authentification (PAP, CHAP)
> - Supporte la compression et le multilink
> - **Utiliser quand :** Liaison avec équipements non-Cisco ou besoin d'authentification

**Configuration PPP avec authentification :**

```bash
# Routeur A - Configuration PPP avec CHAP
RouterA(config)# username RouterB password cisco123
RouterA(config)# interface serial 0/0/0
RouterA(config-if)# encapsulation ppp
RouterA(config-if)# ppp authentication chap
RouterA(config-if)# end

# Routeur B - Configuration PPP avec CHAP
RouterB(config)# username RouterA password cisco123
RouterB(config)# interface serial 0/0/0
RouterB(config-if)# encapsulation ppp
RouterB(config-if)# ppp authentication chap
RouterB(config-if)# end

# Vérification de l'authentification
RouterA# show ppp interface serial 0/0/0
PPP Serial0/0/0
 LCP: Open
 CHAP: SUCCESS
 IPCP: Open
   IP address: 10.1.1.1/30
```

> [!warning] Erreurs courantes avec l'authentification PPP
> 
> - **Mots de passe différents :** L'authentification échoue, interface reste down/down
> - **Username incorrect :** Le username doit correspondre au hostname de l'autre routeur
> - **Casse du mot de passe :** Les mots de passe sont sensibles à la casse
> 
> **Diagnostic d'un échec d'authentification :**
> 
> ```bash
> RouterA# debug ppp authentication
> PPP Serial0/0/0: CHAP authentication failed
> # Vérifier les usernames et passwords
> ```

**Problème de keepalive avec l'encapsulation :**

```bash
# Désaccord de keepalive interval
RouterA# show interfaces serial 0/0/0 | include Keepalive
  Keepalive set (10 sec)

RouterB# show interfaces serial 0/0/0 | include Keepalive
  Keepalive not set
```

> [!example] Impact des keepalive
> 
> Les keepalive sont des trames envoyées périodiquement pour vérifier que la liaison est active.
> 
> **Problème potentiel :**
> 
> - Si un côté envoie des keepalive et l'autre ne répond pas
> - Le côté émetteur peut déclarer la liaison down après expiration du timer
> - Crée une instabilité de liaison (flapping)

```bash
# Désactiver les keepalive (solution temporaire pour test)
Router(config)# interface serial 0/0/0
Router(config-if)# no keepalive
Router(config-if)# end

# Ou ajuster l'intervalle pour correspondre
Router(config)# interface serial 0/0/0
Router(config-if)# keepalive 10
Router(config-if)# end

# Vérification
Router# show interfaces serial 0/0/0 | include Keepalive
  Keepalive set (10 sec)
```

#### 🛠️ Méthodologie de dépannage d'encapsulation

```bash
# 1. Identifier le problème : interface up/down
Router# show ip interface brief
Interface              IP-Address      OK? Method Status                Protocol
Serial0/0/0            10.1.1.1        YES manual up                    down
# up/down = suspect un problème Layer 2

# 2. Vérifier le type d'encapsulation local
Router# show interfaces serial 0/0/0 | include Encapsulation
  Encapsulation HDLC

# 3. Si possible, vérifier l'encapsulation du routeur distant
# (via console, SSH, ou documentation)

# 4. Vérifier les statistiques d'erreurs
Router# show interfaces serial 0/0/0
Serial0/0/0 is up, line protocol is down
  Last input never, output 00:00:05
  # "Last input never" = aucune trame comprise
  0 packets input, 0 bytes
  Received 0 broadcasts, 0 runts, 0 giants, 0 throttles
  0 input errors, 0 CRC, 0 frame, 0 overrun, 0 ignored, 0 abort

# 5. Changer l'encapsulation pour correspondre
Router(config)# interface serial 0/0/0
Router(config-if)# encapsulation ppp
Router(config-if)# end

# 6. Vérifier l'amélioration immédiate
Router# show interfaces serial 0/0/0 | include protocol
Serial0/0/0 is up, line protocol is up

# 7. Tester la connectivité
Router# ping 10.1.1.2
!!!!!
Success rate is 100 percent (5/5)

# 8. Vérifier le détail PPP si applicable
Router# show ppp interface serial 0/0/0
PPP Serial0/0/0
 LCP: Open
 IPCP: Open
```

> [!tip] Commandes clés pour diagnostiquer l'encapsulation
> 
> **Vérification de base :**
> 
> - `show interfaces [nom] | include Encapsulation` : Type d'encapsulation
> - `show interfaces [nom]` : État complet et statistiques
> - `show controllers [nom]` : Vérifier DCE/DTE et clock rate
> 
> **Diagnostic PPP spécifique :**
> 
> - `show ppp interface [nom]` : État des protocoles PPP
> - `debug ppp negotiation` : Voir la négociation PPP en temps réel
> - `debug ppp authentication` : Déboguer l'authentification
> - `debug ppp packet` : Voir les paquets PPP (très verbeux)
> 
> **Attention avec debug :**
> 
> ```bash
> # Toujours désactiver le debug après utilisation
> Router# undebug all
> # Ou
> Router# no debug all
> ```

> [!warning] Pièges courants avec l'encapsulation
> 
> - Oublier que HDLC Cisco est propriétaire (incompatible avec d'autres vendeurs)
> - Configurer l'authentification PPP sans créer les usernames correspondants
> - Ne pas vérifier les deux côtés de la liaison avant de faire des changements
> - Laisser les commandes `debug` actives en production (impact performance)
> - Confondre problème d'encapsulation avec problème de clock rate (DCE/DTE)

---

### Duplex mismatch

#### 📊 Comprendre le duplex mismatch

Le duplex mismatch est un problème de configuration Layer 1/Layer 2 où les deux extrémités d'une liaison Ethernet ne s'accordent pas sur le mode de communication (half-duplex vs full-duplex). Ce problème est insidieux car la liaison peut sembler fonctionner tout en ayant des performances très dégradées.

> [!info] Duplex - Définition
> 
> - **Full-duplex :** Communication bidirectionnelle simultanée (émission et réception en même temps)
> - **Half-duplex :** Communication bidirectionnelle mais non simultanée (soit émission, soit réception)
> - **Auto :** Négociation automatique du mode entre les deux équipements

#### 🔍 Symptômes d'un duplex mismatch

> [!warning] Signes caractéristiques
> 
> - **Performance médiocre :** Débit très inférieur à la normale (souvent 30-50% de perte)
> - **Latence élevée :** Temps de réponse irréguliers et élevés
> - **Collisions excessives :** Nombreuses collisions sur l'interface full-duplex
> - **Late collisions :** Collisions détectées après le début de transmission
> - **Retransmissions fréquentes :** Paquets retransmis à répétition
> - **Interface opérationnelle :** up/up mais performances dégradées

**Vérification des symptômes :**

```bash
# Vérifier l'état de l'interface
Router# show interfaces gigabitEthernet 0/0

GigabitEthernet0/0 is up, line protocol is up
  Hardware is iGbE, address is 0000.1111.2222
  Full-duplex, 1000Mb/s, media type is RJ45
  # Interface en full-duplex
  
  5 minute input rate 50000 bits/sec, 40 packets/sec
  5 minute output rate 45000 bits/sec, 35 packets/sec
     1000000 packets input, 125000000 bytes
     10500 input errors, 0 CRC, 0 frame, 0 overrun, 0 ignored
     8000 late collisions, 2000 deferred
     # ATTENTION : late collisions = indicateur de duplex mismatch !
     900000 packets output, 112500000 bytes
     0 output errors, 5000 collisions, 0 interface resets
```

> [!example] Scénario typique de duplex mismatch
> 
> **Configuration problématique :**
> 
> - Routeur A : Full-duplex (configuré manuellement)
> - Switch B : Half-duplex (auto-négociation échouée ou forcée)
> 
> **Ce qui se passe :**
> 
> 1. Le routeur A envoie et reçoit simultanément (full-duplex)
> 2. Le switch B détecte une "collision" quand il reçoit pendant qu'il envoie
> 3. Le switch B attend avant de retransmettre (algorithme CSMA/CD)
> 4. Résultat : performances catastrophiques mais liaison up/up

#### 🔧 Diagnostic et résolution

**Étape 1 : Vérifier la configuration actuelle**

```bash
# Sur le routeur
Router# show interfaces gigabitEthernet 0/0 | include duplex
  Full-duplex, 1000Mb/s, media type is RJ45

# Vérifier si c'est configuré manuellement ou auto-négocié
Router# show running-config interface gigabitEthernet 0/0
interface GigabitEthernet0/0
 ip address 192.168.1.1 255.255.255.0
 duplex full
 speed 1000
 # Configuré manuellement !

# Sur le switch (si accessible)
Switch# show interfaces gigabitEthernet 0/1
GigabitEthernet0/1 is up, line protocol is up
  Hardware is Gigabit Ethernet, address is 1111.2222.3333
  Configured speed auto, actual 1000Mbit/s
  Configured duplex auto, actual half-duplex
  # PROBLÈME IDENTIFIÉ : half-duplex sur le switch !
```

**Étape 2 : Analyser les compteurs d'erreurs**

```bash
# Vérifier les collisions et erreurs
Router# show interfaces gigabitEthernet 0/0 | include collision|error
     10500 input errors, 0 CRC, 0 frame, 0 overrun, 0 ignored
     8000 late collisions, 2000 deferred
     # Late collisions élevées = duplex mismatch très probable
     0 output errors, 5000 collisions, 0 interface resets

# Sur une liaison correctement configurée, ces valeurs devraient être à 0 ou très faibles
```

> [!info] Comprendre les types de collisions
> 
> **Collisions normales (half-duplex) :**
> 
> - Détectées au début de la transmission
> - Normales sur des liaisons half-duplex partagées
> - Algorithme CSMA/CD se déclenche
> 
> **Late collisions (indicateur de problème) :**
> 
> - Détectées après le début de la transmission (après 64 octets)
> - **Ne devraient JAMAIS se produire en full-duplex**
> - Signe quasi certain d'un duplex mismatch
> 
> **Deferred (transmissions différées) :**
> 
> - Transmission retardée car le média était occupé
> - Normal en half-duplex, mais pas en full-duplex

**Étape 3 : Solutions**

**Solution 1 : Auto-négociation des deux côtés (recommandée)**

```bash
# Sur le routeur
Router(config)# interface gigabitEthernet 0/0
Router(config-if)# duplex auto
Router(config-if)# speed auto
Router(config-if)# end

# Sur le switch
Switch(config)# interface gigabitEthernet 0/1
Switch(config-if)# duplex auto
Switch(config-if)# speed auto
Switch(config-if)# end

# Vérifier que la négociation a réussi
Router# show interfaces gigabitEthernet 0/0 | include duplex
  Full-duplex, 1000Mb/s, media type is RJ45

Switch# show interfaces gigabitEthernet 0/1 | include duplex
  Full-duplex, 1000Mb/s, media type is RJ45
# Les deux en full-duplex = CORRECT
```

> [!tip] Pourquoi préférer l'auto-négociation ?
> 
> - Évite les erreurs de configuration manuelle
> - S'adapte automatiquement aux capacités des équipements
> - Standard IEEE 802.3u
> - Fonctionne dans 99% des cas avec des équipements modernes
> 
> **Exceptions où configurer manuellement :**
> 
> - Équipements très anciens avec auto-négociation défaillante
> - Situations où l'auto-négociation échoue systématiquement
> - Exigences spécifiques de performances ou de compatibilité

**Solution 2 : Configuration manuelle identique des deux côtés**

```bash
# Configuration manuelle IDENTIQUE des deux côtés
# Sur le routeur
Router(config)# interface gigabitEthernet 0/0
Router(config-if)# duplex full
Router(config-if)# speed 1000
Router(config-if)# end

# Sur le switch - DOIT ÊTRE IDENTIQUE
Switch(config)# interface gigabitEthernet 0/1
Switch(config-if)# duplex full
Switch(config-if)# speed 1000
Switch(config-if)# end

# Vérification
Router# show interfaces gigabitEthernet 0/0
  Full-duplex, 1000Mb/s

Switch# show interfaces gigabitEthernet 0/1
  Full-duplex, 1000Mb/s
```

> [!warning] Règle d'or pour la configuration duplex/speed
> 
> **JAMAIS de configuration mixte :**
> 
> - ❌ Un côté en "auto" et l'autre en "manual" = MAUVAIS
> - ✅ Les deux en "auto" = BON
> - ✅ Les deux en "manual" avec paramètres identiques = BON
> 
> **Pourquoi ?** Quand un côté est en auto et l'autre forcé :
> 
> - L'équipement en auto ne reçoit pas de signaux de négociation
> - Il se rabat sur des valeurs par défaut (souvent half-duplex)
> - Résultat : duplex mismatch garanti

**Étape 4 : Vérification et tests après correction**

```bash
# 1. Réinitialiser les compteurs pour un suivi propre
Router# clear counters gigabitEthernet 0/0
Clear "show interface" counters on this interface [confirm] [Enter]

# 2. Vérifier l'état après quelques minutes de trafic
Router# show interfaces gigabitEthernet 0/0

GigabitEthernet0/0 is up, line protocol is up
  Full-duplex, 1000Mb/s
  5 minute input rate 80000 bits/sec, 65 packets/sec
  5 minute output rate 75000 bits/sec, 60 packets/sec
     150000 packets input, 18750000 bytes
     0 input errors, 0 CRC, 0 frame, 0 overrun, 0 ignored
     0 late collisions, 0 deferred
     # Tous les compteurs à 0 = EXCELLENT
     145000 packets output, 18125000 bytes
     0 output errors, 0 collisions, 0 interface resets

# 3. Tester les performances
Router# ping 192.168.1.2 repeat 1000
!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
Success rate is 100 percent (1000/1000), round-trip min/avg/max = 1/1/3 ms
# Latence faible et stable = Bon signe

# 4. Surveiller sur une période
Router# show interfaces gigabitEthernet 0/0 | include collision|error
     0 input errors, 0 CRC, 0 frame, 0 overrun, 0 ignored
     0 late collisions, 0 deferred
     0 output errors, 0 collisions, 0 interface resets
# Aucune erreur après correction = Problème résolu
```

#### 🛠️ Méthodologie complète de diagnostic

```bash
# ÉTAPE 1 : Identifier les symptômes
# - Performances dégradées mais interface up/up ?
# - Utilisateurs se plaignent de lenteur ?
# → Suspecter un duplex mismatch

# ÉTAPE 2 : Vérifier la configuration
Router# show interfaces gigabitEthernet 0/0 | include duplex|speed
Switch# show interfaces gigabitEthernet 0/1 | include duplex|speed
# Comparer les deux côtés

# ÉTAPE 3 : Analyser les compteurs d'erreurs
Router# show interfaces gigabitEthernet 0/0 | include collision|error|deferred
# Chercher : late collisions, deferred, input errors

# ÉTAPE 4 : Vérifier l'historique de configuration
Router# show running-config interface gigabitEthernet 0/0
# Voir si duplex/speed sont forcés manuellement

# ÉTAPE 5 : Comparer avec baseline
# Si disponible, comparer avec des valeurs de référence
# Augmentation soudaine de late collisions = problème récent

# ÉTAPE 6 : Appliquer la solution
# Option A : Auto des deux côtés
# Option B : Manuel identique des deux côtés

# ÉTAPE 7 : Valider
# Clear counters, attendre, recompter les erreurs
# Tester performances (ping, throughput)

# ÉTAPE 8 : Documenter
# Noter la configuration correcte pour référence future
```

> [!tip] Commandes essentielles pour duplex mismatch
> 
> **Diagnostic :**
> 
> - `show interfaces [nom] | include duplex` : Vérifier le mode duplex
> - `show interfaces [nom] | include collision` : Compter les collisions
> - `show running-config interface [nom]` : Voir la config actuelle
> - `show interfaces [nom] counters errors` : Compteurs d'erreurs détaillés
> 
> **Correction :**
> 
> - `duplex auto` : Activer l'auto-négociation (recommandé)
> - `duplex full` : Forcer full-duplex manuellement
> - `duplex half` : Forcer half-duplex (rare)
> - `speed auto` : Auto-négociation de la vitesse
> 
> **Surveillance :**
> 
> - `clear counters [nom]` : Réinitialiser les compteurs
> - `show interfaces [nom] | include rate` : Vérifier le débit actuel

> [!warning] Pièges à éviter
> 
> - **Ne jamais mixer auto et manual** entre deux équipements connectés
> - **Ne pas conclure trop vite :** Des late collisions peuvent avoir d'autres causes (câble défectueux, longueur excessive)
> - **Tester après changement :** Attendre quelques minutes et vérifier que les compteurs restent à zéro
> - **Documenter la config :** Noter quelle configuration fonctionne pour éviter de répéter l'erreur
> - **Vérifier les deux côtés :** Un seul côté visible ne suffit pas pour diagnostiquer
> - **Gigabit Ethernet :** Ne supporte que full-duplex (IEEE 802.3z), half-duplex n'est pas possible en 1000 Mbps

#### 📊 Tableau récapitulatif des configurations duplex

|Équipement A|Équipement B|Résultat|Performance|
|---|---|---|---|
|Auto|Auto|✅ Négociation réussie|100%|
|Full|Full|✅ Compatible|100%|
|Half|Half|✅ Compatible|~50% (CSMA/CD)|
|**Auto**|**Full**|❌ **Mismatch**|**30-50%**|
|**Auto**|**Half**|❌ **Mismatch**|**30-50%**|
|**Full**|**Half**|❌ **Mismatch**|**30-50%**|

> [!example] Cas réel de duplex mismatch
> 
> **Situation :** Un administrateur configure un nouveau routeur et force l'interface en full-duplex 1000 Mbps pour "garantir les meilleures performances". Le switch connecté est en auto-négociation.
> 
> **Résultat :**
> 
> - Le switch, ne recevant pas de signaux d'auto-négociation, se rabat sur half-duplex
> - Le routeur envoie en full-duplex et génère des late collisions côté switch
> - Les utilisateurs rapportent une lenteur extrême du réseau
> - Le trafic semble "saccadé" avec des pics de latence
> 
> **Solution :** Mettre les deux interfaces en auto-négociation. Les performances reviennent immédiatement à la normale.

---

## 🎯 Synthèse du dépannage de connectivité

> [!tip] Check-list rapide de dépannage
> 
> **1. Vérifier les états physiques :**
> 
> - `show ip interface brief` → État global
> - Interface down/down ? → Vérifier câblage, `no shutdown`
> - Interface up/down ? → Vérifier encapsulation, clock rate
> 
> **2. Vérifier l'adressage :**
> 
> - `show ip interface` → Adresse et masque corrects ?
> - Même sous-réseau des deux côtés ?
> - `show arp` → Résolution d'adresse fonctionne ?
> 
> **3. Vérifier le routage :**
> 
> - `show ip route` → Route vers destination existe ?
> - `show ip route [destination]` → Route spécifique valide ?
> - Next-hop joignable ?
> 
> **4. Vérifier l'encapsulation :**
> 
> - `show interfaces [nom] | include Encapsulation`
> - Même type des deux côtés (HDLC, PPP) ?
> - Si PPP : authentification configurée ?
> 
> **5. Vérifier duplex/speed :**
> 
> - `show interfaces [nom] | include duplex`
> - Late collisions présentes ?
> - Auto ou manual des deux côtés ?

---

## 🔄 Approche systématique globale

Lorsque vous êtes confronté à un problème de connectivité, suivez cette méthodologie structurée :

**Phase 1 : Identification**

1. Définir précisément le problème (quoi, où, quand)
2. Déterminer l'étendue (un utilisateur, un segment, tout le réseau)
3. Identifier les changements récents

**Phase 2 : Diagnostic**

1. Commencer par le Layer 1 (physique)
2. Progresser vers Layer 2 (liaison de données)
3. Puis Layer 3 (réseau)
4. Utiliser `show` et `ping` pour isoler

**Phase 3 : Résolution**

1. Appliquer la correction la plus simple d'abord
2. Ne changer qu'une chose à la fois
3. Valider après chaque changement
4. Documenter la solution

**Phase 4 : Vérification**

1. Tester la connectivité complètement
2. Vérifier les compteurs d'erreurs
3. Surveiller sur une période
4. Obtenir confirmation des utilisateurs

> [!info] Modèle OSI et dépannage Le dépannage suit généralement le modèle OSI de bas en haut :
> 
> **Layer 1 (Physique) :** Câbles, interfaces down/down **Layer 2 (Liaison) :** Encapsulation, duplex, up/down **Layer 3 (Réseau) :** Adressage IP, routage
> 
> Résoudre les problèmes de bas niveau en premier, car ils impactent les couches supérieures.

---

_Ce cours a été conçu pour fournir une compréhension approfondie du dépannage de connectivité sur les routeurs Cisco. Chaque section peut être utilisée comme référence indépendante lors de situations réelles de dépannage._