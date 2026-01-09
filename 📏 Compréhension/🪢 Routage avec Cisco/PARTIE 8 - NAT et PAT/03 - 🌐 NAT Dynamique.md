

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

## 🎯 Introduction au NAT Dynamique

Le **NAT dynamique** permet de traduire plusieurs adresses IP privées en plusieurs adresses IP publiques de manière **automatique et temporaire**. Contrairement au NAT statique qui crée des mappages permanents un-à-un, le NAT dynamique attribue dynamiquement une adresse publique disponible depuis un pool d'adresses.

> [!info] Différence clé
> 
> - **NAT statique** : Mapping permanent (1 privée ↔ 1 publique fixe)
> - **NAT dynamique** : Mapping temporaire (plusieurs privées ↔ plusieurs publiques du pool)
> - **PAT** : Plusieurs privées ↔ 1 publique avec des ports différents

### Quand utiliser le NAT dynamique ?

- Vous disposez d'**un bloc d'adresses IP publiques** (pas seulement une)
- Vous avez **plus d'hôtes internes que d'adresses publiques**, mais pas tous simultanément
- Vous voulez une **traduction automatique** sans configuration manuelle pour chaque hôte
- Les connexions sont principalement **initiées depuis l'intérieur** du réseau

> [!warning] Limitation importante Le NAT dynamique ne peut traduire que le nombre d'hôtes égal au nombre d'adresses disponibles dans le pool. Si le pool est épuisé, les nouveaux hôtes ne pourront pas accéder à Internet jusqu'à ce qu'une adresse se libère.

---

## 💾 Pool d'adresses publiques

### Concept et utilité

Un **pool NAT** est un ensemble d'adresses IP publiques contiguës ou non-contiguës que le routeur peut utiliser pour effectuer les traductions d'adresses. Le routeur attribue automatiquement une adresse du pool à chaque hôte interne qui en a besoin.

**Caractéristiques d'un pool :**

- Contient une ou plusieurs adresses IP publiques
- Les adresses peuvent être contiguës (plage) ou définies individuellement
- Chaque adresse du pool ne peut être utilisée que par **un seul hôte à la fois**
- Les mappages sont temporaires et libérés après un timeout

> [!example] Exemple de scénario Vous avez 50 employés mais seulement 10 adresses IP publiques (203.0.113.10 à 203.0.113.19). Avec le NAT dynamique, jusqu'à 10 employés peuvent accéder simultanément à Internet. Les adresses sont réutilisées lorsque les sessions se terminent.

### Configuration d'un pool

La création d'un pool se fait avec la commande `ip nat pool` en mode de configuration globale.

#### Syntaxe de base

```bash
Router(config)# ip nat pool NOM_POOL DEBUT_IP FIN_IP netmask MASQUE
```

ou

```bash
Router(config)# ip nat pool NOM_POOL DEBUT_IP FIN_IP prefix-length LONGUEUR
```

#### Paramètres

|Paramètre|Description|Exemple|
|---|---|---|
|`NOM_POOL`|Nom descriptif du pool (sensible à la casse)|`POOL-PUBLIC`|
|`DEBUT_IP`|Première adresse IP du pool|`203.0.113.10`|
|`FIN_IP`|Dernière adresse IP du pool|`203.0.113.19`|
|`netmask MASQUE`|Masque de sous-réseau en notation décimale|`netmask 255.255.255.0`|
|`prefix-length`|Masque en notation CIDR|`prefix-length 24`|

#### Exemples de configuration de pools

**Pool avec plage contiguë :**

```bash
Router(config)# ip nat pool INTERNET 203.0.113.10 203.0.113.19 netmask 255.255.255.248
```

**Pool avec notation CIDR :**

```bash
Router(config)# ip nat pool PUBLIC-POOL 198.51.100.1 198.51.100.10 prefix-length 28
```

**Pool d'une seule adresse :**

```bash
Router(config)# ip nat pool SINGLE-IP 203.0.113.50 203.0.113.50 netmask 255.255.255.255
```

