

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

## 🆘 Commandes d'aide

### Le point d'interrogation (?)

Le système d'aide intégré de Cisco IOS est un outil fondamental pour tout administrateur réseau. Il permet de découvrir les commandes disponibles et leur syntaxe sans consulter de documentation externe.

> [!info] Pourquoi c'est important Le système d'aide vous permet de travailler de manière autonome et rapide, même sur des équipements que vous découvrez. C'est votre première ligne de support.

#### Types d'aide disponibles

**1. Aide contextuelle complète (`?`)**

Affiche toutes les commandes disponibles au niveau actuel :

```bash
Router> ?
# Affiche toutes les commandes disponibles en mode utilisateur

Router# ?
# Affiche toutes les commandes disponibles en mode privilégié

Router(config)# ?
# Affiche toutes les commandes disponibles en mode configuration globale
```

**2. Aide par mot-clé (`commande ?`)**

Affiche les sous-commandes ou paramètres possibles :

```bash
Router# show ?
# Liste toutes les options possibles après "show"

Router(config)# interface ?
# Liste tous les types d'interfaces disponibles

Router(config-if)# ip address ?
# Montre la syntaxe attendue pour l'adresse IP
```

**3. Aide partielle (`lettre(s) ?`)**

Filtre les commandes commençant par les lettres saisies :

```bash
Router# sh?
# Affiche : show

Router# s?
# Affiche toutes les commandes commençant par "s" : send, setup, show, ssh, etc.

Router(config)# int?
# Affiche : interface
```

> [!example] Exemple pratique
> 
> ```bash
> Router(config)# interface gigabitEthernet ?
>   <0-9>  GigabitEthernet interface number
> 
> Router(config)# interface gigabitEthernet 0/?
>   <0-9>  GigabitEthernet interface number
> 
> Router(config)# interface gigabitEthernet 0/0
> Router(config-if)# ip address ?
>   A.B.C.D  IP address
>   dhcp     IP Address negotiated via DHCP
> 
> Router(config-if)# ip address 192.168.1.1 ?
>   A.B.C.D  IP subnet mask
> ```

### Complétion automatique (Tab)

La touche **Tab** complète automatiquement les commandes partielles si elles sont non ambiguës.

#### Fonctionnement

```bash
Router# conf[Tab]
# Se complète automatiquement en : Router# configure

Router# sh[Tab]
# Se complète automatiquement en : Router# show

Router(config)# int gi0/0[Tab]
# Se complète en : Router(config)# interface gigabitEthernet 0/0
```

> [!tip] Astuce professionnelle Utilisez systématiquement Tab pour :
> 
> - Gagner du temps de frappe
> - Éviter les fautes de frappe
> - Vérifier que votre commande est valide avant de l'exécuter

#### Gestion de l'ambiguïté

Si plusieurs commandes correspondent, rien ne se passe. Utilisez `?` pour voir les options :

```bash
Router# s[Tab]
# Rien ne se passe (plusieurs commandes commencent par "s")

Router# s?
send  setup  show  ssh
# Maintenant vous voyez les options disponibles

Router# sh[Tab]
# Se complète en "show" (non ambigu)
```

> [!warning] Erreurs courantes
> 
> ```bash
> Router# showinterface
> % Invalid input detected at '^' marker.
> ```
> 
> Les commandes Cisco nécessitent des espaces. Utilisez Tab pour éviter ce genre d'erreur.

### Messages d'erreur et aide contextuelle

Le système IOS fournit des messages d'erreur explicites avec un marqueur `^` pointant l'endroit exact du problème.

```bash
Router# show interfaces gigabitEthernet 0/10
                                            ^
% Invalid input detected at '^' marker.
# L'interface 0/10 n'existe pas sur ce routeur

Router(config)# ip addresss 192.168.1.1
                        ^
% Invalid input detected at '^' marker.
# Faute de frappe : "addresss" au lieu de "address"
```

> [!tip] Réflexe professionnel Quand vous voyez une erreur :
> 
> 1. Regardez où se trouve le `^`
> 2. Tapez `?` à cet endroit pour voir les options valides
> 3. Corrigez et réessayez

---

## 📜 Historique des commandes

L'historique des commandes conserve les dernières commandes exécutées, permettant de les rappeler et les réutiliser rapidement.

### Navigation dans l'historique

|Raccourci|Action|Équivalent|
|---|---|---|
|**↑** (Flèche haut)|Commande précédente|`Ctrl+P`|
|**↓** (Flèche bas)|Commande suivante|`Ctrl+N`|

```bash
Router# show running-config
# [Sortie de la configuration...]

Router# show ip interface brief
# [Sortie des interfaces...]

# Appuyez sur ↑ une fois
Router# show ip interface brief

# Appuyez sur ↑ deux fois
Router# show running-config
```

> [!info] Fonctionnement L'historique fonctionne comme une pile LIFO (Last In, First Out). Les commandes les plus récentes sont accessibles en premier.

### Configuration de l'historique

#### Taille du buffer d'historique

Par défaut, IOS conserve les 10 dernières commandes. Vous pouvez modifier cette valeur :

```bash
# Voir la taille actuelle
Router# show terminal
# Cherchez la ligne : History size is 10

# Modifier la taille (entre 0 et 256)
Router# terminal history size 50
# L'historique conserve maintenant 50 commandes

# Configuration permanente
Router(config)# line console 0
Router(config-line)# history size 50

Router(config)# line vty 0 4
Router(config-line)# history size 50
```

