

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

## 🔢 Adressage IP et masque de sous-réseau

### Pourquoi configurer une adresse IP ?

L'attribution d'une adresse IPv4 à une interface est **essentielle** pour permettre au routeur de communiquer sur un réseau IP. Sans adresse IP, l'interface ne peut ni router ni transmettre de paquets IP.

> [!info] Concept clé Chaque interface d'un routeur qui participe au routage IP doit avoir une adresse IP unique appartenant au réseau auquel elle est connectée.

### Syntaxe de base

Pour configurer une adresse IPv4 sur une interface :

```bash
Router> enable
Router# configure terminal
Router(config)# interface <type> <numéro>
Router(config-if)# ip address <adresse-ip> <masque-sous-réseau>
Router(config-if)# exit
```

> [!example] Exemple pratique Configuration de l'interface GigabitEthernet 0/0 avec l'adresse 192.168.1.1/24 :
> 
> ```bash
> Router(config)# interface gigabitethernet 0/0
> Router(config-if)# ip address 192.168.1.1 255.255.255.0
> ```

### Les deux formats de masque de sous-réseau

Cisco IOS accepte uniquement le **format décimal** pour la configuration, mais il est important de comprendre les deux notations :

|Notation CIDR|Masque décimal|Nombre d'hôtes|
|---|---|---|
|/24|255.255.255.0|254|
|/25|255.255.255.128|126|
|/26|255.255.255.192|62|
|/27|255.255.255.224|30|
|/28|255.255.255.240|14|
|/30|255.255.255.252|2|

> [!tip] Astuce professionnelle Pour les liaisons point-à-point entre routeurs, utilisez toujours un **/30** (255.255.255.252) pour économiser les adresses IP. Cela fournit exactement 2 adresses utilisables, parfait pour une liaison entre deux routeurs.

### Configuration avec DHCP (cas particulier)

Dans certains cas (typiquement pour une interface WAN connectée à un FAI), vous pouvez configurer l'interface pour obtenir son adresse via DHCP :

```bash
Router(config-if)# ip address dhcp
```

> [!warning] Attention Cette configuration est rare sur les routeurs d'entreprise et généralement réservée aux routeurs de périphérie (edge routers) ou aux environnements de petite taille.

---

## ⚡ Activation des interfaces (no shutdown)

### Pourquoi les interfaces sont-elles désactivées par défaut ?

Par défaut, **toutes les interfaces d'un routeur Cisco sont administrativement désactivées** (état "administratively down"). C'est une mesure de sécurité pour éviter qu'une interface non configurée ne soit active sur le réseau.

> [!info] État par défaut Après une configuration d'usine ou un reset, toutes les interfaces sont en mode `shutdown`. Vous devez explicitement les activer.

### Syntaxe d'activation

```bash
Router(config)# interface <type> <numéro>
Router(config-if)# no shutdown
```

> [!example] Exemple complet d'activation
> 
> ```bash
> Router(config)# interface gigabitethernet 0/0
> Router(config-if)# ip address 192.168.10.1 255.255.255.0
> Router(config-if)# no shutdown
> Router(config-if)#
> *Mar 1 00:15:23.456: %LINK-3-UPDOWN: Interface GigabitEthernet0/0, changed state to up
> *Mar 1 00:15:24.456: %LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet0/0, changed state to up
> ```

### Messages système après activation

Après l'exécution de `no shutdown`, vous verrez deux messages importants :

1. **%LINK-3-UPDOWN** : Indique que la couche physique (Layer 1) est opérationnelle
2. **%LINEPROTO-5-UPDOWN** : Indique que le protocole de ligne (Layer 2) est opérationnel

> [!tip] Comprendre les états
> 
> - **up/up** : L'interface est physiquement connectée et opérationnelle
> - **up/down** : L'interface est active mais il y a un problème de protocole (Layer 2)
> - **administratively down/down** : L'interface est désactivée avec `shutdown`

### Désactivation d'une interface

Pour désactiver une interface (utile lors de la maintenance ou pour des raisons de sécurité) :

```bash
Router(config-if)# shutdown
```

> [!warning] Bonne pratique de sécurité Il est recommandé de désactiver (`shutdown`) toutes les interfaces inutilisées pour des raisons de sécurité. Cela empêche des connexions non autorisées.

---

## 📝 Description des interfaces

### Pourquoi documenter les interfaces ?

La commande `description` permet d'ajouter une étiquette textuelle à une interface. C'est **indispensable** dans un environnement professionnel pour :

