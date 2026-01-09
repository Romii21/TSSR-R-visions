

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

## 🔌 Introduction aux interfaces série

Les interfaces série sont utilisées pour les connexions WAN (Wide Area Network) point-à-point entre routeurs. Contrairement aux interfaces Ethernet qui sont couramment utilisées dans les LAN, les interfaces série nécessitent une configuration spécifique pour établir la communication.

> [!info] Identification des interfaces série Les interfaces série sont identifiées par la notation **Serial X/Y/Z** ou **S X/Y/Z**
> 
> - Exemples : `Serial 0/0/0`, `Serial 0/1/0`, `S0/0/0`
> - Sur les anciens équipements : `Serial 0`, `Serial 1`

### 🔑 Concepts clés

**DCE vs DTE** : Deux types d'équipements dans une liaison série

- **DCE (Data Circuit-terminating Equipment)** : Fournit l'horloge (clock) pour la synchronisation
- **DTE (Data Terminal Equipment)** : Reçoit l'horloge du DCE

> [!warning] Important Dans un environnement de production, le DCE est généralement le modem ou l'équipement du fournisseur d'accès. Dans un lab, vous devez configurer manuellement quel routeur sera le DCE.

---

## ⏱️ Configuration du Clock Rate (DCE/DTE)

### Pourquoi configurer le clock rate ?

