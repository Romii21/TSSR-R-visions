

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

## 🔄 Running-config vs Startup-config

### Comprendre les deux types de configuration

Sur un routeur Cisco, il existe **deux fichiers de configuration distincts** qui jouent des rôles différents dans le fonctionnement de l'équipement.

|Configuration|Emplacement|Volatilité|Rôle|
|---|---|---|---|
|**Running-config**|RAM|Volatile (perdue au redémarrage)|Configuration active en cours d'exécution|
|**Startup-config**|NVRAM|Non-volatile (persistante)|Configuration chargée au démarrage|

> [!info] Pourquoi deux fichiers ? Cette séparation permet de tester des modifications sans risquer de perdre la connectivité. Si une erreur est commise dans la running-config, un simple redémarrage restaure la configuration précédente (startup-config).

### Running-config (configuration active)

La **running-config** est la configuration actuellement en mémoire vive et utilisée par le routeur. Toute modification effectuée en mode de configuration s'applique immédiatement à ce fichier.

**Caractéristiques :**

- Stockée dans la **RAM** (mémoire volatile)
- Active et en cours d'utilisation
- Modifiable en temps réel
- **Perdue en cas de redémarrage** si non sauvegardée

**Visualiser la running-config :**

```bash
Router# show running-config
```

ou en version abrégée :

```bash
Router# show run
```

> [!tip] Filtrage de l'affichage Vous pouvez filtrer la sortie pour afficher uniquement certaines sections :
> 
> ```bash
> Router# show run | section interface
> Router# show run | include hostname
> Router# show run | begin line vty
> ```

### Startup-config (configuration de démarrage)

La **startup-config** est la configuration sauvegardée qui sera chargée lors du prochain démarrage du routeur.

**Caractéristiques :**

- Stockée dans la **NVRAM** (mémoire non-volatile)
- Persistante après un redémarrage
- Utilisée uniquement au démarrage
- Ne reflète pas les modifications en cours tant qu'elle n'est pas mise à jour

**Visualiser la startup-config :**

```bash
Router# show startup-config
```

ou en version abrégée :

```bash
Router# show start
```

> [!warning] Attention aux modifications non sauvegardées Une erreur fréquente est de modifier la running-config et de redémarrer le routeur en oubliant de sauvegarder. Toutes les modifications seront alors perdues !

### Comparaison entre les deux configurations

Pour vérifier si des modifications non sauvegardées existent :

```bash
Router# show archive config differences
```

Cette commande affiche les différences entre running-config et startup-config.

---

## 💾 Sauvegarde de la configuration

### Pourquoi sauvegarder ?

La sauvegarde de la configuration est une étape **critique** pour :

- Rendre les modifications permanentes après un redémarrage
- Éviter de perdre des heures de configuration
- Garantir la cohérence de la configuration

### Commande principale de sauvegarde

La commande standard pour copier la running-config vers la startup-config :

```bash
Router# copy running-config startup-config
```

**Processus détaillé :**

```bash
Router# copy running-config startup-config
Destination filename [startup-config]? 
Building configuration...
[OK]
```

> [!info] Confirmation de la destination Le système demande confirmation du nom du fichier de destination. Appuyez simplement sur **Entrée** pour accepter le nom par défaut `startup-config`.

### Commande abrégée

La commande la plus couramment utilisée par les administrateurs :

```bash
Router# copy run start
```

Cisco IOS accepte les abréviations tant qu'elles sont non ambiguës.

### Autre méthode : write memory

Une commande héritée mais toujours fonctionnelle :

```bash
Router# write memory
```

ou simplement :

```bash
Router# wr
```

> [!tip] Bonne pratique professionnelle Bien que `write` fonctionne, la commande `copy running-config startup-config` est préférée car elle est plus explicite et conforme aux standards modernes Cisco.

### Vérification après sauvegarde

Après avoir sauvegardé, vérifiez que les deux configurations correspondent :

```bash
Router# show startup-config
Router# show running-config
```

Ou utilisez :

