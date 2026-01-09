# 

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

## 🚀 Activation du processus RIP

### Concept fondamental

L'activation du processus RIP sur un routeur Cisco démarre le protocole de routage dynamique RIP. Cette commande initialise le daemon RIP qui permettra au routeur d'échanger des informations de routage avec ses voisins.

> [!info] Pourquoi activer RIP ?
> 
> - Automatise la découverte et la maintenance des routes
> - Évite la configuration manuelle des routes statiques
> - Permet l'adaptation dynamique aux changements topologiques du réseau

### Syntaxe et commandes

```bash
Router> enable
Router# configure terminal
Router(config)# router rip
Router(config-router)# 
```

> [!example] Exemple pratique
> 
> ```bash
> R1> enable
> R1# configure terminal
> R1(config)# router rip
> R1(config-router)# # Vous êtes maintenant en mode de configuration RIP
> ```

### Points importants

- La commande `router rip` crée une **instance unique** du processus RIP
- Contrairement à OSPF ou EIGRP, RIP n'utilise pas d'identifiant de processus (AS number dans la commande)
- Une seule instance RIP peut tourner simultanément sur un routeur
- Le passage en mode `(config-router)#` indique que le processus est activé

> [!warning] Attention Le simple fait d'entrer la commande `router rip` n'active PAS encore l'envoi de mises à jour. Il faut également déclarer les réseaux (voir section suivante).

---

## 🗺️ Déclaration des réseaux (network)

### Concept fondamental

La commande `network` indique au routeur **sur quelles interfaces** il doit activer le protocole RIP. C'est une étape cruciale qui détermine :

- Quelles interfaces enverront des mises à jour RIP
- Quelles interfaces écouteront les mises à jour RIP entrantes
- Quels réseaux directement connectés seront annoncés aux voisins

### Syntaxe et fonctionnement

```bash
Router(config-router)# network [réseau-classful]
```

> [!info] Particularité de RIP : Adressage Classful RIP utilise l'adressage **classful** pour la déclaration des réseaux, ce qui signifie :
> 
> - Pour une adresse de classe A : on déclare le premier octet (ex: `network 10.0.0.0`)
> - Pour une adresse de classe B : on déclare les deux premiers octets (ex: `network 172.16.0.0`)
> - Pour une adresse de classe C : on déclare les trois premiers octets (ex: `network 192.168.1.0`)
> 
> **RIP convertit automatiquement en adresse classful**, même si vous tapez un sous-réseau spécifique.

### Mécanisme de fonctionnement

Lorsque vous déclarez un réseau avec `network` :

1. **RIP compare le réseau déclaré avec les interfaces du routeur**
2. **Si une interface appartient à ce réseau** → RIP s'active sur cette interface
3. **Le routeur commence alors à** :
    - Envoyer des mises à jour RIP toutes les 30 secondes
    - Écouter les mises à jour provenant des voisins
    - Annoncer les routes qu'il connaît

> [!example] Exemple pratique complet
> 
> ```bash
> R1(config)# router rip
> R1(config-router)# network 192.168.1.0
> R1(config-router)# network 10.0.0.0
> R1(config-router)# network 172.16.0.0
> ```
> 
> **Résultat** :
> 
> - Toutes les interfaces avec une IP en `192.168.1.x` activeront RIP
> - Toutes les interfaces avec une IP en `10.x.x.x` activeront RIP
> - Toutes les interfaces avec une IP en `172.16.x.x` activeront RIP

### Tableau de correspondance classes/masques

|Classe|Premier octet|Masque par défaut|Exemple network|
|---|---|---|---|
|A|1-126|255.0.0.0 (/8)|`network 10.0.0.0`|
|B|128-191|255.255.0.0 (/16)|`network 172.16.0.0`|
|C|192-223|255.255.255.0 (/24)|`network 192.168.1.0`|