Le clock rate (fréquence d'horloge) définit la vitesse de transmission des données sur la liaison série. Il doit être configuré **uniquement sur le côté DCE** de la connexion.

### 🔍 Identifier le côté DCE

Avant de configurer le clock rate, vous devez identifier quel routeur est connecté en tant que DCE :

```bash
Router# show controllers serial 0/0/0
```

> [!example] Exemple de sortie
> 
> ```
> Interface Serial0/0/0
> Hardware is PowerQUICC MPC860
> DCE V.35, clock rate 2000000
> ```
> 
> ou
> 
> ```
> Interface Serial0/0/0
> Hardware is PowerQUICC MPC860
> DTE V.35
> ```

### 📝 Configuration du clock rate

```bash
Router(config)# interface serial 0/0/0
Router(config-if)# clock rate 64000
```

> [!tip] Valeurs courantes de clock rate
> 
> - **64000** : 64 Kbps (ligne RNIS basique)
> - **128000** : 128 Kbps
> - **1544000** : T1 (1.544 Mbps)
> - **2048000** : E1 (2.048 Mbps)
> - **4000000** : 4 Mbps

### 🎯 Configuration complète d'une interface DCE

```bash
Router(config)# interface serial 0/0/0
Router(config-if)# description Liaison WAN vers Routeur_B
Router(config-if)# ip address 10.1.1.1 255.255.255.252
Router(config-if)# clock rate 2000000
Router(config-if)# no shutdown
```

### 🎯 Configuration complète d'une interface DTE

```bash
Router(config)# interface serial 0/0/0
Router(config-if)# description Liaison WAN vers Routeur_A
Router(config-if)# ip address 10.1.1.2 255.255.255.252
Router(config-if)# no shutdown
# Pas de clock rate sur le DTE !
```

> [!warning] Erreur fréquente Ne **jamais** configurer le clock rate sur le côté DTE. Cela n'aura aucun effet et peut créer de la confusion lors du dépannage.

### ✅ Vérification

```bash
Router# show interfaces serial 0/0/0
Router# show ip interface brief
```

---

## 📦 Encapsulation HDLC

### Qu'est-ce que HDLC ?

**HDLC (High-Level Data Link Control)** est le protocole d'encapsulation par défaut sur les interfaces série des routeurs Cisco. C'est un protocole de couche 2 (liaison de données) simple et efficace.

> [!info] Caractéristiques de HDLC Cisco
> 
> - **Propriétaire Cisco** : La version Cisco de HDLC n'est pas compatible avec les autres fabricants
> - **Léger** : Très peu de overhead, excellentes performances
> - **Pas d'authentification** : Aucune sécurité intégrée
> - **Point-à-point uniquement** : Ne supporte que les liaisons directes entre deux routeurs

### 🔄 Quand utiliser HDLC ?

|✅ Utiliser HDLC|❌ Ne pas utiliser HDLC|
|---|---|
|Liaisons entre routeurs Cisco uniquement|Connexion avec équipements non-Cisco|
|Besoin de performances maximales|Besoin d'authentification|
|Liaisons privées sécurisées physiquement|Besoin de fonctionnalités avancées|
|Configuration simple et rapide|Environnements multi-protocoles|

### 📝 Configuration de HDLC

HDLC étant l'encapsulation par défaut, aucune configuration n'est nécessaire sur un nouveau routeur Cisco. Cependant, si vous changez d'encapsulation, voici comment revenir à HDLC :

```bash
Router(config)# interface serial 0/0/0
Router(config-if)# encapsulation hdlc
```

### 🎯 Configuration complète avec HDLC

```bash
Router(config)# interface serial 0/0/0
Router(config-if)# description Liaison HDLC vers Site_B
Router(config-if)# ip address 192.168.1.1 255.255.255.252
Router(config-if)# encapsulation hdlc
Router(config-if)# clock rate 2000000    # Si DCE
Router(config-if)# no shutdown
```

### ✅ Vérification de l'encapsulation

```bash
Router# show interfaces serial 0/0/0
```

> [!example] Sortie attendue
> 
> ```
> Serial0/0/0 is up, line protocol is up
>   Hardware is PowerQUICC Serial
>   Internet address is 192.168.1.1/30
>   MTU 1500 bytes, BW 1544 Kbit/sec, DLY 20000 usec,
>   Encapsulation HDLC, loopback not set
> ```

> [!tip] Astuce de dépannage Si le statut est "Serial0/0/0 is up, line protocol is down", vérifiez :
> 
> - Que les deux côtés utilisent la même encapsulation
> - Que le clock rate est configuré sur le DCE
> - Que les câbles sont correctement connectés

---

## 🔐 Encapsulation PPP

### Qu'est-ce que PPP ?

**PPP (Point-to-Point Protocol)** est un protocole d'encapsulation standard (non propriétaire) pour les liaisons série. Il offre plus de fonctionnalités que HDLC, notamment l'authentification et le support multi-protocoles.

> [!info] Caractéristiques de PPP
> 
> - **Standard ouvert** : Compatible avec tous les fabricants
> - **Authentification** : Support PAP et CHAP
> - **Multi-protocoles** : Supporte IP, IPv6, IPX, etc.
> - **Détection d'erreurs** : Mécanismes de contrôle qualité
> - **Compression** : Support de la compression des données

### 🔄 Comparaison HDLC vs PPP

|Critère|HDLC|PPP|
|---|---|---|
|Compatibilité|Cisco uniquement|Multi-fabricants|
|Authentification|Non|Oui (PAP/CHAP)|
|Overhead|Minimal|Léger|
|Configuration|Très simple|Plus complexe|
|Sécurité|Aucune|Bonne (avec CHAP)|
|Performances|Excellentes|Très bonnes|

### 📝 Configuration de base PPP (sans authentification)

```bash
Router(config)# interface serial 0/0/0
Router(config-if)# encapsulation ppp
```

### 🔐 Configuration PPP avec authentification PAP

**PAP (Password Authentication Protocol)** : Authentification en clair (non sécurisée)

**Sur le Routeur A (celui qui authentifie)** :

```bash
RouterA(config)# username RouterB password cisco123
RouterA(config)# interface serial 0/0/0
RouterA(config-if)# encapsulation ppp
RouterA(config-if)# ppp authentication pap
```

**Sur le Routeur B (celui qui s'authentifie)** :

```bash
RouterB(config)# username RouterA password cisco123
RouterB(config)# interface serial 0/0/0
RouterB(config-if)# encapsulation ppp
RouterB(config-if)# ppp pap sent-username RouterB password cisco123
RouterB(config-if)# ppp authentication pap
```

> [!warning] Sécurité PAP PAP transmet les mots de passe en clair. **N'utilisez PAP que dans des environnements de test.** Privilégiez CHAP en production.

### 🔒 Configuration PPP avec authentification CHAP

**CHAP (Challenge Handshake Authentication Protocol)** : Authentification sécurisée avec hash

**Configuration identique sur les deux routeurs** :

```bash
Router(config)# username Distant_Router password SecretPassword123
Router(config)# interface serial 0/0/0
Router(config-if)# encapsulation ppp
Router(config-if)# ppp authentication chap
```

> [!tip] Règles importantes pour CHAP
> 
> - Le **username** doit correspondre au **hostname** du routeur distant
> - Le **password** doit être **identique** sur les deux routeurs
> - CHAP est sensible à la casse

### 🎯 Configuration complète PPP avec CHAP

**Sur RouterA** :

```bash
RouterA(config)# hostname RouterA
RouterA(config)# username RouterB password MonMotDePasse2024
RouterA(config)# interface serial 0/0/0
RouterA(config-if)# description Liaison PPP-CHAP vers RouterB
RouterA(config-if)# ip address 172.16.1.1 255.255.255.252
RouterA(config-if)# encapsulation ppp
RouterA(config-if)# ppp authentication chap
RouterA(config-if)# clock rate 2000000
RouterA(config-if)# no shutdown
```

**Sur RouterB** :

```bash
RouterB(config)# hostname RouterB
RouterB(config)# username RouterA password MonMotDePasse2024
RouterB(config)# interface serial 0/0/0
RouterB(config-if)# description Liaison PPP-CHAP vers RouterA
RouterB(config-if)# ip address 172.16.1.2 255.255.255.252
RouterB(config-if)# encapsulation ppp
RouterB(config-if)# ppp authentication chap
RouterB(config-if)# no shutdown
```

### ✅ Vérification de PPP

```bash
Router# show interfaces serial 0/0/0
Router# show ppp all
Router# debug ppp authentication    # Pour dépannage en temps réel
```

> [!example] Sortie de show interfaces
> 
> ```
> Serial0/0/0 is up, line protocol is up
>   Hardware is PowerQUICC Serial
>   Internet address is 172.16.1.1/30
>   MTU 1500 bytes, BW 1544 Kbit/sec, DLY 20000 usec,
>   Encapsulation PPP, LCP Open
>   Open: IPCP, CDPCP
> ```

### 🐛 Dépannage PPP

> [!warning] Problèmes courants
> 
> **Line protocol is down** :
> 
> - Vérifier que l'encapsulation PPP est configurée des deux côtés
> - Vérifier le clock rate sur le DCE
> 
> **Authentification échoue** :
> 
> - Vérifier que les usernames correspondent aux hostnames distants
> - Vérifier que les passwords sont identiques (sensible à la casse)
> - Utiliser `debug ppp authentication` pour voir les détails
> 
> **LCP ne s'ouvre pas** :
> 
> - Vérifier la configuration physique (câblage)
> - Vérifier que l'interface n'est pas en shutdown

---

## 📊 Configuration de la Bande Passante

### Pourquoi configurer la bande passante ?

La commande `bandwidth` ne limite **pas** la vitesse réelle de l'interface, mais informe le routeur de la capacité théorique de la liaison. Cette information est utilisée par :

- **Les protocoles de routage** (OSPF, EIGRP) pour calculer les métriques
- **Les mécanismes de QoS** pour gérer la priorité du trafic
- **Les outils de monitoring** pour évaluer l'utilisation des liens

> [!info] Important à comprendre La bande passante configurée est une valeur **logique** utilisée pour les calculs, pas une limitation physique. Pour limiter le débit réel, il faut utiliser des mécanismes de QoS (Traffic Shaping/Policing).

### 📝 Syntaxe de configuration

```bash
Router(config)# interface serial 0/0/0
Router(config-if)# bandwidth KBPS
```

Le paramètre est exprimé en **kilobits par seconde (Kbps)**.

### 🎯 Exemples de configuration

```bash
# Liaison T1 (1.544 Mbps)
Router(config)# interface serial 0/0/0
Router(config-if)# bandwidth 1544

# Liaison E1 (2.048 Mbps)
Router(config)# interface serial 0/1/0
Router(config-if)# bandwidth 2048

# Liaison 512 Kbps
Router(config)# interface serial 0/2/0
Router(config-if)# bandwidth 512

# Liaison 64 Kbps
Router(config)# interface serial 0/3/0
Router(config-if)# bandwidth 64
```

### 📋 Valeurs courantes de bande passante

|Type de liaison|Vitesse|Valeur bandwidth|
|---|---|---|
|Ligne RNIS|64 Kbps|`bandwidth 64`|
|Ligne RNIS double canal|128 Kbps|`bandwidth 128`|
|T1|1.544 Mbps|`bandwidth 1544`|
|E1|2.048 Mbps|`bandwidth 2048`|
|T3|44.736 Mbps|`bandwidth 44736`|

### 🔄 Impact sur les protocoles de routage

#### OSPF

OSPF utilise la bande passante pour calculer le coût des routes :

**Formule** : Coût = 100 000 000 / Bandwidth (en bps)

> [!example] Exemple de calcul OSPF
> 
> - Interface avec `bandwidth 1544` (T1) : Coût = 100 000 000 / 1 544 000 = 64
> - Interface avec `bandwidth 64` : Coût = 100 000 000 / 64 000 = 1562

```bash
# Vérifier le coût OSPF d'une interface
Router# show ip ospf interface serial 0/0/0
```

#### EIGRP

EIGRP utilise la bande passante comme partie de sa métrique composite :

**Métrique** = [(K1 × Bandwidth) + (K2 × Bandwidth)/(256 - Load) + (K3 × Delay)] × 256

> [!tip] Bonne pratique Configurez toujours la bande passante pour refléter la réalité de vos liaisons WAN, surtout si vous utilisez OSPF ou EIGRP.

### 🎯 Configuration complète avec bande passante

```bash
Router(config)# interface serial 0/0/0
Router(config-if)# description Liaison T1 vers Agence_Paris
Router(config-if)# ip address 10.10.10.1 255.255.255.252
Router(config-if)# encapsulation ppp
Router(config-if)# ppp authentication chap
Router(config-if)# bandwidth 1544
Router(config-if)# clock rate 1544000    # Si DCE
Router(config-if)# no shutdown
```

### ✅ Vérification de la bande passante

```bash
Router# show interfaces serial 0/0/0
```

> [!example] Sortie attendue
> 
> ```
> Serial0/0/0 is up, line protocol is up
>   Hardware is PowerQUICC Serial
>   Description: Liaison T1 vers Agence_Paris
>   Internet address is 10.10.10.1/30
>   MTU 1500 bytes, BW 1544 Kbit/sec, DLY 20000 usec,
>   Encapsulation PPP, LCP Open
> ```

```bash
# Affichage condensé de toutes les interfaces
Router# show ip interface brief
```

### ⚠️ Pièges à éviter

> [!warning] Erreurs courantes
> 
> **Ne pas confondre avec clock rate** :
> 
> - `clock rate` : Définit la vitesse physique réelle (DCE uniquement)
> - `bandwidth` : Valeur logique pour les protocoles (DCE et DTE)
> 
> **Oublier de configurer la bande passante** :
> 
> - Par défaut, les interfaces série ont une bande passante de 1544 Kbps
> - Si votre liaison réelle est différente, configurez-la correctement
> 
> **Configurer une valeur incohérente** :
> 
> - La valeur bandwidth devrait correspondre au clock rate configuré
> - Exemple : `clock rate 128000` devrait avoir `bandwidth 128`

> [!tip] Astuce professionnelle Documentez toujours la bande passante réelle de vos liaisons WAN et configurez-la correctement. Cela évite des problèmes de routage difficiles à diagnostiquer, notamment avec OSPF qui pourrait choisir des chemins sous-optimaux.

---

## 🔧 Récapitulatif des commandes essentielles

### Commandes de configuration

```bash
# Identification DCE/DTE
show controllers serial X/Y/Z

# Configuration de base interface série
interface serial X/Y/Z
description [texte]
ip address [IP] [masque]
clock rate [vitesse]           # DCE uniquement
bandwidth [kbps]
no shutdown

# Encapsulation
encapsulation hdlc             # Par défaut
encapsulation ppp

# Authentification PPP
username [nom_distant] password [mot_de_passe]
ppp authentication pap
ppp authentication chap
ppp pap sent-username [nom] password [mdp]
```

### Commandes de vérification

```bash
# État des interfaces
show ip interface brief
show interfaces serial X/Y/Z
show controllers serial X/Y/Z

# Vérification PPP
show ppp all
show ppp interface serial X/Y/Z

# Vérification protocole de routage
show ip ospf interface
show ip eigrp interfaces

# Dépannage
debug ppp authentication
debug ppp negotiation
```

---

## 💡 Bonnes pratiques professionnelles

> [!tip] Recommandations
> 
> **Organisation** :
> 
> - Toujours mettre une description sur chaque interface
> - Utiliser une convention de nommage cohérente
> - Documenter les clock rates et bandwidths réels
> 
> **Sécurité** :
> 
> - Privilégier PPP avec CHAP sur HDLC pour les liaisons WAN
> - Utiliser des mots de passe forts et uniques
> - Changer régulièrement les mots de passe d'authentification
> 
> **Configuration** :
> 
> - Toujours configurer la bande passante pour refléter la réalité
> - Vérifier l'état des interfaces après chaque modification
> - Sauvegarder la configuration après validation
> 
> **Dépannage** :
> 
> - Commencer par vérifier la couche physique (câbles, clock rate)
> - Puis vérifier l'encapsulation (même protocole des deux côtés)
> - Enfin vérifier l'authentification si PPP est utilisé

> [!warning] Attention en production
> 
> - Ne jamais utiliser `debug` en production sans filtres appropriés
> - Toujours tester les changements en maintenance
> - Avoir un plan de rollback avant toute modification
> - Documenter toutes les modifications dans un journal

---

**📚 Fin du cours - Interfaces Série et WAN**