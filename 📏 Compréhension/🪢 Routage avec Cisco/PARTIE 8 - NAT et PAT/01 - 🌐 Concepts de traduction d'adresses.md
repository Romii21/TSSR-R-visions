

> [!info] À propos de ce cours Ce cours couvre les concepts fondamentaux de la traduction d'adresses (NAT/PAT) sur les routeurs Cisco. Vous apprendrez à comprendre les différents types d'adresses et la terminologie essentielle.

---

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

## 🏠 Adresses privées vs publiques (RFC 1918)

### Contexte et problématique

Avec l'explosion d'Internet dans les années 1990, il est devenu évident que les adresses IPv4 publiques (environ 4,3 milliards au total) seraient insuffisantes pour connecter tous les appareils dans le monde. La RFC 1918 a introduit le concept d'**adresses IP privées** pour résoudre ce problème.

### Les plages d'adresses privées

> [!info] RFC 1918 - Plages réservées Trois plages d'adresses IPv4 ont été réservées pour un usage privé (non routables sur Internet) :

|Classe|Plage d'adresses|Masque par défaut|Nombre d'adresses|
|---|---|---|---|
|**Classe A**|`10.0.0.0` - `10.255.255.255`|`/8` (255.0.0.0)|~16,7 millions|
|**Classe B**|`172.16.0.0` - `172.31.255.255`|`/12` (255.240.0.0)|~1 million|
|**Classe C**|`192.168.0.0` - `192.168.255.255`|`/16` (255.255.0.0)|~65 000|

### Caractéristiques des adresses privées

**Adresses privées** :

- ✅ Utilisables librement dans un réseau local (LAN)
- ✅ Peuvent être réutilisées dans différentes organisations
- ✅ Ne nécessitent aucune autorisation d'un organisme
- ❌ **Non routables** sur Internet public
- ❌ Nécessitent une traduction (NAT) pour accéder à Internet

**Adresses publiques** :

- ✅ Routables sur Internet
- ✅ Uniques mondialement (attribuées par les RIR)
- ❌ Ressource limitée et coûteuse
- ❌ Nécessitent une attribution officielle

> [!example] Exemple concret Une entreprise peut utiliser le réseau `192.168.1.0/24` en interne, tout comme des millions d'autres entreprises dans le monde. Ces adresses ne se "voient" jamais entre elles sur Internet car elles sont isolées dans leurs réseaux respectifs.

### Pourquoi cette distinction est cruciale ?

1. **Conservation des adresses IPv4** : Permet à des millions de réseaux privés d'utiliser les mêmes plages
2. **Sécurité** : Les machines privées ne sont pas directement accessibles depuis Internet
3. **Flexibilité** : Vous pouvez restructurer votre réseau interne sans affecter votre présence Internet
4. **Coût** : Pas besoin d'acheter des adresses IP publiques pour chaque appareil

> [!warning] Attention Les adresses privées doivent **obligatoirement** être traduites en adresses publiques via NAT/PAT pour communiquer sur Internet. Sans cette traduction, les paquets seront rejetés par les routeurs Internet.

---

## 🔄 Rôle du NAT/PAT

### Qu'est-ce que le NAT ?

Le **NAT (Network Address Translation)** est un mécanisme qui permet de traduire des adresses IP privées en adresses IP publiques, et inversement. C'est la technologie qui permet aux réseaux privés de communiquer avec Internet.

### Les différents types de NAT

#### 1. NAT Statique (Static NAT)

**Principe** : Correspondance fixe **un-pour-un** entre une adresse privée et une adresse publique.

```
Adresse privée     ←→     Adresse publique
192.168.1.10       ←→     203.0.113.10
192.168.1.11       ←→     203.0.113.11
```

> [!tip] Quand l'utiliser ?
> 
> - Pour des serveurs internes qui doivent être accessibles depuis Internet (serveur web, mail, FTP)
> - Lorsque vous avez suffisamment d'adresses IP publiques
> - Pour maintenir une correspondance permanente

**Avantages** :

- Traduction prévisible et permanente
- Idéal pour héberger des services

**Inconvénients** :

- Consomme une adresse publique par machine
- Coûteux si vous avez beaucoup d'appareils

#### 2. NAT Dynamique (Dynamic NAT)

**Principe** : Affectation temporaire d'une adresse publique depuis un **pool** d'adresses disponibles.

```
Pool d'adresses publiques : 203.0.113.10 - 203.0.113.20 (11 adresses)
                            ↕
Réseau privé : 192.168.1.0/24 (jusqu'à 254 hôtes)
```

