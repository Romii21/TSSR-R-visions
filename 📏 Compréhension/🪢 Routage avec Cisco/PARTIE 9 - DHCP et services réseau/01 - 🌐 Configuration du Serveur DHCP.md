

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

## 🎯 Introduction au DHCP

Le protocole **DHCP (Dynamic Host Configuration Protocol)** permet d'attribuer automatiquement des configurations IP aux clients du réseau. Sur un routeur Cisco, vous pouvez transformer celui-ci en serveur DHCP pour centraliser et simplifier la gestion des adresses IP.

> [!info] Pourquoi utiliser DHCP ?
> 
> - **Automatisation** : Plus besoin de configurer manuellement chaque client
> - **Gestion centralisée** : Toutes les configurations depuis le routeur
> - **Réduction des erreurs** : Évite les conflits d'adresses IP
> - **Flexibilité** : Modification rapide de la configuration réseau

---

## 🔧 Configuration du Pool DHCP

### Concept

Un **pool DHCP** est un ensemble de paramètres réseau que le routeur distribuera aux clients. Chaque pool est associé à un réseau spécifique et porte un nom unique pour l'identifier.

### Syntaxe de base

```bash
Router(config)# ip dhcp pool NOM_DU_POOL
Router(dhcp-config)#
```

> [!example] Exemple pratique
> 
> ```bash
> Router(config)# ip dhcp pool LAN_VENTE
> Router(dhcp-config)# 
> ```

### Points clés

|Élément|Description|
|---|---|
|**Nom du pool**|Identificateur unique, sensible à la casse|
|**Mode dhcp-config**|Mode spécial pour configurer le pool|
|**Convention de nommage**|Utiliser des noms explicites (ex: LAN_RH, VLAN10_ADMIN)|

> [!tip] Bonne pratique Adoptez une convention de nommage cohérente pour vos pools (ex: `ZONE_DEPARTEMENT` ou `VLAN_FONCTION`) pour faciliter la maintenance future.

---

## 📡 Définition de la Plage d'Adresses

### Concept

La commande `network` définit le réseau et le masque de sous-réseau pour le pool DHCP. C'est la plage globale dans laquelle le serveur DHCP pourra attribuer des adresses.

### Syntaxe

```bash
Router(dhcp-config)# network ADRESSE_RESEAU MASQUE
```

> [!example] Exemples selon le masque
> 
> ```bash
> # Réseau /24 (255.255.255.0) - 254 adresses disponibles
> Router(dhcp-config)# network 192.168.1.0 255.255.255.0
> 
> # Réseau /25 (255.255.255.128) - 126 adresses disponibles
> Router(dhcp-config)# network 10.0.10.0 255.255.255.128
> 
> # Réseau /16 (255.255.0.0) - 65534 adresses disponibles
> Router(dhcp-config)# network 172.16.0.0 255.255.0.0
> ```

### Calcul des adresses disponibles

|Masque|Notation CIDR|Adresses totales|Adresses utilisables|
|---|---|---|---|
|255.255.255.0|/24|256|254|
|255.255.255.128|/25|128|126|
|255.255.255.192|/26|64|62|
|255.255.255.224|/27|32|30|

> [!warning] Attention
> 
> - L'adresse réseau doit correspondre à une adresse réseau valide (tous les bits d'hôte à 0)
> - Le masque doit être cohérent avec votre plan d'adressage global
> - Pensez à la croissance future lors du choix de votre masque

---

## 🚪 Configuration de la Passerelle par Défaut

### Concept

La **passerelle par défaut (default gateway)** est l'adresse IP du routeur que les clients utiliseront pour communiquer en dehors de leur réseau local. C'est généralement l'adresse de l'interface du routeur connectée à ce réseau.

### Syntaxe

```bash
Router(dhcp-config)# default-router ADRESSE_IP [IP2] [IP3] [IP4] [IP5] [IP6] [IP7] [IP8]
```

> [!example] Exemple simple
> 
> ```bash
> Router(dhcp-config)# default-router 192.168.1.1
> ```