- Identifier rapidement le rôle de l'interface
- Documenter les connexions physiques
- Faciliter le dépannage
- Maintenir une documentation à jour dans la configuration

> [!info] Documentation vivante La description fait partie de la configuration et est visible dans toutes les commandes `show`. C'est une forme de documentation qui ne peut pas être obsolète car elle voyage avec la config.

### Syntaxe

```bash
Router(config-if)# description <texte-descriptif>
```

Le texte peut contenir jusqu'à **240 caractères** et peut inclure des espaces.

> [!example] Exemples de bonnes descriptions
> 
> ```bash
> Router(config)# interface gigabitethernet 0/0
> Router(config-if)# description LAN Administratif - VLAN 10
> 
> Router(config)# interface gigabitethernet 0/1
> Router(config-if)# description Liaison vers Routeur-Site-Paris - Circuit MPLS 12345
> 
> Router(config)# interface serial 0/0/0
> Router(config-if)# description WAN vers ISP-Telco - ID Client ABC-789
> 
> Router(config)# interface gigabitethernet 0/2
> Router(config-if)# description Connexion vers Switch-Core-01 port Gi1/0/24
> ```

### Conventions de nommage recommandées

> [!tip] Bonnes pratiques professionnelles Incluez dans vos descriptions :
> 
> - **Le rôle** de l'interface (LAN, WAN, Management)
> - **La destination** de la connexion (autre équipement)
> - **L'identifiant du circuit** (pour les WAN)
> - **Le VLAN** associé (pour les interfaces LAN)
> - **Le port distant** pour faciliter le câblage

### Suppression d'une description

Pour retirer une description :

```bash
Router(config-if)# no description
```

---

## 🔍 Vérification de l'état (show ip interface brief)

### La commande essentielle de vérification

La commande `show ip interface brief` est **la commande la plus utilisée** pour vérifier rapidement l'état de toutes les interfaces d'un routeur.

### Syntaxe et utilisation

```bash
Router# show ip interface brief
```

> [!info] Mode d'exécution Cette commande s'exécute en **mode privilégié** (`Router#`), pas en mode configuration.

### Exemple de sortie

```bash
Router# show ip interface brief

Interface              IP-Address      OK? Method Status                Protocol
GigabitEthernet0/0     192.168.1.1     YES manual up                    up
GigabitEthernet0/1     10.0.0.1        YES manual up                    up
GigabitEthernet0/2     unassigned      YES unset  administratively down down
Serial0/0/0            172.16.1.1      YES manual up                    up
Serial0/0/1            unassigned      YES unset  administratively down down
```

### Comprendre les colonnes

|Colonne|Signification|Valeurs possibles|
|---|---|---|
|**Interface**|Nom de l'interface|GigabitEthernet, Serial, FastEthernet, etc.|
|**IP-Address**|Adresse IPv4 configurée|Adresse IP ou "unassigned"|
|**OK?**|Configuration valide|YES/NO|
|**Method**|Méthode d'attribution|manual, DHCP, unset, TFTP, NVRAM|
|**Status**|État de la couche 1 (physique)|up, down, administratively down|
|**Protocol**|État de la couche 2 (liaison)|up, down|

### Interprétation des états

> [!example] Cas d'usage courants
> 
> **Interface opérationnelle :**
> 
> ```
> GigabitEthernet0/0  192.168.1.1  YES manual up  up
> ```
> 
> ✅ Tout est correct, l'interface fonctionne
> 
> **Interface désactivée administrativement :**
> 
> ```
> GigabitEthernet0/2  unassigned  YES unset  administratively down  down
> ```
> 
> 🔒 L'interface est en mode `shutdown`
> 
> **Problème de câble ou connexion physique :**
> 
> ```
> GigabitEthernet0/1  10.0.0.1  YES manual down  down
> ```
> 
> ⚠️ L'interface est active mais aucun signal physique n'est détecté
> 
> **Problème de protocole Layer 2 :**
> 
> ```
> Serial0/0/0  172.16.1.1  YES manual up  down
> ```
> 
> ⚠️ La connexion physique existe mais le protocole ne fonctionne pas

### Variantes de la commande

Pour voir les détails d'une interface spécifique :

```bash
Router# show ip interface gigabitethernet 0/0
```

Cette version détaillée affiche des informations complètes incluant :

