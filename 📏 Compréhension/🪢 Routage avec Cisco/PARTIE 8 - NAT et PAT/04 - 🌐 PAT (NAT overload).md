

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

## 🎯 Introduction au PAT

Le **PAT (Port Address Translation)**, également appelé **NAT Overload**, est une extension du NAT qui permet à plusieurs hôtes d'un réseau privé de partager une seule adresse IP publique. C'est la technologie la plus utilisée aujourd'hui dans les environnements domestiques et d'entreprise.

> [!info] Pourquoi PAT ? Le PAT résout le problème de pénurie d'adresses IPv4 en permettant à des milliers d'appareils internes de communiquer sur Internet avec une seule adresse IP publique. Il utilise les numéros de ports pour différencier les connexions.

---

## 🔄 Principe du PAT

### Fonctionnement technique

Le PAT traduit plusieurs adresses IP privées en une seule adresse IP publique en utilisant des **ports source différents** pour chaque connexion.

**Processus de traduction :**

1. Un hôte interne (ex: 192.168.1.10) initie une connexion vers Internet
2. Le routeur remplace l'IP source privée par l'IP publique
3. Le routeur modifie le port source pour créer une signature unique
4. Le routeur enregistre cette correspondance dans sa table NAT
5. Les réponses sont redirigées vers le bon hôte interne grâce au port

### Exemple de traduction PAT

|IP Source (Interne)|Port Source|IP Traduite (Publique)|Port Traduit|Destination|
|---|---|---|---|---|
|192.168.1.10|3345|203.0.113.5|1025|8.8.8.8:80|
|192.168.1.11|3346|203.0.113.5|1026|8.8.8.8:80|
|192.168.1.12|3347|203.0.113.5|1027|1.1.1.1:443|

> [!tip] Avantage principal Avec le PAT, on peut théoriquement supporter jusqu'à 65 535 connexions simultanées (nombre de ports disponibles) avec une seule adresse IP publique !

### Différence NAT vs PAT

|Critère|NAT classique|PAT (NAT Overload)|
|---|---|---|
|**Ratio IP**|1 privée → 1 publique|N privées → 1 publique|
|**Utilisation ports**|Non|Oui (clé de différenciation)|
|**Économie d'IP**|Faible|Élevée|
|**Cas d'usage**|Serveurs, DMZ|Postes clients, LAN|

---

## ⚙️ Configuration PAT avec pool

Cette méthode utilise un pool d'adresses IP publiques. Le routeur utilise les ports pour multiplexer les connexions sur ces adresses.

### Syntaxe complète

```bash
# 1. Définir les interfaces inside et outside
Router(config)# interface [interface-id]
Router(config-if)# ip nat inside
Router(config-if)# exit

Router(config)# interface [interface-id]
Router(config-if)# ip nat outside
Router(config-if)# exit

# 2. Créer une ACL pour identifier le trafic à traduire
Router(config)# access-list [numéro] permit [adresse-source] [wildcard]

# 3. Créer le pool d'adresses publiques
Router(config)# ip nat pool [nom-pool] [ip-début] [ip-fin] netmask [masque]

# 4. Activer le PAT (overload) avec le pool
Router(config)# ip nat inside source list [numéro-acl] pool [nom-pool] overload
```

### Exemple pratique

```bash
# Configuration du réseau interne 192.168.10.0/24
# Pool d'IP publiques : 203.0.113.10 à 203.0.113.20

Router(config)# interface GigabitEthernet0/0
Router(config-if)# ip address 192.168.10.1 255.255.255.0
Router(config-if)# ip nat inside
Router(config-if)# no shutdown
Router(config-if)# exit

Router(config)# interface GigabitEthernet0/1
Router(config-if)# ip address 203.0.113.1 255.255.255.0
Router(config-if)# ip nat outside
Router(config-if)# no shutdown
Router(config-if)# exit

# Définir le trafic à traduire (tout le réseau 192.168.10.0/24)
Router(config)# access-list 1 permit 192.168.10.0 0.0.0.255

# Créer le pool d'adresses publiques
Router(config)# ip nat pool PUBLIC-POOL 203.0.113.10 203.0.113.20 netmask 255.255.255.0

# Activer le PAT avec overload
Router(config)# ip nat inside source list 1 pool PUBLIC-POOL overload
```