> [!example] Exemple avec redondance
> 
> ```bash
> # Configuration avec passerelle secondaire (haute disponibilité)
> Router(dhcp-config)# default-router 192.168.1.1 192.168.1.2
> ```

### Bonnes pratiques

|Recommandation|Explication|
|---|---|
|**Première adresse utilisable**|Souvent `.1` ou `.254` selon convention|
|**Cohérence**|Utiliser toujours la même convention dans l'organisation|
|**Redondance**|Configurer une passerelle secondaire pour la haute disponibilité|

> [!tip] Astuce professionnelle Documentez votre convention : première adresse (`.1`) pour les petits réseaux, dernière adresse (`.254`) pour les grands réseaux. La cohérence facilite le troubleshooting.

> [!warning] Piège courant La passerelle par défaut DOIT être dans le même réseau que la plage d'adresses définie. Si votre pool est `192.168.1.0/24`, la passerelle doit être entre `192.168.1.1` et `192.168.1.254`.

---

## 🔍 Configuration des Serveurs DNS

### Concept

Les **serveurs DNS (Domain Name System)** permettent la résolution des noms de domaine en adresses IP. Sans DNS, les clients ne pourraient accéder aux sites web qu'en tapant directement leur adresse IP.

### Syntaxe

```bash
Router(dhcp-config)# dns-server ADRESSE_IP [IP2] [IP3] [IP4] [IP5] [IP6] [IP7] [IP8]
```

> [!example] Configuration avec DNS publics
> 
> ```bash
> # Configuration avec Google DNS (primaire et secondaire)
> Router(dhcp-config)# dns-server 8.8.8.8 8.8.4.4
> 
> # Configuration avec Cloudflare DNS
> Router(dhcp-config)# dns-server 1.1.1.1 1.0.0.1
> 
> # Configuration avec OpenDNS
> Router(dhcp-config)# dns-server 208.67.222.222 208.67.220.220
> ```

> [!example] Configuration avec DNS interne et externe
> 
> ```bash
> # DNS interne en priorité, puis DNS publics
> Router(dhcp-config)# dns-server 192.168.1.10 8.8.8.8 8.8.4.4
> ```

### Serveurs DNS populaires

|Fournisseur|DNS Primaire|DNS Secondaire|Caractéristiques|
|---|---|---|---|
|**Google**|8.8.8.8|8.8.4.4|Rapide, fiable, filtrage minimal|
|**Cloudflare**|1.1.1.1|1.0.0.1|Très rapide, focus sur la confidentialité|
|**OpenDNS**|208.67.222.222|208.67.220.220|Filtrage de contenu disponible|
|**Quad9**|9.9.9.9|149.112.112.112|Blocage de sites malveillants|

> [!tip] Bonne pratique
> 
> - **Environnement d'entreprise** : Mettez le DNS interne en premier, puis des DNS publics en backup
> - **Ordre de priorité** : Les clients utiliseront les serveurs dans l'ordre spécifié
> - **Redondance** : Configurez toujours au moins 2 serveurs DNS

> [!warning] Attention Si vous utilisez un DNS interne (Active Directory, serveur local), assurez-vous qu'il est accessible et fonctionnel avant de le configurer, sinon les clients subiront des délais de résolution.

---

## ⛔ Exclusion d'Adresses

### Concept

L'**exclusion d'adresses** permet de réserver certaines adresses IP du pool pour des équipements nécessitant des IP fixes (serveurs, imprimantes, équipements réseau). Ces adresses ne seront jamais distribuées par le DHCP.

### Syntaxe

```bash
Router(config)# ip dhcp excluded-address IP_DEBUT [IP_FIN]
```

> [!info] Important Cette commande se configure en **mode de configuration globale** (pas dans le mode dhcp-config), généralement AVANT la création du pool.

> [!example] Exclusion d'une seule adresse
> 
> ```bash
> Router(config)# ip dhcp excluded-address 192.168.1.1
> ```

