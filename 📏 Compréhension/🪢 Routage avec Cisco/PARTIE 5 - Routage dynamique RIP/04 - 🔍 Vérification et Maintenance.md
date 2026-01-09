

## 📚 Table des matières

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

La vérification et la maintenance du protocole RIP sont essentielles pour s'assurer que le routage fonctionne correctement et pour diagnostiquer les problèmes éventuels. Cisco IOS propose plusieurs commandes permettant d'observer le comportement de RIP en temps réel et d'analyser sa configuration.

> [!info] Pourquoi vérifier RIP ?
> 
> - Confirmer que les réseaux sont bien annoncés
> - Identifier les problèmes de convergence
> - Détecter les boucles de routage
> - Optimiser les performances du réseau
> - Troubleshooter les problèmes de connectivité

---

## Show IP Protocols

### 📖 Présentation

La commande `show ip protocols` affiche des informations globales sur tous les protocoles de routage actifs sur le routeur. Pour RIP, elle fournit une vue d'ensemble de sa configuration et de son état opérationnel.

### 🎯 Quand l'utiliser ?

- Vérification initiale après configuration de RIP
- Diagnostic rapide des paramètres RIP actifs
- Identification des réseaux annoncés
- Vérification des timers et de la version RIP
- Contrôle des sources de mises à jour

### 💻 Syntaxe

```bash
Router# show ip protocols
```

> [!example] Exemple de sortie
> 
> ```bash
> Router# show ip protocols
> 
> Routing Protocol is "rip"
>   Outgoing update filter list for all interfaces is not set
>   Incoming update filter list for all interfaces is not set
>   Sending updates every 30 seconds, next due in 12 seconds
>   Invalid after 180 seconds, hold down 180, flushed after 240
>   Redistributing: rip
>   Default version control: send version 2, receive version 2
>     Interface             Send  Recv  Triggered RIP  Key-chain
>     GigabitEthernet0/0    2     2                                  
>     GigabitEthernet0/1    2     2                                  
>   Automatic network summarization is not in effect
>   Maximum path: 4
>   Routing for Networks:
>     10.0.0.0
>     192.168.1.0
>   Routing Information Sources:
>     Gateway         Distance      Last Update
>     10.0.0.2             120      00:00:15
>     192.168.1.2          120      00:00:23
>   Distance: (default is 120)
> ```

### 🔎 Interprétation des informations

|Élément|Description|Importance|
|---|---|---|
|**Routing Protocol**|Protocole actif (rip)|Confirme que RIP est en cours d'exécution|
|**Sending updates every**|Intervalle d'envoi (30s par défaut)|Indique la fréquence des updates|
|**Invalid/Hold down/Flushed**|Timers (180/180/240s)|Temps avant d'invalider/supprimer une route|
|**Default version control**|Version RIP utilisée|RIPv1 ou RIPv2|
|**Automatic summarization**|État du résumé automatique|Important pour RIPv2|
|**Routing for Networks**|Réseaux annoncés par RIP|Liste des réseaux configurés|
|**Routing Information Sources**|Voisins RIP actifs|Adresses des routeurs qui envoient des updates|
|**Distance**|Distance administrative|120 par défaut pour RIP|

> [!tip] Astuces
> 
> - Cette commande ne nécessite pas le mode privilégié
> - Vérifiez que tous vos réseaux attendus apparaissent dans "Routing for Networks"
> - Si aucun voisin n'apparaît dans "Routing Information Sources", vérifiez la connectivité réseau

> [!warning] Pièges courants
> 
> - Si "Automatic summarization is in effect" apparaît avec RIPv2, cela peut causer des problèmes avec des réseaux discontigus
> - L'absence de voisins peut indiquer un problème de configuration ou de connectivité
> - Des timers modifiés peuvent indiquer une configuration non standard

---

## Show IP RIP Database

### 📖 Présentation

La commande `show ip rip database` affiche le contenu de la base de données RIP, c'est-à-dire toutes les routes apprises par RIP avant qu'elles ne soient insérées dans la table de routage. C'est une vue brute des informations RIP.

### 🎯 Quand l'utiliser ?

- Vérifier les routes reçues par RIP
- Identifier les routes en attente d'insertion
- Diagnostiquer les problèmes de métrique
- Comparer avec la table de routage principale
- Détecter les routes en hold-down

