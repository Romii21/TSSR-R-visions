

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

## Introduction au NAT Statique

Le **NAT statique** (Static NAT) établit une correspondance **permanente** et **un-à-un** entre une adresse IP privée interne et une adresse IP publique externe. Contrairement au NAT dynamique ou au PAT, cette translation est fixe et bidirectionnelle.

> [!info] Pourquoi utiliser le NAT statique ?
> 
> - **Accessibilité permanente** : Les serveurs internes doivent être joignables depuis Internet avec la même IP publique
> - **Translation bidirectionnelle** : Fonctionne aussi bien pour les connexions entrantes que sortantes
> - **Simplicité** : Idéal pour un nombre limité de serveurs publics (web, mail, DNS, etc.)

### Différence avec les autres types de NAT

|Type|Mapping|Direction|Usage typique|
|---|---|---|---|
|**NAT Statique**|1:1 fixe|Bidirectionnel|Serveurs publics|
|NAT Dynamique|1:1 temporaire|Unidirectionnel|Pool d'adresses publiques|
|PAT (NAT Overload)|N:1 avec ports|Unidirectionnel|Poste clients|

---

## Concepts Fondamentaux

### Terminologie Inside/Outside

Le NAT Cisco utilise une terminologie spécifique pour identifier les adresses et interfaces :

```
[Réseau Interne] ←→ [Routeur Cisco] ←→ [Internet]
   (Inside)              (NAT)          (Outside)
```

> [!example] Types d'adresses
> 
> - **Inside Local** : Adresse IP privée réelle du device interne (ex: 192.168.1.10)
> - **Inside Global** : Adresse IP publique représentant le device interne (ex: 203.0.113.5)
> - **Outside Global** : Adresse IP publique réelle du device externe (ex: 8.8.8.8)
> - **Outside Local** : Adresse IP vue par le réseau interne du device externe (généralement identique à Outside Global)

### Flux de translation

```
Client Internet (8.8.8.8) → Serveur Web Public
                              ↓
                         203.0.113.5 (Inside Global)
                              ↓ [Translation NAT]
                         192.168.1.10 (Inside Local)
                              ↓
                         Serveur Web Interne
```

---

## Interfaces Inside/Outside

### Définition des interfaces

Avant toute configuration NAT, vous devez **obligatoirement** définir quelles interfaces sont côté interne et côté externe.

> [!warning] Prérequis Sans la définition des interfaces inside/outside, le NAT ne fonctionnera pas, même si les mappings sont configurés !

### Configuration des interfaces

```bash
Router(config)# interface GigabitEthernet0/0
Router(config-if)# ip address 192.168.1.1 255.255.255.0
Router(config-if)# ip nat inside
Router(config-if)# no shutdown
Router(config-if)# exit

Router(config)# interface GigabitEthernet0/1
Router(config-if)# ip address 203.0.113.1 255.255.255.248
Router(config-if)# ip nat outside
Router(config-if)# no shutdown
Router(config-if)# exit
```

> [!tip] Astuce
> 
> - **Inside** = Interface connectée au réseau privé (LAN)
> - **Outside** = Interface connectée à Internet ou au réseau public (WAN)

### Vérification des interfaces

```bash
Router# show ip nat statistics
```

Cette commande affiche notamment les interfaces configurées pour le NAT.

---

## Configuration du NAT Statique

### Syntaxe de base

```bash
Router(config)# ip nat inside source static [IP_INSIDE_LOCAL] [IP_INSIDE_GLOBAL]
```

- **IP_INSIDE_LOCAL** : Adresse IP privée du serveur interne
- **IP_INSIDE_GLOBAL** : Adresse IP publique exposée

### Exemple pratique : Serveur Web

Vous avez un serveur web interne avec l'IP `192.168.1.10` que vous voulez exposer sur Internet avec l'IP publique `203.0.113.5`.

```bash
! Étape 1 : Définir les interfaces
Router(config)# interface GigabitEthernet0/0
Router(config-if)# ip nat inside
Router(config-if)# exit

Router(config)# interface GigabitEthernet0/1
Router(config-if)# ip nat outside
Router(config-if)# exit

! Étape 2 : Créer le mapping statique
Router(config)# ip nat inside source static 192.168.1.10 203.0.113.5
```

> [!example] Résultat
> 
> - Tout paquet provenant de `192.168.1.10` sortant vers Internet aura `203.0.113.5` comme IP source
> - Tout paquet provenant d'Internet vers `203.0.113.5` sera routé vers `192.168.1.10`

