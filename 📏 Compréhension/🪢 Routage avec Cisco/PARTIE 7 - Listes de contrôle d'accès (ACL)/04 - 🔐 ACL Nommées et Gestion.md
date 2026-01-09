

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

## 🎯 Introduction

Les **ACL nommées** offrent une alternative plus flexible et maintenable aux ACL numérotées traditionnelles. Au lieu d'utiliser un numéro (comme 1 ou 100), vous utilisez un nom descriptif qui facilite la compréhension et la gestion.

> [!info] Pourquoi utiliser des ACL nommées ?
> 
> - **Lisibilité** : Un nom descriptif (`BLOCK_SALES_TO_RH`) est plus parlant qu'un numéro (`100`)
> - **Modification facilitée** : Possibilité d'ajouter, supprimer ou modifier des entrées spécifiques
> - **Pas de limite de nombre** : Contrairement aux ACL numérotées (limites 1-99, 100-199, etc.)
> - **Gestion des séquences** : Contrôle précis de l'ordre des règles

---

## 📝 ACL Standard Nommées

### Syntaxe de base

```bash
Router(config)# ip access-list standard NOM_ACL
Router(config-std-nacl)# [permit|deny] SOURCE [WILDCARD]
Router(config-std-nacl)# exit
```

### Exemple complet

```bash
Router(config)# ip access-list standard AUTORISER_ADMIN
Router(config-std-nacl)# remark === Autoriser le reseau admin ===
Router(config-std-nacl)# permit 192.168.10.0 0.0.0.255
Router(config-std-nacl)# remark === Autoriser le poste du directeur ===
Router(config-std-nacl)# permit host 192.168.20.100
Router(config-std-nacl)# remark === Bloquer tout le reste ===
Router(config-std-nacl)# deny any
Router(config-std-nacl)# exit

! Application sur une interface
Router(config)# interface GigabitEthernet0/1
Router(config-if)# ip access-group AUTORISER_ADMIN out
```

> [!tip] Utilisation de `remark` La commande `remark` permet d'ajouter des commentaires dans votre ACL pour documenter chaque règle. C'est une excellente pratique pour la maintenance future !

### Caractéristiques

|Caractéristique|Description|
|---|---|
|**Portée**|Source uniquement|
|**Position**|Le plus près possible de la destination|
|**Mode configuration**|`config-std-nacl`|
|**Commandes disponibles**|permit, deny, remark|

> [!warning] Attention au `deny any` implicite Même en ACL nommée, il existe toujours un `deny any` implicite à la fin. Pensez à ajouter vos règles `permit` avant !

---

## 🎯 ACL Étendues Nommées

### Syntaxe de base

```bash
Router(config)# ip access-list extended NOM_ACL
Router(config-ext-nacl)# [permit|deny] PROTOCOLE SOURCE [PORTS] DESTINATION [PORTS] [OPTIONS]
Router(config-ext-nacl)# exit
```

### Exemple complet avec plusieurs services

```bash
Router(config)# ip access-list extended FILTRAGE_WEB_MAIL
Router(config-ext-nacl)# remark === Autoriser HTTP/HTTPS vers serveur web ===
Router(config-ext-nacl)# permit tcp 192.168.1.0 0.0.0.255 host 10.0.0.100 eq 80
Router(config-ext-nacl)# permit tcp 192.168.1.0 0.0.0.255 host 10.0.0.100 eq 443
Router(config-ext-nacl)# remark === Autoriser SMTP sortant ===
Router(config-ext-nacl)# permit tcp 192.168.1.0 0.0.0.255 any eq 25
Router(config-ext-nacl)# remark === Autoriser reponses etablies ===
Router(config-ext-nacl)# permit tcp any 192.168.1.0 0.0.0.255 established
Router(config-ext-nacl)# remark === Bloquer tout le reste ===
Router(config-ext-nacl)# deny ip any any log
Router(config-ext-nacl)# exit

! Application sur une interface
Router(config)# interface GigabitEthernet0/0
Router(config-if)# ip access-group FILTRAGE_WEB_MAIL in
```

### Options avancées