> [!warning] Portée de la configuration
> 
> - `terminal history size` : affecte **uniquement** la session actuelle
> - `history size` (dans line config) : configuration **permanente** pour ce type de ligne

#### Désactiver l'historique

```bash
Router# terminal no history
# L'historique est désactivé pour cette session

# Pour le réactiver
Router# terminal history
```

### Visualiser l'historique complet

```bash
Router# show history
  show version
  show running-config
  show ip interface brief
  configure terminal
  interface gigabitEthernet 0/0
  ip address 192.168.1.1 255.255.255.0
  no shutdown
  exit
  show ip interface brief
  show history
```

> [!tip] Cas d'usage
> 
> - **Dépannage** : retrouver la séquence exacte de commandes qui a causé un problème
> - **Documentation** : copier les commandes pour créer des procédures
> - **Apprentissage** : revoir ce qu'un collègue a configuré

### Astuces d'utilisation

**1. Rappel rapide avec recherche partielle**

Bien que Cisco IOS n'ait pas de recherche intégrée comme Bash, vous pouvez :

```bash
# Utiliser ↑ répétitivement pour retrouver une commande longue
# Exemple : chercher "show ip protocols" dans l'historique
Router# [↑][↑][↑]... jusqu'à trouver la commande
```

**2. Modification rapide d'une commande**

```bash
# Dernière commande
Router# show ip interface gigabitEthernet 0/0

# Appuyez sur ↑ pour la rappeler, puis modifiez
Router# show ip interface gigabitEthernet 0/1
#                                          ^ changé de 0/0 à 0/1
```

**3. Répétition de commandes similaires**

```bash
Router(config)# interface gigabitEthernet 0/0
Router(config-if)# ip address 192.168.1.1 255.255.255.0
Router(config-if)# no shutdown
Router(config-if)# exit

# Pour configurer l'interface suivante, rappeler les commandes
Router(config)# [↑][↑][↑]  # Rappelle "interface gigabitEthernet 0/0"
Router(config)# interface gigabitEthernet 0/1  # Modifiez juste le numéro
Router(config-if)# [↑][↑]  # Rappelle "ip address..."
Router(config-if)# ip address 192.168.2.1 255.255.255.0  # Modifiez l'IP
```

---

## ⌨️ Raccourcis clavier

Les raccourcis clavier Cisco IOS augmentent considérablement la vitesse de configuration et de navigation.

### Raccourcis de déplacement du curseur

|Raccourci|Action|Utilité|
|---|---|---|
|**Ctrl+A**|Aller au début de la ligne|Corriger le début d'une longue commande|
|**Ctrl+E**|Aller à la fin de la ligne|Ajouter à la fin d'une commande rappelée|
|**Ctrl+F** ou **→**|Avancer d'un caractère|Navigation fine|
|**Ctrl+B** ou **←**|Reculer d'un caractère|Navigation fine|
|**Esc+F**|Avancer d'un mot|Sauter rapidement dans la commande|
|**Esc+B**|Reculer d'un mot|Sauter rapidement dans la commande|

> [!example] Exemple pratique
> 
> ```bash
> Router(config-if)# ip address 192.168.1.1 255.255.255.0
> #                  ^                                    ^
> #              Ctrl+A                                Ctrl+E
> 
> # Vous êtes ici: ip address 192.168.1.1 255.255.255.0
> #                                     ^
> # Appuyez sur Ctrl+A
> # Vous êtes ici: ip address 192.168.1.1 255.255.255.0
> #                ^
> ```

### Raccourcis d'édition

|Raccourci|Action|Utilité|
|---|---|---|
|**Ctrl+D**|Supprimer le caractère sous le curseur|Correction précise|
|**Backspace**|Supprimer le caractère avant le curseur|Correction standard|
|**Ctrl+K**|Supprimer tout du curseur à la fin de ligne|Réécrire la fin d'une commande|
|**Ctrl+U** ou **Ctrl+X**|Supprimer toute la ligne|Recommencer la commande|
|**Ctrl+W**|Supprimer le mot avant le curseur|Corriger un mot complet|

> [!example] Utilisation de Ctrl+K
> 
> ```bash
> Router(config)# interface gigabitEthernet 0/0
> #                                ^
> # Curseur positionné après "gigabitEthernet"
> # Appuyez sur Ctrl+K
> 
> Router(config)# interface gigabitEthernet
> # Tout après le curseur a été supprimé
> 
> # Vous pouvez maintenant taper le nouvel identifiant
> Router(config)# interface gigabitEthernet 0/1
> ```

### Raccourcis de contrôle de session

|Raccourci|Action|Utilité|
|---|---|---|
|**Ctrl+C**|Interrompre la commande/sortir du mode config|Annuler une commande en cours|
|**Ctrl+Z**|Sortir au mode privilégié (depuis config)|Retour rapide au mode #|
|**Ctrl+Shift+6**|Interrompre un processus (ping, traceroute)|Stopper une commande longue|

> [!tip] Ctrl+Shift+6 : Le sauveur de temps
> 
> ```bash
> Router# ping 10.0.0.1
> Type escape sequence to abort.
> Sending 5, 100-byte ICMP Echos to 10.0.0.1, timeout is 2 seconds:
> .....
> # Si le ping semble ne jamais finir, appuyez sur Ctrl+Shift+6
> # pour l'interrompre immédiatement
> ```

