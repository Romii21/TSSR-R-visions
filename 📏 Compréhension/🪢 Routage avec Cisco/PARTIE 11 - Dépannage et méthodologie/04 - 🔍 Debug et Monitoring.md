

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

## Introduction

Le debug et le monitoring sont des outils essentiels pour le dépannage des équipements Cisco. Ils permettent d'observer en temps réel le comportement du routeur et d'identifier les problèmes. Cependant, ces outils peuvent avoir un impact significatif sur les performances et doivent être utilisés avec précaution, particulièrement en environnement de production.

> [!warning] Attention critique Les commandes de debug consomment énormément de ressources CPU. Une utilisation inappropriée peut rendre un routeur inopérant, provoquer une interruption de service, voire nécessiter un redémarrage physique de l'équipement.

---

## 1. Utilisation prudente du debug

### 🎯 Pourquoi la prudence est essentielle

Le debug sur un routeur Cisco génère des messages en temps réel pour chaque événement surveillé. Sur un équipement en production avec beaucoup de trafic, cela peut signifier des milliers de messages par seconde, saturant le CPU et impactant le forwarding des paquets.

### 📋 Principes de base

**Avant d'activer un debug :**

- Vérifier la charge CPU actuelle (`show processes cpu`)
- Privilégier les heures creuses en production
- Avoir un plan de rollback (savoir comment désactiver rapidement)
- Limiter la portée du debug (utiliser des filtres conditionnels)
- Préférer le logging au debug quand c'est possible

### 💻 Commandes préalables

```bash
# Vérifier la charge CPU avant debug
Router# show processes cpu
CPU utilization for five seconds: 15%/8%; one minute: 12%; five minutes: 10%

# Vérifier la mémoire disponible
Router# show memory
Processor    123456K total, 98765K used, 24691K free

# Activer le timestamp pour corréler les événements
Router# configure terminal
Router(config)# service timestamps debug datetime msec localtime show-timezone
Router(config)# service timestamps log datetime msec localtime show-timezone
```

> [!tip] Astuce professionnelle Sur un équipement de production, documentez toujours votre session de debug dans un ticket ou un journal : heure de début, commandes utilisées, observations, heure de fin. Cela facilite l'audit et le post-mortem en cas d'incident.

### ⚠️ Pièges courants

|Piège|Conséquence|Solution|
|---|---|---|
|Debug sans filtre sur lien haute capacité|CPU à 100%, perte de connectivité|Toujours utiliser debug conditionnel|
|Multiples debugs actifs simultanément|Saturation totale|Activer un debug à la fois|
|Oublier de désactiver le debug|Consommation continue de ressources|Utiliser `undebug all` systématiquement|
|Debug sur console sans `logging synchronous`|Messages illisibles entrecoupés|Configurer `logging synchronous` sur la ligne|

### 🛡️ Bonnes pratiques de sécurité

```bash
# Limiter le temps d'exposition - utiliser un timer
Router# debug ip packet 
Router# ! Effectuer vos tests rapidement
Router# undebug all

# Surveiller le CPU pendant le debug
Router# show processes cpu | include CPU

# En cas d'urgence, désactiver immédiatement
Router# undebug all
Router# no debug all
```

> [!example] Exemple de scenario à risque Un ingénieur active `debug ip packet` sur un routeur backbone sans filtre. Le routeur traite 100 000 paquets/seconde. Le debug génère 100 000 messages/seconde, le CPU monte à 99%, le BGP flap, et tous les voisins se déconnectent. **Toujours filtrer vos debugs !**

---

## 2. Debug conditionnel

### 🎯 Principe et utilité

Le debug conditionnel permet de limiter l'affichage des messages debug à des critères spécifiques (adresse IP, interface, utilisateur, etc.). C'est l'outil indispensable pour utiliser le debug en production de manière ciblée et sécurisée.

### 📋 Types de conditions disponibles

```bash
# Debug par adresse IP source/destination
Router# debug ip packet <ACL-number>

# Debug par interface
Router# debug condition interface <interface-name>

# Debug par adresse MAC
Router# debug condition mac-address <mac-address>

# Debug par appelant (VoIP, PPP)
Router# debug condition called <phone-number>

# Debug par nom d'utilisateur
Router# debug condition username <username>
```

