# 🔐 ACL Étendue (Extended ACL)

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

## 🎯 Introduction aux ACL étendues

Les **ACL étendues** (Extended Access Control Lists) offrent un contrôle granulaire du trafic réseau en permettant de filtrer sur plusieurs critères simultanément. Contrairement aux ACL standard qui filtrent uniquement sur l'adresse IP source, les ACL étendues peuvent examiner :

- Adresse IP source
- Adresse IP de destination
- Type de protocole (TCP, UDP, ICMP, etc.)
- Ports source et destination
- Flags TCP
- Et bien d'autres critères

> [!info] Pourquoi utiliser des ACL étendues ? Les ACL étendues sont le choix privilégié lorsque vous avez besoin d'un contrôle précis du trafic. Par exemple, autoriser uniquement le trafic HTTP vers un serveur web spécifique tout en bloquant le reste du trafic provenant du même réseau.

---

## 🔢 Numérotation des ACL étendues

### Plages de numéros

Les ACL étendues utilisent deux plages de numérotation :

|Type|Plage numérique|Nombre d'ACL possibles|
|---|---|---|
|**Plage standard**|100-199|100 ACL|
|**Plage étendue**|2000-2699|700 ACL|

```bash
# Syntaxe avec numéro
Router(config)# access-list 100 permit tcp any any eq 80

# Syntaxe avec nom (recommandé)
Router(config)# ip access-list extended ACL_WEB_SERVER
```

> [!tip] ACL nommées Il est fortement recommandé d'utiliser des ACL nommées plutôt que numérotées. Les noms descriptifs facilitent la maintenance et la documentation (ex: `ACL_BLOCK_TELNET` au lieu de `101`).

> [!warning] Attention aux plages Ne confondez pas les numéros d'ACL standard (1-99) avec les ACL étendues (100-199). Utiliser le mauvais numéro génère une erreur de syntaxe.

---

## ⚙️ Syntaxe et filtrage

### Syntaxe générale

```bash
access-list [numéro] [permit|deny] [protocole] [source] [wildcard-source] [opérateur port-source] [destination] [wildcard-destination] [opérateur port-destination] [options]
```

### Décomposition des éléments

**1. Numéro ou nom**

```bash
access-list 100 ...                          # Numéroté
ip access-list extended NOM_ACL              # Nommé
```

**2. Action**

- `permit` : Autorise le trafic correspondant
- `deny` : Bloque le trafic correspondant

**3. Protocole**

- `ip` : Tout le trafic IP
- `tcp` : Protocole TCP uniquement
- `udp` : Protocole UDP uniquement
- `icmp` : Protocole ICMP (ping)
- Numéro de protocole (0-255)

**4. Source et destination**

```bash
# Adresse spécifique
192.168.1.10 0.0.0.0

# Réseau
192.168.1.0 0.0.0.255

# N'importe quelle adresse
any

# L'hôte lui-même
host 192.168.1.10   # Équivalent à 192.168.1.10 0.0.0.0
```

**5. Opérateurs de port**

|Opérateur|Signification|Exemple|
|---|---|---|
|`eq`|Égal à (equal)|`eq 80`|
|`neq`|Différent de (not equal)|`neq 23`|
|`lt`|Inférieur à (less than)|`lt 1024`|
|`gt`|Supérieur à (greater than)|`gt 1023`|
|`range`|Plage de ports|`range 20 21`|

### Exemples de base

```bash
# Autoriser tout le trafic HTTP (port 80) vers un serveur web
access-list 100 permit tcp any host 192.168.1.100 eq 80

# Bloquer le trafic Telnet (port 23) depuis un réseau spécifique
access-list 100 deny tcp 10.0.0.0 0.0.0.255 any eq 23

# Autoriser le trafic DNS (UDP port 53)
access-list 100 permit udp any any eq 53

# Bloquer tout le reste (implicite, mais peut être explicite)
access-list 100 deny ip any any
```

> [!example] Exemple complet : Sécuriser un serveur web
> 
> ```bash
> Router(config)# ip access-list extended ACL_WEB_SECURE
> Router(config-ext-nacl)# permit tcp any host 192.168.1.100 eq 80
> Router(config-ext-nacl)# permit tcp any host 192.168.1.100 eq 443
> Router(config-ext-nacl)# deny ip any host 192.168.1.100
> Router(config-ext-nacl)# permit ip any any
> Router(config-ext-nacl)# exit
> Router(config)# interface GigabitEthernet0/0
> Router(config-if)# ip access-group ACL_WEB_SECURE in
> ```
> 
> Cette ACL n'autorise que le trafic HTTP et HTTPS vers le serveur, bloque tout autre trafic vers ce serveur, mais laisse passer le reste.

