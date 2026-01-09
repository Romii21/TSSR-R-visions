

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

## 🔌 Connexion via le port console

### Pourquoi le port console ?

Le port console est l'interface d'administration **locale** et **primaire** d'un routeur Cisco. C'est souvent le seul moyen d'accéder à l'équipement lors de la configuration initiale ou en cas de perte de connectivité réseau.

> [!info] Caractéristiques du port console
> 
> - **Connexion hors-bande** : ne dépend pas du réseau
> - **Accès direct** : même si le routeur n'a aucune configuration IP
> - **Toujours disponible** : fonctionne même si le routeur ne boot pas correctement
> - **Accès physique requis** : nécessite d'être sur site

### Types de connexions console

#### 1. Connexion RJ-45 (classique)

**Matériel nécessaire :**

- Câble console Cisco RJ-45 vers DB-9 (bleu clair, câble rollover)
- Adaptateur DB-9 vers USB (si nécessaire)

**Caractéristiques du câble rollover :**

- Brochage inversé (pin 1 → pin 8, pin 2 → pin 7, etc.)
- Câble plat de couleur bleu clair typiquement
- RJ-45 d'un côté, DB-9 (série) de l'autre

> [!warning] Attention Ne confondez pas le câble console (rollover) avec un câble Ethernet standard ! Le brochage est complètement différent.

#### 2. Connexion USB Mini-B/USB-C (routeurs récents)

Les routeurs modernes (ISR 4000, Catalyst 8000) possèdent souvent un port USB console.

**Avantages :**

- Plus simple : un seul câble USB standard
- Détection automatique sur la plupart des OS
- Pas besoin d'adaptateur série

**Installation du driver :**

- Windows : télécharger le driver Cisco USB Console depuis cisco.com
- Linux/Mac : reconnaissance native dans la majorité des cas

### Localisation physique du port

> [!example] Identification visuelle
> 
> - **Étiquette** : "CONSOLE" ou "CON"
> - **Couleur** : souvent bleu clair sur les anciens modèles
> - **Position** : généralement sur le panneau avant ou arrière, près des ports AUX et management

|Type de routeur|Port console typique|
|---|---|
|ISR 1900/2900/3900|RJ-45 bleu + USB Mini-B|
|ISR 4000|USB Mini-B + RJ-45|
|Catalyst 8000|USB-C + RJ-45|
|Routeurs anciens (2800, 1800)|RJ-45 uniquement|

---

## ⚙️ Paramétrage du terminal

### Choix du logiciel d'émulation terminal

Plusieurs options s'offrent à vous pour établir la connexion série :

|Logiciel|OS|Avantages|Inconvénients|
|---|---|---|---|
|**PuTTY**|Windows, Linux|Gratuit, léger, populaire|Interface basique|
|**SecureCRT**|Windows, Mac, Linux|Professionnel, scriptable|Payant|
|**Tera Term**|Windows|Gratuit, macros|Interface vieillotte|
|**Screen**|Linux, Mac|Natif, simple|Ligne de commande uniquement|
|**Minicom**|Linux|Léger, configurable|Configuration initiale complexe|

### Configuration PuTTY (Windows)

#### Étape 1 : Identifier le port COM

1. **Gestionnaire de périphériques** → **Ports (COM et LPT)**
2. Repérez "USB Serial Port (COMx)" ou "Cisco Serial"
3. Notez le numéro (ex: COM3, COM4)

> [!tip] Astuce Branchez et débranchez le câble USB pour voir quel port apparaît/disparaît dans le Gestionnaire de périphériques.

#### Étape 2 : Paramètres PuTTY

```
Session
├─ Connection type: Serial
├─ Serial line: COM3 (votre port)
└─ Speed: 9600

Category → Connection → Serial
├─ Speed (baud): 9600
├─ Data bits: 8
├─ Stop bits: 1
├─ Parity: None
└─ Flow control: None (ou XON/XOFF)
```

**Résumé des paramètres série :** `9600 8N1`

- **9600** : vitesse en bauds
- **8** : bits de données
- **N** : pas de parité (None)
- **1** : 1 bit d'arrêt