### 💻 Configuration du debug conditionnel

#### Méthode 1 : Avec une ACL

```bash
# Créer une ACL pour filtrer le trafic à débugger
Router(config)# access-list 100 permit ip host 192.168.1.10 host 10.0.0.5

# Activer le debug avec l'ACL
Router# debug ip packet 100

# Vérifier les conditions actives
Router# show debug condition
```

#### Méthode 2 : Avec une condition d'interface

```bash
# Limiter le debug à une interface spécifique
Router# debug condition interface GigabitEthernet0/1

# Activer le debug concerné
Router# debug ip routing

# Vérifier les conditions
Router# show debug condition
Condition 1: interface Gi0/1 (1 flags triggered)
        Flags: Gi0/1
```

#### Méthode 3 : Avec une condition d'adresse IP

```bash
# Debug pour une adresse IP spécifique
Router# debug condition ip 192.168.1.100

# Activer plusieurs types de debug pour cette IP
Router# debug ip packet
Router# debug ip icmp
Router# debug ip routing

# Lister les conditions actives
Router# show debug condition
```

### 🔄 Combinaison de conditions

```bash
# Combiner plusieurs conditions (ET logique)
Router# debug condition interface GigabitEthernet0/1
Router# debug condition ip 10.1.1.5
Router# debug ip packet

# Seul le trafic de 10.1.1.5 sur Gi0/1 sera débugé
```

### ❌ Suppression de conditions

```bash
# Supprimer une condition spécifique
Router# undebug condition 1

# Supprimer toutes les conditions
Router# undebug condition all

# Ou
Router# no debug condition all
```

> [!tip] Astuce avancée Utilisez `debug condition standby` pour débugger HSRP/VRRP sans impacter le CPU avec tout le trafic de données. Ciblez toujours vos debugs au maximum !

### 📊 Vérification et monitoring

```bash
# Voir toutes les conditions actives
Router# show debug condition
Condition 1: interface Gi0/1 (1 flags triggered)
Condition 2: ip 192.168.1.10 (1 flags triggered)

# Voir les debugs actifs
Router# show debug
IP packet debugging is on (detailed) for access list 100
IP routing debugging is on
```

> [!warning] Attention Même avec des conditions, surveillez toujours la charge CPU. Une condition mal configurée (ACL trop permissive) peut quand même générer un volume important de messages.

### 🛠️ Cas d'usage pratiques

**Scénario 1 : Problème de connectivité entre deux hôtes**

```bash
Router(config)# access-list 110 permit ip host 192.168.1.10 host 10.0.0.20
Router# debug ip packet 110 detail
```

**Scénario 2 : Problème sur une interface spécifique**

```bash
Router# debug condition interface GigabitEthernet0/2
Router# debug ip packet
Router# debug arp
```

**Scénario 3 : Problème d'authentification PPP**

```bash
Router# debug condition username client123
Router# debug ppp authentication
Router# debug ppp negotiation
```

---

## 3. Logging vers buffer/console/syslog

### 🎯 Destinations des logs

Cisco IOS permet d'envoyer les messages de log vers plusieurs destinations simultanément. Chaque destination a ses avantages et cas d'usage spécifiques.

|Destination|Avantage|Inconvénient|Usage|
|---|---|---|---|
|**Console**|Immédiat, pas de config réseau|Nécessite accès physique/console, messages perdus à la déconnexion|Dépannage initial, premier boot|
|**Buffer**|Stocké localement, consultation rapide|Taille limitée, perdu au reboot|Debug et dépannage temporaire|
|**Syslog**|Centralisé, historique long terme, corrélation|Dépend du réseau, peut échouer|Production, audit, supervision|
|**Terminal (vty)**|Pratique pour SSH/Telnet|Uniquement pendant la session active|Dépannage à distance|

### 💻 Configuration du logging vers buffer

Le buffer est une zone de mémoire RAM qui stocke les derniers messages de log. C'est l'outil privilégié pour le dépannage car il ne dépend pas du réseau.

```bash
# Activer le logging vers le buffer
Router(config)# logging buffered

# Définir la taille du buffer (en octets)
Router(config)# logging buffered 64000

# Définir le niveau de sévérité
Router(config)# logging buffered 6  # Informational

# Configuration complète recommandée
Router(config)# logging buffered 64000 informational
```