- Adresse IP et masque de sous-réseau
- Broadcast address
- MTU (Maximum Transmission Unit)
- Helper address (pour DHCP relay)
- Access-lists appliquées
- Proxy ARP
- ICMP redirects
- Et bien plus...

> [!tip] Raccourci pratique Vous pouvez abréger la commande : `show ip int br` fonctionne aussi !

### Autres commandes utiles de vérification

```bash
# Voir la configuration en cours d'une interface
Router# show running-config interface gigabitethernet 0/0

# Voir les statistiques d'une interface
Router# show interface gigabitethernet 0/0

# Voir uniquement les interfaces up
Router# show ip interface brief | include up
```

---

## ⚠️ Pièges courants et bonnes pratiques

### Pièges à éviter

> [!warning] Erreur fréquente #1 : Oublier le "no shutdown" La configuration d'une adresse IP seule ne suffit pas. L'interface reste désactivée jusqu'à l'exécution de `no shutdown`. C'est l'erreur la plus courante chez les débutants.

> [!warning] Erreur fréquente #2 : Masque de sous-réseau incorrect Vérifiez toujours que le masque correspond à votre plan d'adressage. Un masque incorrect peut créer des problèmes de routage difficiles à diagnostiquer.
> 
> ```bash
> # INCORRECT - masque trop large
> Router(config-if)# ip address 192.168.1.1 255.255.0.0
> 
> # CORRECT pour un réseau /24
> Router(config-if)# ip address 192.168.1.1 255.255.255.0
> ```

> [!warning] Erreur fréquente #3 : Conflit d'adresse IP Deux interfaces dans le même réseau ne doivent JAMAIS avoir la même adresse IP. Cela cause des problèmes de routage imprévisibles.

> [!warning] Erreur fréquente #4 : Oublier de sauvegarder Après toute modification, pensez à sauvegarder avec `copy running-config startup-config` ou `write memory`. Sinon, vos changements seront perdus au redémarrage.

### Bonnes pratiques professionnelles

> [!tip] Bonne pratique #1 : Standardisez vos descriptions Établissez un format standard pour les descriptions dans votre organisation. Par exemple :
> 
> - Format : `[TYPE] - [DESTINATION] - [CIRCUIT-ID]`
> - Exemple : `WAN - Site-Lyon - MPLS-12345`

> [!tip] Bonne pratique #2 : Utilisez des adresses cohérentes Pour les interfaces de routeur, utilisez généralement :
> 
> - La **première adresse** utilisable (.1) pour la passerelle par défaut du réseau
> - Les **dernières adresses** pour les liaisons point-à-point

> [!tip] Bonne pratique #3 : Documentez au fur et à mesure Ajoutez la description dès la configuration initiale, pas "plus tard". Plus tard n'arrive jamais.

> [!tip] Bonne pratique #4 : Vérifiez systématiquement Après chaque configuration, exécutez `show ip interface brief` pour confirmer l'état. C'est un réflexe à développer.

### Séquence de configuration recommandée

Voici l'ordre optimal pour configurer une interface :

```bash
# 1. Entrer en mode configuration de l'interface
Router(config)# interface gigabitethernet 0/0

# 2. Ajouter une description (documentation)
Router(config-if)# description LAN Utilisateurs - Batiment A

# 3. Configurer l'adresse IP
Router(config-if)# ip address 192.168.10.1 255.255.255.0

# 4. Activer l'interface
Router(config-if)# no shutdown

# 5. Sortir et vérifier
Router(config-if)# exit
Router(config)# exit
Router# show ip interface brief
```

> [!info] Pourquoi cet ordre ?
> 
> - La description d'abord évite de l'oublier
> - L'IP avant le `no shutdown` évite d'activer une interface non configurée
> - La vérification finale confirme que tout fonctionne

---

## 🎯 Récapitulatif des commandes essentielles

```bash
# Configuration complète d'une interface
Router(config)# interface gigabitethernet 0/0
Router(config-if)# description [texte descriptif]
Router(config-if)# ip address [IP] [masque]
Router(config-if)# no shutdown
Router(config-if)# exit

# Vérification
Router# show ip interface brief
Router# show running-config interface gigabitethernet 0/0

# Désactivation
Router(config-if)# shutdown

# Suppression de l'IP
Router(config-if)# no ip address

# Sauvegarde
Router# copy running-config startup-config
```

---

_💡 Ce cours couvre uniquement la configuration IPv4 de base des interfaces. Les aspects avancés comme le routage, les VLANs, ou la sécurité des interfaces sont traités dans d'autres sections._