### Raccourcis de navigation dans l'historique

|Raccourci|Action|Équivalent|
|---|---|---|
|**Ctrl+P**|Commande précédente|Flèche ↑|
|**Ctrl+N**|Commande suivante|Flèche ↓|

### Raccourcis d'affichage

|Raccourci|Action|Utilité|
|---|---|---|
|**Entrée**|Afficher la ligne suivante|Pagination ligne par ligne|
|**Espace**|Afficher la page suivante|Pagination rapide|
|**Ctrl+L** ou **Ctrl+R**|Rafraîchir l'affichage|Réafficher la ligne après un message système|

> [!warning] Messages système interruptifs
> 
> ```bash
> Router(config)# interface gig
> *Dec 21 10:15:32.123: %LINK-3-UPDOWN: Interface GigabitEthernet0/1, changed state to up
> abit0/0
> 
> # Votre commande est interrompue visuellement
> # Appuyez sur Ctrl+L pour réafficher proprement :
> 
> Router(config)# interface gigabit0/0
> ```

### Tableau récapitulatif des raccourcis essentiels

|Catégorie|Raccourci|Action|
|---|---|---|
|**Navigation**|Ctrl+A|Début de ligne|
||Ctrl+E|Fin de ligne|
||Esc+F / Esc+B|Mot suivant/précédent|
|**Édition**|Ctrl+K|Supprimer jusqu'à la fin|
||Ctrl+U|Supprimer toute la ligne|
||Ctrl+W|Supprimer le mot précédent|
|**Contrôle**|Ctrl+C|Annuler|
||Ctrl+Z|Retour mode privilégié|
||Ctrl+Shift+6|Interrompre processus|
|**Historique**|↑ / Ctrl+P|Commande précédente|
||↓ / Ctrl+N|Commande suivante|
|**Affichage**|Ctrl+L|Rafraîchir|

> [!tip] Mémorisation progressive Concentrez-vous d'abord sur ces 5 raccourcis critiques :
> 
> 1. **Ctrl+A** / **Ctrl+E** : Navigation rapide
> 2. **Ctrl+Z** : Retour rapide au mode privilégié
> 3. **Ctrl+Shift+6** : Interrompre les processus
> 4. **↑** : Historique
> 5. **Tab** : Complétion automatique

---

## 🔍 Commandes show essentielles

Les commandes `show` sont les outils de diagnostic et de vérification les plus utilisés sur un routeur Cisco. Elles permettent de visualiser l'état du système, la configuration, et le statut des protocoles.

### Structure des commandes show

```bash
Router# show ?
# Affiche toutes les options disponibles (très longue liste)

Router# show [catégorie] ?
# Affiche les sous-options pour cette catégorie

Router# show [commande complète]
# Affiche les informations demandées
```

> [!info] Niveaux d'accès
> 
> - Mode utilisateur `>` : commandes `show` limitées (version, historique, basiques)
> - Mode privilégié `#` : **toutes** les commandes `show` disponibles

### Commandes show système

#### show version

Affiche les informations matérielles et logicielles du routeur.

```bash
Router# show version

Cisco IOS Software, C2900 Software (C2900-UNIVERSALK9-M), Version 15.1(4)M4
Technical Support: http://www.cisco.com/techsupport
Copyright (c) 1986-2012 by Cisco Systems, Inc.
Compiled Wed 14-Mar-12 15:09 by prod_rel_team

ROM: System Bootstrap, Version 15.0(1r)M15, RELEASE SOFTWARE (fc1)

Router uptime is 2 days, 3 hours, 15 minutes
System returned to ROM by power-on
System image file is "flash:c2900-universalk9-mz.SPA.151-4.M4.bin"

cisco CISCO2911/K9 (revision 1.0) with 487424K/36864K bytes of memory.
Processor board ID FTX1628837W
3 Gigabit Ethernet interfaces
1 Serial(sync/async) interface
DRAM configuration is 64 bits wide with parity disabled.
255K bytes of non-volatile configuration memory.
250880K bytes of ATA System CompactFlash 0 (Read/Write)

Configuration register is 0x2102
```

> [!info] Informations clés fournies
> 
> - **Version IOS** : 15.1(4)M4
> - **Modèle** : CISCO2911/K9
> - **Uptime** : Temps depuis le dernier redémarrage
> - **Mémoire** : RAM disponible (487424K)
> - **Interfaces** : Types et nombre
> - **Image système** : Fichier IOS chargé
> - **Configuration register** : 0x2102 (boot normal)

**Cas d'usage :**

- Vérifier la version IOS avant une mise à jour
- Diagnostiquer des problèmes de mémoire
- Identifier le modèle exact du matériel
- Vérifier le register de configuration

#### show running-config

Affiche la configuration actuellement active en mémoire RAM.

```bash
Router# show running-config

Building configuration...

Current configuration : 1234 bytes
!
version 15.1
service timestamps debug datetime msec
service timestamps log datetime msec
no service password-encryption
!
hostname Router
!
enable secret 5 $1$mERr$hx5rVt7rPNoS4wqbXKX7m0
!
interface GigabitEthernet0/0
 ip address 192.168.1.1 255.255.255.0
 duplex auto
 speed auto
!
interface GigabitEthernet0/1
 no ip address
 shutdown
 duplex auto
 speed auto
!
ip route 0.0.0.0 0.0.0.0 192.168.1.254
!
line con 0
line vty 0 4
 password cisco
 login
!
end
```