Lorsqu'une machine privée initie une connexion, elle "emprunte" une adresse publique du pool. Une fois la session terminée, l'adresse retourne dans le pool.

> [!warning] Limitation importante Si toutes les adresses du pool sont utilisées, les nouvelles connexions échoueront jusqu'à ce qu'une adresse se libère.

**Avantages** :

- Plus économe en adresses publiques que le NAT statique
- Flexible

**Inconvénients** :

- Nécessite tout de même plusieurs adresses publiques
- Possibilité de saturation du pool

#### 3. PAT - Port Address Translation (NAT Overload)

**Principe** : Utilise **une seule adresse IP publique** pour traduire plusieurs adresses privées en utilisant des **ports sources différents**.

```
Connexions internes                    Connexion Internet
192.168.1.10:3500  ──┐
192.168.1.11:3501  ──┤
192.168.1.12:3502  ──┼──→  203.0.113.1:3500
192.168.1.13:3503  ──┤     203.0.113.1:3501
192.168.1.14:3504  ──┘     203.0.113.1:3502
                           203.0.113.1:3503
                           203.0.113.1:3504
```

Le routeur maintient une **table de correspondance** entre :

- (IP privée + port source) ←→ (IP publique + nouveau port source)

> [!tip] PAT : La solution la plus utilisée Le PAT (aussi appelé **NAT Overload**) est de loin la méthode la plus répandue dans les réseaux domestiques et d'entreprise. C'est ce que fait votre box Internet !

**Avantages** :

- ✅ Nécessite **une seule adresse IP publique**
- ✅ Peut gérer des milliers de connexions simultanées (65 000+ ports disponibles)
- ✅ Solution la plus économique
- ✅ Excellente sécurité (machines internes masquées)

**Inconvénients** :

- ❌ Les connexions doivent être initiées depuis l'intérieur (problème pour héberger des services)
- ❌ Peut poser des problèmes avec certains protocoles (FTP, VoIP, jeux en ligne)

### Fonctionnement détaillé du NAT/PAT

#### Flux sortant (Inside → Outside)

1. **Machine interne** envoie un paquet vers Internet
    
    - Source : `192.168.1.10:3500`
    - Destination : `8.8.8.8:80` (serveur Google)
2. **Routeur NAT** reçoit le paquet sur l'interface interne
    
    - Remplace l'IP source `192.168.1.10:3500`
    - Par l'IP publique `203.0.113.1:50000`
    - Enregistre cette correspondance dans sa **table NAT**
3. **Internet** reçoit le paquet avec l'adresse publique
    
    - Source : `203.0.113.1:50000`
    - Destination : `8.8.8.8:80`

#### Flux entrant (Outside → Inside)

1. **Serveur distant** répond au paquet
    
    - Source : `8.8.8.8:80`
    - Destination : `203.0.113.1:50000`
2. **Routeur NAT** consulte sa table
    
    - Trouve que `203.0.113.1:50000` correspond à `192.168.1.10:3500`
    - Remplace l'IP destination par l'adresse privée
3. **Machine interne** reçoit la réponse
    
    - Source : `8.8.8.8:80`
    - Destination : `192.168.1.10:3500`

> [!info] Table NAT Le routeur maintient une **table de traduction** dynamique qui expire après un timeout d'inactivité (généralement 1-5 minutes). Cette table peut contenir des milliers d'entrées.

### Rôle de sécurité du NAT

Bien que le NAT ne soit **pas un pare-feu**, il offre une couche de sécurité par obscurcissement :

- 🛡️ **Masquage** : Les adresses internes sont invisibles depuis Internet
- 🛡️ **Filtrage implicite** : Les connexions non sollicitées depuis l'extérieur sont bloquées
- 🛡️ **Isolation** : Même en cas de compromission d'une IP publique, les machines internes restent séparées

> [!warning] NAT ≠ Pare-feu Le NAT n'est PAS une solution de sécurité complète. Il doit être combiné avec :
> 
> - ACL (Access Control Lists)
> - Pare-feu applicatif
> - IDS/IPS
> - Politiques de sécurité appropriées

---

## 📖 Terminologie NAT

Comprendre la terminologie Cisco est **essentiel** pour configurer et dépanner le NAT. Cisco utilise quatre termes spécifiques qui peuvent sembler déroutants au début.

### Les quatre concepts fondamentaux

