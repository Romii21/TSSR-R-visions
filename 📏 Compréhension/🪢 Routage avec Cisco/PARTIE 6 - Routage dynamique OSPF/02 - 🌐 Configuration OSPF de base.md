

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

## 🚀 Activation du processus OSPF

### Concept fondamental

L'activation d'OSPF sur un routeur Cisco nécessite de démarrer un **processus OSPF** identifié par un numéro de processus. Ce numéro est **localement significatif** : il n'a de sens que pour le routeur lui-même et n'a pas besoin d'être identique sur tous les routeurs du réseau.

> [!info] Pourquoi un numéro de processus ? Le numéro de processus permet d'exécuter plusieurs instances OSPF indépendantes sur le même routeur (bien que ce soit rare en pratique). Chaque processus maintient sa propre base de données de topologie et ses propres tables de routage.

### Syntaxe de base

```bash
Router(config)# router ospf [process-id]
```

- **process-id** : Numéro entre 1 et 65535
- Cette commande active OSPF et vous place en mode de configuration OSPF

> [!example] Exemple d'activation
> 
> ```bash
> Router(config)# router ospf 1
> Router(config-router)# 
> ```
> 
> Le routeur entre maintenant en mode `config-router` où toutes les configurations OSPF seront appliquées.

### Bonnes pratiques

- **Standardisation** : Bien que le numéro soit localement significatif, utilisez le même numéro (souvent 1 ou 10) sur tous vos routeurs pour faciliter la documentation et le dépannage
- **Documentation** : Documentez toujours quel processus OSPF est utilisé pour quel réseau dans votre entreprise

> [!warning] Attention Une fois le processus OSPF activé, il ne participera à aucun échange tant que vous n'aurez pas déclaré au moins un réseau avec la commande `network`.

---

## 🎯 Déclaration des réseaux avec wildcard mask

### Principe de fonctionnement

La déclaration des réseaux indique à OSPF **sur quelles interfaces** il doit être activé. Contrairement à d'autres protocoles, OSPF n'utilise pas de masque de sous-réseau classique mais un **wildcard mask** (masque inversé).

### Syntaxe complète

```bash
Router(config-router)# network [adresse-reseau] [wildcard-mask] area [area-id]
```

- **adresse-reseau** : L'adresse réseau ou une adresse IP d'interface
- **wildcard-mask** : Masque inversé déterminant quelle portion de l'adresse doit correspondre
- **area** : Zone OSPF à laquelle appartient ce réseau (0 pour le backbone)

### Le wildcard mask expliqué

Le wildcard mask fonctionne **à l'inverse** d'un masque de sous-réseau :

- **0** = le bit doit correspondre exactement
- **1** = le bit peut être n'importe quoi (ignoré)

> [!example] Tableau de conversion
> 
> |Masque de sous-réseau|Wildcard mask|CIDR|Utilisation|
> |---|---|---|---|
> |255.255.255.255|0.0.0.0|/32|Une interface spécifique|
> |255.255.255.0|0.0.0.255|/24|Un réseau de classe C|
> |255.255.0.0|0.0.255.255|/16|Un réseau de classe B|
> |255.0.0.0|0.255.255.255|/8|Un réseau de classe A|
> |0.0.0.0|255.255.255.255|/0|Toutes les interfaces|

### Calcul du wildcard mask

**Formule** : Wildcard mask = 255.255.255.255 - Masque de sous-réseau

> [!example] Exemples de calcul
> 
> **Réseau 192.168.10.0/24** :
> 
> ```
> 255.255.255.255
> - 255.255.255.0
> = 0.0.0.255
> ```
> 
> **Réseau 172.16.0.0/16** :
> 
> ```
> 255.255.255.255
> - 255.255.0.0
> = 0.0.255.255
> ```
> 
> **Interface spécifique 10.1.1.1/32** :
> 
> ```
> 255.255.255.255
> - 255.255.255.255
> = 0.0.0.0
> ```

### Méthodes de déclaration

#### Méthode 1 : Par réseau complet

```bash
Router(config-router)# network 192.168.10.0 0.0.0.255 area 0
```

Active OSPF sur toutes les interfaces dont l'IP appartient au réseau 192.168.10.0/24

#### Méthode 2 : Par interface spécifique

```bash
Router(config-router)# network 10.1.1.1 0.0.0.0 area 0
```

Active OSPF uniquement sur l'interface ayant l'IP 10.1.1.1

#### Méthode 3 : Sur toutes les interfaces