```bash
Router(config)# ip access-list extended SECURITE_AVANCEE
Router(config-ext-nacl)# permit tcp any any established
Router(config-ext-nacl)# permit icmp any any echo-reply
Router(config-ext-nacl)# permit icmp any any unreachable
Router(config-ext-nacl)# deny tcp any any eq 23 log
Router(config-ext-nacl)# permit tcp 10.0.0.0 0.0.0.255 any range 20 21
```

> [!example] Options utiles
> 
> - **established** : Autorise uniquement les réponses TCP (connexions déjà établies)
> - **log** : Enregistre les correspondances dans les logs du routeur
> - **eq** : Égal à (equal) - pour un port spécifique
> - **range** : Plage de ports (ex: 20 21 pour FTP)
> - **gt** : Supérieur à (greater than)
> - **lt** : Inférieur à (less than)

### Protocoles courants

|Protocole|Utilisation|
|---|---|
|`tcp`|HTTP, HTTPS, FTP, SSH, Telnet|
|`udp`|DNS, DHCP, TFTP, SNMP|
|`icmp`|Ping, traceroute|
|`ip`|Tout type de trafic IP|

---

## ✏️ Modification et Suppression d'Entrées

### Problème avec les ACL numérotées

> [!warning] Limitation des ACL numérotées Avec les ACL numérotées, vous ne pouvez pas modifier une ligne spécifique. Vous devez :
> 
> 1. Supprimer toute l'ACL (`no access-list 100`)
> 2. Recréer l'ACL complète
> 
> **Risque** : Pendant la recréation, tout le trafic est autorisé !

### Avantage des ACL nommées

Les ACL nommées permettent de **modifier des entrées spécifiques** sans tout supprimer.

#### Supprimer une entrée spécifique

```bash
! Afficher l'ACL avec les numéros de séquence
Router# show ip access-lists FILTRAGE_WEB_MAIL
Extended IP access list FILTRAGE_WEB_MAIL
    10 permit tcp 192.168.1.0 0.0.0.255 host 10.0.0.100 eq www
    20 permit tcp 192.168.1.0 0.0.0.255 host 10.0.0.100 eq 443
    30 deny ip any any log

! Entrer dans l'ACL et supprimer la ligne 20
Router(config)# ip access-list extended FILTRAGE_WEB_MAIL
Router(config-ext-nacl)# no 20
Router(config-ext-nacl)# exit
```

#### Ajouter une entrée à un emplacement précis

```bash
Router(config)# ip access-list extended FILTRAGE_WEB_MAIL
Router(config-ext-nacl)# 15 permit tcp 192.168.2.0 0.0.0.255 any eq 80
Router(config-ext-nacl)# exit
```

> [!tip] Astuce de numérotation Utilisez des intervalles de 10 (10, 20, 30...) pour pouvoir insérer facilement des règles entre deux existantes (ex: 15, 25, 35).

#### Modifier une entrée

```bash
! Supprimer l'ancienne règle
Router(config)# ip access-list extended FILTRAGE_WEB_MAIL
Router(config-ext-nacl)# no 10

! Ajouter la nouvelle règle avec le même numéro
Router(config-ext-nacl)# 10 permit tcp 192.168.1.0 0.0.0.255 host 10.0.0.200 eq www
Router(config-ext-nacl)# exit
```

### Supprimer toute l'ACL

```bash
! Méthode 1 : Depuis le mode configuration globale
Router(config)# no ip access-list extended FILTRAGE_WEB_MAIL

! Méthode 2 : Retirer d'abord de l'interface puis supprimer
Router(config)# interface GigabitEthernet0/0
Router(config-if)# no ip access-group FILTRAGE_WEB_MAIL in
Router(config-if)# exit
Router(config)# no ip access-list extended FILTRAGE_WEB_MAIL
```

> [!warning] Ordre des opérations Bonne pratique : Retirez toujours l'ACL de l'interface **avant** de la supprimer pour éviter des problèmes de trafic.

---

## 🔢 Numéros de Séquence

### Concept

Les **numéros de séquence** définissent l'**ordre d'évaluation** des règles dans une ACL. Le routeur évalue les règles de la plus petite séquence vers la plus grande, et s'arrête à la première correspondance.