### Configuration complète avec plusieurs serveurs

```bash
Router(config)# interface GigabitEthernet0/0
Router(config-if)# description LAN Interne
Router(config-if)# ip address 192.168.1.1 255.255.255.0
Router(config-if)# ip nat inside
Router(config-if)# no shutdown
Router(config-if)# exit

Router(config)# interface GigabitEthernet0/1
Router(config-if)# description WAN vers Internet
Router(config-if)# ip address 203.0.113.1 255.255.255.248
Router(config-if)# ip nat outside
Router(config-if)# no shutdown
Router(config-if)# exit

! Serveur Web
Router(config)# ip nat inside source static 192.168.1.10 203.0.113.5

! Serveur Mail
Router(config)# ip nat inside source static 192.168.1.20 203.0.113.6

! Serveur DNS
Router(config)# ip nat inside source static 192.168.1.30 203.0.113.7
```

### NAT statique avec port spécifique (Port Forwarding)

Vous pouvez également spécifier un port particulier si vous voulez exposer un service spécifique :

```bash
! Syntaxe avec port
Router(config)# ip nat inside source static [protocole] [IP_INSIDE] [PORT_INSIDE] [IP_OUTSIDE] [PORT_OUTSIDE]

! Exemple : Serveur SSH sur port non-standard
Router(config)# ip nat inside source static tcp 192.168.1.10 22 203.0.113.5 2222

! Exemple : Serveur Web sur port 8080 interne, exposé sur port 80
Router(config)# ip nat inside source static tcp 192.168.1.10 8080 203.0.113.5 80
```

> [!info] Quand utiliser le port forwarding ?
> 
> - Exposer plusieurs serveurs internes sur la même IP publique (avec des ports différents)
> - Rediriger un port externe vers un port interne différent
> - Améliorer la sécurité en cachant les ports standards

---

## Cas d'Usage

### 1. Serveur Web Public

**Contexte** : Héberger un site web accessible depuis Internet.

```bash
! Serveur Apache sur 192.168.1.10
Router(config)# ip nat inside source static 192.168.1.10 203.0.113.5
```

**DNS externe** : Le nom de domaine pointe vers `203.0.113.5`, qui est traduit vers `192.168.1.10`.

### 2. Serveur Mail Exchange

**Contexte** : Serveur mail interne devant recevoir des emails depuis Internet.

```bash
! Serveur Exchange sur 192.168.1.20
Router(config)# ip nat inside source static 192.168.1.20 203.0.113.6

! Enregistrement MX du domaine pointe vers 203.0.113.6
```

### 3. Accès VPN Distant

**Contexte** : Serveur VPN accessible depuis l'extérieur.

```bash
! Serveur VPN sur 192.168.1.50
Router(config)# ip nat inside source static tcp 192.168.1.50 1723 203.0.113.5 1723
Router(config)# ip nat inside source static udp 192.168.1.50 500 203.0.113.5 500
Router(config)# ip nat inside source static udp 192.168.1.50 4500 203.0.113.5 4500
```

### 4. Caméras de Surveillance

**Contexte** : Accès distant à des caméras IP.

```bash
! Caméra 1 - Port 8001
Router(config)# ip nat inside source static tcp 192.168.1.100 80 203.0.113.5 8001

! Caméra 2 - Port 8002
Router(config)# ip nat inside source static tcp 192.168.1.101 80 203.0.113.5 8002

! Caméra 3 - Port 8003
Router(config)# ip nat inside source static tcp 192.168.1.102 80 203.0.113.5 8003
```

> [!tip] Astuce En utilisant des ports différents, vous pouvez exposer plusieurs devices similaires sur une seule IP publique.

### 5. Infrastructure DMZ

**Contexte** : Zone démilitarisée avec serveurs publics.

```bash
! Interface vers DMZ
Router(config)# interface GigabitEthernet0/2
Router(config-if)# description DMZ
Router(config-if)# ip address 10.0.0.1 255.255.255.0
Router(config-if)# ip nat inside
Router(config-if)# exit

! Serveurs DMZ exposés
Router(config)# ip nat inside source static 10.0.0.10 203.0.113.5  ! Web
Router(config)# ip nat inside source static 10.0.0.20 203.0.113.6  ! Mail
Router(config)# ip nat inside source static 10.0.0.30 203.0.113.7  ! FTP
```

---

## Vérification et Dépannage

### Commandes de vérification essentielles

