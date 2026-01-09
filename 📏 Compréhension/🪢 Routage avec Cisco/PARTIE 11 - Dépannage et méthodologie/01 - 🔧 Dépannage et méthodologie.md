

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

## 🎯 Méthodologie de dépannage

### Pourquoi une méthodologie ?

Le dépannage réseau sans méthodologie structurée conduit souvent à des pertes de temps considérables, des changements de configuration hasardeux et potentiellement l'aggravation du problème. Une approche méthodique permet de résoudre les incidents plus rapidement et de documenter les solutions pour l'avenir.

### Les étapes fondamentales du dépannage

#### 1. Définir le problème

Avant de chercher une solution, il faut comprendre précisément le problème :

- **Symptômes observés** : Quels sont les comportements anormaux ?
- **Périmètre** : Qui est affecté ? Tous les utilisateurs ou seulement certains ?
- **Moment** : Quand le problème est-il apparu ? Y a-t-il eu des changements récents ?
- **Reproductibilité** : Le problème est-il constant ou intermittent ?

> [!example] Exemple de définition de problème ❌ Mauvaise définition : "Le réseau ne marche pas"
> 
> ✅ Bonne définition : "Depuis ce matin 9h, les utilisateurs du VLAN 20 (192.168.20.0/24) ne peuvent plus accéder au serveur web 10.0.0.50. Les autres VLANs fonctionnent normalement. Aucun changement de configuration n'a été effectué."

#### 2. Rassembler les informations

> [!info] Informations critiques à collecter
> 
> - Schéma réseau et topologie
> - Configurations des équipements impliqués
> - Journaux système (logs)
> - État des interfaces et protocoles
> - Changements récents de configuration

#### 3. Analyser les informations

Examiner les données collectées pour identifier des patterns, des anomalies ou des corrélations. C'est à cette étape que la connaissance du modèle OSI devient cruciale.

#### 4. Éliminer les causes probables

Tester les hypothèses une par une en commençant par les plus probables. Documenter chaque test pour éviter les répétitions.

#### 5. Proposer une hypothèse

Formuler une théorie explicative basée sur les informations et l'analyse effectuée.

#### 6. Tester l'hypothèse

Effectuer des tests ciblés pour confirmer ou infirmer l'hypothèse sans impacter davantage le réseau.

#### 7. Résoudre et documenter

Une fois la cause identifiée, appliquer la solution, vérifier le retour à la normale et documenter l'incident pour référence future.

> [!warning] Pièges courants
> 
> - **Sauter des étapes** : Vouloir aller trop vite conduit souvent à des erreurs
> - **Changer plusieurs paramètres simultanément** : Impossible de savoir quelle modification a résolu le problème
> - **Ne pas documenter** : Les mêmes problèmes reviendront sans trace de la résolution précédente
> - **Négliger les sauvegardes** : Toujours sauvegarder avant de modifier une configuration

---

## 🔺 Modèle OSI et approche par couches

### Le modèle OSI comme référentiel de dépannage

Le modèle OSI (Open Systems Interconnection) avec ses 7 couches fournit un cadre structuré pour comprendre où se situe un problème dans la pile réseau.

|Couche|Nom|Responsabilité|Éléments Cisco concernés|
|---|---|---|---|
|**7**|Application|Services réseau aux applications|Pas directement géré par le routeur|
|**6**|Présentation|Format et chiffrement des données|Pas directement géré par le routeur|
|**5**|Session|Gestion des connexions|Pas directement géré par le routeur|
|**4**|Transport|Fiabilité de bout en bout (TCP/UDP)|ACL, NAT, inspection|
|**3**|Réseau|Routage et adressage IP|Routes, routage, adresses IP|
|**2**|Liaison|Adressage MAC, trames|Interfaces, VLANs, switches|
|**1**|Physique|Câbles, signaux électriques|Câblage, état des interfaces|

### Mapping des problèmes par couche

#### Couche 1 - Physique

**Symptômes typiques :**

- Interface en état "down/down"
- Pas de signal détecté
- Erreurs CRC élevées
- Collisions excessives

**Commandes de diagnostic :**

```bash
# Vérifier l'état physique des interfaces
Router# show interfaces status
Router# show interfaces GigabitEthernet0/0

# Rechercher des erreurs physiques
Router# show interfaces GigabitEthernet0/0 | include error|CRC|collision
```

> [!tip] Astuces Couche 1
> 
> - Vérifier toujours les câbles et connecteurs en premier
> - Utiliser `show controllers` pour des informations bas niveau
> - Les problèmes de couche 1 sont souvent les plus simples à résoudre

#### Couche 2 - Liaison de données

**Symptômes typiques :**

- Interface en état "up/down"
- Problèmes de VLAN
- Erreurs d'encapsulation
- Problèmes de trunking

**Commandes de diagnostic :**

```bash
# État de la couche 2
Router# show interfaces GigabitEthernet0/0 switchport

# Vérifier les VLANs (sur switch)
Switch# show vlan brief

# Vérifier l'encapsulation
Router# show interfaces GigabitEthernet0/0.10
```

> [!example] Exemple de problème Couche 2
> 
> ```bash
> Router# show interfaces GigabitEthernet0/0
> GigabitEthernet0/0 is up, line protocol is down
> # "up/down" indique un problème de couche 2 :
> # - Mauvaise encapsulation
> # - VLAN manquant
> # - Problème de keepalive
> ```