> [!warning] Configuration active vs sauvegardée `show running-config` affiche la configuration **en mémoire RAM** (active). Si le routeur redémarre sans sauvegarde, ces modifications seront **perdues**.
> 
> Utilisez `copy running-config startup-config` pour sauvegarder.

**Options utiles :**

```bash
# Filtrer l'affichage
Router# show running-config | section interface
# Affiche uniquement les sections "interface"

Router# show running-config | include ip address
# Affiche uniquement les lignes contenant "ip address"

Router# show running-config | begin interface GigabitEthernet0/0
# Affiche à partir de cette ligne

Router# show running-config interface gigabitEthernet 0/0
# Affiche uniquement la config de cette interface
```

> [!tip] Filtres de sortie Les pipes `|` permettent de traiter la sortie :
> 
> - `| include` : affiche les lignes contenant le texte
> - `| exclude` : masque les lignes contenant le texte
> - `| section` : affiche la section complète
> - `| begin` : commence l'affichage à partir d'une ligne

#### show startup-config

Affiche la configuration sauvegardée dans la NVRAM, celle qui sera chargée au prochain démarrage.

```bash
Router# show startup-config

Using 1234 bytes
!
version 15.1
service timestamps debug datetime msec
service timestamps log datetime msec
!
hostname Router
!
[... contenu de la configuration ...]
```

**Comparaison running vs startup :**

```bash
# Vérifier si des modifications non sauvegardées existent
Router# show running-config | section interface
Router# show startup-config | section interface

# S'ils diffèrent, il y a des modifications non sauvegardées
```

> [!warning] Pièges courants Après avoir modifié la configuration :
> 
> 1. Les changements sont dans `running-config` (RAM)
> 2. Si vous redémarrez **sans sauvegarder**, les changements sont perdus
> 3. Toujours exécuter `copy running-config startup-config` après des modifications importantes

#### show processes

Affiche les processus en cours d'exécution et leur utilisation CPU.

```bash
Router# show processes

CPU utilization for five seconds: 0%/0%; one minute: 0%; five minutes: 0%
 PID Runtime(ms)   Invoked      uSecs   5Sec   1Min   5Min TTY Process
   1           0        83          0  0.00%  0.00%  0.00%   0 Chunk Manager
   2          12      2847          4  0.00%  0.00%  0.00%   0 Load Meter
   3           0         3          0  0.00%  0.00%  0.00%   0 SpanTree Helper
```

> [!info] Colonnes importantes
> 
> - **PID** : Identifiant du processus
> - **Runtime** : Temps CPU total utilisé
> - **5Sec/1Min/5Min** : Utilisation CPU moyenne
> - **Process** : Nom du processus

**Utilisation pour le dépannage :**

```bash
# Identifier un processus consommant trop de CPU
Router# show processes cpu sorted
# Trie par utilisation CPU décroissante
```

#### show memory

Affiche l'utilisation de la mémoire.

```bash
Router# show memory

          Head    Total(b)     Used(b)     Free(b)   Lowest(b)  Largest(b)
Processor  6F5C7B0  515837968   103235680   412602288   411254996   411226980
      I/O  8F00000   33554432    6684720   26869712   26869712   26865616
```

> [!info] Indicateurs clés
> 
> - **Total** : Mémoire totale disponible
> - **Used** : Mémoire actuellement utilisée
> - **Free** : Mémoire libre
> - **Lowest** : Minimum de mémoire libre depuis le démarrage (alerte si trop bas)

### Commandes show réseau

#### show ip interface brief

Vue résumée de toutes les interfaces et leur état. **La commande la plus utilisée pour un aperçu rapide.**

```bash
Router# show ip interface brief

Interface              IP-Address      OK? Method Status                Protocol
GigabitEthernet0/0     192.168.1.1     YES manual up                    up
GigabitEthernet0/1     unassigned      YES unset  administratively down down
GigabitEthernet0/2     10.0.0.1        YES manual up                    up
Serial0/0/0            unassigned      YES unset  down                  down
```