**Consultation du buffer :**

```bash
# Voir tous les logs du buffer
Router# show logging

# Voir uniquement les dernières lignes
Router# show logging | include ERROR

# Effacer le buffer
Router# clear logging
```

> [!tip] Dimensionnement du buffer Pour un routeur de production, utilisez 64000 à 128000 octets. Pour un switch d'accès peu critique, 16000 octets suffisent. Un buffer trop petit écrase rapidement les anciens messages.

### 💻 Configuration du logging vers console

La console affiche les messages en temps réel. Attention, cela peut rendre la ligne de commande illisible si trop de messages s'affichent.

```bash
# Activer le logging vers la console
Router(config)# logging console

# Définir le niveau de sévérité
Router(config)# logging console warnings  # Niveau 4

# Recommandation : activer le synchronous pour éviter l'interruption
Router(config)# line console 0
Router(config-line)# logging synchronous
Router(config-line)# exec-timeout 30 0
```

> [!warning] Piège console Sans `logging synchronous`, les messages de log interrompent votre saisie de commandes, rendant la console difficile à utiliser. Activez-le systématiquement !

**Désactivation temporaire :**

```bash
# Désactiver l'affichage console sans supprimer la config
Router(config)# no logging console
```

### 💻 Configuration du logging vers terminal (vty)

Le logging vers terminal permet de voir les logs pendant une session SSH/Telnet active.

```bash
# Activer le logging vers le terminal (depuis le mode privilégié)
Router# terminal monitor

# Configuration permanente sur les lignes vty
Router(config)# line vty 0 4
Router(config-line)# logging synchronous
Router(config-line)# exec-timeout 15 0

# Définir le niveau pour les terminaux
Router(config)# logging monitor 6  # Informational
```

**Désactivation :**

```bash
# Désactiver pour la session en cours
Router# terminal no monitor

# Ou simplement
Router# undebug all
```

### 💻 Configuration du logging vers syslog

Le serveur syslog centralise les logs de tous les équipements réseau. C'est la solution professionnelle pour la production.

```bash
# Définir le serveur syslog
Router(config)# logging host 10.1.1.100

# Ou avec un nom (nécessite DNS fonctionnel)
Router(config)# logging host syslog.entreprise.local

# Définir le niveau de sévérité pour syslog
Router(config)# logging trap 5  # Notifications

# Définir l'interface source (pour le routage)
Router(config)# logging source-interface Loopback0

# Activer les timestamps
Router(config)# service timestamps log datetime msec localtime show-timezone
```

**Configuration complète avec plusieurs serveurs :**

```bash
Router(config)# logging host 10.1.1.100
Router(config)# logging host 10.1.1.101
Router(config)# logging trap informational
Router(config)# logging source-interface Loopback0
Router(config)# logging facility local5
```

> [!info] Facility syslog La "facility" est un code qui permet au serveur syslog de catégoriser les messages. `local0` à `local7` sont disponibles pour usage personnalisé. Par convention, on utilise souvent `local5` ou `local6` pour les équipements réseau.

### 📊 Vérification de la configuration

```bash
# Voir toute la configuration de logging
Router# show logging

Syslog logging: enabled (0 messages dropped, 3 messages rate-limited,
                0 flushes, 0 overruns, xml disabled, filtering disabled)

No Active Message Discriminator.

No Inactive Message Discriminator.

    Console logging: level warnings, 127 messages logged, xml disabled,
                     filtering disabled
    Monitor logging: level debugging, 0 messages logged, xml disabled,
                     filtering disabled
    Buffer logging:  level informational, 45 messages logged, xml disabled,
                    filtering disabled
    Exception Logging: size (4096 bytes)
    Count and timestamp logging messages: disabled
    Persistent logging: disabled

No active filter modules.

    Trap logging: level informational, 52 message lines logged
        Logging to 10.1.1.100  (udp port 514, audit disabled,
              link up),
              45 message lines logged,
              0 message lines rate-limited,
              0 message lines dropped-by-MD,
              xml disabled, sequence number disabled
              filtering disabled
        Logging Source-Interface:       VRF Name:
        Loopback0                       
```

### 🛠️ Cas d'usage pratiques

**Scénario 1 : Dépannage rapide sur site**

