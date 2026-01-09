

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

## 🎯 Rôle et utilité des ACL

### Qu'est-ce qu'une ACL ?

Une **Access Control List (ACL)** est un ensemble de règles séquentielles qui permettent de filtrer le trafic réseau sur un équipement Cisco. Les ACL analysent les paquets et décident de les **autoriser (permit)** ou de les **refuser (deny)** selon des critères définis.

> [!info] Définition Une ACL agit comme un gardien de sécurité : elle examine chaque paquet qui passe et décide s'il peut continuer son chemin ou s'il doit être bloqué.

### Utilités principales des ACL

Les ACL sont utilisées dans plusieurs contextes :

**1. Sécurité du réseau**

- Bloquer l'accès à certaines ressources sensibles
- Empêcher le trafic malveillant ou non autorisé
- Segmenter le réseau en zones de sécurité

**2. Contrôle du trafic**

- Limiter la bande passante pour certains types de flux
- Prioriser certains protocoles ou applications
- Empêcher les broadcast storms

**3. Filtrage pour d'autres technologies**

- Définir le trafic intéressant pour les VPN
- Sélectionner les flux pour la QoS (Quality of Service)
- Contrôler les mises à jour de routage

> [!example] Exemple concret Une entreprise souhaite que les employés du département comptabilité (réseau 192.168.10.0/24) n'accèdent PAS au serveur de développement (192.168.50.10). Une ACL peut bloquer ce trafic spécifique tout en permettant le reste des communications.

### Où appliquer une ACL ?

Les ACL s'appliquent sur les **interfaces** d'un routeur, dans deux directions possibles :

|Direction|Description|Symbole|
|---|---|---|
|**Inbound (in)**|Filtre le trafic ENTRANT dans l'interface (avant le routage)|➡️|
|**Outbound (out)**|Filtre le trafic SORTANT de l'interface (après le routage)|⬅️|

> [!tip] Bonne pratique Appliquez les ACL le plus près possible de la source du trafic à filtrer pour économiser les ressources réseau. Si vous devez bloquer un réseau entier, faites-le sur le routeur le plus proche de ce réseau.

### Principe de fonctionnement

```
Paquet entrant
     ↓
[Interface avec ACL]
     ↓
Ligne 1 : Correspond ? → OUI → Action (permit/deny)
     ↓ NON
Ligne 2 : Correspond ? → OUI → Action (permit/deny)
     ↓ NON
Ligne 3 : Correspond ? → OUI → Action (permit/deny)
     ↓ NON
     ...
     ↓
Implicit DENY (refus par défaut)
```

> [!warning] Point critique Dès qu'une ligne correspond au paquet, l'action est exécutée et le traitement s'arrête. Les lignes suivantes ne sont JAMAIS évaluées pour ce paquet.

---

## 🔀 ACL standard vs ACL étendue

Il existe deux types principaux d'ACL sur les routeurs Cisco, chacun offrant un niveau de granularité différent.

### ACL Standard

Les ACL standard filtrent le trafic **uniquement sur l'adresse IP source**.

**Caractéristiques :**

- Simple à configurer
- Filtrage basique (source IP uniquement)
- Numérotation : 1-99 et 1300-1999
- Placement : proche de la **destination**

```bash
# Syntaxe de base
access-list [numéro] {permit|deny} [adresse-source] [wildcard-mask]

# Exemple : Bloquer le réseau 192.168.10.0/24
access-list 10 deny 192.168.10.0 0.0.0.255
access-list 10 permit any
```

> [!info] Wildcard mask Le wildcard mask fonctionne à l'inverse d'un masque de sous-réseau :
> 
> - **0** = ce bit doit correspondre exactement
> - **1** = ce bit peut être n'importe quoi (joker)
> 
> Exemple : `0.0.0.255` signifie "les 3 premiers octets doivent correspondre, le dernier peut être n'importe quoi"

**Quand utiliser une ACL standard ?**