> [!warning] Paramètres critiques Ces paramètres **doivent** correspondre exactement à ceux du routeur (par défaut 9600 8N1). Sinon vous verrez des caractères illisibles ou rien du tout.

#### Étape 3 : Sauvegarder la session

1. Donnez un nom : "Cisco Console"
2. Cliquez sur **Save**
3. Vous pourrez la recharger à chaque connexion

### Configuration Screen (Linux/Mac)

```bash
# Identifier le port série
ls /dev/tty* | grep -i usb
# Exemple de résultat : /dev/ttyUSB0 (Linux) ou /dev/tty.usbserial-XXXX (Mac)

# Se connecter avec screen
screen /dev/ttyUSB0 9600

# Quitter screen
# Ctrl+A puis K (kill), puis Y pour confirmer
```

> [!tip] Permissions sous Linux Si vous obtenez "Permission denied", ajoutez votre utilisateur au groupe dialout :
> 
> ```bash
> sudo usermod -a -G dialout $USER
> # Puis déconnectez-vous et reconnectez-vous
> ```

### Configuration SecureCRT (professionnel)

```
Quick Connect
├─ Protocol: Serial
├─ Port: COM3
├─ Baud rate: 9600
├─ Data bits: 8
├─ Parity: None
├─ Stop bits: 1
└─ Flow control: None
```

**Fonctionnalités avancées de SecureCRT :**

- Logs automatiques de session
- Scripting en Python/VBScript
- Gestion multi-onglets
- Barre de boutons personnalisables avec commandes

### Problèmes courants de connexion

> [!warning] Écran noir ou pas de réponse **Solutions :**
> 
> - Appuyez sur **Entrée** plusieurs fois
> - Vérifiez que le bon port COM est sélectionné
> - Vérifiez les paramètres série (9600 8N1)
> - Essayez un autre câble console
> - Redémarrez le routeur (le boot affiche toujours quelque chose)

> [!warning] Caractères illisibles (garbage) **Cause :** Mauvaise vitesse ou paramètres série incorrects **Solution :** Vérifiez que la vitesse est bien à 9600 bauds avec 8N1

> [!warning] "Port already in use" **Cause :** Une autre application utilise le port **Solution :** Fermez tous les terminaux ouverts, ou redémarrez le PC

---

## 🚀 Premier démarrage et séquence de boot

### Vue d'ensemble du processus de boot

Lorsque vous allumez un routeur Cisco, il exécute une **séquence de démarrage** précise et structurée. Comprendre cette séquence est essentiel pour le dépannage.

```
Étapes du boot
│
├─ 1. POST (Power-On Self-Test)
│   └─ Test du matériel
│
├─ 2. Chargement du Bootstrap
│   └─ Programme minimal en ROM
│
├─ 3. Recherche et chargement de l'IOS
│   └─ Depuis Flash, TFTP, ou ROM
│
├─ 4. Recherche et chargement de la configuration
│   └─ startup-config depuis NVRAM
│
└─ 5. Mode opérationnel
    └─ User EXEC ou Setup mode
```

### Étape 1 : POST (Power-On Self-Test)

**Ce que vous voyez :**

```
System Bootstrap, Version 15.0(1r)M15, RELEASE SOFTWARE (fc1)
Technical Support: http://www.cisco.com/techsupport
Copyright (c) 2010 by cisco Systems, Inc.

Total memory size = 512 MB - On-board = 512 MB, DIMM0 = 0 MB
CISCO2901/K9 platform with 524288 Kbytes of main memory
Main memory is configured to 64/-1(On-board/DIMM0) bit mode with ECC disabled
```

> [!info] Rôle du POST
> 
> - Vérifie l'intégrité du CPU, de la mémoire, des interfaces
> - Affiche la quantité de RAM détectée
> - Identifie le modèle du routeur
> - Durée : 10-30 secondes selon le modèle

**Indicateurs :**

- ✅ POST réussi : passage à l'étape suivante
- ❌ POST échoué : message d'erreur, arrêt possible du boot

### Étape 2 : Chargement du Bootstrap

Le **bootstrap** est un petit programme stocké en **ROM** qui initialise le processus de boot.

**Rôle :**

- Localiser l'IOS (système d'exploitation)
- Charger l'IOS en mémoire RAM
- Transférer le contrôle à l'IOS