---

## 🌐 Filtrage par protocole

### TCP (Transmission Control Protocol)

Le protocole TCP est orienté connexion et garantit la livraison des données.

```bash
# Syntaxe TCP complète
access-list [numéro] permit tcp [source] [port-source] [destination] [port-destination] [flags]

# Exemples pratiques
access-list 110 permit tcp any any eq 80                    # HTTP
access-list 110 permit tcp any any eq 443                   # HTTPS
access-list 110 permit tcp any eq 1024 any gt 1023         # Trafic depuis ports éphémères
access-list 110 permit tcp any any eq 22                    # SSH
access-list 110 deny tcp any any eq 23                      # Bloquer Telnet
```

#### Ports TCP courants

|Service|Port|Exemple de règle|
|---|---|---|
|FTP (données)|20|`permit tcp any any eq 20`|
|FTP (contrôle)|21|`permit tcp any any eq 21`|
|SSH|22|`permit tcp any any eq 22`|
|Telnet|23|`deny tcp any any eq 23`|
|SMTP|25|`permit tcp any any eq 25`|
|HTTP|80|`permit tcp any any eq 80`|
|HTTPS|443|`permit tcp any any eq 443`|

### UDP (User Datagram Protocol)

Le protocole UDP est sans connexion et plus rapide que TCP.

```bash
# Syntaxe UDP
access-list [numéro] permit udp [source] [port-source] [destination] [port-destination]

# Exemples pratiques
access-list 120 permit udp any any eq 53                    # DNS
access-list 120 permit udp any eq 68 any eq 67             # DHCP
access-list 120 permit udp any any eq 69                    # TFTP
access-list 120 permit udp any any eq 161                   # SNMP
access-list 120 permit udp any any range 16384 32767       # RTP (VoIP)
```

#### Ports UDP courants

|Service|Port|Exemple de règle|
|---|---|---|
|DNS|53|`permit udp any any eq 53`|
|DHCP (client)|68|`permit udp any eq 68 any eq 67`|
|DHCP (serveur)|67|`permit udp any eq 67 any eq 68`|
|TFTP|69|`permit udp any any eq 69`|
|NTP|123|`permit udp any any eq 123`|
|SNMP|161|`permit udp any any eq 161`|

### ICMP (Internet Control Message Protocol)

ICMP est utilisé pour les messages de diagnostic et de contrôle réseau.

```bash
# Syntaxe ICMP
access-list [numéro] permit icmp [source] [destination] [type-icmp] [code-icmp]

# Exemples pratiques
access-list 130 permit icmp any any echo                    # Autoriser ping
access-list 130 permit icmp any any echo-reply              # Autoriser réponse ping
access-list 130 permit icmp any any unreachable             # Messages d'erreur
access-list 130 permit icmp any any time-exceeded           # Traceroute
access-list 130 deny icmp any any                           # Bloquer tout autre ICMP
```

#### Types ICMP courants

|Type|Nom|Usage|Exemple|
|---|---|---|---|
|0|Echo Reply|Réponse au ping|`permit icmp any any echo-reply`|
|3|Destination Unreachable|Erreur de routage|`permit icmp any any unreachable`|
|8|Echo Request|Ping|`permit icmp any any echo`|
|11|Time Exceeded|TTL expiré (traceroute)|`permit icmp any any time-exceeded`|

> [!warning] Sécurité ICMP Bien que bloquer complètement ICMP puisse sembler sécurisant, cela peut empêcher le diagnostic réseau. Il est recommandé d'autoriser au minimum `echo-reply` et `unreachable` pour le bon fonctionnement du réseau.

---

## 🔄 Le mot-clé Established

### Concept et utilité

Le mot-clé `established` permet de filtrer les connexions TCP en fonction de leur état. Il autorise uniquement les paquets TCP qui font partie d'une connexion déjà établie (présence des flags ACK ou RST).

> [!info] Fonctionnement technique Une connexion TCP établie a déjà effectué le "three-way handshake" (SYN, SYN-ACK, ACK). Les paquets suivants ont le flag ACK activé. Le mot-clé `established` vérifie la présence de ce flag.

### Syntaxe

```bash
access-list [numéro] permit tcp [source] [destination] established
```

### Cas d'usage typique