- Filtrage simple basé uniquement sur la source
- Bloquer ou autoriser un réseau entier
- Contrôler l'accès à distance (telnet, SSH) sur le routeur lui-même

### ACL Étendue

Les ACL étendues offrent un filtrage beaucoup plus granulaire avec de multiples critères.

**Caractéristiques :**

- Filtrage avancé et précis
- Critères multiples : source, destination, protocole, ports
- Numérotation : 100-199 et 2000-2699
- Placement : proche de la **source**

```bash
# Syntaxe de base
access-list [numéro] {permit|deny} [protocole] [source] [wildcard-source] [destination] [wildcard-destination] [opérateurs]

# Exemple : Bloquer HTTP du réseau 192.168.10.0/24 vers le serveur 192.168.50.10
access-list 100 deny tcp 192.168.10.0 0.0.0.255 host 192.168.50.10 eq 80
access-list 100 permit ip any any
```

**Critères de filtrage disponibles :**

|Critère|Description|Exemples|
|---|---|---|
|**Protocole**|Type de protocole IP|`ip`, `tcp`, `udp`, `icmp`, `eigrp`, `ospf`|
|**Adresse source**|IP ou réseau source|`192.168.1.10`, `10.0.0.0 0.255.255.255`|
|**Adresse destination**|IP ou réseau destination|`host 8.8.8.8`, `any`|
|**Port source**|Port TCP/UDP source|`eq 1024`, `range 1024 65535`|
|**Port destination**|Port TCP/UDP destination|`eq 80`, `eq www`, `eq 443`|
|**Flags TCP**|État de connexion TCP|`established`, `syn`, `ack`|
|**Type ICMP**|Type de message ICMP|`echo`, `echo-reply`, `unreachable`|

> [!example] Exemples d'ACL étendues
> 
> ```bash
> # Autoriser uniquement le ping VERS le serveur 10.0.0.1
> access-list 101 permit icmp any host 10.0.0.1 echo
> access-list 101 permit icmp host 10.0.0.1 any echo-reply
> access-list 101 deny ip any any
> 
> # Autoriser SSH et HTTPS, bloquer le reste
> access-list 102 permit tcp any any eq 22
> access-list 102 permit tcp any any eq 443
> access-list 102 deny ip any any
> 
> # Bloquer Facebook (exemple simplifié)
> access-list 103 deny tcp any host 31.13.64.1 eq 80
> access-list 103 deny tcp any host 31.13.64.1 eq 443
> access-list 103 permit ip any any
> ```

**Quand utiliser une ACL étendue ?**

- Filtrage précis (protocole + ports spécifiques)
- Bloquer un service particulier entre deux réseaux
- Implémenter des politiques de sécurité complexes
- Sélectionner du trafic pour NAT ou VPN

### Comparaison synthétique

|Aspect|ACL Standard|ACL Étendue|
|---|---|---|
|**Critères de filtrage**|Adresse IP source uniquement|Source, destination, protocole, ports, flags|
|**Numérotation**|1-99, 1300-1999|100-199, 2000-2699|
|**Complexité**|Simple|Avancée|
|**Placement recommandé**|Près de la destination|Près de la source|
|**Flexibilité**|Limitée|Très élevée|
|**Usage typique**|Bloquer un réseau entier|Bloquer un service spécifique|

> [!tip] Astuce de placement **ACL Standard** → Près de la **destination** car elle bloque tout depuis une source donnée. Si placée trop tôt, elle bloquerait l'accès à toutes les destinations.
> 
> **ACL Étendue** → Près de la **source** pour économiser la bande passante en bloquant le trafic indésirable avant qu'il ne traverse le réseau.

---

## 🔢 Numérotation des ACL

Cisco utilise un système de numérotation pour identifier le type d'ACL et permettre de les référencer.

### Plages de numérotation

|Type d'ACL|Plage standard|Plage étendue|Total possible|
|---|---|---|---|
|**ACL Standard**|1 - 99|1300 - 1999|799 ACL|
|**ACL Étendue**|100 - 199|2000 - 2699|800 ACL|