```bash
# Console + buffer pour voir tout en temps réel et garder l'historique
Router(config)# logging console informational
Router(config)# logging buffered 64000 debugging
```

**Scénario 2 : Production avec supervision centralisée**

```bash
# Syslog avec buffer de secours
Router(config)# logging host 10.1.1.100
Router(config)# logging trap notifications
Router(config)# logging buffered 128000 warnings
Router(config)# logging console errors
```

**Scénario 3 : Debug distant via SSH**

```bash
Router# terminal monitor
Router(config)# logging monitor debugging
Router# debug ip packet 100
```

> [!tip] Astuce de pro En production, configurez toujours au minimum : buffer (warnings) + syslog (informational). Cela permet d'avoir les événements importants localement et tout l'historique centralisé.

---

## 4. Niveaux de logging (0-7)

### 🎯 Hiérarchie des niveaux

Cisco IOS utilise 8 niveaux de sévérité pour classifier les messages, du plus critique (0) au plus détaillé (7). Lorsqu'un niveau est configuré, tous les messages de ce niveau **et des niveaux supérieurs** (plus critiques) sont inclus.

### 📊 Tableau des niveaux de sévérité

|Niveau|Nom|Mot-clé|Description|Exemple de message|Quand l'utiliser|
|---|---|---|---|---|---|
|**0**|Emergency|`emergencies`|Système inutilisable|System crashed|Jamais filtré - critique absolu|
|**1**|Alert|`alerts`|Action immédiate requise|Temperature critical|Monitoring d'alertes matérielles|
|**2**|Critical|`critical`|Conditions critiques|Memory allocation failure|Problèmes système graves|
|**3**|Error|`errors`|Conditions d'erreur|Interface down|Production - erreurs seulement|
|**4**|Warning|`warnings`|Conditions d'avertissement|Duplex mismatch|Production - recommandé|
|**5**|Notice|`notifications`|Normal mais significatif|BGP neighbor up|Production détaillée|
|**6**|Informational|`informational`|Messages informatifs|Interface up|Debug et dépannage|
|**7**|Debug|`debugging`|Messages de debug|Packet received|Debug uniquement - très verbeux|

> [!info] Principe cumulatif Si vous configurez `logging buffered warnings` (niveau 4), vous recevrez aussi les niveaux 0, 1, 2, et 3. C'est un filtre "jusqu'à ce niveau", pas "uniquement ce niveau".

### 💻 Configuration par destination

```bash
# Console - erreurs uniquement (production)
Router(config)# logging console errors

# Buffer - warnings et supérieur (recommandé)
Router(config)# logging buffered 64000 warnings

# Syslog - informational (supervision complète)
Router(config)# logging trap informational

# Terminal - debugging (dépannage actif)
Router(config)# logging monitor debugging
```

### 🎨 Configuration par numéro ou par nom

```bash
# Les deux syntaxes sont équivalentes
Router(config)# logging buffered 4        # Niveau 4 = warnings
Router(config)# logging buffered warnings # Même résultat

Router(config)# logging trap 6            # Niveau 6 = informational
Router(config)# logging trap informational # Même résultat
```

> [!tip] Bonne pratique Utilisez les noms plutôt que les numéros pour la lisibilité. "warnings" est plus explicite que "4" dans une configuration.

### 📋 Recommandations par environnement

#### 🏢 Production stable

```bash
Router(config)# logging console errors         # 3 - Erreurs uniquement
Router(config)# logging buffered 64000 warnings # 4 - Warnings et erreurs
Router(config)# logging trap notifications      # 5 - Événements notables
```

#### 🔧 Pré-production / Tests

```bash
Router(config)# logging console warnings        # 4
Router(config)# logging buffered 128000 informational # 6
Router(config)# logging trap informational      # 6
```

#### 🐛 Debug / Dépannage actif

```bash
Router(config)# logging console informational   # 6
Router(config)# logging buffered 128000 debugging # 7 - Tout
Router(config)# logging monitor debugging       # 7
Router# terminal monitor                        # Activer sur la session
```

### 📊 Exemples de messages par niveau

**Niveau 0 - Emergency :**

```
%SYS-0-CPUHOG: Task is hogging the CPU
```

**Niveau 1 - Alert :**

```
%SNMP-1-INTERNAL: Internal error
```

**Niveau 2 - Critical :**