> [!info] Incrémentation automatique Par défaut, IOS attribue automatiquement des numéros de séquence par **incrément de 10** :
> 
> - Première règle : 10
> - Deuxième règle : 20
> - Troisième règle : 30
> - ...

### Affichage des numéros de séquence

```bash
Router# show ip access-lists AUTORISER_ADMIN
Standard IP access list AUTORISER_ADMIN
    10 permit 192.168.10.0, wildcard bits 0.0.0.255
    20 permit 192.168.20.100
    30 deny any
```

### Spécifier manuellement un numéro de séquence

```bash
Router(config)# ip access-list extended FILTRAGE_AVANCE
Router(config-ext-nacl)# 5 permit icmp any any echo-reply
Router(config-ext-nacl)# 25 permit tcp any any eq 22
Router(config-ext-nacl)# 100 deny ip any any log
Router(config-ext-nacl)# exit
```

### Réorganiser les séquences

Si vos numéros deviennent désordonnés, vous pouvez les renuméroter :

```bash
Router(config)# ip access-list resequence NOM_ACL DEBUT INCREMENT

! Exemple : Commencer à 10, incrémenter de 10
Router(config)# ip access-list resequence FILTRAGE_WEB_MAIL 10 10
```

> [!example] Exemple de renumérotation **Avant :**
> 
> ```
> 5 permit tcp any any eq 80
> 15 permit tcp any any eq 443
> 25 deny ip any any
> 100 permit icmp any any
> ```
> 
> **Après `ip access-list resequence NOM_ACL 10 10` :**
> 
> ```
> 10 permit tcp any any eq 80
> 20 permit tcp any any eq 443
> 30 deny ip any any
> 40 permit icmp any any
> ```

### Cas d'usage des séquences

#### Insérer une règle prioritaire

```bash
! Situation : Vous avez oublié d'autoriser le trafic SSH
Router# show ip access-lists SECURITE
Extended IP access list SECURITE
    10 permit tcp any any eq 80
    20 permit tcp any any eq 443
    30 deny ip any any log

! Ajouter SSH AVANT le deny any any
Router(config)# ip access-list extended SECURITE
Router(config-ext-nacl)# 25 permit tcp any any eq 22
Router(config-ext-nacl)# exit
```

> [!tip] Numérotation stratégique
> 
> - Règles critiques/spécifiques : séquences basses (5, 10, 15)
> - Règles générales : séquences moyennes (50, 60, 70)
> - Règle de blocage finale : séquence haute (90, 100)

---

## 🔍 Vérification et Dépannage

### Commande `show access-lists`

Affiche **toutes les ACL** configurées sur le routeur.

```bash
Router# show access-lists
```

**Sortie exemple :**

```
Standard IP access list AUTORISER_ADMIN
    10 permit 192.168.10.0, wildcard bits 0.0.0.255 (5 matches)
    20 permit 192.168.20.100 (0 matches)
    30 deny any (12 matches)
Extended IP access list FILTRAGE_WEB_MAIL
    10 permit tcp 192.168.1.0 0.0.0.255 host 10.0.0.100 eq www (150 matches)
    20 permit tcp 192.168.1.0 0.0.0.255 host 10.0.0.100 eq 443 (89 matches)
    30 deny ip any any log (7 matches)
```

> [!info] Compteurs de correspondances Le nombre entre parenthèses `(X matches)` indique combien de paquets ont correspondu à cette règle. C'est très utile pour le dépannage !

### Afficher une ACL spécifique

```bash
Router# show access-lists NOM_ACL

! Exemple
Router# show access-lists FILTRAGE_WEB_MAIL
```

### Afficher uniquement les ACL IP

```bash
Router# show ip access-lists
```

### Commande `show ip interface`

Vérifie **quelle ACL est appliquée sur quelle interface** et dans quelle direction.

```bash
Router# show ip interface GigabitEthernet0/0
```

**Sortie exemple :**

```
GigabitEthernet0/0 is up, line protocol is up
  Internet address is 192.168.1.1/24
  Broadcast address is 255.255.255.255
  Address determined by setup command
  MTU is 1500 bytes
  Helper address is not set
  Directed broadcast forwarding is disabled
  Outgoing access list is not set
  Inbound access list is FILTRAGE_WEB_MAIL
  Proxy ARP is enabled
  ...
```