**Scénario** : Vous voulez autoriser les utilisateurs internes à initier des connexions vers Internet, mais empêcher Internet d'initier des connexions vers le réseau interne.

```bash
# ACL sur l'interface externe (vers Internet)
ip access-list extended ACL_INTERNET_IN
  ! Autoriser uniquement les réponses aux connexions initiées depuis l'intérieur
  permit tcp any any established
  ! Bloquer toute nouvelle connexion depuis Internet
  deny ip any any

interface GigabitEthernet0/1
  description Interface vers Internet
  ip access-group ACL_INTERNET_IN in
```

### Exemples détaillés

**1. Protection basique d'un réseau interne**

```bash
# Interface interne : autoriser tout trafic sortant
ip access-list extended ACL_INTERNAL_OUT
  permit ip 192.168.1.0 0.0.0.255 any

interface GigabitEthernet0/0
  description Interface interne
  ip access-group ACL_INTERNAL_OUT in

# Interface externe : autoriser uniquement les réponses
ip access-list extended ACL_EXTERNAL_IN
  permit tcp any 192.168.1.0 0.0.0.255 established
  deny ip any any

interface GigabitEthernet0/1
  description Interface externe
  ip access-group ACL_EXTERNAL_IN in
```

**2. Autoriser certains services entrants + trafic établi**

```bash
ip access-list extended ACL_DMZ_IN
  ! Autoriser nouvelles connexions HTTP/HTTPS vers serveur web
  permit tcp any host 203.0.113.10 eq 80
  permit tcp any host 203.0.113.10 eq 443
  
  ! Autoriser réponses aux connexions initiées depuis la DMZ
  permit tcp any 203.0.113.0 0.0.0.255 established
  
  ! Bloquer le reste
  deny ip any any
```

> [!tip] Optimisation avec established Placer les règles `established` en début d'ACL améliore les performances car la majorité du trafic dans une connexion existante sera traité rapidement.

> [!warning] Limitations du mot-clé established
> 
> - Ne fonctionne qu'avec TCP (pas UDP ou ICMP)
> - N'est pas un véritable "stateful firewall"
> - Ne vérifie pas la validité réelle de la connexion, seulement la présence des flags ACK/RST
> - Peut être contourné par des paquets malveillants avec flags ACK activés

### Différence avec le firewall stateful

```bash
# ACL avec established (stateless)
access-list 100 permit tcp any any established
# ✗ Vérifie seulement les flags, pas l'état réel de la connexion

# Zone-Based Firewall (stateful) - juste pour information
zone-pair security INSIDE_TO_OUTSIDE source INSIDE destination OUTSIDE
# ✓ Suit réellement l'état des connexions TCP
```

---

## 📍 Placement recommandé

Le placement des ACL étendues est crucial pour l'efficacité et la performance du réseau.

### Règle d'or

> [!tip] Règle de placement **ACL étendues : le plus près possible de la source**
> 
> Placer les ACL étendues près de la source permet de bloquer le trafic indésirable avant qu'il ne consomme de la bande passante sur le réseau.

### Comparaison avec les ACL standard

|Type d'ACL|Placement|Raison|
|---|---|---|
|**Standard**|Près de la destination|Filtrage uniquement sur IP source, placement près de la destination évite de bloquer trop de trafic|
|**Étendue**|Près de la source|Filtrage granulaire permet de bloquer précisément le trafic indésirable dès le départ|

### Exemple de placement optimal

```
[Réseau A] -------- [R1] -------- [R2] -------- [R3] -------- [Serveur Web]
10.0.0.0/24         .1   .2       .1   .2       .1   .2       192.168.1.100
```

**Objectif** : Le réseau A peut accéder uniquement au serveur web en HTTP, rien d'autre.

**❌ Mauvais placement (près de la destination)**

```bash
# Sur R3 (près du serveur) - interface vers le serveur
interface GigabitEthernet0/1
  ip access-group ACL_WEB in

# Le trafic inutile a déjà traversé R1 et R2 !
```

**✅ Bon placement (près de la source)**

```bash
# Sur R1 (près du réseau source) - interface vers le réseau A
ip access-list extended ACL_RESEAU_A
  permit tcp 10.0.0.0 0.0.0.255 host 192.168.1.100 eq 80
  deny ip 10.0.0.0 0.0.0.255 any
  permit ip any any

interface GigabitEthernet0/0
  description Vers Réseau A
  ip access-group ACL_RESEAU_A in

# Le trafic interdit est bloqué immédiatement, économisant la bande passante
```

### Direction d'application

Les ACL peuvent être appliquées dans deux directions sur une interface :