```
%C4K_CHASSIS-2-FANTRAY: Fan tray removed
```

**Niveau 3 - Error :**

```
%LINEPROTO-3-UPDOWN: Line protocol on Interface GigabitEthernet0/1, changed state to down
```

**Niveau 4 - Warning :**

```
%ETHCNTR-4-DUPLEXMISMATCH: Duplex mismatch discovered on Gi0/1
```

**Niveau 5 - Notice (Notification) :**

```
%LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet0/1, changed state to up
```

**Niveau 6 - Informational :**

```
%SYS-6-LOGGINGHOST_STARTSTOP: Logging to host 10.1.1.100 port 514 started
```

**Niveau 7 - Debug :**

```
IP: s=192.168.1.10 (GigabitEthernet0/1), d=10.0.0.1, len 100, rcvd 3
```

### 🔍 Analyse et filtrage

```bash
# Voir les messages d'un niveau spécifique
Router# show logging | include %LINEPROTO-3

# Voir les messages critiques uniquement
Router# show logging | include -2-|-1-|-0-

# Compter les occurrences d'un type d'erreur
Router# show logging | count %ETHCNTR-4-DUPLEXMISMATCH
```

### ⚙️ Configuration granulaire avancée

```bash
# Limiter le rate des messages (éviter le flood)
Router(config)# logging rate-limit all 10 except errors

# Désactiver un type de message spécifique
Router(config)# no logging event link-status

# Réactiver
Router(config)# logging event link-status
```

> [!warning] Attention au niveau 7 (debugging) Ne jamais laisser le logging en mode "debugging" en production. Un seul `debug ip packet` peut générer des millions de messages et saturer le buffer, le syslog et le CPU. Utilisez debugging uniquement pendant des sessions de dépannage actives.

### 🎯 Stratégie optimale multi-niveaux

```bash
# Configuration équilibrée pour production
Router(config)# logging buffered 64000 warnings       # Local : erreurs et warnings
Router(config)# logging trap notifications             # Syslog : événements notables
Router(config)# logging console critical               # Console : urgences seulement
Router(config)# logging monitor informational          # SSH : informationnel si activé
```

Cette stratégie permet de :

- Garder localement (buffer) les problèmes nécessitant une action (warnings+)
- Envoyer au syslog les changements d'état importants (notifications+)
- Éviter de saturer la console (critical uniquement)
- Avoir du détail lors des sessions de dépannage SSH (informational)

---

## 5. Désactivation du debug

### 🎯 Importance de la désactivation

La désactivation rapide et complète du debug est critique pour restaurer les performances normales d'un routeur. Un debug oublié peut continuer à consommer des ressources pendant des jours, causant une dégradation progressive des performances.

> [!warning] Règle d'or Toujours désactiver le debug dès la fin des tests. Un debug actif oublié est l'une des causes les plus fréquentes de dégradation de performance inexpliquée en production.

### 💻 Commandes de désactivation

#### Méthode 1 : undebug all (recommandée)

```bash
# Désactiver tous les debugs actifs
Router# undebug all

# Vérifier que tout est désactivé
Router# show debug
```

Cette commande est la plus sûre car elle désactive **tous** les debugs, quelle que soit leur nature.

#### Méthode 2 : no debug all

```bash
# Alternative équivalente
Router# no debug all

# Vérification
Router# show debug
```

#### Méthode 3 : Désactivation sélective

```bash
# Désactiver un debug spécifique
Router# no debug ip packet
Router# no debug ip routing

# Désactiver une condition spécifique
Router# undebug condition 1

# Désactiver toutes les conditions
Router# undebug condition all
```

> [!tip] Raccourci clavier En mode privilégié, vous pouvez utiliser `u all` comme raccourci de `undebug all`. Pratique en situation d'urgence !

### 📊 Vérification complète

```bash
# Vérifier qu'aucun debug n'est actif
Router# show debug
No debug flags are set

# Vérifier qu'aucune condition n'est active
Router# show debug condition
No debug conditions are set

# Vérifier le retour à la normale du CPU
Router# show processes cpu
CPU utilization for five seconds: 8%/3%; one minute: 7%; five minutes: 6%
```

### 🛡️ Procédure de sécurité en production

**Avant le debug :**

