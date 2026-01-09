

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

## 🎯 Coût et métrique OSPF

### Concept fondamental

OSPF utilise une métrique basée sur le **coût** pour déterminer le meilleur chemin vers une destination. Contrairement à RIP qui compte les sauts, OSPF calcule le coût en fonction de la bande passante des liens.

> [!info] Formule du coût OSPF **Coût = Bande passante de référence / Bande passante de l'interface**
> 
> Par défaut, la bande passante de référence est de **100 Mbps** (100 000 000 bps)

### Calcul du coût par défaut

|Type d'interface|Bande passante|Coût OSPF par défaut|
|---|---|---|
|Fast Ethernet (100 Mbps)|100 Mbps|1|
|Ethernet (10 Mbps)|10 Mbps|10|
|Serial (T1 = 1.544 Mbps)|1.544 Mbps|64|
|Serial (64 Kbps)|64 Kbps|1562|
|Gigabit Ethernet|1000 Mbps|1|
|10 Gigabit Ethernet|10000 Mbps|1|

> [!warning] Problème avec les interfaces rapides Avec la configuration par défaut, toutes les interfaces ≥ 100 Mbps ont un coût de 1, ce qui empêche OSPF de différencier un lien FastEthernet d'un lien 10GigE !

### Vérification du coût actuel

```bash
# Afficher le coût des interfaces
Router# show ip ospf interface brief

# Détails complets d'une interface
Router# show ip ospf interface GigabitEthernet0/0

# Voir le coût dans la table de routage
Router# show ip route ospf
```

### Modifier le coût manuellement

Il existe **deux méthodes** pour influencer le coût OSPF d'une interface :

#### Méthode 1 : Modifier le coût directement

```bash
Router(config)# interface GigabitEthernet0/0
Router(config-if)# ip ospf cost 50
```

> [!tip] Quand utiliser cette méthode ?
> 
> - Contrôle précis sur un lien spécifique
> - Engineering de trafic (forcer un chemin)
> - Valeur arbitraire sans lien avec la bande passante réelle

#### Méthode 2 : Modifier la bande passante de l'interface

```bash
Router(config)# interface GigabitEthernet0/0
Router(config-if)# bandwidth 100000  # en Kbps (ici 100 Mbps)
```

> [!info] Impact de la commande `bandwidth` Cette commande :
> 
> - N'affecte PAS la vitesse réelle de l'interface
> - Influence UNIQUEMENT les calculs de protocoles (OSPF, EIGRP, QoS)
> - Permet à OSPF de recalculer automatiquement le coût

### Vérification du coût modifié

```bash
Router# show ip ospf interface GigabitEthernet0/0
GigabitEthernet0/0 is up, line protocol is up
  Internet Address 10.1.1.1/24, Area 0
  Process ID 1, Router ID 1.1.1.1, Network Type BROADCAST, Cost: 50
  # Le coût a été modifié à 50
```

> [!example] Exemple pratique **Scénario** : Vous avez deux chemins vers un réseau distant :
> 
> - Chemin A : Via un lien FastEthernet (coût 1)
> - Chemin B : Via un lien Gigabit (coût 1)
> 
> OSPF va load-balance entre les deux (même coût), mais vous préférez le Gigabit.
> 
> **Solution** :
> 
> ```bash
> interface FastEthernet0/0
>   ip ospf cost 10
> ```
> 
> Maintenant OSPF préférera le chemin Gigabit (coût 1 < 10)

### Bonnes pratiques

1. **Privilégier `ip ospf cost`** pour un contrôle précis
2. **Utiliser `bandwidth`** si vous voulez aussi influencer d'autres protocoles (QoS, EIGRP)
3. **Documenter** toutes les modifications de coût (pas de valeur par défaut)
4. **Rester cohérent** dans votre politique de coûts à travers le réseau

---

## 📊 Modification de la bande passante de référence

### Pourquoi modifier la bande passante de référence ?