> [!tip] Points clés à vérifier
> 
> - **Inbound access list** : ACL appliquée en entrée (direction `in`)
> - **Outgoing access list** : ACL appliquée en sortie (direction `out`)
> - Si "not set" → Aucune ACL appliquée

### Version abrégée pour toutes les interfaces

```bash
Router# show ip interface brief
```

### Réinitialiser les compteurs

Les compteurs de correspondances s'accumulent. Pour les remettre à zéro :

```bash
Router# clear access-list counters [NOM_ACL]

! Exemple : Réinitialiser tous les compteurs
Router# clear access-list counters

! Exemple : Réinitialiser une ACL spécifique
Router# clear access-list counters FILTRAGE_WEB_MAIL
```

### Tableau récapitulatif des commandes de vérification

|Commande|But|Information affichée|
|---|---|---|
|`show access-lists`|Voir toutes les ACL|Contenu, séquences, matches|
|`show access-lists NOM`|Voir une ACL précise|Détails d'une ACL|
|`show ip access-lists`|Voir ACL IP uniquement|ACL standard/étendue|
|`show ip interface IF`|Vérifier application|ACL in/out sur interface|
|`show run|include access`|Voir config running|
|`clear access-list counters`|Reset compteurs|Remet matches à 0|

### Dépannage courant

#### Problème : L'ACL ne fonctionne pas

```bash
! 1. Vérifier que l'ACL existe
Router# show access-lists NOM_ACL

! 2. Vérifier qu'elle est appliquée
Router# show ip interface GigabitEthernet0/0

! 3. Vérifier les matches (compteurs)
Router# show access-lists NOM_ACL

! 4. Vérifier l'ordre des règles (séquences)
```

> [!warning] Erreurs fréquentes
> 
> - ACL créée mais **non appliquée** sur l'interface
> - Direction incorrecte (`in` vs `out`)
> - Règle trop générale **avant** règle spécifique
> - Oubli du `deny any` explicite si vous voulez logger

#### Problème : Trop de trafic bloqué

```bash
! Vérifier quelle règle bloque
Router# show access-lists
! Regardez les matches sur les deny

! Insérer une règle permit avant le blocage
Router(config)# ip access-list extended NOM_ACL
Router(config-ext-nacl)# 15 permit tcp any any eq 80
```

---

## ✅ Bonnes Pratiques

### 1. Conventions de nommage

> [!tip] Nommage descriptif
> 
> - **Utilisez des MAJUSCULES** : `BLOCK_TELNET`, `ALLOW_ADMIN`
> - **Soyez descriptif** : Indiquez l'objectif de l'ACL
> - **Format cohérent** : `ACTION_SOURCE_DESTINATION` ou `SERVICE_LOCATION`

**Exemples :**

```
ALLOW_ADMIN_TO_SERVERS
BLOCK_INTERNET_FROM_GUESTS
PERMIT_BRANCH_HTTP_HTTPS
DENY_TELNET_ALL
```

### 2. Documentation avec `remark`

```bash
Router(config)# ip access-list extended SECURITE_DMZ
Router(config-ext-nacl)# remark ================================
Router(config-ext-nacl)# remark Autoriser acces web au serveur DMZ
Router(config-ext-nacl)# remark ================================
Router(config-ext-nacl)# 10 permit tcp any host 203.0.113.10 eq 80
Router(config-ext-nacl)# 20 permit tcp any host 203.0.113.10 eq 443
Router(config-ext-nacl)# remark ================================
Router(config-ext-nacl)# remark Bloquer tout le reste vers DMZ
Router(config-ext-nacl)# remark ================================
Router(config-ext-nacl)# 30 deny ip any 203.0.113.0 0.0.0.255 log
```

### 3. Numérotation stratégique

> [!tip] Espacement des séquences
> 
> - Utilisez des **intervalles de 10** : 10, 20, 30...
> - Laissez de l'espace pour insérer des règles ultérieurement
> - Règles critiques en début (séquences basses)
> - Règle de refus finale en fin (séquence haute)