> [!info] Pourquoi deux plages ? Cisco a introduit les plages étendues (1300-1999 et 2000-2699) pour offrir plus de numéros disponibles, car 99 ou 199 ACL peuvent être insuffisants dans de grands réseaux.

### ACL nommées

Au lieu d'utiliser un numéro, vous pouvez donner un **nom** à votre ACL, ce qui améliore la lisibilité.

**Avantages des ACL nommées :**

- Nom descriptif et significatif (ex: `BLOCK_COMPTABILITE`)
- Plus facile à mémoriser et à gérer
- Permet la modification de lignes individuelles
- Permet l'insertion de lignes à des positions spécifiques

```bash
# ACL standard nommée
ip access-list standard AUTORISER_ADMIN
 permit 192.168.1.0 0.0.0.255
 deny any

# ACL étendue nommée
ip access-list extended FILTRAGE_WEB
 deny tcp 192.168.10.0 0.0.0.255 any eq 80
 deny tcp 192.168.10.0 0.0.0.255 any eq 443
 permit ip any any
```

> [!tip] Convention de nommage Utilisez des noms en MAJUSCULES avec des underscores (_) qui décrivent clairement le rôle de l'ACL :
> 
> - `BLOCK_GUEST_NETWORK`
> - `ALLOW_MANAGEMENT`
> - `VPN_INTERESTING_TRAFFIC`

### Comparaison : Numérotées vs Nommées

|Aspect|ACL numérotée|ACL nommée|
|---|---|---|
|**Identification**|Numéro (ex: 10, 100)|Nom (ex: BLOCK_WEB)|
|**Lisibilité**|Faible|Excellente|
|**Modification**|Difficile (recréation complète)|Facile (ligne par ligne)|
|**Syntaxe**|`access-list 10 permit...`|`ip access-list standard NOM`|
|**Usage recommandé**|Petites configs simples|Configurations complexes|

> [!warning] Attention Une fois créée, une ACL numérotée ne peut pas être modifiée ligne par ligne en IOS classique. Vous devez :
> 
> 1. Supprimer toute l'ACL (`no access-list 10`)
> 2. La recréer entièrement
> 
> Les ACL nommées permettent d'ajouter/supprimer des lignes individuellement.

### Gestion avancée des ACL nommées

Avec les ACL nommées, vous pouvez utiliser des **numéros de séquence** pour insérer des règles à des positions précises.

```bash
# Afficher les numéros de séquence
Router# show ip access-lists FILTRAGE_WEB

Extended IP access list FILTRAGE_WEB
    10 deny tcp 192.168.10.0 0.0.0.255 any eq www
    20 deny tcp 192.168.10.0 0.0.0.255 any eq 443
    30 permit ip any any

# Insérer une nouvelle règle entre 10 et 20
Router(config)# ip access-list extended FILTRAGE_WEB
Router(config-ext-nacl)# 15 deny tcp 192.168.10.0 0.0.0.255 any eq 8080

# Supprimer la ligne 20
Router(config-ext-nacl)# no 20
```

> [!tip] Astuce professionnelle Utilisez des numéros de séquence espacés (10, 20, 30...) plutôt que consécutifs (1, 2, 3...) pour pouvoir insérer facilement de nouvelles règles entre les existantes.

---

## 🔄 Ordre de traitement et logique implicite deny

### Ordre de traitement séquentiel

Les ACL traitent les paquets selon un principe **top-down** (de haut en bas) et **first-match** (première correspondance).

**Principe fondamental :**

1. Le paquet est comparé à la **première ligne** de l'ACL
2. Si le paquet correspond → l'action (permit/deny) est exécutée et le traitement **s'arrête**
3. Si le paquet ne correspond pas → passage à la **ligne suivante**
4. Ce processus continue jusqu'à trouver une correspondance
5. Si aucune ligne ne correspond → **implicit deny** (refus automatique)