> [!example] Exclusion d'une plage d'adresses
> 
> ```bash
> # Exclure les 10 premières adresses (1 à 10)
> Router(config)# ip dhcp excluded-address 192.168.1.1 192.168.1.10
> 
> # Exclure les 50 dernières adresses (205 à 254)
> Router(config)# ip dhcp excluded-address 192.168.1.205 192.168.1.254
> ```

### Stratégies d'exclusion courantes

|Stratégie|Plage exclue|Usage typique|
|---|---|---|
|**Début de plage**|`.1` à `.10` ou `.50`|Routeurs, switches, serveurs|
|**Fin de plage**|`.200` à `.254`|Équipements réseau, points d'accès|
|**Milieu de plage**|Variable|Segmentation logique spécifique|

> [!example] Configuration complète réaliste
> 
> ```bash
> Router(config)# ip dhcp excluded-address 192.168.1.1 192.168.1.20
> Router(config)# ip dhcp excluded-address 192.168.1.250 192.168.1.254
> Router(config)#
> Router(config)# ip dhcp pool LAN_BUREAUX
> Router(dhcp-config)# network 192.168.1.0 255.255.255.0
> Router(dhcp-config)# default-router 192.168.1.1
> Router(dhcp-config)# dns-server 192.168.1.10 8.8.8.8
> ```

### Organisation recommandée

```
Réseau 192.168.1.0/24 :
├── .1 - .20     → Exclus (infrastructure)
│   ├── .1       → Routeur (passerelle)
│   ├── .10      → Serveur DNS/AD
│   ├── .11      → Serveur de fichiers
│   └── .20      → Switch principal
├── .21 - .249   → Pool DHCP (229 adresses)
└── .250 - .254  → Exclus (équipements réseau)
    ├── .250     → Point d'accès WiFi 1
    └── .251     → Point d'accès WiFi 2
```

> [!tip] Bonnes pratiques
> 
> - **Documentez** vos exclusions dans un fichier séparé
> - **Standardisez** les plages exclues dans toute l'organisation
> - **Laissez de la marge** : si vous avez 5 serveurs, excluez 10-20 adresses
> - Configurez les exclusions **avant** de créer le pool DHCP

> [!warning] Piège fréquent N'oubliez pas d'exclure l'adresse de la passerelle elle-même ! Si votre routeur utilise `192.168.1.1`, cette adresse doit être exclue du pool.

---

## ⏱️ Durée de Bail (Lease)

### Concept

La **durée de bail (lease time)** définit pendant combien de temps un client peut conserver une adresse IP avant de devoir la renouveler. C'est un équilibre entre stabilité et flexibilité.

### Syntaxe

```bash
Router(dhcp-config)# lease {JOURS [HEURES] [MINUTES] | infinite}
```

> [!example] Différentes configurations
> 
> ```bash
> # Bail de 1 jour (par défaut Cisco)
> Router(dhcp-config)# lease 1
> 
> # Bail de 8 heures
> Router(dhcp-config)# lease 0 8
> 
> # Bail de 2 heures 30 minutes
> Router(dhcp-config)# lease 0 2 30
> 
> # Bail de 7 jours
> Router(dhcp-config)# lease 7
> 
> # Bail infini (permanent jusqu'à libération manuelle)
> Router(dhcp-config)# lease infinite
> ```

### Durées recommandées par type d'environnement

|Environnement|Durée recommandée|Justification|
|---|---|---|
|**Bureau standard**|1-3 jours|Équilibre entre stabilité et flexibilité|
|**WiFi public**|2-4 heures|Rotation rapide des utilisateurs|
|**Réseau étudiant**|8-12 heures|Changements fréquents|
|**Serveurs (DHCP)**|7-30 jours|Stabilité maximale|
|**IoT/Imprimantes**|Infinite ou 30 jours|Adresses quasi-permanentes|
|**Salle de formation**|1-2 heures|Réutilisation rapide|

### Mécanisme de renouvellement