#### 1. Afficher les translations NAT actives

```bash
Router# show ip nat translations
```

**Exemple de sortie** :

```
Pro Inside global      Inside local       Outside local      Outside global
--- 203.0.113.5        192.168.1.10       ---                ---
tcp 203.0.113.5:80     192.168.1.10:80    8.8.8.8:54321      8.8.8.8:54321
```

> [!info] Interprétation
> 
> - La première ligne montre le mapping statique permanent
> - Les lignes suivantes montrent les connexions actives utilisant ce mapping

#### 2. Afficher les statistiques NAT

```bash
Router# show ip nat statistics
```

**Informations affichées** :

- Total de translations actives
- Nombre de translations statiques configurées
- Interfaces inside/outside
- Hits/Misses

#### 3. Afficher uniquement les mappings statiques

```bash
Router# show ip nat translations verbose
```

ou

```bash
Router# show running-config | include ip nat
```

#### 4. Debugger le NAT en temps réel

```bash
Router# debug ip nat
Router# debug ip nat detailed
```

> [!warning] Attention ! Le debug génère beaucoup de logs. Utilisez-le uniquement pour le dépannage et désactivez-le ensuite avec `undebug all`.

### Tester la connectivité

```bash
! Depuis le routeur, tester l'accès au serveur interne
Router# ping 192.168.1.10

! Depuis Internet (simulation), tester l'accès via l'IP publique
! (Nécessite un device externe ou un autre routeur)
External# telnet 203.0.113.5 80
```

### Effacer les translations dynamiques

Si vous testez et voulez réinitialiser les translations dynamiques (pas les statiques) :

```bash
Router# clear ip nat translation *
```

> [!tip] Astuce de dépannage Utilisez `show ip nat translations` avant et après vos tests pour comprendre comment le NAT traduit vos connexions.

---

## Pièges Courants

### 🚨 1. Oublier de définir les interfaces inside/outside

**Erreur** : Configuration du NAT mais rien ne fonctionne.

```bash
! ❌ INCORRECT - Interfaces non définies
Router(config)# ip nat inside source static 192.168.1.10 203.0.113.5
! Le NAT ne fonctionnera pas !

! ✅ CORRECT
Router(config)# interface GigabitEthernet0/0
Router(config-if)# ip nat inside
Router(config-if)# exit
Router(config)# interface GigabitEthernet0/1
Router(config-if)# ip nat outside
Router(config-if)# exit
Router(config)# ip nat inside source static 192.168.1.10 203.0.113.5
```

### 🚨 2. Routage manquant

**Erreur** : Le NAT fonctionne mais les paquets ne sont pas routés correctement.

> [!warning] Vérification nécessaire Le NAT ne crée pas automatiquement les routes ! Assurez-vous que :
> 
> - Le routeur a une route par défaut vers Internet
> - Les devices internes ont le routeur comme passerelle par défaut
> - Les routes de retour existent

```bash
! Vérifier la table de routage
Router# show ip route

! Ajouter une route par défaut si nécessaire
Router(config)# ip route 0.0.0.0 0.0.0.0 203.0.113.1
```

### 🚨 3. Conflit d'adresses IP publiques

**Erreur** : Utiliser la même IP publique pour plusieurs mappings statiques sans port.

```bash
! ❌ INCORRECT - Conflit !
Router(config)# ip nat inside source static 192.168.1.10 203.0.113.5
Router(config)# ip nat inside source static 192.168.1.20 203.0.113.5
! Erreur : La deuxième commande écrase la première

! ✅ CORRECT - Utiliser des IPs différentes
Router(config)# ip nat inside source static 192.168.1.10 203.0.113.5
Router(config)# ip nat inside source static 192.168.1.20 203.0.113.6

! ✅ CORRECT - Ou utiliser des ports différents
Router(config)# ip nat inside source static tcp 192.168.1.10 80 203.0.113.5 80
Router(config)# ip nat inside source static tcp 192.168.1.20 80 203.0.113.5 8080
```

### 🚨 4. Firewall ou ACL bloquant le trafic

**Erreur** : Le NAT est configuré mais les ACL bloquent le trafic.

```bash
! Vérifier les ACL appliquées
Router# show ip access-lists
Router# show ip interface GigabitEthernet0/1

! Si nécessaire, autoriser le trafic
Router(config)# access-list 100 permit ip any host 203.0.113.5
Router(config)# interface GigabitEthernet0/1
Router(config-if)# ip access-group 100 in
```