### 💻 Syntaxe

```bash
Router# show ip rip database
```

> [!example] Exemple de sortie
> 
> ```bash
> Router# show ip rip database
> 
> 10.0.0.0/8    auto-summary
> 10.1.1.0/24    directly connected, GigabitEthernet0/0
> 10.2.2.0/24
>     [1] via 10.1.1.2, 00:00:15, GigabitEthernet0/0
> 192.168.1.0/24    auto-summary
> 192.168.1.0/24    directly connected, GigabitEthernet0/1
> 192.168.2.0/24
>     [2] via 192.168.1.2, 00:00:08, GigabitEthernet0/1
> 172.16.0.0/16    auto-summary
> 172.16.1.0/24
>     [1] via 10.1.1.2, 00:00:15, GigabitEthernet0/0
> 172.16.2.0/24
>     [3] via 10.1.1.2, 00:00:15, GigabitEthernet0/0
> ```

### 🔎 Interprétation des informations

|Élément|Description|Signification|
|---|---|---|
|**directly connected**|Réseau directement connecté|Route locale annoncée par RIP|
|**auto-summary**|Résumé automatique|Entrée de résumé (si activé)|
|**[métrique]**|Nombre de sauts|Distance en hops vers la destination|
|**via X.X.X.X**|Adresse du next-hop|Routeur suivant pour atteindre le réseau|
|**Timestamp**|Temps écoulé|Depuis la dernière mise à jour|
|**Interface**|Interface de sortie|Par où envoyer les paquets|

> [!info] Différence avec la table de routage La base RIP contient **toutes** les routes RIP, même celles qui ne sont pas installées dans la table de routage (par exemple, si une meilleure route existe via un autre protocole avec une meilleure distance administrative).

### 📊 États des routes

Les routes dans la base RIP peuvent avoir différents états :

- **Active** : Route valide et utilisable
- **Hold-down** : Route en période d'attente après avoir été marquée comme inaccessible
- **Possibly down** : Route suspectée d'être inaccessible

> [!tip] Astuces
> 
> - Comparez cette sortie avec `show ip route` pour identifier les routes RIP non installées
> - Une métrique de 16 indique une route inaccessible (métrique infinie dans RIP)
> - Utilisez cette commande pour vérifier que les routes attendues sont bien reçues

> [!warning] Pièges courants
> 
> - Une route présente dans la base RIP mais absente de la table de routage peut indiquer une meilleure route via un autre protocole
> - Des métriques élevées (proches de 16) peuvent indiquer des problèmes de topologie
> - L'absence d'une route attendue peut signifier qu'elle n'est pas annoncée par le voisin

---

## Debug IP RIP

### 📖 Présentation

La commande `debug ip rip` active l'affichage en temps réel des activités RIP : envoi et réception de mises à jour, calcul des métriques, et autres événements du protocole. C'est l'outil de diagnostic le plus détaillé pour RIP.

### 🎯 Quand l'utiliser ?

- Diagnostic approfondi de problèmes RIP
- Observation des mises à jour en temps réel
- Vérification des routes annoncées/reçues
- Détection de problèmes de convergence
- Analyse du comportement des timers

### 💻 Syntaxe

```bash
# Activer le debug RIP
Router# debug ip rip

# Désactiver le debug RIP
Router# no debug ip rip

# Désactiver tous les debugs actifs
Router# undebug all
```

> [!warning] ⚠️ ATTENTION - Impact sur les performances La commande `debug` consomme des ressources CPU importantes et génère beaucoup de messages. À utiliser avec précaution sur des routeurs en production.

### 🔧 Variantes de la commande

```bash
# Debug des événements RIP (envoi/réception)
Router# debug ip rip events

# Debug des bases de données RIP
Router# debug ip rip database

# Debug des transactions RIP (plus détaillé)
Router# debug ip rip transactions
```