```bash
Router# show archive config differences
```

Si aucune différence n'est affichée, les fichiers sont identiques.

> [!warning] Sauvegarde systématique Prenez l'habitude de sauvegarder après **chaque modification importante**. Un redémarrage inattendu (panne de courant, intervention) pourrait sinon faire perdre tout votre travail.

---

## 🗑️ Effacement de la configuration

### Cas d'usage

L'effacement de la configuration est nécessaire dans plusieurs situations :

- Réinitialisation complète d'un routeur
- Préparation avant une nouvelle configuration
- Suppression d'une configuration de test
- Remise à zéro avant revente ou réaffectation

### Effacer la startup-config

Pour supprimer la configuration de démarrage (sans toucher à la running-config) :

```bash
Router# erase startup-config
```

**Processus détaillé :**

```bash
Router# erase startup-config
Erasing the nvram filesystem will remove all configuration files! Continue? [confirm]
[OK]
Erase of nvram: complete
```

> [!warning] Effet immédiat sur la NVRAM Cette commande supprime **immédiatement** le fichier startup-config de la NVRAM. Cependant, la running-config reste active jusqu'au redémarrage.

### Commande alternative

Une autre syntaxe équivalente :

```bash
Router# write erase
```

### Redémarrage après effacement

Pour que l'effacement prenne pleinement effet, redémarrez le routeur :

```bash
Router# reload
```

**Processus de redémarrage :**

```bash
Router# reload
System configuration has been modified. Save? [yes/no]: no
Proceed with reload? [confirm]
```

> [!info] Question de sauvegarde Le système détecte que la running-config diffère de la startup-config et propose de sauvegarder. Répondez **no** si vous souhaitez effectivement perdre la configuration actuelle.

### Effacement complet : running-config + startup-config

Pour remettre un routeur à zéro complet :

```bash
Router# erase startup-config
Router# reload
```

Au redémarrage, le routeur n'aura aucune configuration et démarrera en mode de configuration initiale (Setup Mode).

### Effacer uniquement certaines sections

Vous pouvez également effacer des parties spécifiques :

```bash
Router# configure terminal
Router(config)# no interface GigabitEthernet0/0
Router(config)# no ip route 0.0.0.0 0.0.0.0 192.168.1.1
```

Cependant, ces modifications affectent la **running-config** et doivent être sauvegardées (ou non) selon le besoin.

> [!tip] Snapshot de sécurité Avant d'effacer une configuration, faites toujours une sauvegarde externe (TFTP, USB, copie manuelle) au cas où vous en auriez besoin ultérieurement.

---

## 📤 Export et Import de configurations

### Pourquoi exporter/importer des configurations ?

L'export et l'import permettent :

- **Sauvegarde externe** : protection contre les défaillances matérielles
- **Documentation** : archivage des configurations pour audit
- **Déploiement en masse** : duplication de configurations sur plusieurs équipements
- **Restauration rapide** : récupération après incident
- **Versioning** : gestion de différentes versions de configuration

### Méthodes d'export/import

Cisco IOS supporte plusieurs méthodes pour transférer des configurations :

|Méthode|Protocole|Cas d'usage|
|---|---|---|
|**TFTP**|TFTP|Transferts simples sur réseau local|
|**FTP**|FTP|Transferts avec authentification|
|**SCP**|SSH|Transferts sécurisés|
|**USB**|Local|Transferts physiques sans réseau|
|**Console**|Serial|Copie manuelle via terminal|

---

### 🌐 Export/Import via TFTP

TFTP (Trivial File Transfer Protocol) est la méthode la plus courante pour les équipements Cisco.

#### Prérequis

1. Un serveur TFTP accessible sur le réseau
2. Connectivité IP entre le routeur et le serveur
3. Aucune restriction de pare-feu (port UDP 69)

#### Export de la configuration vers TFTP

**Syntaxe générale :**

```bash
Router# copy running-config tftp:
```

**Processus détaillé :**