> [!tip] Bonnes pratiques pour nommer les pools
> 
> - Utilisez des noms descriptifs et en MAJUSCULES
> - Évitez les espaces (utilisez des tirets ou underscores)
> - Indiquez l'usage dans le nom : `POOL-GUEST`, `POOL-SERVERS`, `POOL-MANAGEMENT`

> [!warning] Attention au masque de sous-réseau Le masque doit correspondre au sous-réseau réel des adresses publiques que vous utilisez. Un masque incorrect peut causer des problèmes de routage.

---

## ⚙️ Configuration du NAT Dynamique

### Étapes de configuration

La mise en place du NAT dynamique suit un processus en **4 étapes principales** :

1. **Créer un pool d'adresses publiques**
2. **Créer une ACL pour identifier le trafic à traduire**
3. **Lier l'ACL au pool avec la commande NAT**
4. **Définir les interfaces inside et outside**

> [!info] Interfaces Inside et Outside
> 
> - **Inside** : Interface connectée au réseau privé (LAN)
> - **Outside** : Interface connectée au réseau public (Internet/WAN)

### Commandes essentielles

#### 1. Créer le pool NAT

```bash
Router(config)# ip nat pool NOM_POOL IP_DEBUT IP_FIN netmask MASQUE
```

#### 2. Créer l'ACL de sélection

```bash
Router(config)# access-list NUMERO permit source WILDCARD
```

ou avec ACL nommée :

```bash
Router(config)# ip access-list standard NOM_ACL
Router(config-std-nacl)# permit source WILDCARD
```

#### 3. Lier l'ACL au pool

```bash
Router(config)# ip nat inside source list ACL pool NOM_POOL
```

**Paramètres :**

- `inside source` : traduit les adresses sources provenant du réseau inside
- `list ACL` : numéro ou nom de l'ACL qui identifie le trafic
- `pool NOM_POOL` : nom du pool à utiliser

#### 4. Configurer les interfaces

```bash
Router(config)# interface TYPE NUMERO
Router(config-if)# ip nat inside

Router(config)# interface TYPE NUMERO
Router(config-if)# ip nat outside
```

### Exemple de configuration complète

#### Scénario

- **Réseau interne** : 192.168.1.0/24
- **Pool d'adresses publiques** : 203.0.113.10 à 203.0.113.19 (10 adresses)
- **Interface inside** : GigabitEthernet0/0
- **Interface outside** : GigabitEthernet0/1

#### Configuration pas à pas

```bash
! Étape 1 : Créer le pool d'adresses publiques
Router(config)# ip nat pool POOL-INTERNET 203.0.113.10 203.0.113.19 netmask 255.255.255.248

! Étape 2 : Créer l'ACL pour identifier le trafic interne
Router(config)# access-list 1 permit 192.168.1.0 0.0.0.255

! Étape 3 : Lier l'ACL au pool NAT
Router(config)# ip nat inside source list 1 pool POOL-INTERNET

! Étape 4 : Configurer l'interface inside (LAN)
Router(config)# interface GigabitEthernet0/0
Router(config-if)# ip address 192.168.1.1 255.255.255.0
Router(config-if)# ip nat inside
Router(config-if)# exit

! Étape 5 : Configurer l'interface outside (WAN)
Router(config)# interface GigabitEthernet0/1
Router(config-if)# ip address 203.0.113.1 255.255.255.248
Router(config-if)# ip nat outside
Router(config-if)# exit
```

> [!example] Fonctionnement Lorsque l'hôte 192.168.1.50 accède à Internet :
> 
> 1. Le routeur vérifie l'ACL 1 → le trafic correspond
> 2. Le routeur attribue une adresse du pool (ex: 203.0.113.10)
> 3. Les paquets sortants ont 203.0.113.10 comme adresse source
> 4. Le mapping reste actif pendant la durée de la session

#### Configuration alternative avec ACL nommée

```bash
! Utilisation d'une ACL nommée (plus lisible)
Router(config)# ip access-list standard NAT-LAN
Router(config-std-nacl)# permit 192.168.1.0 0.0.0.255
Router(config-std-nacl)# exit

Router(config)# ip nat inside source list NAT-LAN pool POOL-INTERNET
```