```bash
Router(config-router)# network 0.0.0.0 255.255.255.255 area 0
```

Active OSPF sur toutes les interfaces du routeur (à utiliser avec précaution)

> [!tip] Astuce de sélection
> 
> - **Petits réseaux** : Utilisez la méthode 3 puis désactivez les interfaces non désirées avec `passive-interface`
> - **Grands réseaux** : Utilisez la méthode 2 pour un contrôle précis
> - **Réseaux structurés** : Utilisez la méthode 1 pour activer OSPF par sous-réseau

> [!example] Configuration complète exemple
> 
> ```bash
> Router(config)# router ospf 1
> Router(config-router)# network 192.168.1.0 0.0.0.255 area 0
> Router(config-router)# network 10.0.0.0 0.0.0.3 area 0
> Router(config-router)# network 172.16.5.1 0.0.0.0 area 1
> ```
> 
> Ce routeur active OSPF sur :
> 
> - Toutes les interfaces du réseau 192.168.1.0/24 (area 0)
> - Les interfaces 10.0.0.0 à 10.0.0.3 (area 0)
> - L'interface spécifique 172.16.5.1 (area 1)

### Pièges courants

> [!warning] Erreurs fréquentes
> 
> **❌ Utiliser un masque de sous-réseau au lieu d'un wildcard** :
> 
> ```bash
> network 192.168.1.0 255.255.255.0 area 0  # INCORRECT
> ```
> 
> **✅ Utiliser le wildcard mask** :
> 
> ```bash
> network 192.168.1.0 0.0.0.255 area 0  # CORRECT
> ```
> 
> **❌ Oublier l'area** :
> 
> ```bash
> network 192.168.1.0 0.0.0.255  # INCOMPLETE
> ```

---

## 🆔 Configuration du Router ID

### Qu'est-ce que le Router ID ?

Le **Router ID** (RID) est un identifiant unique au format d'une adresse IPv4 qui identifie de manière univoque un routeur dans le domaine OSPF. C'est l'un des paramètres les plus importants d'OSPF.

> [!info] Importance du Router ID
> 
> - Identifie de façon unique chaque routeur OSPF
> - Utilisé dans l'élection du DR/BDR (routeurs désignés sur les réseaux multi-accès)
> - Visible dans toutes les communications OSPF
> - Nécessaire pour le bon fonctionnement du protocole

### Méthodes de sélection du Router ID

Si vous ne configurez pas manuellement le Router ID, OSPF le sélectionne automatiquement selon cet ordre de priorité :

#### 1️⃣ Router ID configuré manuellement (recommandé)

```bash
Router(config-router)# router-id [adresse-ip]
```

> [!example] Configuration manuelle
> 
> ```bash
> Router(config)# router ospf 1
> Router(config-router)# router-id 1.1.1.1
> ```

#### 2️⃣ IP la plus élevée sur une interface loopback active

Si aucun Router ID manuel n'est configuré, OSPF choisit l'IP la plus élevée parmi toutes les interfaces loopback actives.

> [!example] Sélection automatique via loopback
> 
> ```bash
> Router(config)# interface loopback 0
> Router(config-if)# ip address 10.10.10.1 255.255.255.255
> Router(config-if)# exit
> Router(config)# router ospf 1
> Router(config-router)# network 0.0.0.0 255.255.255.255 area 0
> # Le Router ID sera 10.10.10.1
> ```

#### 3️⃣ IP la plus élevée sur une interface physique active

Si aucune loopback n'existe, OSPF choisit l'IP la plus élevée parmi toutes les interfaces physiques actives (état up/up).

### Bonnes pratiques

> [!tip] Recommandations professionnelles
> 
> **✅ Toujours configurer manuellement le Router ID** :
> 
> - Utilisez un schéma logique et cohérent (ex: 1.1.1.1, 2.2.2.2, etc.)
> - Ou utilisez des loopbacks avec un plan d'adressage dédié
> 
> **✅ Créer des interfaces loopback dédiées** :
> 
> ```bash
> interface loopback 0
>  ip address 10.255.255.1 255.255.255.255
> ```
> 
> Les loopbacks ne tombent jamais et garantissent la stabilité du Router ID
> 
> **✅ Documentation** : Maintenez un document recensant tous les Router ID de votre infrastructure

### Vérification du Router ID

```bash
Router# show ip ospf
Router# show ip protocols
```