```bash
Router# copy running-config tftp:
Address or name of remote host []? 192.168.1.100
Destination filename [router-confg]? backup-router1-2024.cfg
!!
1234 bytes copied in 2.489 secs (495 bytes/sec)
```

> [!info] Explication des paramètres
> 
> - **Address or name of remote host** : Adresse IP du serveur TFTP
> - **Destination filename** : Nom du fichier sur le serveur (par défaut : hostname + "-confg")
> - **!!** : Indicateurs de progression (chaque ! représente un paquet transféré)

**Syntaxe complète en une ligne :**

```bash
Router# copy running-config tftp://192.168.1.100/backup-router1.cfg
```

#### Import de la configuration depuis TFTP

**Pour remplacer la running-config :**

```bash
Router# copy tftp: running-config
Address or name of remote host []? 192.168.1.100
Source filename []? backup-router1-2024.cfg
Accessing tftp://192.168.1.100/backup-router1-2024.cfg...
Loading backup-router1-2024.cfg from 192.168.1.100 (via GigabitEthernet0/0): !
[OK - 1234 bytes]
1234 bytes copied in 3.214 secs (384 bytes/sec)
```

> [!warning] Fusion de configuration L'import vers `running-config` **fusionne** la configuration importée avec la configuration existante. Les commandes du fichier importé sont ajoutées ou modifient les paramètres existants, mais ne suppriment pas les configurations non présentes dans le fichier.

**Pour remplacer la startup-config :**

```bash
Router# copy tftp: startup-config
```

Puis redémarrez pour appliquer :

```bash
Router# reload
```

#### Syntaxe alternative avec source et destination

```bash
Router# copy tftp://192.168.1.100/backup-router1.cfg running-config
Router# copy tftp://192.168.1.100/backup-router1.cfg startup-config
```

---

### 💿 Export/Import via USB

Les routeurs Cisco modernes supportent les clés USB pour les transferts locaux.

#### Vérifier la présence d'une clé USB

```bash
Router# show file systems
File Systems:

       Size(b)       Free(b)      Type  Flags  Prefixes
*    256487424     201039872     flash     rw   flash:
      33554432      33538048      disk     rw   usbflash0:
```

> [!info] Identification du système de fichiers USB Recherchez `usbflash0:` ou `usb0:` dans la liste des systèmes de fichiers.

#### Lister le contenu de la clé USB

```bash
Router# dir usbflash0:
Directory of usbflash0:/

    1  -rw-     1234  Dec 21 2025 10:30:00 +00:00  backup-old.cfg
    2  -rw-     5678  Dec 20 2025 14:15:00 +00:00  ios-image.bin

33538048 bytes total (33530496 bytes free)
```

#### Export vers USB

```bash
Router# copy running-config usbflash0:backup-router1-21dec2024.cfg
```

**Ou en mode interactif :**

```bash
Router# copy running-config usbflash0:
Destination filename [running-config]? backup-router1-21dec2024.cfg
!!
1234 bytes copied in 1.234 secs (1000 bytes/sec)
```

#### Import depuis USB

```bash
Router# copy usbflash0:backup-router1-21dec2024.cfg running-config
```

> [!tip] Avantage de l'USB L'USB ne nécessite aucune infrastructure réseau et permet des sauvegardes physiques facilement transportables. Idéal pour les interventions sur site ou les environnements isolés.

---

### 🔐 Export/Import via FTP/SCP (sécurisé)

Pour les environnements nécessitant plus de sécurité qu'avec TFTP.

#### Configuration du nom d'utilisateur FTP/SCP

Avant d'utiliser FTP ou SCP, configurez les identifiants :

```bash
Router(config)# ip ftp username admin
Router(config)# ip ftp password cisco123
```

Pour SCP :

```bash
Router(config)# ip scp server enable
Router(config)# ip ssh version 2
```

#### Export via FTP

```bash
Router# copy running-config ftp://admin:cisco123@192.168.1.100/backup-router1.cfg
```

#### Export via SCP

```bash
Router# copy running-config scp://admin@192.168.1.100/backup-router1.cfg
Password: 
```