```
┌─────────────────────────────────────────────────────────────┐
│                    RÉSEAU INTERNE                           │
│  Inside Local          Inside Global                        │
│  (Adresse réelle)      (Adresse traduite)                   │
│                                                              │
│  192.168.1.10    ─────────────►   203.0.113.1               │
│                        NAT                                   │
│                     ROUTEUR                                  │
│                                                              │
└─────────────────────────────────────────────────────────────┘
                            │
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│                    INTERNET / WAN                           │
│  Outside Global        Outside Local                        │
│  (Adresse réelle)      (Adresse traduite)                   │
│                                                              │
│  8.8.8.8        ◄─────────────     8.8.8.8                  │
│                        NAT                                   │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

### 1. Inside Local (IL)

> [!info] Définition **Inside Local** = L'adresse IP **réelle** attribuée à un hôte sur le réseau **interne**, **avant** traduction NAT.

- 📍 **Localisation** : Réseau privé (LAN)
- 🏷️ **Type** : Généralement une adresse privée (RFC 1918)
- 👁️ **Visibilité** : Connue uniquement en interne

**Exemple** : `192.168.1.10`

> [!example] Cas d'usage C'est l'adresse que vous configurez sur votre PC, celle que vous voyez avec `ipconfig` ou `ifconfig`.

### 2. Inside Global (IG)

> [!info] Définition **Inside Global** = L'adresse IP **publique** qui représente une ou plusieurs adresses Inside Local, **après** traduction NAT.

- 📍 **Localisation** : Vue depuis l'extérieur (Internet)
- 🏷️ **Type** : Adresse publique routable
- 👁️ **Visibilité** : Visible sur Internet
- 🔄 **Utilisation** : C'est l'adresse source vue par les serveurs distants

**Exemple** : `203.0.113.1`

> [!example] Cas d'usage C'est l'adresse IP que voit le serveur web quand vous vous connectez depuis votre réseau. Vous pouvez la voir en visitant "whatismyip.com".

### 3. Outside Global (OG)

> [!info] Définition **Outside Global** = L'adresse IP **réelle** attribuée à un hôte sur le réseau **externe** (Internet), telle qu'elle est connue depuis Internet.

- 📍 **Localisation** : Internet / WAN
- 🏷️ **Type** : Adresse publique routable
- 👁️ **Visibilité** : Adresse "officielle" du serveur distant
- 🔄 **Traduction** : Généralement **non traduite** (reste identique)

**Exemple** : `8.8.8.8` (serveur DNS Google)

> [!example] Cas d'usage L'adresse IP d'un serveur web externe (comme google.com, facebook.com). Dans la plupart des cas, cette adresse n'est pas modifiée par le NAT.

### 4. Outside Local (OL)

> [!info] Définition **Outside Local** = L'adresse IP d'un hôte externe **telle qu'elle apparaît** sur le réseau interne, **après** traduction (si une traduction est effectuée).

- 📍 **Localisation** : Vue depuis l'intérieur
- 🏷️ **Type** : Peut être identique à Outside Global (cas le plus fréquent)
- 🔄 **Traduction** : Utilisée dans des scénarios avancés (NAT bidirectionnel, overlapping)

**Exemple** : Généralement `8.8.8.8` (identique à Outside Global)

> [!warning] Concept avancé Dans 95% des cas, **Outside Local = Outside Global**. La distinction devient importante uniquement dans des scénarios complexes comme :
> 
> - Réseaux avec adresses qui se chevauchent (overlapping)
> - NAT bidirectionnel
> - Connexions entre deux réseaux privés via Internet

### Tableau récapitulatif

|Terme|Direction|Localisation|Type d'adresse|Traduction|Exemple|
|---|---|---|---|---|---|
|**Inside Local**|Interne|Réseau privé|Privée (généralement)|AVANT NAT|`192.168.1.10`|
|**Inside Global**|Interne → Externe|Interface publique|Publique|APRÈS NAT|`203.0.113.1`|
|**Outside Global**|Externe|Internet|Publique|Adresse réelle|`8.8.8.8`|
|**Outside Local**|Externe → Interne|Vue interne|Variable|Après traduction (rare)|`8.8.8.8`|

### Exemple de flux complet

> [!example] Scénario : Navigation web depuis un PC interne
> 
> **Configuration** :
> 
> - PC interne : `192.168.1.10` (Inside Local)
> - Interface publique routeur : `203.0.113.1` (Inside Global)
> - Serveur web Google : `8.8.8.8` (Outside Global = Outside Local)
> 
> **Flux sortant** :
> 
> 1. PC envoie un paquet :
>     - Source : `192.168.1.10:3500` (Inside Local)
>     - Destination : `8.8.8.8:80` (Outside Local)
> 2. Routeur traduit (NAT/PAT) :
>     - Source : `203.0.113.1:50000` (Inside Global)
>     - Destination : `8.8.8.8:80` (Outside Global)
> 3. Serveur Google reçoit :
>     - Il voit la source comme `203.0.113.1:50000`
>     - Il répond à cette adresse
> 
> **Flux entrant** : 4. Routeur reçoit la réponse :
> 
> - Source : `8.8.8.8:80` (Outside Global)
>     
> - Destination : `203.0.113.1:50000` (Inside Global)
>     
> 
> 5. Routeur consulte sa table NAT et traduit :
>     - Source : `8.8.8.8:80` (Outside Local)
>     - Destination : `192.168.1.10:3500` (Inside Local)
> 6. PC reçoit la réponse avec les bonnes adresses

### Mnémotechnique pour retenir

> [!tip] Astuce mémorisation
> 
> - **Inside** = Votre réseau (ce que vous contrôlez)
> - **Outside** = Internet (ce que vous ne contrôlez pas)
> - **Local** = Adresse RÉELLE (avant traduction)
> - **Global** = Adresse TRADUITE ou vue depuis l'extérieur (après traduction)
> 
> **Formule simple** :
> 
> - Inside + Local = **Adresse de MA machine RÉELLE**
> - Inside + Global = **Adresse de MA machine vue sur INTERNET**
> - Outside + Global = **Adresse RÉELLE du serveur DISTANT**
> - Outside + Local = **Adresse du serveur distant vue depuis MON réseau**

### Utilisation dans les commandes Cisco

Lorsque vous configurez le NAT sur Cisco IOS, vous utiliserez ces termes :

```bash
# Définir l'interface interne (où sont les Inside Local)
Router(config)# interface GigabitEthernet0/0
Router(config-if)# ip nat inside