### 4. Ajout du `log` sur les deny

```bash
Router(config-ext-nacl)# deny ip any any log
```

Cela permet de **tracer les tentatives bloquées** dans les logs du routeur pour l'audit de sécurité.

### 5. Teste avant production

> [!warning] Toujours tester !
> 
> 1. Créez l'ACL mais ne l'appliquez pas
> 2. Vérifiez la syntaxe avec `show access-lists`
> 3. Appliquez sur l'interface
> 4. Testez la connectivité
> 5. Vérifiez les compteurs `(X matches)`
> 6. Ajustez si nécessaire

### 6. Sauvegarde de configuration

```bash
! Après toute modification d'ACL
Router# copy running-config startup-config
! ou
Router# write memory
```

### 7. Limitation des logs

> [!warning] Attention aux performances L'option `log` génère des messages pour chaque paquet. Sur du trafic intense, cela peut **ralentir le routeur**. Utilisez-la avec parcimonie, surtout sur les règles `permit` à fort trafic.

### 8. Ordre des règles

```bash
! ✅ CORRECT : Spécifique avant général
Router(config-ext-nacl)# 10 permit tcp host 192.168.1.100 any eq 80
Router(config-ext-nacl)# 20 permit tcp 192.168.1.0 0.0.0.255 any eq 80
Router(config-ext-nacl)# 30 deny ip any any

! ❌ INCORRECT : Général avant spécifique (la règle 20 ne servira jamais)
Router(config-ext-nacl)# 10 permit tcp 192.168.1.0 0.0.0.255 any eq 80
Router(config-ext-nacl)# 20 permit tcp host 192.168.1.100 any eq 80
Router(config-ext-nacl)# 30 deny ip any any
```

### 9. Maintenance régulière

- **Révisez vos ACL périodiquement** : Supprimez les règles obsolètes
- **Analysez les compteurs** : Identifiez les règles inutilisées (0 matches)
- **Documentez les changements** : Tenez un journal des modifications
- **Renumérotez si nécessaire** : Gardez une structure claire

### 10. Checkliste de déploiement

```
☐ ACL créée avec nom descriptif
☐ Commentaires remark ajoutés
☐ Règles ordonnées du spécifique au général
☐ Numérotation avec intervalles de 10
☐ deny ip any any log en dernière règle
☐ ACL vérifiée avec show access-lists
☐ Application sur la bonne interface
☐ Direction correcte (in/out)
☐ Tests de connectivité effectués
☐ Compteurs vérifiés (matches)
☐ Configuration sauvegardée
☐ Documentation mise à jour
```

---

## 🎓 Synthèse Rapide

|Aspect|ACL Standard Nommée|ACL Étendue Nommée|
|---|---|---|
|**Création**|`ip access-list standard NOM`|`ip access-list extended NOM`|
|**Filtrage**|Source uniquement|Source, destination, protocole, ports|
|**Modification**|Possible ligne par ligne|Possible ligne par ligne|
|**Séquences**|Automatiques (10, 20, 30) ou manuelles|Automatiques (10, 20, 30) ou manuelles|
|**Application**|`ip access-group NOM in/out`|`ip access-group NOM in/out`|
|**Vérification**|`show access-lists NOM`|`show access-lists NOM`|

---

## 🔑 Points Clés à Retenir

1. **ACL nommées** = Plus flexible que les numérotées pour la modification
2. **Numéros de séquence** = Contrôlent l'ordre d'évaluation (de bas en haut)
3. **Modification ligne par ligne** = Avantage majeur des ACL nommées
4. **Documentation** = Utilisez `remark` généreusement
5. **Vérification** = `show access-lists` + `show ip interface`
6. **Ordre des règles** = Du plus spécifique au plus général
7. **Sauvegarde** = Toujours faire `copy run start` après modification

> [!success] Vous maîtrisez maintenant
> 
> - La création et la gestion des ACL nommées (standard et étendues)
> - La modification précise d'entrées sans tout recréer
> - L'utilisation des numéros de séquence pour contrôler l'ordre
> - Les commandes de vérification essentielles
> - Les bonnes pratiques de nommage et de documentation