#### Couche 3 - Réseau

**Symptômes typiques :**

- Impossible de pinguer des réseaux distants
- Routes manquantes dans la table de routage
- Mauvaise configuration d'adresses IP
- Problèmes de passerelle

**Commandes de diagnostic :**

```bash
# Vérifier la table de routage
Router# show ip route

# Vérifier les adresses IP des interfaces
Router# show ip interface brief

# Tester la connectivité
Router# ping 192.168.1.1
Router# traceroute 192.168.1.1

# Vérifier les protocoles de routage
Router# show ip protocols
Router# show ip ospf neighbor
Router# show ip eigrp neighbors
```

> [!warning] Pièges Couche 3
> 
> - Oublier d'activer le routage IP : `ip routing`
> - Masques de sous-réseau incorrects
> - Passerelle par défaut manquante ou erronée
> - Routes statiques en conflit avec le routage dynamique

#### Couche 4 - Transport

**Symptômes typiques :**

- Connexions TCP qui n'aboutissent pas
- Ports bloqués
- Problèmes d'ACL affectant des services spécifiques

**Commandes de diagnostic :**

```bash
# Vérifier les ACL appliquées
Router# show ip access-lists
Router# show ip interface GigabitEthernet0/0 | include access list

# Vérifier les translations NAT
Router# show ip nat translations

# Tester des ports spécifiques
Router# telnet 192.168.1.1 80
```

### Approche systématique par couches

Quelle que soit l'approche choisie (ascendante ou descendante), toujours :

1. **Vérifier une couche à la fois** de manière exhaustive
2. **Documenter les résultats** de chaque vérification
3. **Ne pas sauter de couches** même si cela semble évident
4. **Comprendre les dépendances** entre couches (une couche supérieure ne peut fonctionner si une couche inférieure est défaillante)

> [!info] Principe fondamental Les couches du modèle OSI sont interdépendantes : si la couche 1 est défaillante, toutes les couches supérieures seront impactées. C'est pourquoi l'approche ascendante est souvent privilégiée.

---

## ⬆️⬇️ Approche descendante vs ascendante

### Approche ascendante (Bottom-Up)

L'approche ascendante consiste à commencer par la couche physique (couche 1) et à remonter progressivement vers les couches supérieures.

#### Quand l'utiliser ?

- **Problème totalement inconnu** : Vous n'avez aucune idée de la cause
- **Panne complète** : Aucune connectivité du tout
- **Nouveau déploiement** : Vérification méthodique d'une nouvelle installation
- **Formation/Apprentissage** : Approche pédagogique pour comprendre les dépendances

#### Méthodologie ascendante

```bash
# ÉTAPE 1 : Couche Physique
Router# show interfaces GigabitEthernet0/0
# Vérifier : "is up" et "line protocol is up"
# Rechercher : erreurs, collisions, resets

# ÉTAPE 2 : Couche Liaison
Router# show interfaces GigabitEthernet0/0 switchport
Router# show vlan brief  # Si switch
# Vérifier : encapsulation, VLANs, adresses MAC

# ÉTAPE 3 : Couche Réseau
Router# show ip interface brief
Router# show ip route
Router# ping <next-hop>
# Vérifier : adresses IP, masques, routes, connectivité locale

# ÉTAPE 4 : Couche Transport et supérieures
Router# show ip access-lists
Router# telnet <destination> <port>
# Vérifier : ACL, services applicatifs
```

#### Avantages

✅ Approche méthodique et exhaustive ✅ Ne rate aucun problème de base ✅ Idéale pour les débutants ✅ Garantit que les fondations sont solides

#### Inconvénients

❌ Peut être longue si le problème est en couche haute ❌ Peut vérifier des éléments qui fonctionnent déjà ❌ Moins efficace pour les techniciens expérimentés

> [!example] Exemple d'utilisation Un nouveau routeur vient d'être installé et aucun service ne fonctionne. L'approche ascendante permet de valider chaque couche systématiquement pour s'assurer que la configuration de base est correcte.

### Approche descendante (Top-Down)

L'approche descendante consiste à commencer par les couches applicatives (couches 7-5) et à descendre vers les couches inférieures uniquement si nécessaire.

#### Quand l'utiliser ?