```
Durée de bail : 8 heures
│
├── T = 0h        → Attribution de l'IP
├── T = 4h (50%)  → Tentative de renouvellement automatique
├── T = 7h (87.5%)→ Deuxième tentative si échec
└── T = 8h (100%) → Expiration, libération si pas renouvelé
```

> [!info] Processus automatique
> 
> - À **50% du bail** : le client tente de renouveler avec le même serveur
> - À **87.5% du bail** : si échec, tentative avec n'importe quel serveur DHCP
> - À **100%** : l'adresse est libérée et le client doit redemander une nouvelle IP

### Impact sur les performances

|Durée courte (< 4h)|Durée moyenne (1-3 jours)|Durée longue (> 7 jours)|
|---|---|---|
|✅ Flexibilité maximale|✅ Bon équilibre|✅ Charge minimale serveur|
|✅ Récupération rapide des IP|✅ Charge serveur modérée|✅ Stabilité maximale|
|❌ Charge serveur élevée|✅ Adapté à la plupart des cas|❌ IP bloquées si client parti|
|❌ Trafic réseau accru||❌ Moins flexible|

> [!tip] Recommandation générale
> 
> - **Réseau stable** avec peu de changements → 3-7 jours
> - **Réseau dynamique** avec beaucoup de mobilité → 8-24 heures
> - **Environnement contrôlé** avec équipements fixes → Infinite ou 30 jours
> - **WiFi invité** ou public → 2-4 heures

> [!warning] Cas particulier : lease infinite Utilisez `lease infinite` avec précaution :
> 
> - ✅ Pour des équipements qui ne bougent jamais
> - ❌ Risque de saturation du pool si les clients ne libèrent pas
> - ❌ Préférez plutôt des IP statiques pour les équipements permanents

---

## 🔎 Vérification et Maintenance

### Commandes de vérification essentielles

```bash
# Afficher la configuration des pools DHCP
Router# show ip dhcp pool [NOM_POOL]

# Afficher les baux actuels (bindings)
Router# show ip dhcp binding

# Afficher les statistiques du serveur DHCP
Router# show ip dhcp server statistics

# Afficher les conflits d'adresses détectés
Router# show ip dhcp conflict
```

> [!example] Interprétation de la sortie
> 
> ```bash
> Router# show ip dhcp binding
> 
> IP address      Client-ID/              Lease expiration        Type
>                 Hardware address
> 192.168.1.25    0100.1234.5678.90       Dec 22 2025 03:15 PM    Automatic
> 192.168.1.48    0100.abcd.ef12.34       Dec 22 2025 05:42 PM    Automatic
> 192.168.1.89    0100.9876.5432.10       Dec 23 2025 09:20 AM    Automatic
> ```

### Maintenance et dépannage

|Commande|Usage|
|---|---|
|`clear ip dhcp binding *`|Libérer tous les baux (attention !)|
|`clear ip dhcp binding ADRESSE_IP`|Libérer un bail spécifique|
|`clear ip dhcp conflict *`|Effacer la base de conflits|
|`debug ip dhcp server events`|Activer le debug (désactiver avec `no debug all`)|

> [!warning] Attention avec clear La commande `clear ip dhcp binding *` va déconnecter TOUS les clients DHCP. Utilisez-la uniquement en maintenance planifiée ou pour un pool spécifique.

> [!tip] Dépannage rapide **Problème** : Un client ne reçoit pas d'adresse IP
> 
> **Vérifications** :
> 
> 1. `show ip dhcp pool` → Pool correctement configuré ?
> 2. `show ip dhcp binding` → Toutes les adresses attribuées ?
> 3. `show ip dhcp conflict` → Conflits détectés ?
> 4. Interface du routeur configurée ? (`show ip interface brief`)
> 5. Exclusions correctes ? (`show run | include excluded`)

---

## 📝 Exemple de Configuration Complète