> [!example] Fonctionnement Avec cette configuration, les hôtes de 192.168.10.0/24 partageront les adresses 203.0.113.10 à 203.0.113.20. Le routeur utilisera les ports pour multiplexer les connexions. Si une IP du pool est saturée (tous les ports utilisés), le routeur passe à l'IP suivante.

> [!warning] Attention au dimensionnement Même avec PAT, un pool limité peut être saturé en cas de très forte charge. Prévoir suffisamment d'adresses dans le pool pour les pics de trafic.

---

## 🔌 Configuration PAT avec interface

Cette méthode est la plus courante et la plus simple : elle utilise directement l'adresse IP de l'interface outside du routeur pour effectuer le PAT.

### Syntaxe complète

```bash
# 1. Définir les interfaces inside et outside
Router(config)# interface [interface-id]
Router(config-if)# ip nat inside
Router(config-if)# exit

Router(config)# interface [interface-id]
Router(config-if)# ip nat outside
Router(config-if)# exit

# 2. Créer une ACL pour identifier le trafic à traduire
Router(config)# access-list [numéro] permit [adresse-source] [wildcard]

# 3. Activer le PAT sur l'interface outside
Router(config)# ip nat inside source list [numéro-acl] interface [interface-outside] overload
```

### Exemple pratique

```bash
# Configuration typique d'un routeur d'entreprise
# Réseau interne : 10.0.0.0/8
# Interface WAN : GigabitEthernet0/1 avec IP publique

Router(config)# interface GigabitEthernet0/0
Router(config-if)# ip address 10.10.10.1 255.255.255.0
Router(config-if)# ip nat inside
Router(config-if)# no shutdown
Router(config-if)# exit

Router(config)# interface GigabitEthernet0/1
Router(config-if)# ip address 198.51.100.5 255.255.255.252
Router(config-if)# ip nat outside
Router(config-if)# no shutdown
Router(config-if)# exit

# Autoriser tout le réseau 10.0.0.0/8 à accéder à Internet
Router(config)# access-list 10 permit 10.0.0.0 0.255.255.255

# Activer PAT sur l'interface outside
Router(config)# ip nat inside source list 10 interface GigabitEthernet0/1 overload
```

> [!tip] Pourquoi cette méthode ?
> 
> - **Simplicité** : pas besoin de gérer un pool d'adresses
> - **Économie** : utilise l'IP déjà attribuée à l'interface WAN
> - **Adaptation automatique** : si l'IP WAN change (DHCP), le PAT s'adapte automatiquement
> - **Cas d'usage** : idéal pour les petites entreprises et les connexions Internet avec une seule IP publique

### Comparaison : Pool vs Interface

|Critère|PAT avec pool|PAT avec interface|
|---|---|---|
|**Complexité**|Plus complexe|Simple|
|**IP utilisées**|Pool dédié|IP de l'interface|
|**Flexibilité**|Haute (plusieurs IP)|Limitée (1 seule IP)|
|**Charge supportée**|Élevée (65k ports × N IP)|Moyenne (65k ports)|
|**Cas d'usage**|Grandes entreprises, ISP|PME, SOHO, succursales|

---

## 🔍 Vérification et diagnostic

### Commande : show ip nat translations

Cette commande affiche toutes les traductions NAT actives dans la table du routeur.

```bash
Router# show ip nat translations
```

**Sortie exemple :**

```
Pro Inside global      Inside local       Outside local      Outside global
tcp 203.0.113.5:1025  192.168.1.10:3345  8.8.8.8:80        8.8.8.8:80
tcp 203.0.113.5:1026  192.168.1.11:3346  8.8.8.8:80        8.8.8.8:80
udp 203.0.113.5:1027  192.168.1.12:3347  1.1.1.1:53        1.1.1.1:53
```

**Interprétation des colonnes :**