> [!tip] ACL numérotée vs nommée
> 
> - **ACL numérotée** (1-99, 1300-1999) : Plus rapide à taper, mais moins descriptive
> - **ACL nommée** : Plus lisible et maintenable, recommandée pour les grandes configurations

---

## 🔒 ACL pour la sélection du trafic

### Rôle des ACL dans le NAT dynamique

Les **Access Control Lists (ACL)** dans le contexte du NAT dynamique servent à **identifier quel trafic doit être traduit**. Elles agissent comme un filtre de sélection, pas comme un outil de sécurité dans ce cas précis.

> [!info] Fonction des ACL pour NAT Les ACL définissent **quelles adresses IP sources** sont éligibles pour la traduction NAT. Seul le trafic correspondant aux règles `permit` de l'ACL sera traduit.

**Caractéristiques importantes :**

- Seules les instructions `permit` sont utilisées par le NAT
- Les instructions `deny` excluent le trafic de la traduction
- Le trafic non traduit peut quand même circuler si une route existe
- L'ordre des règles est important (first-match)

### Types d'ACL utilisables

Pour le NAT dynamique, on utilise principalement des **ACL standard** car on ne se soucie que des adresses sources.

#### ACL Standard

**Numéros autorisés :** 1-99 et 1300-1999

```bash
Router(config)# access-list NUMERO permit SOURCE WILDCARD
```

**Avantages :**