> [!tip] Astuce pratique Si vous avez un doute sur la classe d'une adresse, regardez simplement le premier octet :
> 
> - 10.x.x.x → Classe A → `network 10.0.0.0`
> - 172.16.x.x → Classe B → `network 172.16.0.0`
> - 192.168.1.x → Classe C → `network 192.168.1.0`

### Pièges courants

> [!warning] Piège n°1 : Déclaration trop spécifique
> 
> ```bash
> # ❌ INCORRECT (sera converti automatiquement)
> R1(config-router)# network 192.168.1.1
> # Cisco convertira en 192.168.1.0
> 
> # ✅ CORRECT
> R1(config-router)# network 192.168.1.0
> ```

> [!warning] Piège n°2 : Oublier une interface Si vous ne déclarez pas tous les réseaux de vos interfaces actives, certaines interfaces ne participeront pas au routage dynamique et ne seront pas annoncées aux voisins.

> [!warning] Piège n°3 : Confusion avec OSPF/EIGRP Contrairement à OSPF ou EIGRP qui permettent des déclarations précises avec wildcard masks, RIP est limité aux réseaux classful dans la commande `network`.

---

## 🔄 Version 2 du protocole

### Pourquoi passer à RIPv2 ?

RIP existe en deux versions principales, et RIPv2 corrige plusieurs limitations critiques de RIPv1 :

|Caractéristique|RIPv1|RIPv2|
|---|---|---|
|**Type de mises à jour**|Broadcast (255.255.255.255)|Multicast (224.0.0.9)|
|**Support VLSM**|❌ Non|✅ Oui|
|**Masque de sous-réseau**|Non transmis|Transmis dans les mises à jour|
|**Authentification**|❌ Non supportée|✅ Supportée (MD5, texte clair)|
|**Classful/Classless**|Classful uniquement|Classless|
|**Efficacité réseau**|Moins efficace (broadcasts)|Plus efficace (multicast)|

> [!info] VLSM (Variable Length Subnet Mask) Le support du VLSM permet d'utiliser différents masques de sous-réseau dans le même réseau classful. C'est **indispensable** dans les réseaux modernes pour optimiser l'utilisation des adresses IP.

### Configuration de RIPv2

```bash
Router(config)# router rip
Router(config-router)# version 2
```

> [!example] Configuration complète
> 
> ```bash
> R1(config)# router rip
> R1(config-router)# version 2
> R1(config-router)# network 192.168.1.0
> R1(config-router)# network 10.0.0.0
> ```

### Vérification de la version

```bash
# Afficher la configuration RIP
R1# show ip protocols

# Vous verrez :
# Routing Protocol is "rip"
#   Sending updates every 30 seconds
#   ...
#   Routing for Networks:
#     192.168.1.0
#     10.0.0.0
#   Routing Information Sources:
#     ...
#   Distance: (default is 120)
```

> [!tip] Bonne pratique **Utilisez TOUJOURS RIPv2** dans vos configurations modernes. RIPv1 est obsolète et ne devrait être utilisé que pour maintenir la compatibilité avec de très vieux équipements.

### Impact du passage en version 2

Lorsque vous configurez `version 2` :

1. **Les mises à jour passent en multicast** (224.0.0.9) au lieu de broadcast
    
    - Réduit la charge sur les hôtes non-RIP du réseau
    - Plus efficace en termes de bande passante
2. **Les masques de sous-réseau sont inclus** dans chaque route annoncée
    
    - Permet le support du VLSM
    - Autorise les sous-réseaux de tailles différentes