|Colonne|Description|Exemple|
|---|---|---|
|**Pro**|Protocole (tcp/udp/icmp)|tcp|
|**Inside global**|IP:Port publique utilisée|203.0.113.5:1025|
|**Inside local**|IP:Port privée de l'hôte|192.168.1.10:3345|
|**Outside local**|IP:Port du serveur distant|8.8.8.8:80|
|**Outside global**|IP:Port du serveur distant|8.8.8.8:80|

> [!tip] Variante utile Pour filtrer uniquement les traductions PAT d'un hôte spécifique :
> 
> ```bash
> Router# show ip nat translations | include 192.168.1.10
> ```

### Commande : show ip nat statistics

Cette commande affiche les statistiques globales du NAT/PAT : nombre de traductions actives, hits, miss, etc.

```bash
Router# show ip nat statistics
```

**Sortie exemple :**

```
Total active translations: 347 (0 static, 347 dynamic; 347 extended)
Peak translations: 512, occurred 00:15:23 ago
Outside interfaces:
  GigabitEthernet0/1
Inside interfaces:
  GigabitEthernet0/0
Hits: 25847  Misses: 0
CEF Translated packets: 25847, CEF Punted packets: 0
Expired translations: 1205
Dynamic mappings:
-- Inside Source
[Id: 1] access-list 10 pool PUBLIC-POOL refcount 347
 pool PUBLIC-POOL: netmask 255.255.255.0
        start 203.0.113.10 end 203.0.113.20
        type generic, total addresses 11, allocated 2 (18%), misses 0
```

**Analyse des informations clés :**

> [!info] Métriques importantes
> 
> - **Total active translations** : nombre de connexions NAT en cours
> - **Peak translations** : pic historique de connexions simultanées
> - **Hits** : nombre de paquets traduits avec succès
> - **Misses** : tentatives échouées (problème de configuration si > 0)
> - **Expired translations** : connexions fermées et nettoyées
> - **Pool allocation** : pourcentage d'utilisation du pool d'IP