> [!example] Sortie de vérification
> 
> ```bash
> Router# show ip ospf
>  Routing Process "ospf 1" with ID 1.1.1.1
>  Start time: 00:00:12.456, Time elapsed: 00:05:23.789
>  ...
> ```

### Modification du Router ID

> [!warning] Attention au changement de Router ID
> 
> Modifier le Router ID nécessite de **redémarrer le processus OSPF** pour que le changement prenne effet.
> 
> ```bash
> Router(config-router)# router-id 2.2.2.2
> Reload or use "clear ip ospf process" for this to take effect
> 
> Router(config-router)# end
> Router# clear ip ospf process
> Reset ALL OSPF processes? [no]: yes
> ```
> 
> **⚠️ Impact** : Cette opération rompt temporairement toutes les adjacences OSPF et peut causer une interruption de routage.

### Pièges courants

> [!warning] Erreurs à éviter
> 
> **❌ Router ID en double** : Deux routeurs avec le même Router ID dans le même domaine OSPF causeront des dysfonctionnements majeurs.
> 
> **❌ Utiliser 0.0.0.0 comme Router ID** :
> 
> ```bash
> router-id 0.0.0.0  # INVALIDE
> ```
> 
> **❌ Compter sur l'auto-sélection en production** : L'IP d'une interface physique peut changer ou l'interface peut tomber, modifiant ainsi le Router ID involontairement.

---

## 🚫 Passive-interface

### Concept et utilité

La commande `passive-interface` empêche l'envoi de messages Hello OSPF sur une interface tout en **continuant d'annoncer** le réseau connecté à cette interface dans OSPF.

> [!info] Pourquoi utiliser passive-interface ?
> 
> **Sécurité** : Évite d'établir des adjacences OSPF non désirées
> 
> **Performance** : Réduit le trafic inutile (pas de Hello envoyés)
> 
> **Conception** : Les interfaces connectées à des utilisateurs finaux ou à Internet n'ont pas besoin de participer activement à OSPF
> 
> **Le réseau reste annoncé** : Les autres routeurs OSPF connaissent toujours ce réseau

### Cas d'usage typiques

- Interfaces vers les réseaux LAN (utilisateurs finaux)
- Interfaces vers Internet ou des réseaux externes
- Interfaces de gestion
- Toute interface où aucun autre routeur OSPF n'est connecté

### Syntaxe

#### Configuration par interface spécifique

```bash
Router(config-router)# passive-interface [type] [numero]
```

> [!example] Exemple
> 
> ```bash
> Router(config)# router ospf 1
> Router(config-router)# network 0.0.0.0 255.255.255.255 area 0
> Router(config-router)# passive-interface gigabitEthernet 0/0
> Router(config-router)# passive-interface gigabitEthernet 0/1
> ```

#### Configuration de toutes les interfaces par défaut

```bash
Router(config-router)# passive-interface default
```

Rend toutes les interfaces passives, puis vous activez sélectivement celles qui doivent former des adjacences :

```bash
Router(config-router)# no passive-interface [type] [numero]
```

> [!example] Approche recommandée pour la sécurité
> 
> ```bash
> Router(config)# router ospf 1
> Router(config-router)# network 0.0.0.0 255.255.255.255 area 0
> Router(config-router)# passive-interface default
> Router(config-router)# no passive-interface gigabitEthernet 0/2
> Router(config-router)# no passive-interface serial 0/0/0
> ```
> 
> Ici, toutes les interfaces sont passives SAUF Gi0/2 et Se0/0/0 qui formeront des adjacences OSPF.

### Comparaison des deux approches

|Approche|Avantages|Inconvénients|Recommandation|
|---|---|---|---|
|**Spécifique**|Contrôle précis|Plus de commandes|Petits réseaux avec peu d'interfaces LAN|
|**Default**|Sécurité maximale<br>Moins d'erreurs|Nécessite de réactiver les bonnes interfaces|**Recommandé** pour la production|

### Vérification

```bash
Router# show ip ospf interface brief
Router# show ip protocols
```

> [!example] Sortie de vérification
> 
> ```bash
> Router# show ip ospf interface brief
> Interface    PID   Area  IP Address/Mask    Cost  State Nbrs F/C
> Gi0/2        1     0     10.1.1.1/30        1     P2P   1/1
> Se0/0/0      1     0     10.2.2.1/30        64    P2P   1/1
> Gi0/0        1     0     192.168.1.1/24     1     DR    0/0  (Passive)
> Gi0/1        1     0     192.168.2.1/24     1     DR    0/0  (Passive)
> ```

