

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

## 🎯 Rôle et fonctions d'un routeur

### Définition et rôle principal

Un routeur est un équipement réseau de **couche 3 (réseau)** du modèle OSI qui permet d'interconnecter différents réseaux IP entre eux. Son rôle principal est de **déterminer le meilleur chemin** pour acheminer les paquets de données d'un réseau source vers un réseau de destination.

> [!info] Distinction routeur vs switch
> 
> - **Switch** : Fonctionne en couche 2, connecte des équipements au sein d'un même réseau local (LAN)
> - **Routeur** : Fonctionne en couche 3, interconnecte différents réseaux entre eux (LAN-to-LAN, LAN-to-WAN)

### Fonctions principales

#### 1. 🔀 Routage des paquets

Le routeur analyse l'**adresse IP de destination** de chaque paquet et consulte sa **table de routage** pour déterminer l'interface de sortie appropriée.

**Processus de décision :**

- Réception du paquet sur une interface
- Extraction de l'adresse IP de destination
- Consultation de la table de routage
- Sélection de la meilleure route (selon la métrique)
- Transmission du paquet vers l'interface de sortie

> [!example] Exemple concret Un paquet destiné à 192.168.10.5 arrive sur le routeur. Celui-ci consulte sa table et détermine que ce réseau (192.168.10.0/24) est accessible via l'interface GigabitEthernet0/1.

#### 2. 🛡️ Segmentation des domaines de broadcast

Chaque interface d'un routeur constitue un **domaine de broadcast distinct**. Cela limite la propagation des broadcasts et améliore les performances du réseau.

**Avantages :**

- Réduction du trafic inutile
- Amélioration de la sécurité (isolation des réseaux)
- Meilleure gestion de la bande passante

#### 3. 🔄 Conversion de protocoles

Le routeur peut interconnecter des réseaux utilisant des technologies différentes :

- Ethernet ↔ Série (WAN)
- Ethernet ↔ Fibre optique
- IPv4 ↔ IPv6 (avec mécanismes de transition)

#### 4. 🛠️ Services réseau additionnels

Les routeurs Cisco offrent des fonctionnalités avancées :

|Fonction|Description|Cas d'usage|
|---|---|---|
|**NAT/PAT**|Translation d'adresses|Partage d'une IP publique|
|**DHCP**|Attribution automatique d'IP|Configuration automatique clients|
|**ACL**|Filtrage de trafic|Sécurité, contrôle d'accès|
|**QoS**|Priorisation du trafic|VoIP, applications critiques|
|**VPN**|Tunnels sécurisés|Interconnexion sites distants|

> [!warning] Attention aux performances Plus vous activez de services sur un routeur, plus sa charge CPU augmente. Dimensionnez votre matériel en conséquence.

---

## 🖥️ Modèles Cisco courants

### Gamme ISR (Integrated Services Router)

Les routeurs ISR sont les routeurs **modulaires** de Cisco destinés aux entreprises, offrant haute performance et évolutivité.

#### ISR 4000 Series (génération actuelle)

**Caractéristiques principales :**