> [!example] Configuration complète d'un réseau d'entreprise
> 
> ```bash
> ! ========================================
> ! Configuration du serveur DHCP
> ! Réseau : LAN Bureaux (192.168.1.0/24)
> ! ========================================
> 
> ! Étape 1 : Exclusions (adresses réservées)
> Router(config)# ip dhcp excluded-address 192.168.1.1 192.168.1.20
> Router(config)# ip dhcp excluded-address 192.168.1.250 192.168.1.254
> 
> ! Étape 2 : Création du pool DHCP
> Router(config)# ip dhcp pool LAN_BUREAUX
> Router(dhcp-config)# network 192.168.1.0 255.255.255.0
> Router(dhcp-config)# default-router 192.168.1.1
> Router(dhcp-config)# dns-server 192.168.1.10 8.8.8.8 8.8.4.4
> Router(dhcp-config)# lease 3
> Router(dhcp-config)# exit
> 
> ! Étape 3 : Vérification
> Router(config)# exit
> Router# show ip dhcp pool
> Router# show run | section dhcp
> ```

### Structure d'un déploiement multi-VLAN

> [!example] Plusieurs pools pour différents départements
> 
> ```bash
> ! ========== VLAN 10 - Administration ==========
> ip dhcp excluded-address 10.0.10.1 10.0.10.10
> !
> ip dhcp pool VLAN10_ADMIN
>  network 10.0.10.0 255.255.255.0
>  default-router 10.0.10.1
>  dns-server 10.0.1.10 8.8.8.8
>  lease 7
> 
> ! ========== VLAN 20 - Ventes ==========
> ip dhcp excluded-address 10.0.20.1 10.0.20.10
> !
> ip dhcp pool VLAN20_VENTES
>  network 10.0.20.0 255.255.255.0
>  default-router 10.0.20.1
>  dns-server 10.0.1.10 8.8.8.8
>  lease 2
> 
> ! ========== VLAN 30 - WiFi Invités ==========
> ip dhcp excluded-address 10.0.30.1 10.0.30.5
> !
> ip dhcp pool VLAN30_INVITES
>  network 10.0.30.0 255.255.255.0
>  default-router 10.0.30.1
>  dns-server 8.8.8.8 8.8.4.4
>  lease 0 4
> ```

---

## ⚙️ Options DHCP Avancées (Mention)

> [!info] Options supplémentaires disponibles Bien que non couvertes en détail dans cette partie, sachez que d'autres options DHCP existent :
> 
> - `domain-name` : Nom de domaine pour les clients
> - `netbios-name-server` : Serveurs WINS pour environnements Windows
> - `option` : Options DHCP personnalisées (code option)
> 
> Ces configurations seront abordées dans les parties avancées du cours.

---

## 🎯 Récapitulatif des Commandes Principales

```bash
# Configuration globale - Exclusions
ip dhcp excluded-address IP_DEBUT [IP_FIN]

# Création du pool
ip dhcp pool NOM_POOL

# Dans le pool (mode dhcp-config)
network ADRESSE_RESEAU MASQUE
default-router IP_PASSERELLE
dns-server IP_DNS1 [IP_DNS2] [IP_DNS3]...
lease {JOURS [HEURES] [MINUTES] | infinite}

# Vérification
show ip dhcp pool
show ip dhcp binding
show ip dhcp server statistics
show ip dhcp conflict

# Maintenance
clear ip dhcp binding {* | ADRESSE_IP}
clear ip dhcp conflict *
```

---

> [!tip] Mémo pour la configuration rapide **Ordre recommandé** :
> 
> 1. Planifier le plan d'adressage (réseau, masque, exclusions)
> 2. Configurer les exclusions (`ip dhcp excluded-address`)
> 3. Créer le pool (`ip dhcp pool`)
> 4. Définir le réseau (`network`)
> 5. Configurer la passerelle (`default-router`)
> 6. Ajouter les DNS (`dns-server`)
> 7. Ajuster le bail (`lease`)
> 8. Vérifier (`show ip dhcp pool`)

---

**📌 Fin du module DHCP et Services Réseau**