```bash
# 1. Noter la charge CPU de base
Router# show processes cpu | include CPU
CPU utilization for five seconds: 12%/5%

# 2. Préparer la commande de désactivation
Router# ! Taper "undebug all" et laisser dans le buffer

# 3. Définir un timer mental (ex: 2 minutes maximum)
```

**Pendant le debug :**

```bash
# Surveiller le CPU en continu (autre session SSH recommandée)
Router# show processes cpu | include CPU

# Si CPU > 80%, désactiver immédiatement
Router# undebug all
```

**Après le debug :**

```bash
# 1. Désactiver immédiatement
Router# undebug all

# 2. Vérifier la désactivation
Router# show debug

# 3. Vérifier le retour du CPU à la normale
Router# show processes cpu

# 4. Nettoyer les conditions
Router# undebug condition all
```

### ⚠️ Situations d'urgence

**Scénario 1 : CPU à 100%, console bloquée**

```bash
# Si la console ne répond plus, tenter plusieurs fois
Router# undebug all
Router# undebug all
Router# no debug all

# Si toujours bloqué, utiliser le bouton Break pour entrer en ROMMON
# Puis reload l'équipement (dernier recours)
```

**Scénario 2 : Debug réseau actif, perte de connexion SSH**

```bash
# Avoir toujours une seconde session SSH ouverte en parallèle
# Ou configurer un timer automatique via EEM (Embedded Event Manager)

Router(config)# event manager applet DISABLE_DEBUG
Router(config-applet)# event timer countdown time 300
Router(config-applet)# action 1.0 cli command "enable"
Router(config-applet)# action 2.0 cli command "undebug all"
```

> [!example] EEM - Timer de sécurité L'exemple ci-dessus configure un script EEM qui exécutera automatiquement `undebug all` après 5 minutes (300 secondes). C'est un filet de sécurité si vous perdez la connexion pendant un debug.

### 🔄 Workflow complet recommandé

```bash
# === PHASE 1 : PRÉPARATION ===
Router# show processes cpu                    # Noter le CPU de base
Router# show debug                             # Vérifier qu'aucun debug actif

# === PHASE 2 : CONFIGURATION ===
Router# debug condition interface Gi0/1       # Appliquer conditions
Router(config)# access-list 100 permit ip host 192.168.1.10 any

# === PHASE 3 : ACTIVATION ===
Router# debug ip packet 100 detail            # Activer le debug ciblé
Router# terminal monitor                       # Si session SSH

# === PHASE 4 : OBSERVATION ===
# Effectuer les tests (maximum 2-3 minutes)
# Capturer les outputs nécessaires

# === PHASE 5 : DÉSACTIVATION IMMÉDIATE ===
Router# undebug all                            # Tout désactiver
Router# undebug condition all                  # Nettoyer les conditions
Router# terminal no monitor                    # Désactiver terminal

# === PHASE 6 : VÉRIFICATION ===
Router# show debug                             # Confirmer désactivation
Router# show processes cpu                     # Vérifier retour CPU normal
```

### 📋 Checklist post-debug

- [ ] `undebug all` exécuté
- [ ] `show debug` retourne "No debug flags are set"
- [ ] `undebug condition all` exécuté
- [ ] CPU retourné au niveau de base (< 20% idéalement)
- [ ] Captures/logs sauvegardés si nécessaire
- [ ] Documentation de l'incident mise à jour

> [!tip] Automatisation Pour les équipements critiques, envisagez de créer des scripts EEM qui désactivent automatiquement les debugs après un délai défini. C'est une protection contre les oublis humains.

### 🚨 Messages de confirmation

Après `undebug all`, vous devriez voir :

```
All possible debugging has been turned off
```

Vérification :

```bash
Router# show debug
No debug flags are set
```

Si vous voyez encore des flags, relancez `undebug all` ou redémarrez l'équipement en dernier recours.

---

> [!info] 📌 Points clés à retenir
> 
> - **Debug = Impact CPU** : Toujours utiliser avec prudence et conditions
> - **Logging multi-destinations** : Buffer + Syslog = couverture optimale
> - **Niveaux appropriés** : Production = warnings/notifications, Debug = debugging
> - **Désactivation systématique** : `undebug all` dès la fin des tests
> - **Surveillance continue** : Vérifier le CPU avant, pendant et après le debug