> [!example] Exemple de sortie
> 
> ```bash
> Router# debug ip rip
> 
> RIP: received v2 update from 10.1.1.2 on GigabitEthernet0/0
>       172.16.1.0/24 via 0.0.0.0 in 1 hops
>       172.16.2.0/24 via 0.0.0.0 in 2 hops
>       192.168.2.0/24 via 0.0.0.0 in 1 hops
> 
> RIP: sending v2 update to 224.0.0.9 via GigabitEthernet0/0 (10.1.1.1)
>       192.168.1.0/24 via 0.0.0.0, metric 1, tag 0
>       192.168.3.0/24 via 0.0.0.0, metric 2, tag 0
> 
> RIP: received v2 update from 192.168.1.2 on GigabitEthernet0/1
>       192.168.2.0/24 via 0.0.0.0 in 1 hops
>       172.16.3.0/24 via 0.0.0.0 in 3 hops
> ```

### 🔎 Interprétation des messages

#### Messages de réception

```bash
RIP: received v2 update from 10.1.1.2 on GigabitEthernet0/0
      172.16.1.0/24 via 0.0.0.0 in 1 hops
```

|Élément|Description|
|---|---|
|**received v2 update**|Type de mise à jour reçue (v1 ou v2)|
|**from 10.1.1.2**|Adresse source du voisin RIP|
|**on GigabitEthernet0/0**|Interface de réception|
|**172.16.1.0/24**|Réseau annoncé|
|**via 0.0.0.0**|Next-hop (0.0.0.0 = utiliser l'adresse source)|
|**in 1 hops**|Métrique reçue|

#### Messages d'envoi

```bash
RIP: sending v2 update to 224.0.0.9 via GigabitEthernet0/0 (10.1.1.1)
      192.168.1.0/24 via 0.0.0.0, metric 1, tag 0
```

|Élément|Description|
|---|---|
|**sending v2 update**|Type de mise à jour envoyée|
|**to 224.0.0.9**|Adresse multicast RIPv2|
|**via GigabitEthernet0/0**|Interface d'envoi|
|**(10.1.1.1)**|Adresse source de l'interface|
|**192.168.1.0/24**|Réseau annoncé|
|**metric 1**|Métrique envoyée|
|**tag 0**|Tag de route (pour filtrage avancé)|

### 🛠️ Bonnes pratiques

> [!tip] Conseils d'utilisation
> 
> - **Toujours désactiver le debug après usage** avec `undebug all`
> - Utilisez `terminal monitor` si vous êtes connecté en Telnet/SSH pour voir les messages
> - Limitez la durée du debug (quelques minutes maximum)
> - Sur un routeur de production, préférez `show` aux commandes `debug`
> - Redirigez les logs vers un buffer pour analyse ultérieure

```bash
# Configurer un buffer de logs
Router(config)# logging buffered 50000
Router(config)# exit

# Activer le debug
Router# debug ip rip

# Attendre quelques secondes/minutes

# Désactiver le debug
Router# undebug all

# Consulter les logs capturés
Router# show logging
```

> [!warning] Pièges courants
> 
> - **Ne jamais laisser un debug actif** : risque de saturation CPU
> - Les messages de debug peuvent submerger la console : utilisez `no logging console` temporairement
> - Sur des réseaux à fort trafic, le debug peut générer des milliers de lignes par minute
> - Le debug ne montre que les événements futurs, pas l'état actuel

### 🔍 Scénarios de diagnostic

#### Scénario 1 : Aucune mise à jour reçue

```bash
Router# debug ip rip
# Aucun message "received" n'apparaît
```

**Causes possibles** :

- Problème de connectivité physique
- RIP non configuré sur le voisin
- ACL bloquant les mises à jour RIP (UDP 520)
- Versions RIP incompatibles

#### Scénario 2 : Routes reçues mais non installées

```bash
RIP: received v2 update from 10.1.1.2 on GigabitEthernet0/0
      172.16.1.0/24 via 0.0.0.0 in 1 hops
# Mais la route n'apparaît pas dans "show ip route"
```

**Causes possibles** :

- Route avec meilleure distance administrative via un autre protocole
- Route filtrée par une distribute-list
- Problème de split horizon

#### Scénario 3 : Métrique qui augmente continuellement

```bash
# Première update
RIP: received update from 10.1.1.2: 172.16.1.0/24 in 2 hops
# 30 secondes plus tard
RIP: received update from 10.1.1.2: 172.16.1.0/24 in 3 hops
# 30 secondes plus tard
RIP: received update from 10.1.1.2: 172.16.1.0/24 in 4 hops
```

**Cause probable** : Boucle de routage (counting to infinity)

---

## Analyse des mises à jour RIP

### 📖 Présentation

L'analyse des mises à jour RIP consiste à comprendre le contenu, la fréquence et le comportement des messages RIP échangés entre routeurs. Cette analyse permet d'optimiser le réseau et de détecter les anomalies.

### 🔄 Cycle de vie d'une mise à jour RIP

```
┌─────────────────────────────────────────────────────────────┐
│  Routeur A                          Routeur B                │
│                                                               │
│  1. Timer = 30s                                               │
│     ↓                                                         │
│  2. Envoi update ──────────────────→ 3. Réception            │
│     (224.0.0.9)                         ↓                     │
│                                      4. Validation            │
│                                         ↓                     │
│                                      5. Calcul métrique       │
│                                         ↓                     │
│                                      6. Mise à jour table     │
│                                         ↓                     │
│  7. Réception ACK ←───────────────  8. Reset timer          │
│     (implicite)                         (180s)                │
└─────────────────────────────────────────────────────────────┘
```

### 📊 Structure d'une mise à jour RIP

Une mise à jour RIP contient plusieurs champs importants :

|Champ|Taille|Description|Exemple|
|---|---|---|---|
|**Command**|1 octet|Request (1) ou Response (2)|2|
|**Version**|1 octet|Version RIP (1 ou 2)|2|
|**Address Family**|2 octets|Protocole (2 = IPv4)|2|
|**Route Tag**|2 octets|Tag pour filtrage|0|
|**IP Address**|4 octets|Adresse réseau|192.168.1.0|
|**Subnet Mask**|4 octets|Masque (RIPv2 uniquement)|255.255.255.0|
|**Next Hop**|4 octets|Prochain routeur|0.0.0.0|
|**Metric**|4 octets|Nombre de sauts|1|

> [!info] Différence RIPv1 vs RIPv2
> 
> - **RIPv1** : Pas de champ masque de sous-réseau (classful)
> - **RIPv2** : Inclut le masque (classless), permet VLSM et CIDR

### 🕐 Analyse des timers RIP

RIP utilise quatre timers principaux qui régissent son comportement :

#### 1. Update Timer (30 secondes)

```bash
Sending updates every 30 seconds, next due in 12 seconds
```

- **Fonction** : Fréquence d'envoi des mises à jour périodiques
- **Comportement** : Les updates sont décalés aléatoirement (±5s) pour éviter la synchronisation
- **Impact** : Plus le timer est court, plus la convergence est rapide, mais plus le trafic augmente

#### 2. Invalid Timer (180 secondes)

```bash
Invalid after 180 seconds
```

- **Fonction** : Temps avant de marquer une route comme invalide si aucune mise à jour n'est reçue
- **Calcul** : Par défaut = 6 × Update Timer
- **Comportement** : La route reste dans la table mais est marquée comme "possibly down"

#### 3. Hold-down Timer (180 secondes)

```bash
hold down 180
```

- **Fonction** : Période pendant laquelle les mises à jour d'une route invalide sont ignorées
- **Objectif** : Prévenir les boucles de routage lors de la convergence
- **Comportement** : Seules les mises à jour avec une meilleure métrique sont acceptées

#### 4. Flush Timer (240 secondes)

```bash
flushed after 240
```

- **Fonction** : Temps avant suppression définitive de la route de la table
- **Calcul** : Par défaut = Invalid Timer + 60 secondes
- **Comportement** : La route est complètement retirée de la base RIP

### 📈 Diagramme temporel des timers

```
Time (s)  0    30    60    90   120   150   180   210   240
          │    │     │     │     │     │     │     │     │
Update    ●────●─────●─────●─────●─────●─────●─────●─────●
          │                                    │           │
Route     ├────────── VALIDE ──────────────────┤           │
                                               │           │
Invalid                                        ├─ INVALID ─┤
                                               │           │
Hold-down                                      ├─HOLD-DOWN─┤
                                                           │
Flush                                                      ●
                                                    (suppression)
```

### 🔍 Analyse de scénarios typiques

#### Scénario 1 : Convergence normale

```bash
00:00:00 - Route 192.168.2.0/24 reçue avec métrique 1
00:00:30 - Mise à jour reçue, métrique toujours 1 (refresh)
00:01:00 - Mise à jour reçue, métrique toujours 1
00:01:30 - Mise à jour reçue, métrique toujours 1
# Cycle normal, timer Invalid reseté à chaque update
```

#### Scénario 2 : Perte de connectivité

```bash
00:00:00 - Dernière mise à jour reçue pour 192.168.2.0/24
00:00:30 - Pas de mise à jour (problème réseau)
00:01:00 - Pas de mise à jour
00:01:30 - Pas de mise à jour
00:03:00 - INVALID TIMER expire → route marquée "possibly down"
00:03:00 - HOLD-DOWN démarre (180s)
00:06:00 - FLUSH TIMER expire → route supprimée
```

#### Scénario 3 : Route qui oscille (flapping)

```bash
00:00:00 - Route 10.0.0.0/8 reçue, métrique 2
00:00:30 - Route invalide (métrique 16)
00:01:00 - Route reçue, métrique 2
00:01:30 - Route invalide (métrique 16)
00:02:00 - Route reçue, métrique 2
# Problème de stabilité : lien intermittent ou boucle
```

> [!warning] Indicateur de problème Les routes qui oscillent entre valide et invalide indiquent généralement :
> 
> - Un lien physique instable
> - Une boucle de routage partielle
> - Un routeur qui redémarre fréquemment
> - Des problèmes de filtre/ACL intermittents

### 📊 Métriques et calculs

#### Calcul de la métrique

La métrique RIP est simple : **nombre de sauts (hop count)**

```
Métrique finale = Métrique reçue + 1
```

> [!example] Exemple de calcul
> 
> ```
> Routeur A ─(1)─ Routeur B ─(1)─ Routeur C ─(1)─ Réseau X
> 
> Du point de vue de A :
> - Réseau directement connecté : métrique 0 (interne)
> - Réseau via B : métrique 1
> - Réseau via C : métrique 2
> - Réseau X : métrique 3
> ```

#### Métrique maximale

```bash
Métrique maximale = 15
Métrique infinie (inaccessible) = 16
```

> [!warning] Limitation de RIP La métrique maximale de 15 limite RIP aux réseaux de petite à moyenne taille. Au-delà de 15 sauts, le réseau est considéré comme inaccessible.

### 🎯 Critères de sélection des routes

Lorsque RIP reçoit plusieurs routes vers la même destination, il applique les critères suivants dans l'ordre :

1. **Métrique la plus basse** : La route avec le moins de sauts est préférée
2. **En cas d'égalité** : RIP peut utiliser jusqu'à 4 chemins (load balancing)
3. **Si > 4 chemins à égalité** : Les 4 premiers sont conservés

```bash
# Configuration du nombre de chemins maximum (1 à 16)
Router(config-router)# maximum-paths 6
```

### 🔄 Mécanismes de prévention des boucles

RIP implémente plusieurs mécanismes pour éviter les boucles de routage :

#### 1. Split Horizon

```
Règle : Ne pas renvoyer une route par l'interface par laquelle elle a été apprise
```

> [!example] Exemple
> 
> ```
> Routeur A ←──── Routeur B ←──── Routeur C
>            Gi0/0          Gi0/1
> 
> B apprend 10.0.0.0/8 via A sur Gi0/0
> → B n'annonce PAS 10.0.0.0/8 à A via Gi0/0
> → B annonce 10.0.0.0/8 à C via Gi0/1
> ```

#### 2. Poison Reverse

```
Règle : Annoncer une route avec métrique 16 (infinie) vers l'interface source
```

> [!info] Différence avec Split Horizon Split Horizon = ne pas annoncer Poison Reverse = annoncer avec métrique infinie

#### 3. Hold-down Timer

```
Règle : Ignorer les mises à jour d'une route invalide pendant 180s
Exception : Accepter uniquement les updates avec meilleure métrique
```

#### 4. Triggered Updates

```
Règle : Envoyer une mise à jour immédiate lors d'un changement (sans attendre les 30s)
```

### 📉 Analyse des problèmes courants

#### Problème 1 : Convergence lente

**Symptômes** :

```bash
# Les routes mettent 3-4 minutes à se stabiliser après un changement
```

**Causes** :

- Timers par défaut trop longs (180s invalid + hold-down)
- Trop de sauts entre routeurs
- Problèmes de split horizon sur des topologies complexes

**Solutions** :

- Réduire les timers (attention aux impacts)
- Simplifier la topologie
- Envisager un protocole plus performant (OSPF, EIGRP)

#### Problème 2 : Boucles de routage

**Symptômes** :

```bash
# Métriques qui augmentent jusqu'à 16
# Paquets qui rebondissent entre routeurs
```

**Causes** :

- Split horizon désactivé
- Configuration incorrecte des routes statiques
- Redistribution mal configurée

**Solutions** :

- Vérifier `split-horizon` sur les interfaces
- Éviter les routes statiques conflictuelles
- Utiliser le poison reverse

#### Problème 3 : Routes absentes de la table

**Symptômes** :

```bash
# Route visible dans "show ip rip database"
# Mais absente de "show ip route"
```

**Causes** :

- Distance administrative défavorable (RIP = 120)
- Route filtrée par distribute-list
- Problème de masque avec auto-summary

**Solutions** :

- Vérifier les autres protocoles de routage
- Contrôler les filtres RIP
- Désactiver auto-summary si nécessaire

### 📝 Checklist de vérification des mises à jour

Lors de l'analyse des mises à jour RIP, vérifiez systématiquement :

- [ ] **Fréquence** : Les updates arrivent-elles toutes les 30 secondes ?
- [ ] **Voisins** : Tous les voisins attendus envoient-ils des updates ?
- [ ] **Contenu** : Les routes attendues sont-elles présentes ?
- [ ] **Métriques** : Les métriques correspondent-elles à la topologie ?
- [ ] **Version** : RIPv1 ou RIPv2 cohérent sur tout le réseau ?
- [ ] **Timers** : Les timers sont-ils cohérents entre routeurs ?
- [ ] **Split Horizon** : Activé sur les interfaces appropriées ?
- [ ] **Convergence** : Le réseau converge-t-il dans un délai acceptable ?

> [!tip] Méthode d'analyse
> 
> 1. Commencez par `show ip protocols` pour une vue globale
> 2. Utilisez `show ip rip database` pour voir toutes les routes RIP
> 3. Comparez avec `show ip route` pour identifier les écarts
> 4. Si nécessaire, activez `debug ip rip` pour une analyse temps réel
> 5. Désactivez toujours le debug après diagnostic

---

## Synthèse des commandes

### 🎯 Commandes essentielles

|Commande|Mode|Fonction|Usage|
|---|---|---|---|
|`show ip protocols`|EXEC|Vue globale de la config RIP|Vérification initiale|
|`show ip rip database`|EXEC|Contenu de la base RIP|Diagnostic routes|
|`debug ip rip`|Privileged|Monitoring temps réel|Troubleshooting|
|`undebug all`|Privileged|Désactiver tous les debugs|Après diagnostic|

### 🔄 Workflow de diagnostic

```
1. Problème détecté
         ↓
2. show ip protocols
   (vérifier config générale)
         ↓
3. show ip rip database
   (vérifier routes apprises)
         ↓
4. show ip route
   (comparer avec table finale)
         ↓
5. Si nécessaire : debug ip rip
   (observer en temps réel)
         ↓
6. undebug all
   (toujours nettoyer)
```

### 🛡️ Bonnes pratiques de maintenance

> [!tip] Recommandations
> 
> - Vérifiez régulièrement l'état de RIP avec `show ip protocols`
> - Documentez la topologie et les métriques attendues
> - Surveillez les changements fréquents de routes (instabilité)
> - Limitez l'usage de `debug` en production
> - Gardez des logs des configurations pour comparaison
> - Testez les changements en environnement de lab avant production
> - Utilisez des outils de monitoring pour détecter les anomalies
> - Configurez des syslog pour archiver les événements RIP
> - Planifiez des maintenances régulières pour vérifier la santé du réseau

### ⚡ Commandes rapides de dépannage

```bash
# Vérification complète en 4 commandes
Router# show ip protocols
Router# show ip rip database
Router# show ip route rip
Router# show ip interface brief

# Si problème persistant
Router# debug ip rip
# Attendre 1-2 minutes
Router# undebug all
```

### 📊 Tableau récapitulatif des timers

|Timer|Valeur défaut|Fonction|Impact si modifié|
|---|---|---|---|
|**Update**|30s|Fréquence d'envoi|↓ = convergence rapide mais + trafic|
|**Invalid**|180s|Marquer route invalide|↓ = détection rapide des pannes|
|**Hold-down**|180s|Ignorer updates invalides|↓ = risque de boucles|
|**Flush**|240s|Suppression définitive|↓ = nettoyage rapide|

> [!warning] Modification des timers Tous les routeurs RIP d'un réseau doivent avoir les **mêmes valeurs de timers** pour éviter les problèmes de synchronisation et de convergence.

### 🔧 Commandes de modification des timers

```bash
Router(config)# router rip
Router(config-router)# timers basic update invalid holddown flush

# Exemple : réduire les timers pour convergence plus rapide
Router(config-router)# timers basic 10 60 60 90
# Update=10s, Invalid=60s, Hold-down=60s, Flush=90s
```

> [!warning] Attention La réduction des timers augmente la charge CPU et le trafic réseau. À n'utiliser que sur des réseaux petits et bien contrôlés.

---

## 🎓 Points clés à retenir

### ✅ Ce qu'il faut savoir

1. **Show ip protocols** : Première commande pour une vue globale de RIP
2. **Show ip rip database** : Voir toutes les routes RIP avant insertion dans la table
3. **Debug ip rip** : Outil puissant mais à utiliser avec précaution
4. **Timers RIP** : 30s (update), 180s (invalid/hold-down), 240s (flush)
5. **Métrique** : Hop count, maximum 15, infinie = 16
6. **Toujours désactiver les debugs** après utilisation

### ⚠️ Erreurs à éviter

1. ❌ Laisser un `debug ip rip` actif en production
2. ❌ Ne pas comparer database RIP et table de routage
3. ❌ Modifier les timers sans coordination entre routeurs
4. ❌ Ignorer les routes en hold-down (signe de problème)
5. ❌ Ne pas vérifier les versions RIP (v1 vs v2)
6. ❌ Oublier de vérifier split-horizon sur les interfaces

### 🎯 Méthodologie de diagnostic

```
Symptôme observé
       ↓
Collecte d'informations (show commands)
       ↓
Analyse comparative (database vs routing table)
       ↓
Hypothèses de cause
       ↓
Debug ciblé si nécessaire
       ↓
Correction
       ↓
Vérification post-correction
```

### 📋 Checklist finale de vérification

Avant de valider une configuration RIP, vérifiez :

- [ ] Tous les réseaux attendus sont listés dans `show ip protocols`
- [ ] Tous les voisins RIP apparaissent dans "Routing Information Sources"
- [ ] Les routes de la database RIP sont bien dans la table de routage
- [ ] Les métriques correspondent à la topologie réelle
- [ ] Les timers sont identiques sur tous les routeurs
- [ ] Split-horizon est activé (sauf cas particuliers)
- [ ] Aucune route n'oscille entre valide/invalide (flapping)
- [ ] La convergence se fait en moins de 3-4 minutes
- [ ] Pas de boucles de routage (métriques qui montent à 16)
- [ ] Les updates sont reçues toutes les 30 secondes

---

## 💡 Astuces professionnelles

### 🔍 Surveillance proactive

Mettez en place une surveillance régulière :

```bash
# Script de monitoring basique
Router# show ip protocols | include Routing Protocol
Router# show ip rip database | count
Router# show ip route rip | count
```

### 📝 Documentation

Maintenez à jour :

- Schéma de topologie avec métriques attendues
- Liste des réseaux annoncés par chaque routeur
- Historique des modifications de configuration
- Baselines des sorties de commandes en état stable

### 🚨 Alertes à surveiller

Configurez des alertes ou vérifiez régulièrement :

1. **Changements fréquents de routes** : Signe d'instabilité
2. **Métriques anormalement élevées** : Problème de topologie
3. **Disparition soudaine de voisins** : Problème de connectivité
4. **Routes en hold-down prolongé** : Boucle ou problème de convergence

### 🎪 Cas d'usage réels

#### Cas 1 : Réseau ne converge pas

```bash
# Diagnostic
Router# show ip protocols
# Vérifier : "Routing Information Sources" est vide
Router# show ip interface brief
# Vérifier : Les interfaces sont up/up
Router# debug ip rip
# Observer : Aucun message "received" n'apparaît

# Cause probable : Problème de connectivité ou RIP non configuré sur voisin
```

#### Cas 2 : Route présente mais pas utilisée

```bash
# Diagnostic
Router# show ip rip database
# La route 10.0.0.0/8 est présente avec métrique 2
Router# show ip route
# La route n'apparaît pas

# Cause probable : Route avec meilleure AD via un autre protocole
Router# show ip route 10.0.0.0
# Vérifier quel protocole est utilisé et pourquoi
```

#### Cas 3 : Convergence très lente

```bash
# Observation
Router# debug ip rip
# Les routes mettent 4-5 minutes à se stabiliser après un changement

# Analyse
Router# show ip protocols | include Invalid
# "Invalid after 180 seconds, hold down 180, flushed after 240"

# Cause : Timers par défaut trop longs pour l'environnement
# Solution : Réduire les timers (avec coordination)
```

---

## 🔬 Exercices d'analyse

### Exercice 1 : Interpréter une sortie

```bash
Router# show ip protocols

Routing Protocol is "rip"
  Sending updates every 30 seconds, next due in 8 seconds
  Invalid after 180 seconds, hold down 180, flushed after 240
  Default version control: send version 2, receive version 2
  Automatic network summarization is in effect
  Maximum path: 4
  Routing for Networks:
    10.0.0.0
    192.168.1.0
  Routing Information Sources:
    Gateway         Distance      Last Update
  Distance: (default is 120)
```

**Question** : Quel est le problème majeur visible ici ?

<details> <summary>💡 Réponse</summary>

**Problème** : Aucun voisin RIP n'apparaît dans "Routing Information Sources"

**Causes possibles** :

1. RIP non configuré sur les routeurs voisins
2. Problème de connectivité physique
3. Versions RIP incompatibles
4. ACL bloquant le trafic RIP (UDP 520)

**Actions correctives** :

- Vérifier la configuration RIP des voisins
- Tester la connectivité (ping)
- Vérifier les ACLs sur les interfaces

</details>

### Exercice 2 : Analyser un debug

```bash
Router# debug ip rip

RIP: received v2 update from 10.1.1.2 on GigabitEthernet0/0
      192.168.1.0/24 via 0.0.0.0 in 2 hops
      192.168.2.0/24 via 0.0.0.0 in 3 hops
      172.16.0.0/16 via 0.0.0.0 in 16 hops

RIP: sending v2 update to 224.0.0.9 via GigabitEthernet0/1
      192.168.1.0/24 via 0.0.0.0, metric 3, tag 0
      172.16.0.0/16 via 0.0.0.0, metric 16, tag 0
```

**Question** : Que remarquez-vous d'anormal et quelle en est la conséquence ?

<details> <summary>💡 Réponse</summary>

**Anomalie** : Le réseau 172.16.0.0/16 a une métrique de 16

**Signification** :

- Métrique 16 = route inaccessible (métrique infinie dans RIP)
- Cette route ne sera pas installée dans la table de routage
- Le routeur propage l'information d'inaccessibilité à ses voisins

**Causes possibles** :

- Le réseau 172.16.0.0/16 est réellement down
- Il y a plus de 15 sauts jusqu'à cette destination
- Boucle de routage détectée (count to infinity)

**Action** : Investiguer pourquoi ce réseau est inaccessible

</details>

---

## 🎯 Conclusion

La vérification et la maintenance de RIP reposent sur trois piliers :

1. **Observation régulière** avec `show ip protocols` et `show ip rip database`
2. **Diagnostic ciblé** avec `debug ip rip` en cas de problème
3. **Compréhension profonde** des timers, métriques et mécanismes anti-boucles

> [!tip] Conseil final La maîtrise de ces outils de vérification vous permettra non seulement de maintenir RIP efficacement, mais aussi de développer une méthodologie applicable à d'autres protocoles de routage (OSPF, EIGRP, BGP).

### 📚 Commandes de référence rapide

```bash
# Vérification de base
show ip protocols
show ip rip database
show ip route rip

# Diagnostic approfondi
debug ip rip
show ip interface brief
show running-config | section router rip

# Nettoyage
undebug all
clear ip route *
```

> [!warning] Rappel de sécurité
> 
> - Ne jamais laisser un debug actif en production
> - Toujours documenter les changements de configuration
> - Tester en environnement de lab avant déploiement
> - Maintenir des backups de configuration à jour

---

**Fin du cours - Vérification et Maintenance RIP** ✅