3. **Possibilité d'activer l'authentification** (non détaillée ici, fait partie d'une autre section)
    

> [!warning] Compatibilité Si vous avez des routeurs avec RIPv1 et RIPv2 dans le même réseau :
> 
> - Par défaut, RIPv2 peut recevoir à la fois v1 et v2
> - Mais RIPv2 n'envoie QUE des mises à jour v2
> - Les routeurs RIPv1 ignoreront les mises à jour RIPv2

---

## 🎯 Désactivation de l'auto-summary

### Comprendre l'auto-summary

L'**auto-summary** (résumé automatique) est un comportement hérité de l'époque classful du routage. Lorsqu'il est activé, RIP :

1. **Résume automatiquement les sous-réseaux** en réseaux classful aux frontières de réseaux majeurs
2. **N'annonce que le réseau classful** au lieu des sous-réseaux spécifiques

> [!info] Définition : Frontière de réseau majeur Une frontière de réseau majeur se produit quand une interface d'un routeur appartient à un réseau classful différent de celui d'une autre interface.
> 
> **Exemple** : Si un routeur a une interface en `192.168.1.0/24` et une autre en `10.1.1.0/24`, il y a une frontière entre le réseau de classe C et le réseau de classe A.

### Problème posé par l'auto-summary

```
Exemple de topologie problématique :

R1: 192.168.1.0/24 ---- R2 ---- R3: 192.168.2.0/24
                    10.0.0.0/30   10.0.0.4/30

AVEC auto-summary activé :
- R2 annonce à R3 : "192.168.0.0/16" (résumé classful)
- R2 annonce à R1 : "192.168.0.0/16" (résumé classful)
- R3 ne peut pas distinguer 192.168.1.0 de 192.168.2.0
- Résultat : PERTE DE CONNECTIVITÉ
```

> [!warning] Danger critique de l'auto-summary L'auto-summary peut causer :
> 
> - **Perte de connectivité** entre sous-réseaux
> - **Boucles de routage** dans certaines topologies
> - **Blackholes** (trafic envoyé vers une destination inexistante)
> - **Routes ambiguës** quand plusieurs chemins existent vers le même résumé

### Configuration : Désactiver l'auto-summary

```bash
Router(config)# router rip
Router(config-router)# no auto-summary
```

> [!example] Configuration recommandée complète
> 
> ```bash
> R1(config)# router rip
> R1(config-router)# version 2
> R1(config-router)# no auto-summary
> R1(config-router)# network 192.168.1.0
> R1(config-router)# network 10.0.0.0
> ```

### Comportement avant/après

|Situation|Avec auto-summary|Sans auto-summary (no auto-summary)|
|---|---|---|
|Sous-réseau `192.168.1.0/24`|Annoncé comme `192.168.0.0/16`|Annoncé comme `192.168.1.0/24`|
|Sous-réseau `192.168.2.0/24`|Annoncé comme `192.168.0.0/16`|Annoncé comme `192.168.2.0/24`|
|Sous-réseau `10.1.1.0/24`|Annoncé comme `10.0.0.0/8`|Annoncé comme `10.1.1.0/24`|
|**Précision**|❌ Faible (perte d'info)|✅ Élevée (info complète)|

### Vérification

```bash
# Vérifier que l'auto-summary est désactivé
R1# show ip protocols

# Vous devriez voir :
# Routing Protocol is "rip"
#   ...
#   Automatic network summarization is not in effect
#   ...
```

```bash
# Vérifier les routes annoncées
R1# show ip rip database

# Avec "no auto-summary", vous verrez les sous-réseaux spécifiques
# Sans "no auto-summary", vous verriez les réseaux classful résumés
```

> [!tip] Règle d'or **TOUJOURS utiliser `no auto-summary` avec RIPv2** dans les réseaux modernes. C'est devenu un standard de configuration, et Cisco l'a même rendu par défaut dans les versions IOS récentes.

### Historique et évolution

- **IOS ancien** : `auto-summary` activé par défaut (comportement classful)
- **IOS récent (15.0+)** : `no auto-summary` par défaut (comportement classless)
- **Meilleure pratique** : Toujours spécifier explicitement `no auto-summary` pour éviter toute confusion

---

## 🔇 Passive-interface

### Concept et utilité

La commande `passive-interface` permet de **désactiver l'envoi de mises à jour RIP** sur une interface spécifique, tout en continuant à annoncer le réseau de cette interface aux autres routeurs.

> [!info] Qu'est-ce qu'une interface passive ? Une interface passive dans RIP :
> 
> - ✅ **Annonce toujours** son réseau aux voisins via les autres interfaces actives
> - ❌ **N'envoie PLUS** de mises à jour RIP sur cette interface
> - ❌ **N'écoute PLUS** les mises à jour RIP sur cette interface
> - ✅ **Reste opérationnelle** pour le trafic de données normal

### Cas d'usage principaux

#### 1. Interfaces vers les utilisateurs finaux

```
        [R1]
         |
    [Switch]
    /   |   \
[PC1][PC2][PC3]
```

Les PCs n'ont pas besoin de recevoir les mises à jour RIP (ils ne sont pas des routeurs). Envoyer des mises à jour RIP vers eux est :

- Inutile (ils ne les traiteront pas)
- Gaspillage de bande passante
- Risque de sécurité (exposition de la topologie)

#### 2. Interfaces vers l'Internet

```
[ISP Router]
      |
    [R1] --- [R2] (réseau interne)
```

Vous ne voulez PAS envoyer vos routes internes vers l'ISP via RIP.

#### 3. Interfaces de management

Les interfaces utilisées uniquement pour l'administration ne doivent pas participer au routage dynamique.

### Syntaxe et configuration

```bash
# Rendre une interface spécifique passive
Router(config)# router rip
Router(config-router)# passive-interface [type] [numéro]
```

```bash
# Rendre TOUTES les interfaces passives par défaut
# puis réactiver seulement celles nécessaires
Router(config)# router rip
Router(config-router)# passive-interface default
Router(config-router)# no passive-interface [type] [numéro]
```

> [!example] Exemple : Interface spécifique passive
> 
> ```bash
> R1(config)# router rip
> R1(config-router)# version 2
> R1(config-router)# no auto-summary
> R1(config-router)# network 192.168.1.0
> R1(config-router)# network 10.0.0.0
> R1(config-router)# passive-interface GigabitEthernet0/0
> ```
> 
> **Résultat** : GigabitEthernet0/0 ne recevra plus de mises à jour RIP, mais son réseau sera toujours annoncé aux autres routeurs.

> [!example] Exemple : Approche "passive par défaut"
> 
> ```bash
> R1(config)# router rip
> R1(config-router)# version 2
> R1(config-router)# no auto-summary
> R1(config-router)# network 192.168.1.0
> R1(config-router)# network 10.0.0.0
> R1(config-router)# passive-interface default
> R1(config-router)# no passive-interface GigabitEthernet0/1
> R1(config-router)# no passive-interface Serial0/0/0
> ```
> 
> **Résultat** : Seules Gi0/1 et S0/0/0 envoient/reçoivent des mises à jour RIP.

### Avantages de passive-interface

|Avantage|Description|
|---|---|
|**Sécurité**|Empêche la diffusion d'informations de routage sur des réseaux non fiables|
|**Performance**|Réduit le trafic inutile (mises à jour toutes les 30s évitées)|
|**Stabilité**|Évite les adjacences RIP non désirées|
|**Bande passante**|Économise de la bande passante sur les liens limités|
|**CPU**|Réduit légèrement la charge CPU (moins de traitement de paquets)|

### Vérification

```bash
# Lister toutes les interfaces passives
R1# show ip protocols

# Sortie typique :
# Routing Protocol is "rip"
#   ...
#   Passive Interface(s):
#     GigabitEthernet0/0
#   Routing for Networks:
#     192.168.1.0
#     10.0.0.0
#   ...
```

```bash
# Vérifier les voisins RIP actifs
R1# show ip rip neighbors
# (cette commande n'existe pas directement dans RIP, 
#  utilisez plutôt "show ip route rip" pour voir les routes apprises)
```

### Comparaison des approches

#### Approche 1 : Passive par interface

```bash
router rip
 passive-interface Gi0/0
 passive-interface Gi0/2
 # Gi0/1 et S0/0/0 restent actives
```

**Avantages** :

- ✅ Clair et explicite
- ✅ Facile à comprendre

**Inconvénients** :

- ❌ Doit être mis à jour si on ajoute des interfaces
- ❌ Plus de lignes de configuration

#### Approche 2 : Passive par défaut

```bash
router rip
 passive-interface default
 no passive-interface Gi0/1
 no passive-interface S0/0/0
```

**Avantages** :

- ✅ Plus sécurisé (nouvelles interfaces passives par défaut)
- ✅ Moins de lignes pour beaucoup d'interfaces passives
- ✅ Approche "deny all, allow few"

**Inconvénients** :

- ❌ Moins intuitif au premier abord
- ❌ Risque d'oublier d'activer une interface légitime

> [!tip] Recommandation professionnelle Dans les environnements de production, privilégiez l'approche **passive-interface default** puis réactivez explicitement les interfaces nécessaires. C'est une approche "secure by default" qui évite les erreurs de configuration.

### Pièges courants

> [!warning] Piège n°1 : Oublier de déclarer le network
> 
> ```bash
> # ❌ INCOMPLET - L'interface est passive mais le réseau n'est pas déclaré
> router rip
>  passive-interface Gi0/0
>  # Manque : network 192.168.1.0
> 
> # ✅ CORRECT
> router rip
>  network 192.168.1.0
>  passive-interface Gi0/0
> ```
> 
> Sans la commande `network`, le réseau ne sera PAS annoncé du tout !

> [!warning] Piège n°2 : Rendre passive la mauvaise interface Si vous rendez passive une interface censée communiquer avec un autre routeur RIP, l'adjacence ne se formera jamais et les routes ne seront pas échangées.

> [!warning] Piège n°3 : Confusion entre passive et shutdown
> 
> - `passive-interface` : L'interface reste UP, mais RIP ne communique plus dessus
> - `shutdown` : L'interface est complètement désactivée (DOWN)
> 
> Ce sont deux commandes **très différentes** !

---

## 📝 Configuration complète type

Voici une configuration RIPv2 complète intégrant toutes les bonnes pratiques :

```bash
!
hostname R1
!
interface GigabitEthernet0/0
 description LAN Utilisateurs
 ip address 192.168.1.1 255.255.255.0
 no shutdown
!
interface GigabitEthernet0/1
 description Vers R2
 ip address 10.1.1.1 255.255.255.252
 no shutdown
!
interface Serial0/0/0
 description Vers R3
 ip address 10.1.2.1 255.255.255.252
 no shutdown
!
router rip
 version 2                          ! Utiliser RIPv2 obligatoirement
 no auto-summary                    ! Désactiver le résumé automatique
 network 192.168.1.0                ! Déclarer le réseau LAN
 network 10.0.0.0                   ! Déclarer tous les liens inter-routeurs
 passive-interface default          ! Toutes les interfaces passives par défaut
 no passive-interface Gi0/1         ! Activer RIP vers R2
 no passive-interface Serial0/0/0   ! Activer RIP vers R3
!
end
```

> [!tip] Points clés de cette configuration
> 
> 1. ✅ RIPv2 activé pour le support VLSM
> 2. ✅ Auto-summary désactivé pour la précision
> 3. ✅ Approche "passive par défaut" pour la sécurité
> 4. ✅ Seules les interfaces nécessaires communiquent en RIP
> 5. ✅ Les réseaux utilisateurs sont protégés (pas de mises à jour RIP)

---

## 🎓 Récapitulatif des commandes essentielles

|Commande|Fonction|Mode|
|---|---|---|
|`router rip`|Active le processus RIP|Global config|
|`version 2`|Force l'utilisation de RIPv2|Router config|
|`no auto-summary`|Désactive le résumé automatique|Router config|
|`network [réseau]`|Déclare un réseau (classful)|Router config|
|`passive-interface [int]`|Rend une interface passive|Router config|
|`passive-interface default`|Rend toutes les interfaces passives|Router config|
|`no passive-interface [int]`|Réactive RIP sur une interface|Router config|
|`show ip protocols`|Affiche la config RIP active|Privileged EXEC|
|`show ip rip database`|Affiche la base de données RIP|Privileged EXEC|
|`show ip route rip`|Affiche uniquement les routes RIP|Privileged EXEC|

---

## ⚠️ Pièges globaux et bonnes pratiques

### Pièges à éviter absolument

> [!warning] Les 5 erreurs les plus fréquentes
> 
> 1. **Oublier `version 2`** → Utilisation de RIPv1 par défaut sur ancien IOS
> 2. **Oublier `no auto-summary`** → Perte de sous-réseaux spécifiques
> 3. **Ne pas utiliser passive-interface** → Mises à jour envoyées partout inutilement
> 4. **Mauvaise déclaration network** → Interfaces non activées pour RIP
> 5. **Rendre passive la mauvaise interface** → Perte d'adjacence RIP

### Checklist de configuration RIPv2

- [ ] Activer `router rip`
- [ ] Configurer `version 2`
- [ ] Ajouter `no auto-summary`
- [ ] Déclarer tous les réseaux avec `network` (format classful)
- [ ] Identifier les interfaces qui ne doivent PAS échanger de mises à jour
- [ ] Configurer `passive-interface` pour ces interfaces
- [ ] Vérifier avec `show ip protocols`
- [ ] Vérifier les routes RIP avec `show ip route rip`
- [ ] Tester la connectivité avec `ping` entre réseaux distants

### Bonnes pratiques professionnelles

> [!tip] Recommandations avancées
> 
> - **Toujours commenter** vos interfaces (`description`)
> - **Documenter** pourquoi une interface est passive
> - **Utiliser passive-interface default** dans les environnements complexes
> - **Vérifier systématiquement** après chaque modification
> - **Préférer RIPv2** même si RIPv1 suffirait techniquement
> - **Planifier la migration vers OSPF/EIGRP** si le réseau grandit (RIP limité à 15 sauts)

---

## 🔍 Commandes de vérification détaillées

### Vérifier la configuration RIP complète

```bash
R1# show ip protocols

# Informations affichées :
# - Version de RIP utilisée
# - État de l'auto-summary
# - Réseaux déclarés
# - Interfaces passives
# - Timers (update, invalid, holddown, flush)
# - Distance administrative (120 par défaut)
```

### Afficher les routes RIP uniquement

```bash
R1# show ip route rip

# Exemple de sortie :
# R    192.168.2.0/24 [120/1] via 10.1.1.2, 00:00:15, GigabitEthernet0/1
# R    192.168.3.0/24 [120/2] via 10.1.1.2, 00:00:15, GigabitEthernet0/1
#
# Légende :
# R = Route RIP
# [120/1] = [Distance administrative / Métrique (hop count)]
# via 10.1.1.2 = Next-hop
# 00:00:15 = Temps depuis dernière mise à jour
# GigabitEthernet0/1 = Interface de sortie
```

### Examiner la base de données RIP

```bash
R1# show ip rip database

# Affiche toutes les routes connues par RIP
# avant qu'elles soient insérées dans la table de routage
```

### Debug RIP (avec précaution)

```bash
R1# debug ip rip
# ⚠️ ATTENTION : Peut générer BEAUCOUP de sortie
# À utiliser uniquement en environnement de test

R1# undebug all
# Désactive tous les debugs actifs
```

---

_Configuration RIPv2 - Guide complet pour Cisco IOS_