> [!warning] Signes de problèmes
> 
> - **Misses élevés** : ACL mal configurée ou pool épuisé
> - **Peak proche de 65535** : saturation des ports (besoin de plus d'IP)
> - **Pool à 100%** : toutes les IP publiques sont utilisées

### Autres commandes de vérification

```bash
# Afficher uniquement le résumé des statistiques
Router# show ip nat statistics | begin Total

# Vérifier les interfaces NAT configurées
Router# show ip nat statistics | include interfaces

# Afficher les traductions dynamiques uniquement
Router# show ip nat translations verbose

# Afficher la configuration NAT active
Router# show running-config | include nat
Router# show running-config | section nat
```

---

## 🧹 Gestion des traductions NAT

### Clear des traductions NAT

Les traductions NAT sont conservées en mémoire tant que les connexions sont actives. Parfois, il est nécessaire de les supprimer manuellement.

#### Supprimer toutes les traductions dynamiques

```bash
Router# clear ip nat translation *
```

> [!warning] Impact Cette commande coupe toutes les connexions NAT actives ! Les utilisateurs perdront leurs connexions en cours (téléchargements, sessions SSH, etc.). À utiliser avec précaution en production.

#### Supprimer une traduction spécifique

```bash
# Supprimer par adresse IP interne
Router# clear ip nat translation inside [adresse-ip-locale]

# Supprimer par adresse IP externe
Router# clear ip nat translation outside [adresse-ip-globale]

# Exemple : supprimer les traductions de 192.168.1.10
Router# clear ip nat translation inside 192.168.1.10
```

#### Supprimer une traduction précise (protocole + ports)

```bash
Router# clear ip nat translation protocol inside [ip-local] [port-local] outside [ip-global] [port-global]

# Exemple : supprimer une connexion TCP spécifique
Router# clear ip nat translation tcp inside 192.168.1.10 3345 outside 8.8.8.8 80
```

### Timeouts des traductions

Le routeur supprime automatiquement les traductions inactives après un délai configurable.

#### Afficher les timeouts actuels

```bash
Router# show ip nat statistics | include timeout
```

**Valeurs par défaut :**

- **TCP** : 86400 secondes (24 heures)
- **UDP** : 300 secondes (5 minutes)
- **ICMP** : 60 secondes (1 minute)

#### Modifier les timeouts

```bash
# Modifier le timeout global NAT
Router(config)# ip nat translation timeout [secondes]

# Modifier le timeout TCP
Router(config)# ip nat translation tcp-timeout [secondes]

# Modifier le timeout UDP
Router(config)# ip nat translation udp-timeout [secondes]

# Modifier le timeout ICMP
Router(config)# ip nat translation icmp-timeout [secondes]

# Modifier le timeout pour les connexions TCP établies
Router(config)# ip nat translation finrst-timeout [secondes]
```

**Exemple :**

```bash
# Réduire le timeout UDP à 2 minutes pour libérer les ports plus vite
Router(config)# ip nat translation udp-timeout 120

# Réduire le timeout TCP à 2 heures
Router(config)# ip nat translation tcp-timeout 7200
```

> [!tip] Optimisation
> 
> - **Timeouts courts** : libèrent les ports plus vite (utile si pénurie de ports)
> - **Timeouts longs** : évitent les reconnexions fréquentes (meilleur pour la stabilité)
> - **Compromis** : UDP 3-5 min, TCP 1-2h pour la plupart des environnements

### Gestion de la table NAT

```bash
# Afficher le nombre maximum de traductions
Router# show ip nat statistics | include translations

# Modifier la taille maximale de la table NAT (si supporté)
Router(config)# ip nat translation max-entries [nombre]
```

> [!warning] Limites matérielles La capacité maximale de traductions dépend du modèle de routeur et de sa RAM. Les routeurs d'entrée de gamme supportent généralement quelques milliers de traductions, tandis que les routeurs haut de gamme peuvent gérer des centaines de milliers.

---

## 🎯 Bonnes pratiques PAT

> [!tip] Recommandations de configuration
> 
> **1. Choix de la méthode**
> 
> - PAT avec interface : pour les petites structures (< 200 utilisateurs)
> - PAT avec pool : pour les moyennes/grandes structures ou charges élevées
> 
> **2. ACL strictes**
> 
> - Ne pas utiliser `permit any` : spécifier précisément les réseaux internes
> - Utiliser des ACL nommées pour plus de clarté
> 
> **3. Monitoring régulier**
> 
> - Vérifier les statistiques NAT quotidiennement en production
> - Surveiller le ratio hits/misses
> - Alerter si utilisation > 80% du pool
> 
> **4. Documentation**
> 
> - Documenter les pools d'IP utilisés
> - Noter les timeouts personnalisés
> - Garder trace des ACL NAT

> [!warning] Pièges courants
> 
> **Erreur 1 : Oublier le mot-clé `overload`** Sans `overload`, c'est du NAT classique (1:1) et non du PAT !
> 
> **Erreur 2 : ACL trop permissive** Une ACL `permit any` peut traduire du trafic non désiré.
> 
> **Erreur 3 : Pool trop petit** Calculer : nombre d'utilisateurs × connexions moyennes / 60000 ports
> 
> **Erreur 4 : Clear en production sans prévenir** `clear ip nat translation *` coupe toutes les connexions actives !
> 
> **Erreur 5 : Ne pas vérifier après modification** Toujours tester avec `show ip nat translations` et un ping/traceroute

---

## 📊 Tableau récapitulatif des commandes

|Objectif|Commande|
|---|---|
|Définir interface inside|`ip nat inside`|
|Définir interface outside|`ip nat outside`|
|Créer pool d'IP|`ip nat pool [nom] [début] [fin] netmask [masque]`|
|PAT avec pool|`ip nat inside source list [ACL] pool [nom] overload`|
|PAT avec interface|`ip nat inside source list [ACL] interface [int] overload`|
|Voir traductions actives|`show ip nat translations`|
|Voir statistiques|`show ip nat statistics`|
|Supprimer toutes traductions|`clear ip nat translation *`|
|Supprimer traduction IP|`clear ip nat translation inside [IP]`|
|Modifier timeout TCP|`ip nat translation tcp-timeout [sec]`|
|Modifier timeout UDP|`ip nat translation udp-timeout [sec]`|

---

_Configuration mise à jour : Décembre 2025_