- Architecture moderne multi-cœurs
- Performances de 100 Mbps à 4 Gbps
- Support IPv6 natif
- Virtualisation intégrée (possibilité d'héberger des machines virtuelles)

**Modèles courants :**

|Modèle|Débit WAN|Utilisation typique|Particularités|
|---|---|---|---|
|**ISR 4221**|Jusqu'à 300 Mbps|PME (50-100 utilisateurs)|Compact, 2 ports GE|
|**ISR 4321**|Jusqu'à 500 Mbps|Moyennes entreprises|3 ports GE, slots modulaires|
|**ISR 4331**|Jusqu'à 1 Gbps|Grandes entreprises|3 ports GE, haute performance|
|**ISR 4451**|Jusqu'à 4 Gbps|Datacenter, campus|4 ports GE, très haute capacité|

> [!tip] Astuce de sélection Pour choisir le bon modèle, calculez votre besoin en bande passante WAN et ajoutez 30-50% de marge pour anticiper la croissance.

#### ISR 2900/3900 Series (ancienne génération, encore très répandue)

Ces modèles restent largement utilisés et sont excellents pour l'apprentissage.

**Série 2900 :**

- **2901** : 2 ports GE intégrés, 2-4 slots
- **2911** : 3 ports GE intégrés, 4 slots
- **2921** : 3 ports GE intégrés, 4 slots (plus performant)

**Série 3900 :**

- **3925E** : 3 ports GE, 4 slots, performances élevées
- **3945E** : 3 ports GE, 4 slots, très haute performance

> [!info] Notation des modèles Le "E" dans "3925E" signifie "Enhanced" (performances améliorées). Les modèles sans "E" sont des versions standard.

### Autres gammes importantes

#### Routeurs Branch (petites agences)

- **ISR 1000 Series** : Routeurs SD-WAN compacts
- **Cisco 800 Series** : Très petites entreprises, SOHO

#### Routeurs d'agrégation/cœur

- **ASR 1000 Series** : Agrégation, fournisseurs d'accès
- **ASR 9000 Series** : Cœur de réseau, opérateurs

> [!warning] IOS vs IOS XE
> 
> - Anciens modèles (2900/3900) : **Cisco IOS** classique
> - Nouveaux modèles (ISR 4000) : **Cisco IOS XE** (architecture Linux modernisée) Les commandes de base restent identiques, mais certaines fonctionnalités avancées diffèrent.

---

## 🔧 Composants physiques

### Architecture matérielle d'un routeur

Un routeur Cisco se compose de plusieurs éléments matériels essentiels.

#### 1. 💾 Types de mémoires

|Mémoire|Acronyme|Fonction|Persistante ?|Contenu typique|
|---|---|---|---|---|
|**ROM**|Read-Only Memory|Stocke le bootstrap et POST|✅ Oui|Programme de démarrage, diagnostics|
|**Flash**|Flash Memory|Stocke l'IOS|✅ Oui|Image IOS (ex: c2900-universalk9-mz.SPA.157-3.M6.bin)|
|**NVRAM**|Non-Volatile RAM|Stocke la configuration|✅ Oui|startup-config (configuration de démarrage)|
|**RAM**|Random Access Memory|Mémoire de travail|❌ Non|running-config, table de routage, cache ARP|

> [!info] Processus de démarrage
> 
> 1. **POST** (Power-On Self Test) depuis ROM
> 2. Chargement du **bootstrap** depuis ROM
> 3. Localisation et chargement de l'**IOS** depuis Flash vers RAM
> 4. Chargement de la **startup-config** depuis NVRAM vers RAM (devient running-config)

#### 2. 🔌 Processeur et composants

**CPU (Central Processing Unit) :**

- Traite les paquets
- Exécute les processus de routage
- Gère les protocoles
- Performance critique pour le débit du routeur

**Exemples de commandes pour vérifier les ressources :**

```bash
# Afficher l'utilisation CPU
Router# show processes cpu

# Afficher l'utilisation mémoire
Router# show memory

# Afficher les détails du matériel
Router# show version
```

> [!warning] Utilisation CPU élevée Une utilisation CPU constamment >75% indique soit un problème de configuration (boucle de routage, ACL inefficaces) soit un sous-dimensionnement matériel.

### Interfaces physiques

#### Types d'interfaces courantes

##### 🌐 Interfaces Ethernet

**FastEthernet (FE) :** 100 Mbps

- Notation : `FastEthernet0/0`, `Fa0/0`
- Ancienne norme, encore présente sur vieux équipements

**GigabitEthernet (GE) :** 1 Gbps

- Notation : `GigabitEthernet0/0`, `Gi0/0` ou `GigabitEthernet0/0/0`
- Standard actuel pour les connexions LAN

**10GigabitEthernet :** 10 Gbps

- Notation : `TenGigabitEthernet0/0`, `Te0/0`
- Utilisé pour liens haute capacité

> [!example] Décryptage de la notation `GigabitEthernet0/0/1` :
> 
> - **0** = Slot du routeur (position du module)
> - **0** = Numéro du sous-module
> - **1** = Numéro du port sur le module

##### 📡 Interfaces série (WAN)

Utilisées pour connexions WAN longue distance.

**Types de connecteurs :**

- **Serial** : Interface série synchrone standard
- **T1/E1** : Connexions téléphoniques numériques
- **BRI/PRI (ISDN)** : Lignes RNIS (moins courantes aujourd'hui)

**Notation :** `Serial0/0/0` ou `Se0/0/0`

> [!info] DCE vs DTE Sur les liens série, un équipement fournit l'horloge (**DCE** - Data Communication Equipment), l'autre la reçoit (**DTE** - Data Terminal Equipment). En production, c'est le fournisseur d'accès (modem, CSU/DSU) qui est DCE. En lab, vous devez définir manuellement qui fournit l'horloge.

```bash
# Vérifier le type de câble (DCE ou DTE)
Router# show controllers serial 0/0/0

# Si DCE, définir la vitesse d'horloge (obligatoire)
Router(config)# interface serial 0/0/0
Router(config-if)# clock rate 64000
```

##### 🔌 Interfaces auxiliaires et console

**Console (CON) :**

- Port RJ45 bleu
- Connexion locale pour configuration initiale
- Câble console (RJ45-to-DB9 ou RJ45-to-USB)

**Auxiliary (AUX) :**

- Port pour modem externe
- Gestion à distance via ligne téléphonique
- Peu utilisé aujourd'hui

### Slots et modules d'extension

#### Architecture modulaire

Les routeurs ISR permettent l'ajout de modules pour étendre les capacités.

**Types de slots :**

|Type|Description|Utilisation|
|---|---|---|
|**EHWIC**|Enhanced High-Speed WAN Interface Card|Cartes WAN (Serial, T1/E1, ADSL)|
|**SM**|Service Module|Modules services (Switch intégré, VoIP)|
|**ISM**|Internal Service Module|Services intégrés (pare-feu, IPS)|
|**NIM**|Network Interface Module|Interfaces réseau (utilisé sur ISR 4000)|

> [!tip] Vérifier les modules installés
> 
> ```bash
> Router# show diag
> # ou
> Router# show inventory
> ```

#### Exemples de modules populaires

- **EHWIC-4ESG** : 4 ports GigabitEthernet supplémentaires
- **EHWIC-1GE-SFP** : 1 port GE avec slot SFP (fibre)
- **NIM-2GE-CU-SFP** : 2 ports GE combo (cuivre ou fibre)

---

## 🔗 Types de connexions

### Vue d'ensemble des méthodes d'accès

Un routeur Cisco peut être configuré et géré via plusieurs méthodes, chacune adaptée à un contexte spécifique.

|Méthode|Accès|Sécurité|Cas d'usage|
|---|---|---|---|
|**Console**|Local, câble série|⭐⭐⭐⭐⭐|Configuration initiale, dépannage|
|**Telnet**|Réseau (TCP 23)|⭐|⚠️ **Obsolète** (clair)|
|**SSH**|Réseau (TCP 22)|⭐⭐⭐⭐⭐|Gestion à distance sécurisée|
|**HTTP/HTTPS**|Navigateur web|⭐⭐⭐|Interface graphique (moins courant)|

### 1. 🖥️ Connexion console

#### Caractéristiques

La connexion console est le mode d'accès **le plus fondamental** et **le plus fiable** :

- **Accès physique direct** au routeur
- **Aucune configuration réseau** nécessaire
- Fonctionne même si le routeur est **complètement vierge**
- Utilisée pour la **configuration initiale** et le **dépannage critique**

#### Matériel nécessaire

**Câble console Cisco :**

- Côté routeur : RJ45 (port console bleu)
- Côté PC :
    - Ancien : DB9 (port série)
    - Moderne : USB (avec adaptateur USB-to-Serial)

> [!tip] Câble console universel Les câbles "console USB" Cisco modernes (bleu clair) intègrent directement la puce USB-to-Serial et sont plug-and-play sous Windows/macOS/Linux.

#### Configuration du logiciel terminal

**Logiciels recommandés :**

- **Windows** : PuTTY, TeraTerm, SecureCRT
- **macOS/Linux** : Screen, Minicom, PuTTY
- **Multi-plateforme** : SecureCRT (payant mais très complet)

**Paramètres de connexion série :**

```
Bits par seconde (Baud rate) : 9600
Bits de données : 8
Parité : Aucune (None)
Bits d'arrêt : 1
Contrôle de flux : Aucun (None)
```

> [!warning] Paramètres obligatoires Ces paramètres sont standards et **doivent être respectés exactement**. Tout écart empêchera la communication (vous verrez des caractères illisibles ou rien du tout).

**Exemple avec PuTTY (Windows) :**

1. Sélectionnez "Serial"
2. Port COM : `COM3` (vérifiez dans Gestionnaire de périphériques)
3. Speed : `9600`
4. Cliquez sur "Open"

**Exemple avec Screen (macOS/Linux) :**

```bash
# Identifier le port
ls /dev/tty.*                    # macOS
ls /dev/ttyUSB*                  # Linux

# Se connecter
screen /dev/tty.usbserial 9600   # macOS
screen /dev/ttyUSB0 9600         # Linux

# Quitter screen : Ctrl+A puis K
```

#### Avantages et limitations

**✅ Avantages :**

- Accès garanti, même si configuration réseau corrompue
- Sécurité maximale (accès physique requis)
- Affichage des messages de démarrage (logs boot)
- Pas besoin d'authentification pour la première connexion

**❌ Limitations :**

- Nécessite présence physique
- Un seul utilisateur à la fois
- Vitesse limitée (9600 bps)

### 2. 📞 Connexion Telnet

#### Fonctionnement

Telnet est un protocole de **terminal virtuel** permettant l'accès distant via réseau.

**Caractéristiques techniques :**

- Protocole : **TCP port 23**
- Transmission : **texte clair** (non chiffré)
- Standard : RFC 854

> [!warning] Sécurité critique **N'utilisez JAMAIS Telnet en production !** Toutes les données (y compris mots de passe) transitent en clair sur le réseau. Utilisez SSH à la place.

#### Cas d'usage (rares)

Telnet peut être acceptable **uniquement** dans ces situations :

- Réseau de **laboratoire isolé**
- Dépannage rapide dans environnement **100% contrôlé**
- Équipement ancien ne supportant **pas SSH**

#### Configuration basique Telnet

```bash
# Activer le mot de passe sur les lignes VTY
Router(config)# line vty 0 4
Router(config-line)# password cisco123
Router(config-line)# login
Router(config-line)# exit

# Se connecter depuis un PC
C:\> telnet 192.168.1.1
```

> [!info] Lignes VTY Les lignes **VTY** (Virtual TeletYpe) sont des lignes de terminal virtuel. `line vty 0 4` signifie que 5 connexions simultanées (numérotées 0 à 4) sont possibles. Certains modèles supportent `vty 0 15` (16 connexions).

### 3. 🔐 Connexion SSH (Secure Shell)

#### Pourquoi SSH ?

SSH est le **standard moderne** pour l'administration à distance sécurisée.

**Avantages clés :**

- **Chiffrement complet** de la session (authentification + données)
- Authentification forte (mot de passe + clés publiques)
- Protection contre l'interception (man-in-the-middle)
- Tunnel sécurisé pour d'autres protocoles (SCP, SFTP)

**Versions :**

- **SSH v1** : Obsolète, vulnérable, ne plus utiliser
- **SSH v2** : Standard actuel, utilisez toujours cette version

#### Prérequis pour activer SSH

SSH nécessite plusieurs éléments configurés sur le routeur :

1. **Nom d'hôte** défini (hostname)
2. **Nom de domaine** défini (ip domain-name)
3. **Clés RSA** générées
4. **Authentification locale** configurée (base de données utilisateurs)

> [!info] Pourquoi ces prérequis ?
> 
> - Le **hostname + domain-name** forment le FQDN utilisé comme identifiant pour générer les clés
> - Les **clés RSA** sont nécessaires pour établir le tunnel chiffré
> - L'**authentification locale** permet de définir qui peut se connecter

#### Configuration complète SSH

```bash
# Étape 1 : Définir hostname et domain-name
Router(config)# hostname R1
R1(config)# ip domain-name entreprise.local

# Étape 2 : Générer les clés RSA
R1(config)# crypto key generate rsa
# Quand demandé, entrez 2048 bits minimum (1024 minimum, 2048 recommandé)
How many bits in the modulus [512]: 2048

# Étape 3 : Créer un compte utilisateur local
R1(config)# username admin privilege 15 secret Cisco@2024
# privilege 15 = droits administrateur complets
# secret = chiffrement fort (au lieu de password)

# Étape 4 : Configurer les lignes VTY
R1(config)# line vty 0 15
R1(config-line)# transport input ssh     # Autoriser UNIQUEMENT SSH
R1(config-line)# login local             # Utiliser base de données locale
R1(config-line)# exit

# Étape 5 (Optionnel) : Forcer SSH version 2
R1(config)# ip ssh version 2

# Étape 6 (Optionnel) : Configurer timeout et tentatives
R1(config)# ip ssh time-out 60           # Timeout connexion (secondes)
R1(config)# ip ssh authentication-retries 3
```

> [!tip] Bonnes pratiques SSH
> 
> - Utilisez toujours `secret` au lieu de `password` (algorithme MD5 vs plaintext)
> - Définissez `transport input ssh` (et PAS `transport input all` qui autoriserait Telnet)
> - Créez des comptes utilisateur distincts (évitez le compte unique "admin")
> - Utilisez des mots de passe complexes (12+ caractères, majuscules, chiffres, symboles)

#### Connexion SSH depuis un client

**Depuis Windows (PuTTY) :**

1. Type de connexion : SSH
2. Hostname : IP du routeur (ex: 192.168.1.1)
3. Port : 22
4. Cliquez "Open"
5. Entrez username/password

**Depuis Linux/macOS (terminal) :**

```bash
# Connexion simple
ssh admin@192.168.1.1

# Connexion avec port personnalisé
ssh -p 2222 admin@192.168.1.1

# Première connexion : accepter la clé (fingerprint)
The authenticity of host '192.168.1.1' can't be established.
RSA key fingerprint is SHA256:xxxxx...
Are you sure you want to continue connecting (yes/no)? yes
```

#### Vérification et dépannage SSH

```bash
# Vérifier la configuration SSH
R1# show ip ssh

# Vérifier les connexions SSH actives
R1# show ssh

# Vérifier les utilisateurs connectés
R1# show users

# Vérifier les clés crypto générées
R1# show crypto key mypubkey rsa
```

> [!warning] Erreur courante : clés RSA manquantes Si vous obtenez l'erreur "Please create RSA keys", c'est que vous n'avez pas exécuté `crypto key generate rsa`. Sans clés, SSH ne peut pas fonctionner.

### 4. 🌐 Connexion HTTP/HTTPS

#### Présentation

Cisco propose une interface web (HTTP/HTTPS) pour la gestion graphique des routeurs, notamment via **Cisco Configuration Professional (CCP)** ou **Cisco Prime Infrastructure**.

**Caractéristiques :**

- Interface graphique (GUI) dans le navigateur
- Moins utilisée que CLI en environnement professionnel
- Utile pour débutants ou tâches de monitoring
- Certaines configurations avancées nécessitent toujours CLI

#### Activation du serveur HTTP

```bash
# Activer le serveur HTTP (NON chiffré - éviter en production)
Router(config)# ip http server

# Activer le serveur HTTPS (chiffré - recommandé)
Router(config)# ip http secure-server

# Définir l'authentification
Router(config)# ip http authentication local

# Créer un utilisateur
Router(config)# username webadmin privilege 15 secret Web@Secure2024
```

#### Accès via navigateur

**HTTP :** `http://192.168.1.1`  
**HTTPS :** `https://192.168.1.1`

> [!warning] Certificat auto-signé Le routeur utilise un certificat auto-signé, votre navigateur affichera un avertissement de sécurité. C'est normal pour un équipement réseau, vous pouvez accepter l'exception.

#### Avantages et inconvénients

**✅ Avantages :**

- Interface conviviale pour débutants
- Visualisation graphique de la topologie
- Monitoring en temps réel (graphiques CPU, bande passante)
- Assistants de configuration guidés

**❌ Inconvénients :**

- Moins complet que CLI pour configurations avancées
- Consomme ressources routeur (CPU/mémoire)
- Moins flexible pour automatisation (scripts)
- Nécessite connectivité réseau fonctionnelle

### Comparaison et choix de la méthode

> [!tip] Quand utiliser quelle méthode ?
> 
> **Console :**
> 
> - Configuration initiale (routeur neuf)
> - Dépannage critique (routeur inaccessible par réseau)
> - Password recovery
> - Mise à jour IOS
> 
> **SSH :**
> 
> - Administration quotidienne à distance
> - Automatisation (scripts Python/Ansible)
> - Toute opération depuis le réseau
> - Production (99% des cas)
> 
> **HTTP/HTTPS :**
> 
> - Monitoring rapide
> - Configuration guidée pour débutants
> - Visualisation topologie
> 
> **Telnet :**
> 
> - ⚠️ Éviter absolument sauf lab isolé

---

## 🎓 Récapitulatif des points essentiels

### Routeur : rôle et importance

Un routeur est l'équipement fondamental qui **interconnecte les réseaux** en travaillant au **niveau 3 (IP)**. Il prend des décisions de routage intelligentes, segmente les domaines de broadcast et offre de nombreux services réseau (NAT, DHCP, sécurité).

### Choix du matériel

Les gammes **ISR 2900/3900** restent très courantes dans les entreprises existantes, tandis que les **ISR 4000** représentent la génération actuelle avec des performances et une architecture modernisées. Le choix dépend du débit WAN nécessaire et du nombre d'utilisateurs.

### Composants à connaître

Les quatre types de mémoires (**ROM, Flash, NVRAM, RAM**) jouent des rôles distincts dans le fonctionnement du routeur. Comprendre leur fonction aide à diagnostiquer les problèmes de démarrage ou de configuration.

Les interfaces physiques (**GigabitEthernet, Serial**) sont vos points de connexion vers les réseaux. Leur notation peut sembler complexe au début mais suit une logique claire : slot/sous-module/port.

### Méthodes de connexion

- **Console** : accès de base garanti, indispensable pour configuration initiale
- **SSH** : standard professionnel pour gestion à distance sécurisée
- **Telnet** : obsolète et dangereux, n'utiliser qu'en lab isolé
- **HTTP/HTTPS** : interface graphique occasionnelle, pas le cœur du métier

> [!tip] Prochaine étape Maintenant que vous connaissez le matériel et les méthodes d'accès, vous êtes prêt à vous connecter à un routeur et découvrir l'interface en ligne de commande (CLI) et les modes de configuration IOS.