> [!warning] Problème avec la valeur par défaut La bande passante de référence par défaut (100 Mbps) date d'une époque où FastEthernet était rapide. Aujourd'hui :
> 
> - Gigabit Ethernet = coût 1
> - 10 Gigabit Ethernet = coût 1
> - 40 Gigabit Ethernet = coût 1
> 
> **OSPF ne peut pas différencier ces liens !**

### Calcul avec une nouvelle référence

Si vous définissez la référence à 10 Gbps (10 000 Mbps) :

|Type d'interface|Bande passante|Nouveau coût OSPF|
|---|---|---|
|10 Gigabit Ethernet|10 Gbps|1|
|Gigabit Ethernet|1 Gbps|10|
|Fast Ethernet|100 Mbps|100|
|Ethernet|10 Mbps|1000|

### Configuration globale

```bash
# Entrer dans le processus OSPF
Router(config)# router ospf 1

# Modifier la bande passante de référence (en Mbps)
Router(config-router)# auto-cost reference-bandwidth 10000
% OSPF: Reference bandwidth is changed.
      Please ensure reference bandwidth is consistent across all routers.
```

> [!info] Unité de mesure La valeur est en **Mbps** (pas en Kbps !)
> 
> - Pour 1 Gbps : `auto-cost reference-bandwidth 1000`
> - Pour 10 Gbps : `auto-cost reference-bandwidth 10000`
> - Pour 100 Gbps : `auto-cost reference-bandwidth 100000`

### Vérification

```bash
# Vérifier la configuration OSPF
Router# show ip protocols
*** IP Routing is NSF aware ***

Routing Protocol is "ospf 1"
  Outgoing update filter list for all interfaces is not set
  Incoming update filter list for all interfaces is not set
  Router ID 1.1.1.1
  Number of areas in this router is 1. 1 normal 0 stub 0 nssa
  Maximum path: 4
  Routing for Networks:
  Reference bandwidth unit is 10000 mbps
  # Ici on voit la nouvelle référence
```

> [!warning] Règle d'or : COHÉRENCE ! **TOUS les routeurs OSPF du domaine de routage doivent utiliser la même bande passante de référence !**
> 
> Sinon :
> 
> - Les calculs de coût seront incohérents
> - Les chemins optimaux ne seront pas les mêmes partout
> - Risque de boucles de routage ou de chemins sous-optimaux

### Exemple de configuration réseau

```bash
# À configurer sur TOUS les routeurs du réseau OSPF
Router1(config)# router ospf 1
Router1(config-router)# auto-cost reference-bandwidth 10000

Router2(config)# router ospf 1
Router2(config-router)# auto-cost reference-bandwidth 10000

Router3(config)# router ospf 1
Router3(config-router)# auto-cost reference-bandwidth 10000
```

### Recommandations

> [!tip] Choix de la valeur de référence
> 
> - **Réseau moderne (Gigabit+)** : 10000 Mbps (10 Gbps)
> - **Datacenter (10G+)** : 100000 Mbps (100 Gbps)
> - **Réseau mixte** : Choisir en fonction du lien le plus rapide × 10
> 
> **Formule recommandée** : Référence = Lien le plus rapide du réseau

> [!example] Exemple de décision Votre réseau contient :
> 
> - Quelques liens 10 GigE au cœur
> - Majoritairement du Gigabit en distribution
> - Du FastEthernet en accès
> 
> **Recommandation** : `auto-cost reference-bandwidth 10000`
> 
> Résultat :
> 
> - 10 GigE : coût 1 (optimal)
> - 1 GigE : coût 10
> - 100 Mbps : coût 100

---

## 👑 Priority et élection DR/BDR

### Concept du DR et BDR

Sur les réseaux multi-accès (Ethernet), OSPF élit un **Designated Router (DR)** et un **Backup Designated Router (BDR)** pour optimiser les échanges de LSA.

> [!info] Rôles
> 
> - **DR** : Collecte et redistribue les LSA à tous les routeurs
> - **BDR** : Prend le relais si le DR tombe
> - **DROTHER** : Tous les autres routeurs (communiquent uniquement avec DR/BDR)

### Pourquoi contrôler l'élection ?