```bash
# Exemple d'ordre de traitement
access-list 100 deny tcp host 192.168.1.10 any eq 80        # Ligne 1
access-list 100 permit tcp 192.168.1.0 0.0.0.255 any eq 80  # Ligne 2
access-list 100 deny tcp any any eq 80                      # Ligne 3
access-list 100 permit ip any any                           # Ligne 4
# (implicit deny all) ← Ligne invisible mais toujours présente
```

**Analyse d'un paquet source 192.168.1.10 vers port 80 :**

- Ligne 1 : Correspond ! → Action = DENY → **Paquet bloqué, traitement terminé**
- Lignes 2, 3, 4 : Ne seront JAMAIS évaluées pour ce paquet

**Analyse d'un paquet source 192.168.1.20 vers port 80 :**

- Ligne 1 : Ne correspond pas (IP différente) → Ligne suivante
- Ligne 2 : Correspond ! → Action = PERMIT → **Paquet autorisé, traitement terminé**
- Lignes 3, 4 : Ne seront JAMAIS évaluées pour ce paquet

> [!warning] Piège fréquent L'ordre des lignes est CRITIQUE. Une règle trop générale placée en premier peut rendre les règles suivantes inutiles.
> 
> ```bash
> # ❌ MAUVAIS EXEMPLE
> access-list 110 permit ip any any           # Cette ligne autorise TOUT
> access-list 110 deny tcp any any eq 80      # Cette ligne ne sera JAMAIS évaluée !
> 
> # ✅ BON EXEMPLE
> access-list 110 deny tcp any any eq 80      # Blocage spécifique en premier
> access-list 110 permit ip any any           # Autorisation générale en dernier
> ```

### L'implicit deny (deny any any)

À la fin de **toute ACL**, Cisco ajoute automatiquement une règle invisible :

```bash
deny ip any any
```

Cette règle est **implicite** : elle n'apparaît pas dans la configuration mais elle est toujours active.

> [!info] Pourquoi l'implicit deny ? Cette logique suit le principe de sécurité **"deny by default"** (refuser par défaut). Seul le trafic explicitement autorisé peut passer. C'est l'approche la plus sûre en sécurité réseau.

**Conséquences pratiques :**

```bash
# Cette ACL bloque TOUT le trafic !
access-list 10 deny 192.168.10.0 0.0.0.255
# (implicit deny any) ← Bloque tout le reste

# Pour permettre le reste du trafic, il FAUT ajouter :
access-list 10 deny 192.168.10.0 0.0.0.255
access-list 10 permit any                      # ← Obligatoire !
```

> [!example] Visualisation de l'implicit deny
> 
> ```bash
> Router# show access-lists 10
> Standard IP access list 10
>     10 deny   192.168.10.0, wildcard bits 0.0.0.255
>     20 permit any
>     (implicit deny)  ← Affiché pour information
> ```

### Patterns courants et bonnes pratiques

**Pattern 1 : Blacklist (liste noire)** Bloquer des sources/destinations spécifiques, autoriser le reste.

```bash
# Bloquer deux réseaux, autoriser le reste
access-list 20 deny 192.168.10.0 0.0.0.255
access-list 20 deny 172.16.0.0 0.0.255.255
access-list 20 permit any  # ← ESSENTIEL sinon tout est bloqué
```

**Pattern 2 : Whitelist (liste blanche)** Autoriser des sources/destinations spécifiques, bloquer le reste.

```bash
# Autoriser deux réseaux, bloquer le reste
access-list 30 permit 192.168.1.0 0.0.0.255
access-list 30 permit 192.168.2.0 0.0.0.255
# (implicit deny any) ← Bloque automatiquement tout le reste
```

**Pattern 3 : Bloquer un service, autoriser le reste**

```bash
# Bloquer HTTP et HTTPS, autoriser le reste
access-list 120 deny tcp any any eq 80
access-list 120 deny tcp any any eq 443
access-list 120 permit ip any any  # ← Autorise tout le reste
```