> [!info] Colonnes expliquées
> 
> - **Interface** : Nom de l'interface
> - **IP-Address** : Adresse IP configurée (unassigned = pas d'IP)
> - **OK?** : Configuration valide (YES) ou invalide (NO)
> - **Method** : Comment l'IP a été configurée (manual, DHCP, unset)
> - **Status** : État administratif (up, down, administratively down)
> - **Protocol** : État du protocole de ligne (up = câble connecté et actif)

**Interprétation des états :**

|Status|Protocol|Signification|
|---|---|---|
|up|up|✅ Interface fonctionnelle|
|up|down|⚠️ Interface activée mais problème de couche physique (câble déconnecté)|
|administratively down|down|🔒 Interface désactivée par commande `shutdown`|
|down|down|❌ Interface inactive (désactivée ou problème matériel)|

> [!tip] Réflexe professionnel Exécutez `show ip interface brief` :
> 
> - Après chaque modification d'interface
> - Pour vérifier rapidement l'état du routeur
> - Avant de commencer un dépannage réseau

#### show interfaces

Affiche des informations détaillées sur une ou toutes les interfaces.

```bash
Router# show interfaces gigabitEthernet 0/0

GigabitEthernet0/0 is up, line protocol is up
  Hardware is CN Gigabit Ethernet, address is c471.fe6b.5000 (bia c471.fe6b.5000)
  Internet address is 192.168.1.1/24
  MTU 1500 bytes, BW 1000000 Kbit/sec, DLY 10 usec,
     reliability 255/255, txload 1/255, rxload 1/255
  Encapsulation ARPA, loopback not set
  Keepalive set (10 sec)
  Full-duplex, 1000Mb/s, media type is RJ45
  output flow-control is unsupported, input flow-control is unsupported
  ARP type: ARPA, ARP Timeout 04:00:00
  Last input 00:00:00, output 00:00:00, output hang never
  Last clearing of "show interface" counters never
  Input queue: 0/75/0/0 (size/max/drops/flushes); Total output drops: 0
  Queueing strategy: fifo
  Output queue: 0/40 (size/max)
  5 minute input rate 0 bits/sec, 0 packets/sec
  5 minute output rate 0 bits/sec, 0 packets/sec
     12345 packets input, 1234567 bytes, 0 no buffer
     Received 123 broadcasts (0 IP multicasts)
     0 runts, 0 giants, 0 throttles
     0 input errors, 0 CRC, 0 frame, 0 overrun, 0 ignored
     0 watchdog, 456 multicast, 0 pause input
     0 input packets with dribble condition detected
     67890 packets output, 7654321 bytes, 0 underruns
     0 output errors, 0 collisions, 1 interface resets
     0 unknown protocol drops
     0 babbles, 0 late collision, 0 deferred
     0 lost carrier, 0 no carrier, 0 pause output
     0 output buffer failures, 0 output buffers swapped out
```

> [!info] Informations critiques
> 
> - **État** : up/down (physique et protocole)
> - **Adresse MAC** : c471.fe6b.5000
> - **MTU** : 1500 bytes (taille maximale de trame)
> - **Bandwidth** : 1000000 Kbit/sec (1 Gbps)
> - **Duplex** : Full-duplex
> - **Erreurs** : Input/Output errors, CRC, collisions

**Analyse des erreurs :**

```bash
# Rechercher des erreurs spécifiques
Router# show interfaces gigabitEthernet 0/0 | include error
     0 input errors, 0 CRC, 0 frame, 0 overrun, 0 ignored
     0 output errors, 0 collisions, 1 interface resets
```

> [!warning] Erreurs à surveiller
> 
> - **CRC errors** : Problèmes de câblage ou d'interférences
> - **Collisions** : Problème de duplex mismatch
> - **Input errors** : Trames corrompues reçues
> - **Runts/Giants** : Trames de taille anormale

#### show ip route

Affiche la table de routage IP.

```bash
Router# show ip route

Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       + - replicated route, % - next hop override

Gateway of last resort is 192.168.1.254 to network 0.0.0.0

S*    0.0.0.0/0 [1/0] via 192.168.1.254
      10.0.0.0/8 is variably subnetted, 2 subnets, 2 masks
C        10.0.0.0/24 is directly connected, GigabitEthernet0/2
L        10.0.0.1/32 is directly connected, GigabitEthernet0/2
      192.168.1.0/24 is variably subnetted, 2 subnets, 2 masks
C        192.168.1.0/24 is directly connected, GigabitEthernet0/0
L        192.168.1.1/32 is directly connected, GigabitEthernet0/0
```

> [!info] Codes de routage
> 
> - **S*** : Route statique par défaut (0.0.0.0/0)
> - **C** : Réseau directement connecté
> - **L** : Adresse IP locale de l'interface
> - **S** : Route statique
> - **R** : Route apprise via RIP
> - **D** : Route apprise via EIGRP
> - **O** : Route apprise via OSPF

**Lecture d'une entrée :**

```bash
S*    0.0.0.0/0 [1/0] via 192.168.1.254
# S* = Route statique par défaut
# 0.0.0.0/0 = Toutes les destinations
# [1/0] = [Distance administrative / Métrique]
# via 192.168.1.254 = Next-hop (passerelle)

C     192.168.1.0/24 is directly connected, GigabitEthernet0/0
# C = Réseau connecté directement
# 192.168.1.0/24 = Réseau
# GigabitEthernet0/0 = Interface de sortie
```

> [!tip] Commandes complémentaires
> 
> ```bash
> # Afficher uniquement les routes statiques
> Router# show ip route static
> 
> # Afficher uniquement les routes connectées
> Router# show ip route connected
> 
> # Afficher les détails d'une route spécifique
> Router# show ip route 10.0.0.0
> ```

#### show arp

Affiche la table ARP (correspondances IP ↔ MAC).

```bash
Router# show arp

Protocol  Address          Age (min)  Hardware Addr   Type   Interface
Internet  192.168.1.1             -   c471.fe6b.5000  ARPA   GigabitEthernet0/0
Internet  192.168.1.10           15   0050.56c0.0001  ARPA   GigabitEthernet0/0
Internet  192.168.1.254           5   0050.56c0.0008  ARPA   GigabitEthernet0/0
Internet  10.0.0.1                -   c471.fe6b.5001  ARPA   GigabitEthernet0/2
```

> [!info] Colonnes expliquées
> 
> - **Address** : Adresse IP
> - **Age** : Temps depuis la dernière mise à jour (- = entrée locale)
> - **Hardware Addr** : Adresse MAC
> - **Interface** : Interface où la correspondance a été apprise

**Dépannage avec ARP :**

```bash
# Vider la table ARP (forcer une nouvelle résolution)
Router# clear arp-cache

# Vérifier si une IP spécifique est dans le cache
Router# show arp | include 192.168.1.10
```

> [!warning] Âge des entrées ARP
> 
> - **Age = -** : Adresse locale du routeur
> - **Age récent (0-5 min)** : Communication active
> - **Age ancien (>10 min)** : Entrée expirée bientôt
> - Timeout par défaut : **4 heures** (240 minutes)

#### show protocols

Affiche le statut des protocoles de routage configurés et l'état des interfaces.

```bash
Router# show protocols

Global values:
  Internet Protocol routing is enabled
GigabitEthernet0/0 is up, line protocol is up
  Internet address is 192.168.1.1/24
GigabitEthernet0/1 is administratively down, line protocol is down
GigabitEthernet0/2 is up, line protocol is up
  Internet address is 10.0.0.1/24
Serial0/0/0 is down, line protocol is down
```

> [!tip] Cas d'usage Vue rapide pour vérifier :
> 
> - Quelles interfaces ont une adresse IP
> - L'état de chaque interface
> - Si le routage IP est activé

#### show ip protocols

Affiche les informations sur les protocoles de routage dynamique actifs.

```bash
Router# show ip protocols

*** IP Routing is NSF aware ***

Routing Protocol is "rip"
  Outgoing update filter list for all interfaces is not set
  Incoming update filter list for all interfaces is not set
  Sending updates every 30 seconds, next due in 12 seconds
  Invalid after 180 seconds, hold down 180, flushed after 240
  Redistributing: rip
  Default version control: send version 2, receive version 2
    Interface             Send  Recv  Triggered RIP  Key-chain
    GigabitEthernet0/0    2     2
    GigabitEthernet0/2    2     2
  Automatic network summarization is not in effect
  Maximum path: 4
  Routing for Networks:
    10.0.0.0
    192.168.1.0
  Routing Information Sources:
    Gateway         Distance      Last Update
    192.168.1.254        120      00:00:15
  Distance: (default is 120)
```

> [!info] Informations clés
> 
> - **Protocole actif** : RIP, EIGRP, OSPF, etc.
> - **Réseaux annoncés** : Networks configurés
> - **Timers** : Intervalles de mise à jour
> - **Sources** : Voisins de routage
> - **Distance administrative** : 120 pour RIP

### Commandes show de connectivité

#### show cdp neighbors

CDP (Cisco Discovery Protocol) découvre les équipements Cisco directement connectés.

```bash
Router# show cdp neighbors

Capability Codes: R - Router, T - Trans Bridge, B - Source Route Bridge
                  S - Switch, H - Host, I - IGMP, r - Repeater

Device ID        Local Intrfce     Holdtme    Capability  Platform  Port ID
Switch1          Gig 0/0           120        S I         WS-C2960  Fas 0/1
Router2          Ser 0/0/0         150        R           CISCO2911 Ser 0/0/0
```

> [!info] Informations fournies
> 
> - **Device ID** : Nom d'hôte du voisin
> - **Local Interface** : Interface locale connectée
> - **Holdtime** : Temps avant expiration de l'information
> - **Capability** : Type d'équipement (R=Router, S=Switch)
> - **Platform** : Modèle du voisin
> - **Port ID** : Interface du voisin connectée

**Version détaillée :**

```bash
Router# show cdp neighbors detail

-------------------------
Device ID: Switch1
Entry address(es):
  IP address: 192.168.1.2
Platform: cisco WS-C2960-24TT-L,  Capabilities: Switch IGMP
Interface: GigabitEthernet0/0,  Port ID (outgoing port): FastEthernet0/1
Holdtime : 120 sec

Version :
Cisco IOS Software, C2960 Software (C2960-LANBASEK9-M), Version 15.0(2)SE
```

> [!tip] Cas d'usage CDP
> 
> - Cartographier rapidement la topologie physique
> - Vérifier les connexions sans accéder physiquement aux équipements
> - Identifier le modèle et l'IOS des voisins
> - Dépanner les problèmes de câblage

> [!warning] Sécurité CDP CDP transmet des informations sensibles (modèle, IOS, IP). Dans un environnement de production, désactivez-le sur les interfaces exposées :
> 
> ```bash
> Router(config-if)# no cdp enable
> ```

#### show users

Affiche les utilisateurs actuellement connectés au routeur.

```bash
Router# show users

    Line       User       Host(s)              Idle       Location
*  0 con 0                idle                 00:00:00
   2 vty 0     admin      idle                 00:05:23   192.168.1.100

  Interface    User               Mode         Idle     Peer Address
```

> [!info] Colonnes expliquées
> 
> - **Line** : Type de connexion (con = console, vty = telnet/SSH)
> - **User** : Nom d'utilisateur connecté
> - **Idle** : Temps d'inactivité
> - **Location** : Adresse IP source (pour vty)
> - ***** : Indique votre session actuelle

**Gestion des sessions :**

```bash
# Déconnecter un utilisateur spécifique
Router# clear line vty 0
[confirm]

# Voir qui est connecté en temps réel
Router# show users
```

### Commandes show de sécurité

#### show privilege

Affiche le niveau de privilège actuel de votre session.

```bash
Router> show privilege
Current privilege level is 1

Router# show privilege
Current privilege level is 15
```

> [!info] Niveaux de privilège
> 
> - **Niveau 1** : Mode utilisateur (User EXEC) - accès limité
> - **Niveau 15** : Mode privilégié (Privileged EXEC) - accès complet
> - **Niveaux 2-14** : Niveaux personnalisables (rarement utilisés)

#### show sessions

Affiche les connexions Telnet/SSH sortantes actives depuis ce routeur.

```bash
Router# show sessions

Conn Host                Address             Byte  Idle Conn Name
   1 192.168.1.10        192.168.1.10           0     0 192.168.1.10
*  2 10.0.0.2            10.0.0.2               0     0 10.0.0.2

# * indique la session active
```

**Navigation entre sessions :**

```bash
# Suspendre une session Telnet : Ctrl+Shift+6 puis X
Router# telnet 192.168.1.10
[...session ouverte...]
[Ctrl+Shift+6, puis X]

Router# 
# Vous êtes revenu au routeur d'origine

# Reprendre une session
Router# resume 1
# ou simplement taper le numéro
Router# 1

# Fermer une session
Router# disconnect 1
```

### Commandes show de configuration avancée

#### show flash

Affiche le contenu de la mémoire Flash (où est stocké l'IOS).

```bash
Router# show flash:

System flash directory:
File  Length   Name/status
  3   33591768 c2900-universalk9-mz.SPA.151-4.M4.bin
  2   28282    sigdef-category.xml
  1   227537   sigdef-default.xml
[33847587 bytes used, 216760109 available, 250607696 total]
249856K bytes of processor board System flash (Read/Write)
```

> [!info] Informations clés
> 
> - **Fichiers IOS** : Images système disponibles
> - **Taille** : Espace utilisé et disponible
> - **Total** : Capacité totale de la Flash

**Gestion de la Flash :**

```bash
# Lister le contenu avec plus de détails
Router# dir flash:

# Supprimer un fichier (attention !)
Router# delete flash:ancien-ios.bin

# Vérifier l'intégrité d'une image IOS
Router# verify flash:c2900-universalk9-mz.SPA.151-4.M4.bin
```

#### show clock

Affiche l'horloge système du routeur.

```bash
Router# show clock
*10:45:23.456 UTC Sun Dec 21 2025
```

> [!warning] Importance de l'horloge
> 
> - Essentielle pour les logs horodatés
> - Critique pour les protocoles de sécurité (certificats, Kerberos)
> - Synchronisation recommandée via NTP

**Configuration de l'horloge :**

```bash
# Définir manuellement (non persistant au redémarrage)
Router# clock set 10:45:00 21 December 2025

# Configurer NTP (persistant)
Router(config)# ntp server 129.6.15.28
```

#### show logging

Affiche les messages de log et la configuration du système de logging.

```bash
Router# show logging

Syslog logging: enabled (0 messages dropped, 3 messages rate-limited,
                0 flushes, 0 overruns, xml disabled, filtering disabled)

No Active Message Discriminator.

No Inactive Message Discriminator.

    Console logging: level debugging, 145 messages logged, xml disabled,
                     filtering disabled
    Monitor logging: level debugging, 0 messages logged, xml disabled,
                     filtering disabled
    Buffer logging:  level debugging, 145 messages logged, xml disabled,
                    filtering disabled
    Exception Logging: size (8192 bytes)
    Count and timestamp logging messages: disabled
    Persistent logging: disabled

No active filter modules.

    Trap logging: level informational, 89 message lines logged
        Logging to 192.168.1.100  (udp port 514, audit disabled,
              link up),
              89 message lines logged,
              0 message lines rate-limited,
              0 message lines dropped-by-MD,
              xml disabled, sequence number disabled
              filtering disabled
Log Buffer (8192 bytes):

*Dec 21 10:30:15.123: %LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet0/0, changed state to up
*Dec 21 10:30:16.456: %LINK-3-UPDOWN: Interface GigabitEthernet0/0, changed state to up
```

> [!info] Niveaux de logging (0-7)
> 
> |Niveau|Nom|Signification|
> |---|---|---|
> |0|Emergency|Système inutilisable|
> |1|Alert|Action immédiate requise|
> |2|Critical|Condition critique|
> |3|Error|Message d'erreur|
> |4|Warning|Message d'avertissement|
> |5|Notice|Normal mais significatif|
> |6|Informational|Messages informatifs|
> |7|Debug|Messages de débogage|

**Commandes de logging complémentaires :**

```bash
# Afficher uniquement les logs en mémoire tampon
Router# show logging | include %

# Effacer les logs
Router# clear logging

# Désactiver temporairement les logs sur la console
Router(config)# no logging console
```

### Tableau récapitulatif des commandes show essentielles

|Catégorie|Commande|Information fournie|
|---|---|---|
|**Système**|`show version`|Version IOS, modèle, uptime, mémoire|
||`show running-config`|Configuration active (RAM)|
||`show startup-config`|Configuration sauvegardée (NVRAM)|
||`show processes`|Processus et utilisation CPU|
||`show memory`|Utilisation de la mémoire|
|**Réseau**|`show ip interface brief`|⭐ Résumé des interfaces|
||`show interfaces`|Détails d'une interface|
||`show ip route`|Table de routage|
||`show arp`|Cache ARP|
||`show protocols`|Statut des protocoles réseau|
||`show ip protocols`|Protocoles de routage dynamique|
|**Connectivité**|`show cdp neighbors`|Équipements Cisco voisins|
||`show users`|Utilisateurs connectés|
||`show sessions`|Sessions Telnet/SSH actives|
|**Configuration**|`show flash`|Contenu de la mémoire Flash|
||`show clock`|Horloge système|
||`show logging`|Messages de log|
|**Sécurité**|`show privilege`|Niveau de privilège actuel|

> [!tip] Les 5 commandes à maîtriser en priorité
> 
> 1. **`show running-config`** : Configuration active
> 2. **`show ip interface brief`** : État des interfaces
> 3. **`show ip route`** : Table de routage
> 4. **`show interfaces`** : Détails et erreurs
> 5. **`show version`** : Informations système

### Techniques avancées de filtrage

Les commandes `show` peuvent produire des sorties très longues. Utilisez les pipes pour filtrer efficacement.

#### Pipe include

Affiche uniquement les lignes contenant le texte spécifié.

```bash
Router# show running-config | include interface
interface GigabitEthernet0/0
interface GigabitEthernet0/1
interface GigabitEthernet0/2
interface Serial0/0/0

Router# show running-config | include ip address
 ip address 192.168.1.1 255.255.255.0
 ip address 10.0.0.1 255.255.255.0
```

#### Pipe exclude

Masque les lignes contenant le texte spécifié.

```bash
Router# show running-config | exclude !
# Affiche la config sans les lignes de commentaire "!"

Router# show interfaces | exclude 0 packets
# Masque les statistiques à zéro
```

#### Pipe section

Affiche la section complète contenant le texte.

```bash
Router# show running-config | section interface GigabitEthernet0/0
interface GigabitEthernet0/0
 ip address 192.168.1.1 255.255.255.0
 duplex auto
 speed auto
 no shutdown
```

#### Pipe begin

Commence l'affichage à partir de la première occurrence.

```bash
Router# show running-config | begin interface
interface GigabitEthernet0/0
 ip address 192.168.1.1 255.255.255.0
[... reste de la configuration ...]
```

#### Combinaison de filtres

```bash
# Afficher les interfaces avec des erreurs
Router# show interfaces | include error|CRC

# Afficher les routes statiques uniquement
Router# show running-config | section ip route

# Afficher la config sans les lignes vides
Router# show running-config | exclude ^$
```

> [!tip] Expressions régulières Les pipes Cisco supportent les regex basiques :
> 
> - **^** : Début de ligne
> - **$** : Fin de ligne
> - **.** : N'importe quel caractère
> - ***** : Zéro ou plusieurs occurrences

---

## 🎯 Synthèse des bonnes pratiques

### Workflow efficace avec les commandes de base

1. **Toujours commencer par une vue d'ensemble**
    
    ```bash
    Router# show ip interface brief
    Router# show ip route
    ```
    
2. **Utiliser l'aide avant de chercher la documentation**
    
    ```bash
    Router# commande ?
    Router# comm[Tab]
    ```
    
3. **Exploiter l'historique pour les tâches répétitives**
    
    ```bash
    # Configurez la première interface
    # Puis utilisez ↑ pour répéter sur les autres interfaces
    ```
    
4. **Filtrer intelligemment les sorties longues**
    
    ```bash
    Router# show running-config | section interface
    Router# show interfaces | include error
    ```
    
5. **Vérifier avant de sauvegarder**
    
    ```bash
    Router# show running-config
    # Vérifiez que tout est correct
    Router# copy running-config startup-config
    ```
    

### Commandes réflexes pour le dépannage

```bash
# 1. État général du système
Router# show version
Router# show processes cpu sorted
Router# show memory

# 2. État des interfaces
Router# show ip interface brief
Router# show interfaces gigabitEthernet 0/0

# 3. Connectivité réseau
Router# show ip route
Router# show arp
Router# show cdp neighbors

# 4. Configuration active
Router# show running-config | section interface
Router# show ip protocols
```

### Pièges à éviter

> [!warning] Erreurs fréquentes
> 
> 1. **Oublier de sauvegarder** après une modification
>     - Toujours : `copy running-config startup-config`
> 2. **Confondre running-config et startup-config**
>     - Running = RAM (volatile)
>     - Startup = NVRAM (persistante)
> 3. **Ne pas vérifier les erreurs d'interface**
>     - Toujours consulter : `show interfaces | include error`
> 4. **Ignorer les messages système**
>     - Appuyez sur Ctrl+L pour réafficher votre commande proprement
> 5. **Ne pas utiliser Tab et ?**
>     - Vous perdez du temps et faites plus d'erreurs

---

## 🔑 Points clés à retenir

✅ Le système d'aide (`?`) et la complétion (`Tab`) sont vos meilleurs alliés

✅ L'historique des commandes (↑/↓) accélère considérablement votre travail

✅ Les raccourcis clavier (Ctrl+A, Ctrl+E, Ctrl+Z, Ctrl+Shift+6) sont essentiels pour l'efficacité

✅ `show ip interface brief` est la commande la plus utilisée au quotidien

✅ Les filtres de sortie (`| include`, `| section`, etc.) sont indispensables pour les longues sorties

✅ Toujours vérifier `running-config` avant de sauvegarder vers `startup-config`

✅ Les commandes `show` ne modifient jamais la configuration - utilisez-les sans crainte

---

_Ce cours couvre les commandes de base essentielles pour travailler efficacement sur un routeur Cisco. La maîtrise de ces commandes est fondamentale avant d'aborder la configuration avancée des protocoles et services réseau._