### Étape 3 : Localisation et chargement de l'IOS

Le routeur cherche l'image IOS selon l'ordre de priorité défini dans le **registre de configuration** (config-register).

#### Registre de configuration (config-register)

Valeur hexadécimale stockée en NVRAM qui contrôle le comportement du boot.

|Valeur|Comportement|
|---|---|
|**0x2102**|**Défaut** : boot normal depuis Flash|
|0x2101|Boot en mode ROMMON|
|0x2142|Ignore startup-config (récupération de mot de passe)|
|0x2100|Boot en mode ROM Monitor manuel|

```
Processus de recherche IOS

1. Flash (par défaut avec 0x2102)
   └─ boot system flash:c2900-universalk9-mz.SPA.151-4.M4.bin

2. TFTP (si configuré)
   └─ boot system tftp://192.168.1.1/ios-image.bin

3. ROM (mode de secours limité)
   └─ IOS minimal si aucune autre source disponible
```

**Messages typiques lors du chargement :**

```
Loading "flash:c2900-universalk9-mz.SPA.151-4.M4.bin" from flash
##################################################
[OK - 53325324 bytes]
```

> [!tip] Décompression de l'IOS Certaines images IOS sont compressées. Vous verrez des "######" pendant la décompression. C'est normal et peut prendre 1-3 minutes.

### Étape 4 : Chargement de la configuration

Une fois l'IOS chargé, le routeur cherche le fichier **startup-config** en NVRAM.

#### Scénario 1 : startup-config existe

```
Loading network-confg from nvram
[OK - 1821 bytes]

Router>
```

Le routeur démarre avec la configuration sauvegardée et entre en **mode User EXEC** (`Router>`).

#### Scénario 2 : startup-config n'existe pas (premier boot)

```
--- System Configuration Dialog ---

Would you like to enter the initial configuration dialog? [yes/no]: 
```

**Setup mode** (assistant de configuration) :

- Proposé lors du premier démarrage
- Permet une configuration de base guidée
- **Recommandation** : tapez `no` et configurez manuellement (plus de contrôle)

> [!tip] Bonnes pratiques Les administrateurs expérimentés répondent toujours **no** au setup mode et préfèrent configurer manuellement en ligne de commande pour un contrôle total.

### Étape 5 : Mode opérationnel

Le routeur est prêt. Vous êtes en **mode User EXEC**.

```
Router>
```

**Prompt :** Le nom du routeur suivi de `>`

Pour accéder au mode privilégié :

```bash
Router> enable
Router#
```

Le prompt change en `#` (mode privilégié / Privileged EXEC).

### Temps de boot typiques

|Modèle|Temps de boot|
|---|---|
|ISR 1900|2-3 minutes|
|ISR 2900|3-5 minutes|
|ISR 4000|4-6 minutes|
|Anciens modèles (2800)|3-4 minutes|

> [!warning] Patience ! Ne débranchez **JAMAIS** un routeur pendant le boot. Cela peut corrompre le système de fichiers Flash et nécessiter une réinstallation complète de l'IOS.

### Messages de boot complets (exemple)

```
System Bootstrap, Version 15.0(1r)M15, RELEASE SOFTWARE (fc1)
Technical Support: http://www.cisco.com/techsupport
Copyright (c) 2010 by cisco Systems, Inc.

Total memory size = 512 MB - On-board = 512 MB

Cisco IOS Software, C2900 Software (C2900-UNIVERSALK9-M), Version 15.1(4)M4
Technical Support: http://www.cisco.com/techsupport
Copyright (c) 1986-2012 by Cisco Systems, Inc.

Cisco CISCO2901/K9 (revision 1.0) with 487424K/36864K bytes of memory.
Processor board ID FTX1234567
3 Gigabit Ethernet interfaces
1 terminal line
DRAM configuration is 64 bits wide with parity disabled.
255K bytes of non-volatile configuration memory.
250880K bytes of ATA System CompactFlash 0 (Read/Write)

Press RETURN to get started!

Router>
```

---

## 🔧 Mode ROMMON et récupération

### Qu'est-ce que le mode ROMMON ?