> [!tip] Règle d'or Placez toujours les règles **du plus spécifique au plus général** :
> 
> 1. Blocages/autorisations d'hôtes spécifiques (host)
> 2. Blocages/autorisations de sous-réseaux
> 3. Règle générale "permit any" ou laisser l'implicit deny
> 
> Plus la règle est spécifique, plus elle doit être haute dans l'ACL.

### Debugging : Comment vérifier l'ordre de traitement ?

```bash
# Afficher l'ACL avec les compteurs de hits (paquets correspondants)
Router# show access-lists 100
Extended IP access list 100
    10 deny tcp host 192.168.1.10 any eq www (5 matches)
    20 permit tcp 192.168.1.0 0.0.0.255 any eq www (142 matches)
    30 deny tcp any any eq www (0 matches)
    40 permit ip any any (25896 matches)
```

Les **matches** indiquent combien de paquets ont correspondu à chaque ligne. Si une ligne a 0 match alors qu'elle devrait en avoir, c'est qu'une règle précédente intercepte déjà ces paquets.

> [!warning] Pièges courants **Piège 1 : Règle générale en premier**
> 
> ```bash
> # ❌ Le "permit any" rend les autres lignes inutiles
> access-list 40 permit any
> access-list 40 deny 192.168.10.0 0.0.0.255
> ```
> 
> **Piège 2 : Oublier le "permit any" final**
> 
> ```bash
> # ❌ Cette ACL bloque TOUT sauf 192.168.1.10
> access-list 50 permit host 192.168.1.10
> # (implicit deny) ← Bloque tout le reste !
> ```
> 
> **Piège 3 : ACL vide appliquée**
> 
> ```bash
> # ❌ Une ACL sans règles bloque TOUT (implicit deny)
> ip access-list extended VIDE
> # (pas de règles)
> # (implicit deny)
> ```

### Application d'une ACL sur une interface

Une fois l'ACL créée, elle doit être **appliquée sur une interface** pour être active.

```bash
# Créer l'ACL
Router(config)# access-list 10 permit 192.168.1.0 0.0.0.255
Router(config)# access-list 10 deny any

# L'appliquer sur une interface
Router(config)# interface GigabitEthernet0/0
Router(config-if)# ip access-group 10 in   # Trafic ENTRANT
# OU
Router(config-if)# ip access-group 10 out  # Trafic SORTANT
```

> [!info] Une ACL par direction Sur une même interface, vous pouvez avoir :
> 
> - **Une seule ACL inbound (in)**
> - **Une seule ACL outbound (out)**
> 
> Si vous appliquez une nouvelle ACL dans la même direction, elle remplace l'ancienne.

---

## 📊 Récapitulatif visuel

```
┌─────────────────────────────────────────────────────────────┐
│                    FLUX D'UN PAQUET                         │
└─────────────────────────────────────────────────────────────┘

Paquet arrive sur l'interface
         ↓
    [ACL Inbound ?]
         ↓ OUI
    ┌────────────────┐
    │ Ligne 1 match? │ → OUI → [PERMIT] → Continue
    │       ↓ NON    │              ↓
    │ Ligne 2 match? │         [DENY] → Bloqué ❌
    │       ↓ NON    │
    │      ...       │
    │       ↓ NON    │
    │ Implicit DENY  │ → Bloqué ❌
    └────────────────┘
         ↓
    [Décision de routage]
         ↓
    [ACL Outbound ?]
         ↓ OUI
    (Même processus)
         ↓
    Paquet sort de l'interface ✅
```

> [!tip] Mémo final
> 
> - **ACL Standard** : Filtre sur IP source → Placer près de la destination
> - **ACL Étendue** : Filtre multi-critères → Placer près de la source
> - **Ordre crucial** : Spécifique → Général, first-match gagne
> - **Implicit deny** : Toujours présent en fin d'ACL
> - **Permit any** : Obligatoire si vous voulez autoriser le reste du trafic
> - **ACL nommées** : Préférables pour la maintenance et la lisibilité