### Impact sur le routage

> [!info] Comportement d'une interface passive
> 
> **✅ Ce qui continue de fonctionner** :
> 
> - Le réseau connecté est annoncé dans OSPF
> - Les routes OSPF sont installées dans la table de routage
> - Le trafic peut traverser l'interface normalement
> 
> **❌ Ce qui est désactivé** :
> 
> - Envoi de paquets Hello OSPF
> - Réception de paquets Hello OSPF (ignorés)
> - Formation d'adjacences OSPF
> - Élection DR/BDR sur cette interface

### Bonnes pratiques

> [!tip] Recommandations
> 
> **🔐 Sécurité d'abord** : Utilisez `passive-interface default` puis activez uniquement les interfaces nécessaires
> 
> **📝 Documentation** : Documentez clairement quelles interfaces doivent former des adjacences et pourquoi
> 
> **🎯 Systematique** : Toute interface LAN sans routeur voisin devrait être passive
> 
> **✅ Vérification** : Après configuration, vérifiez avec `show ip ospf interface brief` que seules les bonnes interfaces forment des adjacences

### Pièges courants

> [!warning] Erreurs fréquentes
> 
> **❌ Oublier de rendre les interfaces LAN passives** : Risque de sécurité et consommation de ressources inutile
> 
> **❌ Rendre passive une interface WAN par erreur** :
> 
> ```bash
> passive-interface serial 0/0/0  # Si un routeur voisin est connecté !
> ```
> 
> Résultat : Aucune adjacence ne se formera et les routes ne seront pas échangées
> 
> **❌ Confondre passive-interface et no network** :
> 
> - `passive-interface` : Le réseau est annoncé mais pas d'adjacence
> - Retirer le `network` : Le réseau n'est ni annoncé ni actif dans OSPF

### Annulation de la configuration

```bash
Router(config-router)# no passive-interface [type] [numero]
Router(config-router)# no passive-interface default
```

---

## 📊 Configuration complète exemple

> [!example] Scénario complet
> 
> **Topologie** :
> 
> - Router R1 avec 3 interfaces :
>     - Gi0/0 : 192.168.10.0/24 (LAN utilisateurs)
>     - Gi0/1 : 192.168.20.0/24 (LAN serveurs)
>     - Gi0/2 : 10.1.1.0/30 (WAN vers R2)
> - Loopback0 : 1.1.1.1/32 (Router ID)
> 
> **Configuration complète** :
> 
> ```bash
> ! Création de la loopback pour le Router ID
> Router(config)# interface loopback 0
> Router(config-if)# ip address 1.1.1.1 255.255.255.255
> Router(config-if)# description Router ID OSPF
> Router(config-if)# exit
> 
> ! Configuration OSPF
> Router(config)# router ospf 1
> Router(config-router)# router-id 1.1.1.1
> 
> ! Déclaration des réseaux
> Router(config-router)# network 192.168.10.0 0.0.0.255 area 0
> Router(config-router)# network 192.168.20.0 0.0.0.255 area 0
> Router(config-router)# network 10.1.1.0 0.0.0.3 area 0
> Router(config-router)# network 1.1.1.1 0.0.0.0 area 0
> 
> ! Sécurisation : toutes les interfaces passives par défaut
> Router(config-router)# passive-interface default
> 
> ! Activation OSPF uniquement sur l'interface WAN
> Router(config-router)# no passive-interface gigabitEthernet 0/2
> Router(config-router)# end
> 
> ! Vérification
> Router# show ip ospf neighbor
> Router# show ip ospf interface brief
> Router# show ip protocols
> ```
> 
> **Résultat attendu** :
> 
> - Router ID : 1.1.1.1
> - Adjacence OSPF uniquement avec R2 via Gi0/2
> - Les réseaux 192.168.10.0/24 et 192.168.20.0/24 sont annoncés dans OSPF
> - Aucun Hello OSPF envoyé sur Gi0/0 et Gi0/1

---

## ✅ Résumé des commandes essentielles

```bash
! Activation OSPF
router ospf [process-id]

! Configuration Router ID
router-id [adresse-ip]

! Déclaration réseau
network [adresse] [wildcard-mask] area [area-id]

! Interface passive (spécifique)
passive-interface [type] [numero]

! Toutes interfaces passives par défaut
passive-interface default

! Réactiver une interface
no passive-interface [type] [numero]

! Vérification
show ip ospf
show ip ospf interface brief
show ip ospf neighbor
show ip protocols
```

---