### 🚨 5. L'IP publique n'est pas sur le routeur

**Erreur** : Utiliser une IP publique non configurée sur une interface.

> [!warning] Important L'IP publique utilisée pour le NAT statique doit soit :
> 
> - Être configurée sur l'interface outside
> - Faire partie d'un subnet directement connecté
> - Être routée vers le routeur par l'ISP

```bash
! ❌ INCORRECT - 203.0.113.5 n'est pas configurée
Router(config)# interface GigabitEthernet0/1
Router(config-if)# ip address 203.0.113.1 255.255.255.248
Router(config-if)# exit
Router(config)# ip nat inside source static 192.168.1.10 203.0.113.50
! 203.0.113.50 n'est pas dans le subnet !

! ✅ CORRECT
Router(config)# ip nat inside source static 192.168.1.10 203.0.113.5
! 203.0.113.5 est dans le subnet 203.0.113.0/29
```

### 🚨 6. Désactiver le NAT statique

Pour supprimer un mapping statique :

```bash
! Syntaxe
Router(config)# no ip nat inside source static [IP_INSIDE] [IP_OUTSIDE]

! Exemple
Router(config)# no ip nat inside source static 192.168.1.10 203.0.113.5
```

### 🚨 7. Hairpin NAT / NAT Loopback

**Problème** : Les clients internes ne peuvent pas accéder aux serveurs internes via leur IP publique.

> [!info] Explication Par défaut, Cisco ne permet pas le "hairpin NAT" (connexion d'un client interne vers l'IP publique d'un serveur interne).

**Solution** : Activer le NAT sur la même interface ou utiliser un DNS split-horizon.

```bash
! Activer le hairpin NAT (pas recommandé, impact performance)
Router(config)# interface GigabitEthernet0/0
Router(config-if)# ip nat inside
Router(config-if)# ip nat enable
Router(config-if)# exit

! ✅ MEILLEURE SOLUTION : DNS interne
! Configurer un serveur DNS interne qui résout le nom de domaine
! vers l'IP privée (192.168.1.10) pour les clients internes
! et l'IP publique (203.0.113.5) depuis Internet (Split DNS)
```

---

## Bonnes Pratiques

> [!tip] Recommandations professionnelles
> 
> 1. **Documentation** : Maintenir un tableau de correspondance IP privée ↔ IP publique
> 2. **Sécurité** : Toujours combiner le NAT avec des ACL appropriées
> 3. **Monitoring** : Surveiller régulièrement les translations avec `show ip nat statistics`
> 4. **Sauvegarde** : Sauvegarder la configuration après chaque modification NAT
> 5. **Nomenclature** : Utiliser des descriptions d'interface claires
> 6. **DNS** : Configurer des enregistrements DNS appropriés pour les IPs publiques
> 7. **Plan d'adressage** : Réserver un bloc d'IPs publiques dédié au NAT statique
> 8. **Tests** : Toujours tester depuis l'extérieur après configuration

### Template de configuration

```bash
! ============================================
! NAT STATIQUE - Configuration Template
! Date : [DATE]
! Administrateur : [NOM]
! ============================================

! Étape 1 : Interfaces
interface [INTERFACE_INSIDE]
 description [LAN/DMZ]
 ip address [IP_PRIVEE] [MASQUE]
 ip nat inside
 no shutdown
!
interface [INTERFACE_OUTSIDE]
 description [WAN vers Internet]
 ip address [IP_PUBLIQUE] [MASQUE]
 ip nat outside
 no shutdown
!

! Étape 2 : Mappings statiques
! Format : ip nat inside source static [PRIVEE] [PUBLIQUE]
ip nat inside source static [IP_SERVEUR_1] [IP_PUB_1]
ip nat inside source static [IP_SERVEUR_2] [IP_PUB_2]
!

! Étape 3 : Route par défaut
ip route 0.0.0.0 0.0.0.0 [NEXT_HOP_ISP]
!

! Étape 4 : ACL (optionnel mais recommandé)
access-list [NUM] permit [CONDITIONS]
interface [INTERFACE_OUTSIDE]
 ip access-group [NUM] in
!
end
```

---

> [!success] Résumé Le **NAT statique** crée une correspondance permanente un-à-un entre une IP privée et une IP publique. Il est essentiel pour exposer des serveurs internes sur Internet tout en préservant l'infrastructure réseau privée. La configuration nécessite la définition des interfaces inside/outside et l'établissement des mappings appropriés.