> [!tip] Raisons de modifier la priority
> 
> 1. **Performance** : Désigner le routeur le plus puissant comme DR
> 2. **Stabilité** : Éviter qu'un routeur peu fiable devienne DR
> 3. **Topologie** : Le routeur le mieux connecté doit être DR
> 4. **Design** : Forcer un routeur spécifique dans un rôle précis

### Processus d'élection DR/BDR

L'élection se base sur (dans cet ordre) :

1. **Priority la plus élevée** (0-255, défaut = 1)
2. En cas d'égalité : **Router ID le plus élevé**

> [!warning] L'élection n'est PAS préemptive !
> 
> - L'élection se fait au démarrage du réseau
> - Si un nouveau routeur avec une priority plus élevée arrive, il ne devient PAS automatiquement DR
> - Le DR actuel garde son rôle jusqu'à ce qu'il tombe ou redémarre
> - Pour forcer une nouvelle élection : redémarrer OSPF sur tous les routeurs

### Configuration de la priority

```bash
# Configurer la priority sur une interface
Router(config)# interface GigabitEthernet0/0
Router(config-if)# ip ospf priority 100

# Priority de 0 = Le routeur ne peut JAMAIS être DR ou BDR
Router(config-if)# ip ospf priority 0
```

> [!info] Valeurs de priority
> 
> - **0** : Le routeur ne participe PAS à l'élection (jamais DR/BDR)
> - **1** : Valeur par défaut
> - **1-255** : Plus la valeur est élevée, plus le routeur a de chances d'être élu DR
> - **255** : Priority maximale (quasi-garanti d'être DR)

### Vérification de l'état DR/BDR

```bash
# Voir l'état sur une interface
Router# show ip ospf interface GigabitEthernet0/0
GigabitEthernet0/0 is up, line protocol is up
  Internet Address 10.1.1.1/24, Area 0
  Process ID 1, Router ID 1.1.1.1, Network Type BROADCAST, Cost: 1
  Designated Router (ID) 2.2.2.2, Interface address 10.1.1.2
  Backup Designated router (ID) 3.3.3.3, Interface address 10.1.1.3
  Timer intervals configured, Hello 10, Dead 40, Wait 40, Retransmit 5
    oob-resync timeout 40
  Priority 100
  # Ce routeur a une priority de 100

# Vue d'ensemble des voisins et de leur rôle
Router# show ip ospf neighbor
Neighbor ID     Pri   State           Dead Time   Address         Interface
2.2.2.2         100   FULL/DR         00:00:35    10.1.1.2        GigabitEthernet0/0
3.3.3.3          50   FULL/BDR        00:00:38    10.1.1.3        GigabitEthernet0/0
4.4.4.4           1   FULL/DROTHER    00:00:32    10.1.1.4        GigabitEthernet0/0
```

### Forcer une nouvelle élection

```bash
# Méthode 1 : Redémarrer le processus OSPF (recommandé)
Router# clear ip ospf process
Reset ALL OSPF processes? [no]: yes

# Méthode 2 : Désactiver/réactiver l'interface
Router(config)# interface GigabitEthernet0/0
Router(config-if)# shutdown
Router(config-if)# no shutdown

# Méthode 3 : Redémarrer le routeur (dernière option)
Router# reload
```

> [!example] Scénario d'utilisation **Situation** : Vous avez 4 routeurs sur un segment Ethernet
> 
> - R1 : Routeur de cœur puissant
> - R2 : Routeur de cœur secondaire
> - R3 : Routeur de distribution
> - R4 : Routeur d'accès
> 
> **Objectif** : R1 doit être DR, R2 doit être BDR, R4 ne doit jamais être DR/BDR
> 
> **Configuration** :
> 
> ```bash
> # Sur R1
> interface GigabitEthernet0/0
>   ip ospf priority 200
> 
> # Sur R2
> interface GigabitEthernet0/0
>   ip ospf priority 150
> 
> # Sur R3
> interface GigabitEthernet0/0
>   ip ospf priority 50
> 
> # Sur R4
> interface GigabitEthernet0/0
>   ip ospf priority 0
> ```

### Bonnes pratiques

> [!tip] Recommandations
> 
> 1. **Toujours définir des priorities explicites** sur les réseaux multi-accès
> 2. **Utiliser priority 0** sur les routeurs qui ne doivent jamais être DR/BDR
> 3. **Espacer les valeurs** (ex: 200, 150, 100, 50) pour une hiérarchie claire
> 4. **Documenter** le rôle attendu de chaque routeur
> 5. **Tester** l'élection après toute modification (avec `clear ip ospf process`)

> [!warning] Pièges courants
> 
> - Ne pas oublier que l'élection n'est **pas préemptive**
> - Une priority de 0 signifie "jamais DR/BDR", pas "faible priorité"
> - La priority est configurée **par interface**, pas globalement
> - Sur les liaisons point-à-point, il n'y a **pas d'élection DR/BDR** (inutile)

---

## ⏱️ Hello et Dead intervals

### Rôle des timers OSPF

OSPF utilise deux timers critiques pour maintenir les adjacences :

> [!info] Les deux timers
> 
> - **Hello interval** : Fréquence d'envoi des paquets Hello (maintien de la relation de voisinage)
> - **Dead interval** : Délai avant de déclarer un voisin "mort" si aucun Hello n'est reçu

### Valeurs par défaut

|Type de réseau|Hello interval|Dead interval|Relation|
|---|---|---|---|
|Broadcast (Ethernet)|10 secondes|40 secondes|Dead = 4 × Hello|
|Point-to-Point|10 secondes|40 secondes|Dead = 4 × Hello|
|NBMA|30 secondes|120 secondes|Dead = 4 × Hello|

> [!warning] Règle impérative **Les timers Hello et Dead DOIVENT être identiques sur les deux extrémités d'un lien !**
> 
> Sinon, l'adjacence ne se formera pas (état bloqué à INIT ou 2WAY).

### Pourquoi modifier les timers ?

> [!tip] Cas d'usage **Réduire les timers** (convergence plus rapide) :
> 
> - Réseaux critiques nécessitant une détection rapide des pannes
> - Liens instables où vous voulez détecter rapidement les problèmes
> - Environnements à haute disponibilité
> 
> **Augmenter les timers** (réduire la charge CPU) :
> 
> - Liens lents ou saturés
> - Routeurs avec ressources limitées
> - Réseaux stables où la convergence rapide n'est pas critique

### Configuration des timers

```bash
# Modifier le Hello interval (en secondes)
Router(config)# interface GigabitEthernet0/0
Router(config-if)# ip ospf hello-interval 5

# Modifier le Dead interval (en secondes)
Router(config-if)# ip ospf dead-interval 20
```

> [!info] Configuration automatique Si vous modifiez uniquement le Hello interval, Cisco IOS ajuste automatiquement le Dead interval à 4 × Hello.
> 
> Vous pouvez aussi définir manuellement les deux pour un contrôle total.

### Configuration avancée

```bash
# Dead interval minimal (1 seconde) avec sub-second hello
Router(config-if)# ip ospf dead-interval minimal hello-multiplier 4
# Envoie 4 Hellos par seconde (250ms entre chaque)
# Dead interval = 1 seconde
```

> [!warning] Sub-second timers : attention ! Les timers sub-second (< 1s) :
> 
> - **Augmentent significativement la charge CPU**
> - Sont sensibles à la gigue réseau
> - Peuvent causer des flaps d'adjacence
> - Réservés aux environnements très contrôlés (datacenter haute dispo)

### Vérification des timers

```bash
# Voir les timers sur une interface
Router# show ip ospf interface GigabitEthernet0/0
GigabitEthernet0/0 is up, line protocol is up
  Internet Address 10.1.1.1/24, Area 0
  Process ID 1, Router ID 1.1.1.1, Network Type BROADCAST, Cost: 1
  Timer intervals configured, Hello 5, Dead 20, Wait 20, Retransmit 5
  # Hello = 5s, Dead = 20s

# Vérifier que les voisins ont les mêmes timers
Router# show ip ospf neighbor detail
Neighbor 2.2.2.2, interface address 10.1.1.2
    Dead timer due in 00:00:17
    # Le dead timer compte à rebours depuis 20s
```

> [!example] Scénario de convergence rapide **Besoin** : Datacenter critique, convergence < 2 secondes
> 
> **Configuration** :
> 
> ```bash
> # Sur les deux routeurs
> interface GigabitEthernet0/0
>   ip ospf hello-interval 1
>   ip ospf dead-interval 3
> ```
> 
> **Résultat** :
> 
> - Hello toutes les 1 seconde
> - Panne détectée en maximum 3 secondes
> - Convergence totale en ~4-5 secondes

### Diagnostic des problèmes de timers

```bash
# Si l'adjacence ne se forme pas, vérifier les timers
Router# debug ip ospf hello
OSPF-1 HELLO Gi0/0: Rcv hello from 2.2.2.2 area 0 10.1.1.2
OSPF-1 HELLO Gi0/0: Mismatched hello parameters from 10.1.1.2
OSPF-1 HELLO Gi0/0: Dead timer mismatch 40 from 10.1.1.2 configured 20
# Le voisin a un Dead de 40s, nous avons 20s → adjacence impossible !
```

### Bonnes pratiques

> [!tip] Recommandations
> 
> 1. **Garder la relation 4:1** entre Dead et Hello (standard et prévisible)
> 2. **Modifier les timers sur les DEUX extrémités** du lien simultanément
> 3. **Tester** avant de déployer en production (risque de perte d'adjacence)
> 4. **Éviter** les valeurs extrêmes sans raison valable
> 5. **Documenter** toute modification par rapport aux valeurs par défaut
> 6. **Monitorer** la charge CPU après avoir réduit les timers

> [!warning] Pièges à éviter
> 
> - Ne JAMAIS modifier un seul côté du lien
> - Les timers trop courts peuvent causer des flaps sur liens instables
> - Les timers trop longs retardent la détection de pannes réelles
> - Sub-second timers = charge CPU élevée (attention aux routeurs anciens)

### Tableau récapitulatif des timers courants

|Scénario|Hello (s)|Dead (s)|Convergence|Charge CPU|
|---|---|---|---|---|
|Défaut|10|40|~50s|Faible|
|Convergence rapide|1|3|~5s|Moyenne|
|Convergence très rapide|Sub-second|1|~2s|Élevée|
|Lien lent/stable|30|120|~150s|Très faible|

---

## 🔐 Authentication OSPF

### Pourquoi authentifier OSPF ?

> [!warning] Menaces sans authentication Sans authentication, un attaqueur peut :
> 
> - Injecter de fausses routes dans le réseau
> - Créer un routeur malveillant et devenir DR
> - Effectuer une attaque DoS en envoyant des paquets OSPF
> - Modifier la topologie réseau
> - Intercepter du trafic via route hijacking

L'authentication OSPF garantit que seuls les routeurs autorisés participent au domaine OSPF.

### Types d'authentication OSPF

OSPF supporte trois modes d'authentication :

|Type|Numéro|Sécurité|Usage|
|---|---|---|---|
|**Null** (aucune)|0|❌ Aucune|Par défaut, non recommandé|
|**Clear-text**|1|⚠️ Faible|Obsolète, mot de passe en clair|
|**MD5**|2|✅ Bonne|Recommandé, hash cryptographique|

> [!tip] Recommandation moderne **Toujours utiliser MD5** (Type 2) pour l'authentication OSPF.
> 
> Clear-text (Type 1) envoie le mot de passe en clair sur le réseau = vulnérable au sniffing.

### Configuration : Authentication par interface

```bash
# Méthode la plus flexible et recommandée

# Étape 1 : Activer l'authentication MD5 sur l'interface
Router(config)# interface GigabitEthernet0/0
Router(config-if)# ip ospf authentication message-digest

# Étape 2 : Définir la clé d'authentication
Router(config-if)# ip ospf message-digest-key 1 md5 MonMotDePasse123
```

> [!info] Paramètres de la clé
> 
> - **Key-id** (1 dans l'exemple) : Numéro d'identification de la clé (1-255)
> - Permet de faire tourner les clés (migration sans interruption)
> - Tous les routeurs doivent utiliser le même Key-id ET le même mot de passe

### Configuration : Authentication par area

```bash
# Configure l'authentication pour toute l'area
# Toutes les interfaces dans cette area utiliseront l'authentication

Router(config)# router ospf 1
Router(config-router)# area 0 authentication message-digest

# Puis définir la clé sur chaque interface
Router(config)# interface GigabitEthernet0/0
Router(config-if)# ip ospf message-digest-key 1 md5 MonMotDePasse123
```

> [!tip] Quelle méthode choisir ?
> 
> - **Par interface** : Plus flexible, authentication différente par lien
> - **Par area** : Plus simple à gérer, cohérence garantie dans toute l'area

### Vérification de l'authentication

```bash
# Vérifier la configuration sur une interface
Router# show ip ospf interface GigabitEthernet0/0
GigabitEthernet0/0 is up, line protocol is up
  Internet Address 10.1.1.1/24, Area 0
  Process ID 1, Router ID 1.1.1.1, Network Type BROADCAST, Cost: 1
  Message digest authentication enabled
    Youngest key id is 1
  # L'authentication MD5 est activée avec la clé 1

# Voir les voisins et leur état
Router# show ip ospf neighbor
Neighbor ID     Pri   State           Dead Time   Address         Interface
2.2.2.2          1   FULL/DR         00:00:35    10.1.1.2        GigabitEthernet0/0
# Si l'adjacence est FULL, l'authentication fonctionne
```

### Rotation de clés (Key Rollover)

L'un des avantages de l'authentication OSPF est la possibilité de changer les clés **sans interruption de service**.

```bash
# Étape 1 : Ajouter une nouvelle clé sur TOUS les routeurs (en plus de l'ancienne)
Router1(config-if)# ip ospf message-digest-key 2 md5 NouveauMotDePasse456

Router2(config-if)# ip ospf message-digest-key 2 md5 NouveauMotDePasse456

# À ce stade : Les routeurs acceptent les clés 1 ET 2, mais envoient avec la plus récente (2)

# Étape 2 : Attendre quelques minutes pour s'assurer que tout fonctionne

# Étape 3 : Supprimer l'ancienne clé sur TOUS les routeurs
Router1(config-if)# no ip ospf message-digest-key 1

Router2(config-if)# no ip ospf message-digest-key 1
```

> [!example] Scénario de migration Vous avez 10 routeurs avec la clé 1, vous voulez migrer vers la clé 2.
> 
> **Procédure** :
> 
> 1. Ajouter `key 2` sur les 10 routeurs (les adjacences restent up)
> 2. Vérifier que toutes les adjacences sont FULL
> 3. Retirer `key 1` sur les 10 routeurs
> 
> **Résultat** : Migration sans perte de connectivité !

### Authentication Clear-text (à éviter)

```bash
# Pour information uniquement - NE PAS UTILISER EN PRODUCTION

# Par interface
Router(config-if)# ip ospf authentication
Router(config-if)# ip ospf authentication-key MotDePasseClair

# Par area
Router(config-router)# area 0 authentication
Router(config-if)# ip ospf authentication-key MotDePasseClair
```

> [!warning] Danger du Clear-text Le mot de passe est visible en clair dans :
> 
> - Les paquets OSPF sur le réseau (Wireshark)
> - La configuration du routeur
> 
> **Ne JAMAIS utiliser en production moderne !**

### Dépannage de l'authentication

```bash
# Activer le debug OSPF
Router# debug ip ospf adj
OSPF-1 ADJ Gi0/0: Rcv pkt from 10.1.1.2 : Mismatched Authentication type.
Input packet specified type 0, we use type 2
# Le voisin n'a pas d'authentication, nous utilisons MD5

# Autre erreur courante
OSPF-1 ADJ Gi0/0: Rcv pkt from 10.1.1.2 : Mismatched Authentication Key
# Le mot de passe ne correspond pas
```

> [!tip] Checklist de dépannage Si l'adjacence ne se forme pas :
> 
> 1. ✅ Vérifier que l'authentication est activée des DEUX côtés
> 2. ✅ Vérifier que le TYPE d'authentication est identique (MD5 vs clear-text)
> 3. ✅ Vérifier que le Key-ID est identique
> 4. ✅ Vérifier que le mot de passe est identique (sensible à la casse !)
> 5. ✅ Vérifier avec `debug ip ospf adj`

### Bonnes pratiques de sécurité

> [!tip] Recommandations
> 
> 1. **Toujours utiliser MD5** (jamais clear-text)
> 2. **Mots de passe forts** : 16+ caractères, alphanumériques + symboles
> 3. **Rotation régulière** : Changer les clés tous les 6-12 mois
> 4. **Documentation sécurisée** : Stocker les mots de passe dans un gestionnaire sécurisé
> 5. **Clés différentes par area** : Isolation en cas de compromission
> 6. **Monitoring** : Alerter sur les échecs d'authentication
> 7. **Tests** : Toujours tester sur un lien non-critique avant déploiement massif

### Exemple complet de configuration sécurisée

```bash
# Configuration complète d'un routeur avec authentication OSPF

# Définir le processus OSPF avec authentication par area
Router(config)# router ospf 1
Router(config-router)# router-id 1.1.1.1
Router(config-router)# area 0 authentication message-digest
Router(config-router)# network 10.1.1.0 0.0.0.255 area 0
Router(config-router)# network 10.2.1.0 0.0.0.255 area 0

# Configuration des interfaces
Router(config)# interface GigabitEthernet0/0
Router(config-if)# ip address 10.1.1.1 255.255.255.0
Router(config-if)# ip ospf message-digest-key 1 md5 Secure_P@ssw0rd_2024
Router(config-if)# ip ospf 1 area 0

Router(config)# interface GigabitEthernet0/1
Router(config-if)# ip address 10.2.1.1 255.255.255.0
Router(config-if)# ip ospf message-digest-key 1 md5 Secure_P@ssw0rd_2024
Router(config-if)# ip ospf 1 area 0
```

### Vérification finale

```bash
# Commandes de vérification complète
Router# show ip ospf interface brief
Router# show ip ospf neighbor
Router# show ip ospf
Router# show ip protocols

# S'assurer qu'il n'y a pas d'erreurs d'authentication
Router# show ip ospf statistics
```

> [!example] Scénario réel : Sécurisation d'un réseau OSPF **Situation initiale** : Réseau de 20 routeurs OSPF sans authentication
> 
> **Objectif** : Déployer MD5 authentication sans interruption
> 
> **Plan d'action** :
> 
> 1. **Fenêtre de maintenance** : Planifier en heures creuses
> 2. **Configuration en bloc** :
>     - Activer `area 0 authentication message-digest` sur tous les routeurs
>     - Configurer la clé MD5 sur toutes les interfaces
> 3. **Validation** : Vérifier toutes les adjacences avec `show ip ospf neighbor`
> 4. **Monitoring** : Surveiller les logs pendant 24h
> 5. **Documentation** : Mettre à jour les diagrammes et la base de données
> 
> **Résultat** : Migration réussie en 30 minutes, aucune interruption

---

## 📝 Récapitulatif des commandes essentielles

### Coût OSPF

```bash
# Modifier le coût directement
ip ospf cost <1-65535>

# Modifier via la bande passante
bandwidth <kbps>

# Modifier la référence globale (en Mbps)
auto-cost reference-bandwidth <1-4294967>

# Vérification
show ip ospf interface [interface]
show ip route ospf
show ip protocols
```

### Priority DR/BDR

```bash
# Configurer la priority
ip ospf priority <0-255>

# Forcer une nouvelle élection
clear ip ospf process

# Vérification
show ip ospf interface
show ip ospf neighbor
```

### Timers Hello/Dead

```bash
# Modifier les timers
ip ospf hello-interval <secondes>
ip ospf dead-interval <secondes>

# Timers sub-second
ip ospf dead-interval minimal hello-multiplier <3-20>

# Vérification
show ip ospf interface
debug ip ospf hello
```

### Authentication OSPF

```bash
# Par interface (MD5)
ip ospf authentication message-digest
ip ospf message-digest-key <key-id> md5 <password>

# Par area
area <area-id> authentication message-digest
ip ospf message-digest-key <key-id> md5 <password>

# Vérification
show ip ospf interface
show ip ospf
debug ip ospf adj
```

---

## 🎯 Points clés à retenir

> [!tip] Optimisation OSPF : Les essentiels
> 
> **Coût et métrique** :
> 
> - Formule : Coût = Référence / Bande passante
> - Modifier la référence pour les réseaux modernes (10000+ Mbps)
> - Utiliser `ip ospf cost` pour un contrôle précis
> 
> **DR/BDR** :
> 
> - Priority 0 = Jamais DR/BDR
> - Plus haute priority = DR, seconde = BDR
> - Élection NON préemptive (redémarrer OSPF pour forcer)
> 
> **Timers** :
> 
> - Hello : Fréquence des paquets (défaut 10s)
> - Dead : Délai avant déclaration "mort" (défaut 40s)
> - Relation recommandée : Dead = 4 × Hello
> - Doivent être IDENTIQUES des deux côtés
> 
> **Authentication** :
> 
> - Toujours utiliser MD5 (Type 2)
> - Rotation de clés sans interruption possible
> - Même clé ET même key-id sur les deux routeurs

---

## ⚠️ Erreurs fréquentes à éviter

> [!warning] Pièges courants
> 
> 1. **Référence de bande passante** : Oublier de la configurer sur TOUS les routeurs
> 2. **Priority** : Croire que l'élection est préemptive (elle ne l'est pas !)
> 3. **Timers** : Modifier un seul côté du lien (adjacence cassée)
> 4. **Authentication** : Utiliser clear-text ou des mots de passe faibles
> 5. **Coût** : Ne pas vérifier l'impact sur les chemins de routage après modification
> 6. **DR/BDR** : Configurer priority 0 sur tous les routeurs (aucun DR/BDR possible)
> 7. **Sub-second timers** : Les utiliser sans mesurer la charge CPU

---

## 🔍 Méthodologie de dépannage

Quand OSPF ne fonctionne pas comme attendu après optimisation :

```bash
# 1. Vérifier la configuration de base
show running-config | section router ospf
show ip ospf

# 2. Vérifier les interfaces
show ip ospf interface brief
show ip ospf interface <interface>

# 3. Vérifier les adjacences
show ip ospf neighbor
show ip ospf neighbor detail

# 4. Vérifier les routes
show ip route ospf
show ip ospf database

# 5. Activer les debugs (avec prudence !)
debug ip ospf adj
debug ip ospf hello
debug ip ospf packet

# 6. Désactiver les debugs
undebug all
```

> [!tip] Approche systématique
> 
> 1. **Identifier** : Quel paramètre a été modifié ?
> 2. **Vérifier** : Les deux côtés du lien ont-ils la même config ?
> 3. **Tester** : Isoler le problème sur un seul lien
> 4. **Corriger** : Appliquer la modification correcte
> 5. **Valider** : Vérifier que l'adjacence est FULL et stable
> 6. **Documenter** : Noter la modification et sa raison

---

## 📊 Tableau de décision rapide

|Objectif|Action recommandée|Commande clé|
|---|---|---|
|Forcer un chemin spécifique|Modifier le coût|`ip ospf cost X`|
|Réseau avec liens 10G+|Augmenter la référence|`auto-cost reference-bandwidth 10000`|
|Désigner un DR spécifique|Augmenter sa priority|`ip ospf priority 200`|
|Empêcher d'être DR/BDR|Priority à 0|`ip ospf priority 0`|
|Convergence plus rapide|Réduire les timers|`ip ospf hello-interval 1` + `dead-interval 3`|
|Sécuriser OSPF|Activer MD5|`ip ospf authentication message-digest`|
|Lien saturé/lent|Augmenter les timers|`ip ospf hello-interval 30`|
|Rotation de clé|Ajouter nouvelle clé|`ip ospf message-digest-key 2 md5 ...`|

---

_Fin du cours - Optimisation OSPF_