```bash
interface GigabitEthernet0/0
  ip access-group ACL_ENTRANT in       # Trafic entrant dans l'interface
  ip access-group ACL_SORTANT out      # Trafic sortant de l'interface
```

> [!info] Quelle direction choisir ?
> 
> - **IN** : Filtre le trafic qui entre dans l'interface (le plus courant)
> - **OUT** : Filtre le trafic qui sort de l'interface
> 
> En général, on applique les ACL en **IN** pour économiser les ressources du routeur.

### Scénarios de placement complexes

**Scénario 1 : Plusieurs réseaux sources**

```bash
# Si vous avez plusieurs réseaux à filtrer, une ACL par interface source
interface GigabitEthernet0/0
  description Réseau Invités
  ip access-group ACL_GUESTS in

interface GigabitEthernet0/1
  description Réseau Employés
  ip access-group ACL_EMPLOYEES in
```

**Scénario 2 : Protection périmétrique**

```bash
# Sur le routeur de bordure (entre LAN et Internet)
# Interface interne
interface GigabitEthernet0/0
  description LAN Interne
  ip access-group ACL_LAN_TO_INTERNET in    # Contrôle ce qui sort

# Interface externe
interface GigabitEthernet0/1
  description Internet
  ip access-group ACL_INTERNET_TO_LAN in    # Contrôle ce qui entre
```

> [!warning] Erreur courante Ne pas appliquer la même ACL en IN et OUT sur la même interface. Cela peut créer des blocages inattendus et des problèmes de connectivité difficiles à diagnostiquer.

---

## ✅ Bonnes pratiques

### 1. Organisation et nommage

```bash
# ✅ BON : Noms descriptifs
ip access-list extended ACL_BLOCK_TELNET_FROM_INTERNET
ip access-list extended ACL_PERMIT_WEB_TO_DMZ
ip access-list extended ACL_SALES_DEPARTMENT

# ❌ MAUVAIS : Noms vagues
ip access-list extended ACL1
ip access-list extended TEMP
access-list 100...
```

### 2. Documentation inline

```bash
ip access-list extended ACL_WEB_SERVER
  ! === HTTP/HTTPS vers serveur web public ===
  permit tcp any host 203.0.113.10 eq 80
  permit tcp any host 203.0.113.10 eq 443
  
  ! === Autoriser gestion SSH depuis admin uniquement ===
  permit tcp host 10.0.0.5 host 203.0.113.10 eq 22
  
  ! === Bloquer tout le reste ===
  deny ip any host 203.0.113.10
  
  ! === Autoriser autre trafic Internet ===
  permit ip any any
```

### 3. Ordre des règles

Les ACL sont évaluées de haut en bas, la première règle correspondante est appliquée.

```bash
# ✅ BON : Spécifique en premier, général ensuite
ip access-list extended ACL_ORDRE_CORRECT
  permit tcp host 10.0.0.5 any eq 22          ! Règle spécifique
  deny tcp 10.0.0.0 0.0.0.255 any eq 22       ! Règle plus large
  permit ip any any                            ! Règle générale

# ❌ MAUVAIS : Général en premier bloque les règles suivantes
ip access-list extended ACL_ORDRE_INCORRECT
  permit ip any any                            ! Cette règle autorise tout
  deny tcp 10.0.0.0 0.0.0.255 any eq 22       ! Jamais évaluée !
```

### 4. Règle deny implicite

```bash
# Toute ACL se termine par un "deny ip any any" implicite

# Si vous voulez voir les paquets bloqués dans les logs :
ip access-list extended ACL_WITH_LOGGING
  permit tcp any any eq 80
  deny ip any any log    ! Explicite avec logging
```

### 5. Modification des ACL nommées

```bash
# Afficher les numéros de séquence
Router# show ip access-lists ACL_WEB_SERVER

Extended IP access list ACL_WEB_SERVER
    10 permit tcp any host 192.168.1.100 eq 80
    20 permit tcp any host 192.168.1.100 eq 443
    30 deny ip any any

# Insérer une règle entre deux existantes
Router(config)# ip access-list extended ACL_WEB_SERVER
Router(config-ext-nacl)# 15 permit tcp any host 192.168.1.100 eq 8080

# Supprimer une règle spécifique
Router(config-ext-nacl)# no 20

# Réorganiser avec sequence-number
Router(config-ext-nacl)# no 15
Router(config-ext-nacl)# 25 permit tcp any host 192.168.1.100 eq 8080
```