**ROMMON** (ROM Monitor) est un mode de diagnostic de bas niveau, équivalent au BIOS d'un PC. C'est une interface minimale stockée en **ROM** qui permet d'intervenir quand l'IOS ne peut pas démarrer.

> [!info] Quand utiliser ROMMON ?
> 
> - **Récupération de mot de passe** oublié
> - **Réinstallation de l'IOS** après corruption
> - **Changement du config-register** en urgence
> - **Diagnostic matériel** avancé
> - **Boot depuis TFTP** si Flash corrompue

### Accéder au mode ROMMON

#### Méthode 1 : Interruption du boot (la plus courante)

1. **Redémarrez** le routeur
2. **Dans les 60 premières secondes**, appuyez sur **Ctrl+Break** (ou Ctrl+Pause)
3. Le routeur interrompt le boot et entre en ROMMON

```
System Bootstrap, Version 15.0(1r)M15, RELEASE SOFTWARE (fc1)
...
monitor: command "boot" aborted due to user interrupt

rommon 1 >
```

> [!warning] Timing critique Vous devez appuyer sur Ctrl+Break **pendant le POST**, avant que l'IOS ne commence à charger. Si vous êtes trop tard, le routeur continue de booter normalement.

> [!tip] Combinaisons selon les logiciels
> 
> - **PuTTY** : Ctrl+Break fonctionne directement
> - **SecureCRT** : Ctrl+Break ou configurer "Send Break" dans les options
> - **Tera Term** : Alt+B pour envoyer un signal Break
> - **Screen** : Ctrl+A puis F

#### Méthode 2 : Config-register à 0x2100 ou 0x2101

Si configuré en avance, le routeur boot directement en ROMMON.

```bash
Router(config)# config-register 0x2101
Router(config)# exit
Router# reload
```

Au prochain démarrage → mode ROMMON automatiquement.

### Prompt ROMMON

```
rommon 1 >
rommon 2 >
rommon 3 >
```

Le chiffre incrémente à chaque commande saisie.

### Commandes ROMMON essentielles

#### Afficher les variables d'environnement

```bash
rommon 1 > set
PS1=rommon ! > 
BOOTLDR=
RET_2_RCALTS=
?=0
RANDOM_NUM=123456
CONFIG_FILE=
BOOTLDR_UPGRADE_ATTEMPT=0
```

**Variables importantes :**

- `BOOT` : chemin de l'image IOS à charger
- `CONFIG_FILE` : chemin du fichier de configuration
- `BAUD` : vitesse console (défaut 9600)

#### Modifier le config-register

```bash
rommon 2 > confreg 0x2142
```

Définit le registre à 0x2142 (ignore startup-config au prochain boot).

**Confirmation :**

```
You must reset or power cycle for new config to take effect.
```

> [!example] Cas d'usage : Récupération de mot de passe En bootant avec 0x2142, le routeur ignore la configuration et vous pouvez accéder sans mot de passe.

#### Réinitialiser le routeur

```bash
rommon 3 > reset
```

Équivalent à un redémarrage.

#### Booter manuellement depuis Flash

```bash
rommon 4 > boot flash:c2900-universalk9-mz.SPA.151-4.M4.bin
```

Force le boot d'une image IOS spécifique.

#### Lister le contenu de la Flash

```bash
rommon 5 > dir flash:
```

Affiche les fichiers disponibles en mémoire Flash.

#### Obtenir de l'aide

```bash
rommon 6 > ?

alias               set and display aliases command
boot                boot up an external process
break               set/show/clear the breakpoint
confreg             configuration register utility
cont                continue executing a downloaded image
...
```

### Scénario : Récupération de mot de passe

#### Étape par étape

**1. Entrer en ROMMON**

- Redémarrez le routeur
- Appuyez sur **Ctrl+Break** pendant le POST

**2. Changer le config-register**

```bash
rommon 1 > confreg 0x2142
rommon 2 > reset
```

**3. Le routeur redémarre et ignore startup-config**

```
Would you like to enter the initial configuration dialog? [yes/no]: no
Router>
```

**4. Entrer en mode privilégié et copier startup vers running**

```bash
Router> enable
Router# copy startup-config running-config
Destination filename [running-config]? [Entrée]
```

**5. Changer le mot de passe**