# Définir l'interface externe (où sont les Outside Global)
Router(config)# interface GigabitEthernet0/1
Router(config-if)# ip nat outside

# Vérifier les traductions actives
Router# show ip nat translations
```

La sortie de `show ip nat translations` affichera ces quatre colonnes :

- **Inside global** : L'IP publique utilisée
- **Inside local** : L'IP privée d'origine
- **Outside local** : L'IP du serveur distant (vue interne)
- **Outside global** : L'IP réelle du serveur distant

> [!info] Dans la pratique Comprendre cette terminologie est crucial pour :
> 
> - Configurer correctement les règles NAT
> - Déboguer les problèmes de connectivité
> - Analyser les logs et les tables de traduction
> - Communiquer avec d'autres administrateurs réseau

---

## 🎯 Points clés à retenir

> [!tip] Résumé essentiel
> 
> **Adresses privées (RFC 1918)** :
> 
> - Trois plages : `10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16`
> - Non routables sur Internet, nécessitent NAT/PAT
> - Réutilisables dans différentes organisations
> 
> **Types de NAT** :
> 
> - **NAT Statique** : 1 IP privée ↔ 1 IP publique (permanente)
> - **NAT Dynamique** : Pool d'IPs publiques partagées temporairement
> - **PAT (NAT Overload)** : Plusieurs IPs privées → 1 IP publique (avec ports)
> 
> **Terminologie** :
> 
> - **Inside Local** : IP réelle interne (avant NAT)
> - **Inside Global** : IP publique (après NAT)
> - **Outside Global** : IP réelle du serveur distant
> - **Outside Local** : IP du serveur distant vue depuis l'intérieur

---

## ⚠️ Pièges courants

> [!warning] Erreurs fréquentes
> 
> 1. **Confondre les termes** Local/Global
>     - Local = adresse RÉELLE
>     - Global = adresse TRADUITE ou vue depuis l'extérieur
> 2. **Oublier que Outside Local ≈ Outside Global**
>     - Dans 95% des cas, ces deux adresses sont identiques
> 3. **Penser que NAT = Sécurité**
>     - Le NAT offre de l'obscurcissement, pas une vraie protection
>     - Toujours utiliser des ACL et pare-feux en complément
> 4. **Utiliser des adresses privées en Outside**
>     - Les adresses `192.168.x.x` ne sont PAS routables sur Internet
>     - Vérifier que l'interface externe utilise une adresse publique valide
> 5. **Saturer le pool NAT dynamique**
>     - Dimensionner correctement le pool d'adresses publiques
>     - Préférer PAT si le nombre d'adresses publiques est limité

---

_Cours créé pour Obsidian - Configuration Routeur Cisco_