> [!tip] Astuce séquence Les numéros de séquence par défaut sont incrémentés de 10. Cela laisse de la place pour insérer des règles ultérieurement sans tout recréer.

### 6. Test et validation

```bash
# Vérifier la configuration
Router# show ip access-lists
Router# show ip access-lists ACL_WEB_SERVER

# Vérifier l'application sur les interfaces
Router# show ip interface GigabitEthernet0/0 | include access list

# Voir les statistiques (compteurs de paquets)
Router# show ip access-lists ACL_WEB_SERVER

# Réinitialiser les compteurs
Router# clear ip access-list counters ACL_WEB_SERVER
```

### 7. Sauvegarde avant modification

```bash
# Toujours sauvegarder avant une modification majeure
Router# show running-config | include ip access-list
Router# copy running-config startup-config

# Ou exporter la configuration
Router# show running-config | redirect tftp://192.168.1.10/backup-config.txt
```

### 8. Éviter les erreurs courantes

> [!warning] Erreurs fréquentes à éviter
> 
> **1. Oublier le "permit ip any any"**
> 
> ```bash
> # ❌ Bloque tout sauf HTTP
> access-list 100 permit tcp any any eq 80
> # Le "deny ip any any" implicite bloque tout le reste !
> 
> # ✅ Autoriser le reste du trafic
> access-list 100 permit tcp any any eq 80
> access-list 100 permit ip any any
> ```
> 
> **2. Inverser source et destination**
> 
> ```bash
> # ❌ MAUVAIS
> access-list 100 permit tcp host 192.168.1.100 any eq 80
> # Autorise le serveur à initier HTTP vers n'importe qui
> 
> # ✅ BON
> access-list 100 permit tcp any host 192.168.1.100 eq 80
> # Autorise n'importe qui à accéder au serveur en HTTP
> ```
> 
> **3. Mauvais masque wildcard**
> 
> ```bash
> # ❌ MAUVAIS : utilise un masque de sous-réseau
> access-list 100 permit ip 192.168.1.0 255.255.255.0 any
> 
> # ✅ BON : masque wildcard (inverse du masque de sous-réseau)
> access-list 100 permit ip 192.168.1.0 0.0.0.255 any
> ```

### 9. Performance et optimisation

```bash
# Placer les règles les plus fréquemment utilisées en premier
ip access-list extended ACL_OPTIMIZED
  ! 80% du trafic = HTTP/HTTPS
  permit tcp any any eq 80
  permit tcp any any eq 443
  
  ! 15% du trafic = DNS
  permit udp any any eq 53
  
  ! 5% du trafic = Autres services
  permit tcp any any eq 22
  permit tcp any any eq 25
  
  deny ip any any
```

### 10. Utilisation du logging

```bash
ip access-list extended ACL_SECURITY_MONITORING
  ! Log les tentatives d'accès SSH
  permit tcp any any eq 22 log
  
  ! Log les tentatives de scan de ports
  deny tcp any any range 1 1023 log
  
  ! Log tout le trafic bloqué
  deny ip any any log

# Attention : le logging peut impacter les performances !
```

> [!warning] Impact du logging L'utilisation excessive du mot-clé `log` peut surcharger le processeur du routeur car chaque paquet correspondant génère un message de log. Utilisez-le avec parcimonie et uniquement pour le débogage ou la sécurité critique.

---

## 🔍 Commandes de vérification essentielles

```bash
# Afficher toutes les ACL
show ip access-lists

# Afficher une ACL spécifique
show ip access-lists ACL_WEB_SERVER

# Afficher les ACL appliquées aux interfaces
show ip interface

# Afficher la configuration running
show running-config | include access-list
show running-config | section ip access-list

# Compter les correspondances
show ip access-lists | include match

# Debug ACL (utiliser avec précaution en production)
debug ip packet detail
```

---

## 📊 Tableau récapitulatif

|Critère|ACL Standard|ACL Étendue|
|---|---|---|
|**Numérotation**|1-99, 1300-1999|100-199, 2000-2699|
|**Filtrage**|IP source uniquement|Source, destination, protocole, ports|
|**Granularité**|Faible|Élevée|
|**Placement**|Près de la destination|Près de la source|
|**Complexité**|Simple|Avancée|
|**Usage typique**|Bloquer un réseau entier|Contrôle précis par service|

---

> [!tip] Astuce finale Les ACL étendues sont puissantes mais complexes. Commencez par des règles simples, testez-les dans un environnement de lab, et documentez systématiquement chaque règle pour faciliter la maintenance future.