> [!info] Sécurité SCP SCP utilise SSH et chiffre les données pendant le transfert, contrairement à TFTP et FTP qui transmettent en clair.

---

### 📋 Copie manuelle via console

Pour les petites configurations ou en l'absence de serveur, vous pouvez copier manuellement.

#### Export manuel

1. Affichez la configuration :

```bash
Router# show running-config
```

2. Sélectionnez tout le texte dans votre terminal et copiez-le
3. Collez dans un éditeur de texte et sauvegardez

#### Import manuel

1. Ouvrez le fichier de configuration dans un éditeur
2. Copiez l'intégralité du contenu
3. Connectez-vous au routeur en mode configuration :

```bash
Router# configure terminal
```

4. Collez le contenu dans la console
5. Sortez du mode configuration :

```bash
Router(config)# end
```

> [!warning] Limitations de la copie manuelle Cette méthode est sujette aux erreurs (caractères spéciaux, délais de tampon) et n'est recommandée que pour de petites configurations ou en dépannage.

---

### 🔍 Bonnes pratiques pour les sauvegardes

> [!tip] Stratégie de sauvegarde recommandée
> 
> - **Sauvegardez régulièrement** : après chaque modification majeure
> - **Utilisez des noms explicites** : incluez le hostname, la date et la version
> - **Automatisez** : scriptez les sauvegardes TFTP via des tâches planifiées
> - **Versionnez** : conservez plusieurs versions pour pouvoir revenir en arrière
> - **Testez la restauration** : vérifiez périodiquement que vos sauvegardes sont exploitables
> - **Sauvegardez hors site** : stockez des copies sur des serveurs distants pour la reprise après sinistre

**Exemple de convention de nommage :**

```
hostname-YYYYMMDD-HHmm-version.cfg
router1-20241221-1430-v2.3.cfg
```

---

### ⚠️ Pièges courants

> [!warning] Erreurs fréquentes à éviter
> 
> **1. Oublier de sauvegarder avant un reload**
> 
> - Toujours faire `copy run start` avant `reload`
> 
> **2. Confondre running et startup**
> 
> - Les modifications s'appliquent à running, pas à startup automatiquement
> 
> **3. Serveur TFTP inaccessible**
> 
> - Vérifiez la connectivité avec `ping` avant de transférer
> - Assurez-vous que le service TFTP est actif
> 
> **4. Permissions insuffisantes sur le serveur TFTP**
> 
> - Le répertoire TFTP doit être accessible en lecture/écriture
> 
> **5. Importer directement dans startup sans tester**
> 
> - Importez d'abord dans running pour tester, puis sauvegardez
> 
> **6. Ne pas vérifier l'espace disponible**
> 
> - Utilisez `show file systems` pour vérifier l'espace libre sur flash/USB

---

## 📊 Tableau récapitulatif des commandes

|Action|Commande|
|---|---|
|Afficher la running-config|`show running-config` ou `show run`|
|Afficher la startup-config|`show startup-config` ou `show start`|
|Sauvegarder running → startup|`copy running-config startup-config` ou `copy run start`|
|Effacer startup-config|`erase startup-config`|
|Exporter vers TFTP|`copy running-config tftp:`|
|Importer depuis TFTP|`copy tftp: running-config`|
|Exporter vers USB|`copy running-config usbflash0:nomfichier.cfg`|
|Importer depuis USB|`copy usbflash0:nomfichier.cfg running-config`|
|Comparer running et startup|`show archive config differences`|
|Lister fichiers USB|`dir usbflash0:`|
|Redémarrer|`reload`|

---

## 🎯 Points clés à retenir

- **Running-config** (RAM) = configuration active, volatile
- **Startup-config** (NVRAM) = configuration de démarrage, persistante
- Toujours **sauvegarder** après des modifications importantes : `copy run start`
- Les imports fusionnent avec la configuration existante (ne remplacent pas)
- TFTP est la méthode standard, USB est pratique sur site
- Une stratégie de sauvegarde rigoureuse est essentielle en production