```bash
Router# configure terminal
Router(config)# enable secret NouveauMotDePasse123
Router(config)# exit
```

**6. Restaurer le config-register par défaut**

```bash
Router# configure terminal
Router(config)# config-register 0x2102
Router(config)# exit
```

**7. Sauvegarder et redémarrer**

```bash
Router# copy running-config startup-config
Router# reload
```

> [!warning] Sécurité physique Cette procédure montre l'importance de **sécuriser physiquement** vos équipements. Toute personne ayant accès au port console peut réinitialiser le mot de passe !

### Scénario : Réinstallation de l'IOS via TFTP

Si la Flash est corrompue ou l'IOS supprimé, vous pouvez charger un nouveau système via le réseau depuis ROMMON.

**Prérequis :**

- Un serveur TFTP sur le réseau
- L'image IOS disponible sur ce serveur
- Une connexion réseau fonctionnelle (câble Ethernet branché)

**Commandes ROMMON :**

```bash
rommon 1 > IP_ADDRESS=192.168.1.10
rommon 2 > IP_SUBNET_MASK=255.255.255.0
rommon 3 > DEFAULT_GATEWAY=192.168.1.1
rommon 4 > TFTP_SERVER=192.168.1.100
rommon 5 > TFTP_FILE=c2900-universalk9-mz.SPA.151-4.M4.bin

rommon 6 > tftpdnld
```

Le routeur télécharge l'IOS via TFTP :

```
Receiving c2900-universalk9-mz.SPA.151-4.M4.bin from 192.168.1.100
!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
Download Complete!
```

**Ensuite, copier en Flash et booter :**

```bash
rommon 7 > boot
```

> [!tip] Sauvegarde préventive Gardez toujours une copie de votre image IOS sur un serveur TFTP ou un PC pour ce type de situation d'urgence.

### Sortir du mode ROMMON

Pour sortir de ROMMON et continuer le boot normal :

```bash
rommon 1 > boot
```

Ou si vous voulez restaurer le config-register et redémarrer :

```bash
rommon 1 > confreg 0x2102
rommon 2 > reset
```

### Pièges courants en ROMMON

> [!warning] Pas de touche Backspace Dans ROMMON, la touche Backspace peut ne pas fonctionner correctement. Si vous faites une erreur de frappe :
> 
> - Utilisez **Ctrl+U** pour effacer toute la ligne
> - Ou retapez toute la commande

> [!warning] Sensibilité à la casse Les commandes et noms de fichiers sont **sensibles à la casse** en ROMMON. `BOOT` ≠ `boot`.

> [!warning] Pas d'auto-complétion Contrairement au CLI IOS normal, il n'y a **pas de touche Tab** pour l'auto-complétion. Vous devez taper les noms complets.

---

## 🎯 Résumé des points clés

### Connexion console

- Port RJ-45 ou USB selon le modèle
- Paramètres série : **9600 8N1**
- Logiciels : PuTTY, SecureCRT, Screen, Tera Term
- Câble rollover pour RJ-45, USB standard pour USB console

### Séquence de boot

1. **POST** → test matériel
2. **Bootstrap** → programme ROM minimal
3. **Chargement IOS** → depuis Flash (par défaut)
4. **Chargement config** → startup-config depuis NVRAM
5. **Mode opérationnel** → User EXEC ou Setup

### Mode ROMMON

- Accès via **Ctrl+Break** pendant le boot
- Utilisé pour récupération et diagnostic
- Commandes clés : `confreg`, `boot`, `reset`, `dir flash:`
- Config-register 0x2142 pour bypass du startup-config

### Config-register importants

|Valeur|Usage|
|---|---|
|0x2102|Boot normal (défaut)|
|0x2142|Récupération mot de passe|
|0x2101|Boot en ROMMON|

---

> [!tip] 💡 Mémo rapide
> 
> - **Connexion console** : câble rollover + PuTTY 9600 8N1
> - **Boot normal** : POST → Bootstrap → IOS → Config → Prompt
> - **ROMMON** : Ctrl+Break pendant POST, pour urgences
> - **Récup mot de passe** : ROMMON → confreg 0x2142 → reset → nouveau mdp → confreg 0x2102