- **Problème applicatif spécifique** : Un service particulier ne fonctionne pas mais le reste fonctionne
- **Symptômes clairs** : Les utilisateurs signalent un problème précis (ex: impossible d'accéder à un site web)
- **Techniciens expérimentés** : Permet d'aller plus vite quand on a de l'expérience
- **Problème intermittent** : Souvent lié à des couches hautes

#### Méthodologie descendante

```bash
# ÉTAPE 1 : Test applicatif
Router# telnet 192.168.1.100 80  # Tester le service HTTP
Router# ping www.example.com     # Tester la résolution DNS

# Si échec, descendre à la couche Transport/Réseau
# ÉTAPE 2 : Couche Transport et Réseau
Router# show ip access-lists              # Vérifier les ACL
Router# show ip nat translations          # Vérifier NAT
Router# ping 192.168.1.100               # Test ICMP couche 3
Router# traceroute 192.168.1.100         # Tracer le chemin

# Si échec, descendre à la couche Liaison
# ÉTAPE 3 : Couche Liaison
Router# show arp                          # Vérifier la résolution ARP
Router# show mac address-table            # Sur switch

# Si échec, vérifier la couche Physique
# ÉTAPE 4 : Couche Physique
Router# show interfaces GigabitEthernet0/0
```

#### Avantages

✅ Plus rapide pour les problèmes ciblés ✅ Évite de vérifier des couches qui fonctionnent ✅ Efficace pour les techniciens expérimentés ✅ Correspond à la perception utilisateur du problème

#### Inconvénients

❌ Peut rater des problèmes de couches basses ❌ Nécessite de l'expérience pour être efficace ❌ Risque de fausses pistes si plusieurs problèmes coexistent

> [!example] Exemple d'utilisation Les utilisateurs ne peuvent plus accéder à un serveur web spécifique (port 80), mais les pings fonctionnent. L'approche descendante permet de vérifier d'abord les ACL et les règles de pare-feu avant de descendre aux couches inférieures.

### Approche "Diviser pour régner"

Une troisième approche hybride consiste à identifier le milieu de la pile OSI (généralement couche 3) et à vérifier dans les deux directions.

#### Méthodologie

```bash
# ÉTAPE 1 : Commencer par la couche 3 (Réseau)
Router# ping <destination>

# Si le ping fonctionne :
#   → Le problème est probablement en couches 4-7
#   → Utiliser l'approche descendante

# Si le ping échoue :
#   → Le problème est probablement en couches 1-3
#   → Utiliser l'approche ascendante
```

#### Avantages

✅ Très efficace pour les techniciens expérimentés ✅ Élimine rapidement la moitié des possibilités ✅ Flexible selon les résultats

> [!tip] Conseil pratique La plupart des techniciens expérimentés utilisent inconsciemment l'approche "diviser pour régner" : un simple ping permet de déterminer rapidement s'il faut chercher en haut ou en bas de la pile OSI.

### Comparaison récapitulative

|Critère|Ascendante|Descendante|Diviser pour régner|
|---|---|---|---|
|**Vitesse**|🐢 Lente|🚀 Rapide|⚡ Très rapide|
|**Exhaustivité**|✅✅✅ Complète|⚠️ Partielle|✅✅ Bonne|
|**Niveau requis**|👶 Débutant|👨‍💼 Expérimenté|👨‍💼 Expérimenté|
|**Risque d'erreur**|✅ Faible|⚠️ Moyen|✅ Faible|

> [!warning] Recommandation Pour les débutants, privilégiez **toujours l'approche ascendante**. Une fois que vous maîtrisez le dépannage, vous pourrez naturellement adapter votre approche selon le contexte.

---

## 📊 Collecte d'informations

La collecte d'informations est l'étape la plus critique du dépannage. Des informations incomplètes ou incorrectes conduiront à des hypothèses erronées et à une perte de temps considérable.

### Informations à collecter AVANT le dépannage

#### 1. Documentation réseau

> [!info] Documents essentiels
> 
> - **Schéma de topologie physique** : Où sont les équipements ?
> - **Schéma de topologie logique** : Comment les réseaux sont-ils interconnectés ?
> - **Plan d'adressage IP** : Quels sont les sous-réseaux utilisés ?
> - **Configuration des VLANs** : Quels VLANs existent et sur quels ports ?
> - **Historique des changements** : Quelles modifications ont été faites récemment ?

#### 2. État de référence (Baseline)

Un état de référence du réseau en fonctionnement normal est invaluable pour comparer avec la situation actuelle.

```bash
# Créer un état de référence régulièrement
Router# show version | redirect flash:baseline-version.txt
Router# show running-config | redirect flash:baseline-config.txt
Router# show ip route | redirect flash:baseline-routes.txt
Router# show interfaces | redirect flash:baseline-interfaces.txt
```

> [!tip] Bonnes pratiques de baseline
> 
> - Créer une baseline après chaque changement majeur de configuration
> - Automatiser la collecte hebdomadaire avec des scripts
> - Sauvegarder les baselines hors du routeur (serveur TFTP/FTP)
> - Inclure les statistiques d'utilisation et de performance

### Commandes essentielles de collecte d'informations

#### Informations générales sur le système

```bash
# Version IOS et matériel
Router# show version
# Informations : Version IOS, uptime, mémoire, interfaces disponibles

# Configuration active
Router# show running-config
# Toute la configuration actuellement en mémoire

# Configuration sauvegardée
Router# show startup-config
# Configuration qui sera chargée au prochain redémarrage

# Historique des commandes
Router# show history
# Liste des dernières commandes tapées (utile pour tracer les actions)
```

#### Informations sur les interfaces

```bash
# Vue d'ensemble de toutes les interfaces
Router# show ip interface brief
# Format compact : nom, IP, statut (up/down)

# Détails complets d'une interface
Router# show interfaces GigabitEthernet0/0
# Statistiques détaillées : erreurs, paquets, bande passante, etc.

# Statistiques d'erreurs uniquement
Router# show interfaces GigabitEthernet0/0 | include error|drop|collision
# Filtre les lignes contenant les mots-clés importants

# État de toutes les interfaces (format tableau)
Router# show interfaces status
# Vue compacte et lisible de l'état de chaque port
```

> [!example] Interpréter les états d'interface
> 
> ```bash
> Router# show ip interface brief
> Interface         IP-Address      OK? Method Status                Protocol
> GigabitEthernet0/0 192.168.1.1    YES manual up                    up
> GigabitEthernet0/1 unassigned     YES unset  administratively down down
> Serial0/0/0       10.0.0.1        YES manual up                    down
> ```
> 
> - **up/up** : Interface fonctionne correctement (couches 1 et 2 OK)
> - **up/down** : Problème de couche 2 (encapsulation, keepalive)
> - **down/down** : Problème de couche 1 (câble, pas de signal)
> - **administratively down** : Interface désactivée volontairement (commande `shutdown`)

#### Informations sur le routage

```bash
# Table de routage complète
Router# show ip route
# Toutes les routes connues (connectées, statiques, dynamiques)

# Routes d'un protocole spécifique
Router# show ip route ospf
Router# show ip route eigrp
Router# show ip route static

# Détails d'une route spécifique
Router# show ip route 192.168.1.0
# Affiche comment atteindre ce réseau

# Protocoles de routage actifs
Router# show ip protocols
# Quels protocoles sont configurés et leurs paramètres
```

#### Informations sur les protocoles de routage

```bash
# OSPF
Router# show ip ospf neighbor          # Voisins OSPF
Router# show ip ospf interface         # Interfaces OSPF
Router# show ip ospf database          # Base de données LSDB

# EIGRP
Router# show ip eigrp neighbors        # Voisins EIGRP
Router# show ip eigrp topology         # Topologie EIGRP
Router# show ip eigrp interfaces       # Interfaces EIGRP

# RIP (si utilisé)
Router# show ip rip database           # Base de données RIP
```

#### Informations sur la connectivité

```bash
# Test de connectivité basique
Router# ping 192.168.1.1
# Vérifie si la destination est joignable (couche 3)

# Test de connectivité étendu
Router# ping
Protocol [ip]: 
Target IP address: 192.168.1.1
Repeat count [5]: 100
Datagram size [100]: 
Timeout in seconds [2]: 
Extended commands [n]: y
Source address or interface: 10.0.0.1
# Permet de spécifier l'interface source et autres paramètres

# Tracer le chemin vers une destination
Router# traceroute 192.168.1.1
# Affiche tous les sauts (routeurs) entre la source et la destination

# Table ARP
Router# show arp
Router# show ip arp
# Correspondances IP <-> MAC apprises
```

> [!tip] Astuces de connectivité
> 
> ```bash
> # Ping étendu pour tester depuis une interface spécifique
> Router# ping 192.168.1.1 source GigabitEthernet0/0
> 
> # Ping avec taille de paquet personnalisée (tester MTU)
> Router# ping 192.168.1.1 size 1500 df-bit
> 
> # Traceroute avec résolution DNS désactivée (plus rapide)
> Router# traceroute 192.168.1.1 numeric
> ```

#### Informations sur les ACL et la sécurité

```bash
# Toutes les ACL configurées
Router# show access-lists
Router# show ip access-lists

# ACL appliquées sur une interface
Router# show ip interface GigabitEthernet0/0 | include access list

# Statistiques NAT
Router# show ip nat translations
Router# show ip nat statistics
```

#### Journaux et événements système

```bash
# Afficher les logs
Router# show logging
# Tous les messages système enregistrés

# Logs en temps réel (Ctrl+C pour arrêter)
Router# terminal monitor
Router# debug ip routing  # ATTENTION : impacte les performances
Router# undebug all       # Désactiver tous les debugs

# Messages critiques uniquement
Router# show logging | include %ERR|%WARN
```

> [!warning] Attention aux commandes debug Les commandes `debug` génèrent énormément de trafic et peuvent surcharger le processeur d'un routeur en production. Utilisez-les avec précaution et désactivez-les immédiatement après usage avec `undebug all` ou `no debug all`.

### Techniques de collecte efficaces

#### Utiliser les filtres de sortie

```bash
# Inclure uniquement les lignes contenant un mot-clé
Router# show running-config | include interface|ip address

# Exclure certaines lignes
Router# show interfaces | exclude Serial

# Afficher uniquement une section
Router# show running-config | section router ospf

# Commencer l'affichage à partir d'une ligne
Router# show running-config | begin line vty
```

#### Rediriger les sorties vers des fichiers

```bash
# Sauvegarder dans la flash interne
Router# show running-config | redirect flash:config-backup.txt

# Ajouter à un fichier existant
Router# show interfaces | append flash:interface-stats.txt

# Envoyer vers un serveur TFTP
Router# show running-config | redirect tftp://192.168.1.100/config.txt
```

#### Comparer deux configurations

```bash
# Comparer configuration active vs sauvegardée
Router# show archive config differences nvram:startup-config system:running-config

# Afficher les différences
# Les lignes avec + ont été ajoutées
# Les lignes avec - ont été supprimées
# Les lignes avec ! sont identiques
```

### Collecte d'informations structurée

> [!info] Template de collecte Lors d'un incident, collectez systématiquement :
> 
> 1. **Contexte** : Date, heure, qui a signalé le problème
> 2. **Symptômes** : Description précise du dysfonctionnement
> 3. **Périmètre** : Équipements et utilisateurs affectés
> 4. **État système** : `show version`, `show running-config`
> 5. **État interfaces** : `show ip interface brief`, `show interfaces`
> 6. **État routage** : `show ip route`, `show ip protocols`
> 7. **Connectivité** : Résultats de ping et traceroute
> 8. **Logs** : Messages d'erreur récents
> 9. **Changements** : Toute modification effectuée avant l'incident

### Automatisation de la collecte

```bash
# Script de collecte d'informations (à exécuter en mode EXEC)
Router# show version | redirect flash:info-version.txt
Router# show running-config | redirect flash:info-config.txt
Router# show ip interface brief | redirect flash:info-interfaces.txt
Router# show ip route | redirect flash:info-routes.txt
Router# show logging | redirect flash:info-logs.txt
```

> [!tip] Créer un alias pour la collecte rapide
> 
> ```bash
> Router(config)# alias exec collect-info do show version | append flash:diagnostic.txt; do show running-config | append flash:diagnostic.txt; do show ip interface brief | append flash:diagnostic.txt; do show ip route | append flash:diagnostic.txt
> 
> # Utilisation :
> Router# collect-info
> # Toutes les informations sont ajoutées au fichier diagnostic.txt
> ```

---

## 🎯 Isolement du problème

L'isolement du problème consiste à réduire progressivement le périmètre de recherche jusqu'à identifier précisément la cause racine du dysfonctionnement.

### Principe de l'isolement

> [!info] Objectif Diviser le réseau en segments logiques et déterminer où se situe exactement le problème en éliminant les parties qui fonctionnent correctement.

### Techniques d'isolement

#### 1. Isolement par segmentation réseau

Diviser le réseau en segments et tester chaque segment individuellement.

```bash
# Tester le segment local (même sous-réseau)
Router# ping 192.168.1.2  # Hôte sur le même LAN

# Tester la passerelle par défaut
Router# ping 192.168.1.254  # Gateway local

# Tester le routeur suivant
Router# ping 10.0.0.2  # Next hop

# Tester la destination finale
Router# ping 172.16.0.1  # Destination distante
```

**Interprétation :**

- Si le ping local échoue → problème dans le segment local (couche 1-2)
- Si seul le ping distant échoue → problème de routage (couche 3)
- Si tous les pings échouent sauf local → problème de passerelle

#### 2. Isolement par substitution

Remplacer ou contourner des composants pour identifier celui qui pose problème.

> [!example] Exemple de substitution
> 
> ```bash
> # Test original échoue :
> PC1 ---[Switch]---[Router A]---[Router B]--- PC2
> 
> # Méthode 1 : Contourner Router A
> PC1 ---[Switch]---[Router B]--- PC2
> 
> # Méthode 2 : Remplacer le câble entre Switch et Router A
> # Méthode 3 : Tester depuis Router A directement (exclure Switch)
> ```

#### 3. Isolement par comparaison

Comparer un élément fonctionnel avec un élément défaillant pour identifier les différences.

```bash
# Comparer deux interfaces (une qui marche, une qui ne marche pas)
Router# show interfaces GigabitEthernet0/0  # Fonctionne
Router# show interfaces GigabitEthernet0/1  # Ne fonctionne pas

# Comparer les configurations
Router# show running-config interface GigabitEthernet0/0
Router# show running-config interface GigabitEthernet0/1
# Rechercher les différences de configuration
```

#### 4. Isolement par simplification

Réduire la configuration à sa forme la plus simple pour éliminer les variables.

> [!warning] Attention en production Cette technique nécessite de modifier temporairement la configuration. Toujours sauvegarder avant et avoir un plan de retour arrière.

```bash
# Exemple : Problème avec une route statique complexe
# Configuration actuelle (complexe)
Router(config)# ip route 172.16.0.0 255.255.0.0 10.0.0.2
Router(config)# ip route 172.16.0.0 255.255.0.0 10.0.0.3 10
Router(config)# ip route 172.16.0.0 255.255.0.0 10.0.0.4 20

# Simplification : Supprimer les routes de backup temporairement
Router(config)# no ip route 172.16.0.0 255.255.0.0 10.0.0.3 10
Router(config)# no ip route 172.16.0.0 255.255.0.0 10.0.0.4 20
# Tester avec uniquement la route primaire

# Si ça fonctionne, rajouter les routes une par une
Router(config)# ip route 172.16.0.0 255.255.0.0 10.0.0.3 10
# Tester à nouveau pour identifier laquelle pose problème
```

### Méthodologie d'isolement systématique

#### Étape 1 : Définir le périmètre initial

Déterminer tous les éléments potentiellement impliqués dans le problème.

```bash
# Exemple : Impossible d'accéder au serveur 172.16.0.50
# Périmètre :
# - PC client (192.168.1.10)
# - Switch d'accès
# - Routeur A (gateway 192.168.1.254)
# - Routeur B (intermédiaire)
# - Routeur C (réseau serveur)
# - Serveur (172.16.0.50)
```

#### Étape 2 : Tester les extrémités

Commencer par vérifier que les points de départ et d'arrivée fonctionnent.

```bash
# Depuis le PC client
PC> ping 192.168.1.254  # Tester la gateway locale

# Depuis le routeur proche du serveur
RouterC# ping 172.16.0.50  # Tester le serveur depuis le routeur adjacent
```

**Interprétation :**

- Si le serveur répond depuis RouterC → Le problème est dans le chemin réseau
- Si le serveur ne répond pas depuis RouterC → Le problème est local au serveur ou à son segment

#### Étape 3 : Tester le chemin segment par segment

```bash
# Depuis RouterA
RouterA# ping 10.0.0.2  # Next hop vers RouterB
RouterA# traceroute 172.16.0.50  # Identifier où ça bloque

# Analyser le traceroute
# Chaque ligne = un routeur traversé
# * * * = timeout à ce niveau → problème localisé
```

> [!example] Exemple de traceroute révélateur
> 
> ```bash
> RouterA# traceroute 172.16.0.50
> 
> Type escape sequence to abort.
> Tracing the route to 172.16.0.50
> 
>   1 10.0.0.2 4 msec 4 msec 4 msec      # RouterB OK
>   2 10.0.1.2 8 msec 8 msec 8 msec      # RouterC OK
>   3  *  *  *                            # Timeout ici
>   4  *  *  *
>   5  *  *  *
> 
> # Le problème se situe entre RouterC (10.0.1.2) et le serveur (172.16.0.50)
> ```

#### Étape 4 : Isoler la couche OSI concernée

Une fois le segment problématique identifié, déterminer quelle couche est en cause.

```bash
# Sur le routeur où le problème est localisé (RouterC)

# Test Couche 1
RouterC# show interfaces GigabitEthernet0/1
# Vérifier : "is up" et "line protocol is up"

# Test Couche 2
RouterC# show arp | include 172.16.0.50
# Vérifier si l'adresse MAC est présente dans la table ARP

# Test Couche 3
RouterC# show ip route 172.16.0.0
# Vérifier si la route existe vers le réseau du serveur

# Test Couche 4
RouterC# telnet 172.16.0.50 80
# Tester l'accès à un port spécifique (si ICMP ne fonctionne pas)
```

#### Étape 5 : Réduire progressivement les variables

Éliminer les éléments qui fonctionnent pour se concentrer sur ceux qui posent problème.

> [!tip] Matrice d'élimination
> 
> |Élément testé|Fonctionne ?|Action|
> |---|---|---|
> |Interface physique|✅ Oui|Éliminer de la recherche|
> |Configuration IP|✅ Oui|Éliminer de la recherche|
> |Table de routage|❌ Non|**INVESTIGUER**|
> |ACL|❓ À tester|Tester ensuite|

### Techniques avancées d'isolement

#### Isolement par analyse de logs

Les logs système peuvent révéler instantanément où se situe le problème.

```bash
# Rechercher des erreurs spécifiques
Router# show logging | include %
Router# show logging | include ERR|WARN

# Exemples de messages révélateurs
%LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet0/0, changed state to down
# → Problème de couche 2 sur cette interface

%LINK-3-UPDOWN: Interface GigabitEthernet0/1, changed state to down
# → Problème de couche 1 sur cette interface

%OSPF-5-ADJCHG: Process 1, Nbr 10.0.0.2 on GigabitEthernet0/0 from FULL to DOWN
# → Relation de voisinage OSPF perdue

%SYS-5-CONFIG_I: Configured from console by admin on vty0
# → Changement de configuration récent (à corréler avec l'incident)
```

> [!warning] Corrélation temporelle Toujours vérifier l'horodatage des logs pour identifier si les erreurs correspondent au moment où le problème est apparu.

#### Isolement par test de connectivité bidirectionnelle

Tester la connectivité dans les deux sens peut révéler des problèmes asymétriques.

```bash
# Test A → B
RouterA# ping 172.16.0.1

# Test B → A
RouterB# ping 192.168.1.1

# Si A→B fonctionne mais pas B→A :
# → Problème de routage asymétrique
# → Route de retour manquante sur RouterB
# → ACL bloquant le trafic dans un sens uniquement
```

#### Isolement par capture de paquets

Sur certains équipements ou avec des outils externes (Wireshark sur PC), analyser les paquets.

```bash
# Sur un routeur Cisco (fonctionnalité limitée)
Router# debug ip packet
Router# debug ip icmp

# Analyser :
# - Les paquets arrivent-ils sur l'interface d'entrée ?
# - Sont-ils routés vers l'interface de sortie ?
# - Sont-ils bloqués par une ACL ?

# IMPORTANT : Désactiver après usage
Router# undebug all
```

### Isolement des problèmes courants

#### Problème : Pas de connectivité réseau

**Méthode d'isolement :**

```bash
# 1. Vérifier la couche physique
Router# show ip interface brief
# Si "down/down" → problème physique

# 2. Vérifier l'adressage IP
Router# show running-config interface GigabitEthernet0/0
# Vérifier IP et masque corrects

# 3. Vérifier la table de routage
Router# show ip route
# La route vers la destination existe-t-elle ?

# 4. Vérifier les ACL
Router# show ip access-lists
Router# show ip interface GigabitEthernet0/0 | include access list
# Une ACL bloque-t-elle le trafic ?

# 5. Tester avec ping étendu
Router# ping 172.16.0.1 source GigabitEthernet0/0
# Forcer l'interface source pour tester un chemin spécifique
```

#### Problème : Routage ne fonctionne pas

**Méthode d'isolement :**

```bash
# 1. Vérifier que le routage IP est activé
Router# show ip protocols
# Doit afficher les protocoles de routage actifs

# 2. Vérifier les voisinages (pour routage dynamique)
Router# show ip ospf neighbor
Router# show ip eigrp neighbors
# Les voisins sont-ils présents ?

# 3. Vérifier les routes apprises
Router# show ip route ospf
Router# show ip route eigrp
# Les routes attendues sont-elles présentes ?

# 4. Comparer avec un routeur fonctionnel
Router# show ip route  # Sur routeur OK
Router# show ip route  # Sur routeur problématique
# Identifier les routes manquantes
```

#### Problème : Connexion intermittente

**Méthode d'isolement :**

```bash
# 1. Ping prolongé pour identifier la fréquence
Router# ping 172.16.0.1 repeat 1000
# Analyser le taux de perte et les patterns

# 2. Vérifier les compteurs d'erreurs
Router# show interfaces GigabitEthernet0/0 | include error|drop|collision
# Des erreurs augmentent-elles en continu ?

# 3. Monitorer les logs en temps réel
Router# terminal monitor
# Observer si des messages apparaissent lors des pertes de connectivité

# 4. Vérifier la charge CPU et mémoire
Router# show processes cpu history
Router# show memory statistics
# Une charge élevée peut causer des pertes intermittentes
```

### Documentation de l'isolement

> [!info] Template de documentation Lors de l'isolement d'un problème, documentez :
> 
> **1. Tests effectués**
> 
> - Commandes exécutées
> - Résultats obtenus
> - Interprétation
> 
> **2. Éléments éliminés**
> 
> - Composants vérifiés et fonctionnels
> - Pourquoi ils ont été écartés
> 
> **3. Périmètre restant**
> 
> - Où le problème est localisé
> - Hypothèses restantes
> 
> **4. Prochaines étapes**
> 
> - Tests à effectuer ensuite
> - Actions correctives envisagées

### Matrice de décision pour l'isolement

```
┌─────────────────────────────────────────────────────────┐
│         MATRICE D'ISOLEMENT DE PROBLÈME                 │
└─────────────────────────────────────────────────────────┘

Ping local (même sous-réseau) ?
├─ OUI → Ping gateway ?
│   ├─ OUI → Ping destination distante ?
│   │   ├─ OUI → Problème applicatif (Couches 4-7)
│   │   └─ NON → Problème de routage (Couche 3)
│   └─ NON → Problème de gateway
│       └─ Vérifier configuration gateway et routes
└─ NON → Problème local (Couches 1-2)
    ├─ show interfaces → État physique
    └─ show arp → Résolution adresses
```

### Outils d'isolement rapide

#### Checklist d'isolement express

> [!tip] Checklist 5 minutes Pour isoler rapidement un problème de connectivité :
> 
> ```bash
> ☐ 1. show ip interface brief          # État global
> ☐ 2. ping <gateway>                    # Connectivité locale
> ☐ 3. ping <destination>                # Connectivité distante
> ☐ 4. show ip route                     # Routes présentes
> ☐ 5. show logging | include %ERR       # Erreurs récentes
> ```
> 
> Ces 5 commandes révèlent 90% des problèmes courants en moins de 5 minutes.

#### Commandes combinées pour isolement rapide

```bash
# Vérification complète d'une interface en une commande
Router# show interfaces GigabitEthernet0/0 | include line protocol|error|drop|collision|Input queue|Output queue

# Vérification routing + connectivité combinée
Router# show ip route 172.16.0.0 ; ping 172.16.0.1

# Vérification complète OSPF
Router# show ip ospf neighbor ; show ip route ospf ; show ip ospf interface brief
```

### Pièges courants lors de l'isolement

> [!warning] Erreurs fréquentes à éviter
> 
> **1. Tirer des conclusions hâtives**
> 
> - Ne pas assumer qu'un ping qui fonctionne = tout fonctionne
> - ICMP peut être autorisé alors que d'autres protocoles sont bloqués
> 
> **2. Négliger les problèmes multiples**
> 
> - Plusieurs problèmes peuvent coexister
> - Résoudre un problème peut en révéler un autre
> 
> **3. Modifier sans tester**
> 
> - Toujours tester après chaque modification
> - Ne jamais faire plusieurs changements simultanés
> 
> **4. Ignorer les dépendances**
> 
> - Un problème DNS peut sembler être un problème réseau
> - Un problème d'authentification peut ressembler à un problème de connectivité
> 
> **5. Oublier le contexte temporel**
> 
> - Corréler les problèmes avec les changements récents
> - Vérifier si le problème est nouveau ou préexistant

### Exemple complet d'isolement méthodique

> [!example] Cas pratique : Utilisateurs ne peuvent pas accéder à Internet
> 
> **Symptôme** : Les PC du VLAN 10 (192.168.10.0/24) ne peuvent plus accéder à Internet depuis ce matin
> 
> **Isolement étape par étape :**
> 
> ```bash
> # ÉTAPE 1 : Vérifier connectivité locale
> PC> ping 192.168.10.254  # Gateway OK ✅
> 
> # ÉTAPE 2 : Depuis le routeur, vérifier interfaces
> Router# show ip interface brief
> GigabitEthernet0/0  192.168.10.254  YES manual up    up  ✅
> GigabitEthernet0/1  203.0.113.2     YES manual up    up  ✅
> 
> # ÉTAPE 3 : Vérifier la table de routage
> Router# show ip route
> Gateway of last resort is 203.0.113.1 to network 0.0.0.0
> S*    0.0.0.0/0 [1/0] via 203.0.113.1  ✅
> C     192.168.10.0/24 is directly connected, GigabitEthernet0/0  ✅
> 
> # ÉTAPE 4 : Ping depuis le routeur vers Internet
> Router# ping 8.8.8.8
> Success rate is 100 percent  ✅
> # → Le routeur peut accéder à Internet
> 
> # ÉTAPE 5 : Ping depuis le routeur avec IP source du VLAN 10
> Router# ping 8.8.8.8 source 192.168.10.254
> Success rate is 0 percent  ❌
> # → Problème identifié : avec IP source du VLAN 10, ça ne fonctionne pas
> 
> # ÉTAPE 6 : Vérifier NAT
> Router# show ip nat translations
> # Aucune translation ! ❌
> 
> Router# show running-config | section nat
> # Aucune configuration NAT ! ❌
> 
> # CAUSE IDENTIFIÉE : Configuration NAT manquante ou supprimée
> # SOLUTION : Reconfigurer NAT pour le VLAN 10
> ```

---

## 🎓 Récapitulatif et bonnes pratiques

### Principes fondamentaux du dépannage

1. **Toujours suivre une méthodologie structurée** - Pas de dépannage "au feeling"
2. **Documenter chaque étape** - Pour soi-même et pour les autres
3. **Ne changer qu'une seule variable à la fois** - Pour identifier la cause exacte
4. **Toujours sauvegarder avant de modifier** - Pour pouvoir revenir en arrière
5. **Vérifier la solution** - S'assurer que le problème est vraiment résolu

### Approche recommandée selon le contexte

|Situation|Approche recommandée|
|---|---|
|**Nouveau déploiement**|Ascendante (bottom-up)|
|**Panne complète**|Ascendante (bottom-up)|
|**Problème applicatif**|Descendante (top-down)|
|**Problème de performance**|Diviser pour régner|
|**Formation/Apprentissage**|Ascendante (bottom-up)|
|**Dépannage rapide**|Diviser pour régner|

### Commandes essentielles à maîtriser

```bash
# Top 10 des commandes de dépannage
1. show ip interface brief          # Vue d'ensemble rapide
2. show interfaces                  # Détails interface
3. show ip route                    # Table de routage
4. ping                            # Test connectivité
5. traceroute                      # Tracer le chemin
6. show running-config             # Configuration active
7. show logging                    # Logs système
8. show ip protocols               # Protocoles de routage
9. show ip access-lists            # ACL appliquées
10. show version                   # Infos système
```

### Workflow de dépannage universel

```
┌─────────────────────────────────────────────────────────┐
│              WORKFLOW DE DÉPANNAGE                      │
└─────────────────────────────────────────────────────────┘

1. DÉFINIR LE PROBLÈME
   ├─ Symptômes exacts
   ├─ Périmètre affecté
   └─ Moment d'apparition

2. COLLECTER LES INFORMATIONS
   ├─ État des équipements
   ├─ Configurations
   ├─ Logs et historique
   └─ Topologie réseau

3. ANALYSER LES DONNÉES
   ├─ Identifier les anomalies
   ├─ Corréler les événements
   └─ Formuler des hypothèses

4. ISOLER LA CAUSE
   ├─ Tester segment par segment
   ├─ Éliminer ce qui fonctionne
   └─ Localiser le composant défaillant

5. RÉSOUDRE
   ├─ Appliquer la correction
   ├─ Vérifier le retour à la normale
   └─ Documenter la solution

6. SUIVRE
   ├─ Monitorer la stabilité
   ├─ Mettre à jour la documentation
   └─ Analyser la cause racine
```

### Erreurs à éviter absolument

> [!warning] Les 10 erreurs fatales du dépannage
> 
> 1. **Modifier sans sauvegarder** - Risque de perdre la configuration fonctionnelle
> 2. **Ne pas documenter** - Impossible de tracer ce qui a été fait
> 3. **Changer plusieurs paramètres en même temps** - On ne sait plus ce qui a fonctionné
> 4. **Ignorer les logs** - Perdre des indices précieux
> 5. **Ne pas vérifier les bases** - Couche 1 oubliée = temps perdu
> 6. **Faire confiance aux autres sans vérifier** - "Personne n'a touché" ≠ réalité
> 7. **Utiliser debug en production sans précaution** - Risque de surcharge CPU
> 8. **Tester en heures de pointe** - Maximise l'impact en cas d'erreur
> 9. **Ne pas avoir de plan de retour arrière** - Si ça empire, comment revenir ?
> 10. **Négliger la communication** - Informer les utilisateurs et la hiérarchie

### Mindset du bon dépanneur

> [!tip] Qualités essentielles
> 
> **🧠 Méthodique** - Suivre un processus structuré **🔍 Observateur** - Ne rien tenir pour acquis **📝 Rigoureux** - Documenter systématiquement **🤔 Analytique** - Comprendre le "pourquoi" **💪 Patient** - Le dépannage prend du temps **🎯 Persévérant** - Ne pas abandonner face à la complexité **🗣️ Communicatif** - Expliquer clairement le problème et la solution

### Ressources pour aller plus loin

Le dépannage est une compétence qui s'améliore avec l'expérience. Chaque problème résolu enrichit votre base de connaissances et vous rend plus efficace pour les suivants.

> [!info] Développer ses compétences
> 
> - **Pratiquer régulièrement** dans un lab
> - **Documenter tous les incidents** résolus
> - **Créer une base de connaissances** personnelle
> - **Analyser les post-mortems** d'incidents majeurs
> - **Simuler des pannes** pour s'entraîner
> - **Partager les solutions** avec la communauté

---

**📌 Points clés à retenir :**

✅ Le dépannage est un processus structuré, pas une improvisation ✅ Le modèle OSI est votre meilleur allié pour comprendre où chercher ✅ La collecte d'informations est l'étape la plus importante ✅ L'isolement méthodique fait gagner un temps considérable ✅ Documenter aujourd'hui = gagner du temps demain

---

_Fin du module : Dépannage et méthodologie - Configuration routeur Cisco_