- Simple et rapide à configurer
- Suffisant pour la sélection NAT (basée sur l'IP source uniquement)
- Consomme peu de ressources

#### ACL Étendue (moins courant pour NAT)

**Numéros autorisés :** 100-199 et 2000-2699

```bash
Router(config)# access-list NUMERO permit PROTOCOLE SOURCE WILDCARD DEST WILDCARD
```

**Usage pour NAT :**

- Permet de traduire seulement certains types de trafic
- Utile pour des scénarios complexes (ex: traduire seulement HTTP)

> [!tip] Recommandation Utilisez des **ACL standard** pour le NAT dynamique dans la majorité des cas. Les ACL étendues ne sont nécessaires que pour des besoins de granularité très spécifiques.

### Configuration des ACL

#### Syntaxe de l'ACL standard

```bash
Router(config)# access-list NUMERO permit ADRESSE_RESEAU WILDCARD_MASK
```

**Wildcard mask :** L'inverse du masque de sous-réseau

- `0` = doit correspondre exactement
- `255` = peut être n'importe quoi

#### Exemples d'ACL pour NAT

**Permettre un réseau entier :**

```bash
Router(config)# access-list 10 permit 192.168.1.0 0.0.0.255
```

→ Traduit toutes les adresses de 192.168.1.0 à 192.168.1.255

**Permettre plusieurs sous-réseaux :**

```bash
Router(config)# access-list 15 permit 192.168.1.0 0.0.0.255
Router(config)# access-list 15 permit 192.168.2.0 0.0.0.255
Router(config)# access-list 15 permit 192.168.3.0 0.0.0.255
```

**Permettre une plage spécifique :**

```bash
Router(config)# access-list 20 permit 10.0.0.0 0.255.255.255
```

→ Traduit tout le réseau de classe A privé (10.0.0.0/8)

**Exclure certaines adresses :**

```bash
Router(config)# access-list 25 deny 192.168.1.10 0.0.0.0
Router(config)# access-list 25 permit 192.168.1.0 0.0.0.255
```

→ Traduit tout le réseau 192.168.1.0/24 **sauf** l'hôte 192.168.1.10

> [!warning] Ordre des règles Les ACL sont évaluées de haut en bas. Une fois qu'une règle correspond, les suivantes sont ignorées. Placez toujours les règles les plus spécifiques en premier.

#### ACL nommée standard

**Syntaxe :**

```bash
Router(config)# ip access-list standard NOM_ACL
Router(config-std-nacl)# [sequence] permit SOURCE WILDCARD
Router(config-std-nacl)# [sequence] deny SOURCE WILDCARD
```

**Exemple complet :**

```bash
Router(config)# ip access-list standard NAT-ALLOWED
Router(config-std-nacl)# 10 deny 192.168.1.100 0.0.0.0
Router(config-std-nacl)# 20 permit 192.168.1.0 0.0.0.255
Router(config-std-nacl)# exit

Router(config)# ip nat inside source list NAT-ALLOWED pool INTERNET-POOL
```

> [!tip] Numéros de séquence Les numéros de séquence (10, 20, 30...) permettent d'insérer ou supprimer des règles facilement sans tout recréer. Par défaut, IOS incrémente par 10.

#### Raccourcis utiles

**Permettre un hôte unique :**

```bash
Router(config)# access-list 30 permit host 192.168.1.50
! Équivalent à :
Router(config)# access-list 30 permit 192.168.1.50 0.0.0.0
```

**Permettre tout :**

```bash
Router(config)# access-list 35 permit any
! Équivalent à :
Router(config)# access-list 35 permit 0.0.0.0 255.255.255.255
```

#### Exemple avancé : Sélection par sous-réseau

**Scénario :** Vous avez 3 départements et voulez traduire seulement les départements A et B.

```bash
! Département A : 10.1.0.0/24
! Département B : 10.2.0.0/24
! Département C : 10.3.0.0/24 (ne sera PAS traduit)

Router(config)# ip access-list standard NAT-DEPTS-A-B
Router(config-std-nacl)# permit 10.1.0.0 0.0.0.255
Router(config-std-nacl)# permit 10.2.0.0 0.0.0.255
Router(config-std-nacl)# exit

Router(config)# ip nat pool POOL-AB 203.0.113.20 203.0.113.39 netmask 255.255.255.224
Router(config)# ip nat inside source list NAT-DEPTS-A-B pool POOL-AB
```

> [!info] Trafic non traduit Le département C (10.3.0.0/24) ne sera pas traduit car il n'est pas dans l'ACL. Ce trafic peut toujours sortir si une route et une politique de sécurité le permettent, mais avec son adresse privée.

---

## 🔍 Vérification et dépannage

### Commandes de vérification

#### Afficher les traductions actives

```bash
Router# show ip nat translations
```

**Sortie exemple :**

```
Pro Inside global      Inside local       Outside local      Outside global
--- 203.0.113.10       192.168.1.50       ---                ---
--- 203.0.113.11       192.168.1.51       ---                ---
```

**Colonnes :**

- **Inside local** : Adresse privée de l'hôte interne
- **Inside global** : Adresse publique traduite du pool
- **Outside local/global** : Adresses de destination (souvent identiques)

#### Afficher les statistiques NAT

```bash
Router# show ip nat statistics
```

**Informations affichées :**

- Nombre total de traductions actives
- Nombre d'adresses utilisées dans le pool
- Nombre de traductions créées et expirées
- Configuration des pools et ACL

**Sortie exemple :**

```
Total active translations: 5 (0 static, 5 dynamic; 0 extended)
Peak translations: 8, occurred 00:12:34 ago
Outside interfaces:
  GigabitEthernet0/1
Inside interfaces:
  GigabitEthernet0/0
Hits: 1523  Misses: 0
```

#### Vérifier la configuration des pools

```bash
Router# show ip nat pool
```

**Exemple de sortie :**

```
Pool POOL-INTERNET
  netmask 255.255.255.248
  start 203.0.113.10 end 203.0.113.19
  type generic, total addresses 10, allocated 5 (50%), misses 0
```

#### Vérifier les ACL

```bash
Router# show access-lists
```

ou pour une ACL spécifique :

```bash
Router# show access-lists 1
Router# show ip access-list NAT-LAN
```

### Commandes de débogage

> [!warning] Attention avec le debug Les commandes `debug` peuvent générer beaucoup de trafic sur la console et impacter les performances. Utilisez-les avec précaution en production.

#### Activer le debug NAT

```bash
Router# debug ip nat
```

**Désactiver :**

```bash
Router# no debug ip nat
! ou
Router# undebug all
```

#### Debug détaillé

```bash
Router# debug ip nat detailed
```

### Résolution de problèmes courants

#### Problème : Aucune traduction ne se crée

**Vérifications :**

1. **ACL correcte ?**

```bash
Router# show access-lists
```

→ Vérifiez que le trafic correspond aux règles `permit`

2. **Interfaces correctement configurées ?**

```bash
Router# show ip interface brief | include nat
```

→ Vérifiez que inside/outside sont bien configurés

3. **Pool épuisé ?**

```bash
Router# show ip nat statistics
```

→ Regardez "allocated" vs "total addresses"

#### Problème : Pool d'adresses épuisé

**Symptôme :** Les nouveaux hôtes ne peuvent pas accéder à Internet

**Solutions :**

1. **Augmenter le timeout :**

```bash
Router(config)# ip nat translation timeout 300
```

(par défaut 86400 secondes = 24h)

2. **Agrandir le pool :**

```bash
Router(config)# no ip nat pool POOL-INTERNET
Router(config)# ip nat pool POOL-INTERNET 203.0.113.10 203.0.113.29 netmask 255.255.255.224
```

3. **Passer au PAT (overload) :**

```bash
Router(config)# ip nat inside source list 1 pool POOL-INTERNET overload
```

#### Effacer les traductions

```bash
Router# clear ip nat translation *
```

> [!warning] Impact Cette commande coupe toutes les sessions actives. Utilisez-la uniquement en cas de nécessité ou lors de tests.

**Effacer une traduction spécifique :**

```bash
Router# clear ip nat translation inside 192.168.1.50
```

---

## ⚠️ Pièges courants et bonnes pratiques

### Pièges courants

#### 1. Pool trop petit

❌ **Erreur :**

```bash
ip nat pool MINI 203.0.113.10 203.0.113.12 netmask 255.255.255.252
! Seulement 3 adresses pour 50 utilisateurs
```

✅ **Bonne pratique :**

- Calculez le nombre de connexions simultanées maximales attendues
- Ajoutez une marge de 20-30% pour la croissance
- Surveillez les statistiques régulièrement

#### 2. Oublier de configurer inside/outside

❌ **Erreur fréquente :**

```bash
! Configuration NAT correcte MAIS...
! Oubli de définir les interfaces
```

✅ **Toujours vérifier :**

```bash
Router# show ip interface brief
! Vérifier que "(NAT inside)" et "(NAT outside)" apparaissent
```

#### 3. ACL trop restrictive ou mal ordonnée

❌ **Erreur :**

```bash
access-list 1 permit 192.168.1.0 0.0.0.255
access-list 1 deny 192.168.1.100 0.0.0.0
! Le deny n'aura jamais d'effet car le permit est avant
```

✅ **Correct :**

```bash
access-list 1 deny 192.168.1.100 0.0.0.0
access-list 1 permit 192.168.1.0 0.0.0.255
! Les règles spécifiques AVANT les règles générales
```

#### 4. Masque de sous-réseau incorrect dans le pool

❌ **Erreur :**

```bash
ip nat pool MAUVAIS 203.0.113.10 203.0.113.19 netmask 255.255.255.0
! Le masque ne correspond pas à la plage d'adresses
```

✅ **Correct :**

```bash
ip nat pool BON 203.0.113.10 203.0.113.19 netmask 255.255.255.248
! ou prefix-length 29
```

#### 5. Confondre NAT dynamique et PAT

❌ **Mauvaise compréhension :**

- "J'ai 100 utilisateurs et 1 IP publique, je peux utiliser le NAT dynamique"
- → NON, il faut du PAT (overload)

✅ **Règle :**

- **NAT dynamique** = nombre d'hôtes ≤ nombre d'IPs publiques
- **PAT** = nombre d'hôtes > nombre d'IPs publiques

### Bonnes pratiques

#### 1. Nommage cohérent

```bash
! Utilisez des conventions de nommage claires
ip nat pool POOL-GUEST ...
ip nat pool POOL-SERVERS ...
ip nat pool POOL-MANAGEMENT ...

ip access-list standard NAT-GUEST
ip access-list standard NAT-SERVERS
```

#### 2. Documentation

```bash
! Ajoutez des commentaires dans la configuration
access-list 10 remark === ACL pour NAT dynamique - LAN principal ===
access-list 10 permit 192.168.1.0 0.0.0.255
```

#### 3. Monitoring

```bash
! Créez un script pour surveiller l'utilisation du pool
Router# show ip nat statistics | include allocated
! Configurez des alertes si > 80% utilisé
```

#### 4. Timeout adapté

```bash
! Ajustez le timeout selon vos besoins
ip nat translation timeout 1800  ! 30 minutes pour libérer plus vite
! Ou
ip nat translation timeout 3600  ! 1 heure pour plus de stabilité
```

**Valeurs recommandées :**

- **Utilisateurs web** : 1800-3600 secondes (30min - 1h)
- **Applications métier** : 7200-14400 secondes (2h - 4h)
- **Par défaut** : 86400 secondes (24h) - souvent trop long

#### 5. Segmentation par département

```bash
! Créez des pools séparés pour différents groupes
ip nat pool POOL-DIRECTION 203.0.113.10 203.0.113.14 netmask 255.255.255.248
ip nat pool POOL-EMPLOYES 203.0.113.15 203.0.113.30 netmask 255.255.255.224

ip access-list standard NAT-DIRECTION
 permit 10.10.0.0 0.0.255.255

ip access-list standard NAT-EMPLOYES
 permit 10.20.0.0 0.0.255.255

ip nat inside source list NAT-DIRECTION pool POOL-DIRECTION
ip nat inside source list NAT-EMPLOYES pool POOL-EMPLOYES
```

#### 6. Plan de secours avec PAT

```bash
! Configurer un fallback en PAT si le pool est épuisé
ip nat inside source list NAT-LAN pool POOL-INTERNET overload
! Le mot-clé "overload" active le PAT automatiquement si nécessaire
```

> [!tip] Astuce de production Combinez NAT dynamique + PAT : utilisez d'abord le pool (NAT dynamique), puis si épuisé, basculez en PAT sur une IP de secours. Cela optimise l'usage des IPs publiques tout en garantissant la continuité de service.

#### 7. Sauvegarde de la configuration

```bash
! Toujours sauvegarder après modification
Router# copy running-config startup-config
! ou
Router# write memory
```

---

## 📊 Tableau récapitulatif

|Élément|Commande|Objectif|
|---|---|---|
|**Pool NAT**|`ip nat pool NOM IP_DEBUT IP_FIN netmask MASQUE`|Définir les adresses publiques disponibles|
|**ACL standard**|`access-list NUM permit SOURCE WILDCARD`|Identifier le trafic à traduire|
|**Liaison ACL-Pool**|`ip nat inside source list ACL pool POOL`|Activer la traduction dynamique|
|**Interface inside**|`ip nat inside`|Marquer l'interface LAN|
|**Interface outside**|`ip nat outside`|Marquer l'interface WAN|
|**Vérification**|`show ip nat translations`|Voir les traductions actives|
|**Statistiques**|`show ip nat statistics`|Analyser l'utilisation du NAT|
|**Effacer traductions**|`clear ip nat translation *`|Réinitialiser les sessions|

---

## 🎓 Points clés à retenir

> [!important] Essentiels du NAT Dynamique
> 
> 1. **Pool = ensemble d'IPs publiques** disponibles pour traduction
> 2. **ACL = filtre** qui détermine quel trafic est éligible
> 3. **Limitation** : Maximum d'hôtes simultanés = nombre d'IPs dans le pool
> 4. **Ordre important** : Pool → ACL → Liaison → Interfaces
> 5. **Temporaire** : Les mappages sont créés/détruits dynamiquement
> 6. **Inside/Outside** : Toujours configurer les interfaces correctement

> [!tip] Pour aller plus loin Le NAT dynamique est idéal quand vous avez plusieurs IPs publiques. Si vous n'en avez qu'une seule ou très peu, le **PAT (overload)** sera plus approprié et sera abordé dans la section suivante du cours.

---

_Cours réalisé pour Obsidian - Configuration Routeur Cisco - NAT Dynamique_