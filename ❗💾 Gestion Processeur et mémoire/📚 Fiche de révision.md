# Gestion des Processus et de la Mémoire

## 📑 Sommaire

1. [Concepts Fondamentaux](https://claude.ai/chat/8187d618-3e80-49e5-9c95-bc086f7d6c76#concepts-fondamentaux)
2. [Le Problème du Partage du Processeur](https://claude.ai/chat/8187d618-3e80-49e5-9c95-bc086f7d6c76#le-probl%C3%A8me-du-partage-du-processeur)
3. [Notion de Processus](https://claude.ai/chat/8187d618-3e80-49e5-9c95-bc086f7d6c76#notion-de-processus)
4. [Gestion de la Mémoire](https://claude.ai/chat/8187d618-3e80-49e5-9c95-bc086f7d6c76#gestion-de-la-m%C3%A9moire)
5. [L'Ordonnancement](https://claude.ai/chat/8187d618-3e80-49e5-9c95-bc086f7d6c76#lordonnancement)
6. [Approche GNU/Linux](https://claude.ai/chat/8187d618-3e80-49e5-9c95-bc086f7d6c76#approche-gnulinux)
7. [Approche Windows PowerShell](https://claude.ai/chat/8187d618-3e80-49e5-9c95-bc086f7d6c76#approche-windows-powershell)
8. [Le Démarrage du Système](https://claude.ai/chat/8187d618-3e80-49e5-9c95-bc086f7d6c76#le-d%C3%A9marrage-du-syst%C3%A8me)

---

## Concepts Fondamentaux

### Qu'est-ce qu'un programme ?

Un **programme** est une séquence d'instructions machines (en binaire) stockée dans une mémoire. C'est quelque chose de **statique** destiné à être exécuté par un processeur.

### La RAM, à quoi ça sert ?

La **mémoire RAM** est le stockage temporaire et rapide utilisé pour :

- Exécuter les programmes
- Stocker les données en cours d'utilisation
- Permettre une communication rapide avec le processeur

**Représentation graphique :**

```
┌─────────────────────────────────────────┐
│         MÉMOIRE VIVE (RAM)              │
├─────────────────────────────────────────┤
│ Adresse 0x000  │ Byte 1                 │
├────────────────┼─────────────────────────┤
│ Adresse 0x001  │ Byte 2                 │
├────────────────┼─────────────────────────┤
│ Adresse 0x002  │ Byte 3                 │
│      ...       │  ...                   │
└─────────────────────────────────────────┘
```

---

## Le Problème du Partage du Processeur

### Une Première Approche Naïve : "Chacun Son Tour"

À l'origine, on pensait simplement :

1. Lancer le système d'exploitation (noyau)
2. Le noyau lance un programme utilisateur
3. Le programme rend la main au noyau
4. Recommencer

**Problèmes identifiés :**

- ⚠️ Et si le programme **ne rend pas la main** ? (blocage)
- ⚠️ Efficacité réduite (gaspillage CPU lors des attentes)
- ⚠️ Très peu réactif (l'utilisateur doit attendre)

### Inefficacité du CPU : Le Problème I/O

```
Timeline d'exécution classique :

CPU    [Travail] [Attente disque] [Travail] [Attente réseau]
       ✓         ✗                ✓         ✗
       
⚠️ Pendant les attentes, le CPU reste inactif !
```

### Réactivité : Cas des Applications Interactives

Quand un utilisateur clique sur un bouton :

- L'action se lance (calcul long, téléchargement...)
- **Problème** : doit-on attendre la fin pour pouvoir faire autre chose ?
- **Solution** : Non ! Il faut pouvoir continuer à utiliser l'application

---

## Notion de Processus

### Qu'est-ce qu'un Processus ?

**Un processus, c'est un programme en cours d'exécution.**

Plus techniquement, c'est une **structure de données du système d'exploitation** contenant toutes les informations nécessaires pour le gérer.

### La Solution : Le Système à Temps Partagé

**Idée clé** : Simuler des exécutions simultanées en basculant très rapidement entre les processus.

```
Schéma du multitâche préemptif :

┌─────────────────────────────────┐
│   Système d'Exploitation        │
│   (Ordonnanceur)                │
└──────────────┬──────────────────┘
               │
     ┌─────────┼─────────┐
     ▼         ▼         ▼
┌─────────┐ ┌─────────┐ ┌─────────┐
│Process A│ │Process B│ │Process C│
│ En cours│ │ En pause│ │ En pause│
└─────────┘ └─────────┘ └─────────┘

L'OS alterne très rapidement entre eux,
créant l'illusion d'exécutions parallèles
```

### Les Métadonnées d'un Processus

Un processus est caractérisé par :

|Métadonnée|Signification|
|---|---|
|**PID**|Identifiant unique du processus|
|**PPID**|Identifiant du processus parent|
|**CMD**|Commande de lancement|
|**UID**|Identifiant utilisateur associé|
|**GID**|Groupe associé|
|**État**|En cours, en attente, suspendu, etc.|
|**Priorité**|Importance relative du processus|

### Le Soutien Matériel : Interruptions

Le processeur dispose d'un **mode protégé** :

- Certaines instructions sont interdites aux programmes utilisateurs
- Force l'appel au système pour accéder au matériel
- **Interruptions matérielles** : événements déclenchant automatiquement le changement de processus
    - Horloge (timer interrupt)
    - Appels système
    - Signaux externes

---

## Gestion de la Mémoire

### Le Problème : Accès à la Mémoire

Plusieurs processus partagent la même RAM. **Question critique** :

- Comment garantir qu'un processus n'accède **pas aux données** d'un autre processus ?
- Comment éviter qu'il ne les **modifie** ?

### La Solution : Mémoire Virtuelle

**Concept** : Chaque processus dispose de son propre **espace d'adressage virtuel**.

```
Mémoire Virtuelle vs Physique :

Processus A        Processus B        Mémoire Physique
┌─────────┐        ┌─────────┐        ┌───────────────┐
│0x000    │        │0x000    │        │Bloc 1 : ProA  │
│0x004    │  ──┐   │0x004    │  ──┐   │Bloc 2 : ProA  │
│0x008    │    ├──>│0x008    │    ├──>│Bloc 3 : ProB  │
│  ...    │  ──┘   │  ...    │  ──┘   │Bloc 4 : ProB  │
└─────────┘        └─────────┘        │  ...          │
                                      └───────────────┘
                   MMU (Memory Management Unit)
                   Traduit adresses virtuelles 
                   en adresses physiques
```

### Mécanismes de Protection

**Pagination** :

- La mémoire est divisée en **pages** (petits blocs)
- Chaque page peut être chargée/déchargée
- Table des pages = correspondance adresses virtuelles ↔ physiques
- Droits d'accès associés (lecture, écriture, exécution)

**Swap** :

- Si la RAM manque, les pages peu utilisées sont déportées sur le **disque**
- Récupération automatique lors d'un **défaut de page**
- Crée plus de mémoire disponible qu'en réalité

### Allocation d'un Processus

Quand un processus est créé :

1. **Allocation mémoire** pour :
    
    - Code
    - Données statiques
    - Pile (stack)
    - Tas (heap)
2. **Création du PCB** (Process Control Block) :
    
    - PID, PPID, priorité
    - Adresses des zones mémoires
    - Sauvegarde du contexte
    - Insertion dans la table des processus
3. **Protection** : Mise en place des droits d'accès
    

---

## L'Ordonnancement

### À Qui le Tour ?

L'**ordonnanceur** décide quel processus en attente obtient l'accès au processeur après chaque interruption.

```
Flux d'ordonnancement :

Interruption (timer, appel sys, etc.)
        ↓
Sauvegarder l'état du processus actuel
        ↓
Consulter la table des processus
        ↓
Appliquer la politique d'ordonnancement
        ↓
Charger l'état du processus sélectionné
        ↓
Reprendre l'exécution
```

### Politiques d'Ordonnancement

|Politique|Fonctionnement|Avantages|Inconvénients|
|---|---|---|---|
|**FIFO**|Premier arrivé, premier servi|Simple|Pas juste si processus long|
|**Tourniquet**|Chacun son tour, quantum égal|Réactif|Overhead important|
|**Priorités**|Processus haute priorité d'abord|Flexible|Risque de famine|
|**CFS** (Linux)|Partage équitable du CPU|Juste et efficace|Complexe|

**⭐ CFS (Completely Fair Scheduler)** : Ordonnanceur principal de Linux

- Chaque processus dispose d'une part équitable du CPU
- Adapte dynamiquement les priorités
- Algorithme arbre rouge-noir

### Gestion du Swap

**Deux approches** :

1. **À la demande** : Décharger quand la RAM est pleine
2. **Préventive** : Décharger certaines pages avant

**Algorithmes de remplacement** :

- **FIFO** : Remplacer la plus ancienne page
- **LRU** (Least Recently Used) : Remplacer la moins récemment utilisée ⭐ Plus efficace
- **Autres** : Clock, Second Chance, etc.

**Défaut de page** : Tentative d'accès à une page en swap

- Le processus est mis en attente
- La page est rechargée en mémoire
- Le processus reprend

---

## Les Threads

### Processus vs Threads

**Constatation** :

- Créer un processus est **coûteux** (allocation mémoire, PCB, etc.)
- Communication entre processus est **délicate**

**Solution** : Les **threads** (processus légers)

### Qu'est-ce qu'un Thread ?

```
Comparaison :

PROCESSUS                   THREADS (dans même processus)

Processus A                 Processus A
┌─────────────┐            ┌────────────────────┐
│Code         │            │Code (partagé)      │
│Données      │            ├────────────────────┤
│Pile (ProcA) │            │Thread 1: Pile      │
│Tas          │            │Thread 2: Pile      │
│PCB          │            │Thread 3: Pile      │
└─────────────┘            │Données (partagées) │
                           │Tas (partagé)       │
Isolement complet          └────────────────────┘
                           Zones mémoire partagées
```

### Avantages des Threads

✅ Légers (peu coûteux à créer)  
✅ Partage facile de données (même mémoire)  
✅ Changement de contexte rapide  
✅ Programmation parallèle simplifiée

⚠️ **Attention** : Synchronisation nécessaire pour éviter les conflits !

---

## Approche GNU/Linux

### Les Processus Linux

Structure hiérarchique avec **init** (PID 1) à la racine :

```
init (PID 1)
├── systemd
├── NetworkManager
├── cron
├── lightdm
│   └── login
│       └── bash
│           └── job.sh
├── rsyslogd
├── sshd
└── ...
```

**Commande** : `pstree` pour visualiser cette arborescence

### Métadonnées Linux

|Terme|Signification|
|---|---|
|**PID**|Process ID (identifiant unique)|
|**PPID**|Parent Process ID|
|**UID**|User ID (propriétaire)|
|**GID**|Group ID|
|**TTY**|Terminal de contrôle|
|**État**|Running, Sleeping, Stopped, etc.|

Ces infos sont accessibles dans `/proc/[PID]/`

### Signaux Linux

Les processus communiquent via des **signaux** :

|Signal|Nom|Effet|
|---|---|---|
|**SIGINT**|Interruption|Ctrl+C : demande d'arrêt|
|**SIGTERM**|Terminaison|Arrêt gracieux|
|**SIGKILL**|Destruction|Force le noyau à arrêter (ne peut pas être ignoré)|
|**SIGTSTP**|Pause|Ctrl+Z : suspension|
|**SIGQUIT**|Quit|Ctrl+\ : arrêt + core dump|
|**SIGSEGV**|Erreur segmentation|Accès mémoire interdit|

### Commandes Essentielles

**Consultation** :

```bash
ps              # Liste les processus
pstree          # Arborescence des processus
top             # Vue dynamique (consommation CPU)
htop            # Interface améliorée (plus interactive)
```

**Gestion** :

```bash
kill PID        # Envoyer un signal (SIGTERM par défaut)
killall nom     # Tuer tous les processus du nom
fg              # Mettre en premier plan
bg              # Reprendre en arrière plan
```

**Lancement** :

```bash
command &                           # Lancer en arrière plan
nohup command &                     # Lancer détaché (ignorer SIGHUP)
nohup command > log.txt 2>&1 &     # Rediriger sortie et erreurs
```

### Planification de Tâches

```bash
cron            # Démon exécutant tâches récurrentes
crontab -e      # Éditer les tâches planifiées

at HH:MM        # Lancer une tâche à heure précise
atq             # Voir les tâches en attente
atrm ID         # Supprimer une tâche
```

### Gestion du Swap

```bash
free            # Afficher utilisation RAM et swap
swapon -a       # Activer tous les espaces swap
swapoff /dev/sd# # Désactiver un swap
mkswap /path    # Initialiser une partition/fichier swap
```

Configuration persistante : `/etc/fstab`

---

## Approche Windows PowerShell

### Métadonnées Windows

|Terme|Signification|
|---|---|
|**Id**|Process ID (équivalent PID)|
|**ParentId**|Identifiant du processus parent|
|**UserName**|Utilisateur propriétaire|
|**Group**|Groupe associé|
|**Handles**|Nombre de fichiers/ressources ouverts|
|**State**|État (Running, Stopped, etc.)|
|**Path**|Répertoire de travail|

### Cmdlets PowerShell

**Consultation** :

```powershell
Get-Process                  # Liste les processus
Get-Process -Name notepad    # Info sur un processus spécifique
Get-ComputerInfo            # Infos système globales
Get-CimInstance Win32_Process # Via WMI
```

**Gestion** :

```powershell
Start-Process notepad.exe               # Lancer un processus
Stop-Process -Id 1234                   # Arrêter un processus
Wait-Process -Name notepad              # Attendre la fin
Invoke-Command -ScriptBlock {...}       # Exécuter localement ou à distance
```

**Services** :

```powershell
Get-Service                 # Liste les services
Start-Service ServiceName   # Démarrer un service
Stop-Service ServiceName    # Arrêter un service
Restart-Service ServiceName # Redémarrer
Suspend-Service ServiceName # Mettre en pause
Set-Service -Name ServiceName -StartupType Automatic
```

---

## Le Démarrage du Système

### Boot Classique : MBR/BIOS

```
Séquence de démarrage MBR/BIOS :

1. BIOS démarre
   ↓
2. Choisit le périphérique de démarrage (configuration)
   ↓
3. Charge le MBR (Master Boot Record - premier secteur)
   ↓
4. MBR charge le bootloader
   ↓
5. Bootloader charge le noyau de l'OS
   ↓
6. Noyau prend la main
```

**Limitations** : Peu flexible, norme ancienne (années 1980)

### Boot UEFI (Moderne)

```
Séquence de démarrage UEFI :

1. UEFI démarre
   ↓
2. Choisit le périphérique de démarrage
   ↓
3. Accède à l'ESP (EFI System Partition)
   ↓
4. Charge le firmware UEFI + bootloader depuis ESP
   ↓
5. Bootloader charge le noyau
   ↓
6. Noyau prend la main
```

**Avantages** : Flexible, modulaire, sécurisé

### Secure Boot (UEFI)

**Objectif** : Interdire le démarrage de noyaux compromis/malveillants

**Mécanisme** :

- UEFI n'accepte que les bootloaders **signés numériquement**
- Vérification de signature avant chargement
- Mis en place par Microsoft (peut poser problèmes pour Linux)

### Amorçage GNU/Linux

**GRUB** (GRand Unified Bootloader) :

- Très flexible
- Peut démarrer pratiquement n'importe quel OS
- Configuration dans `/boot/grub/grub.cfg`
- Versions : GRUB Legacy (ancien) et GRUB2 (moderne)

### Amorçage Windows

**Bootmgr.exe** :

- Situé à la racine C:\ ou dans partition système
- Lit le **BCD** (Boot Configuration Data)
- Lance **winload.exe** (charge les pilotes système)
- Lance le **SCM** (Service Control Manager)
- SCM démarre les services de démarrage

```
Boot Windows (simplifié) :

UEFI/BIOS
    ↓
Bootmgr.exe
    ↓
Lit BCD
    ↓
Winload.exe (charge drivers + OS)
    ↓
SCM (Service Control Manager)
    ↓
Services de démarrage lancés
```

### Démarrage du Système d'Exploitation

Une fois le noyau chargé, deux approches :

**Linux : Systemd**

- Système de démarrage moderne
- Gestion des dépendances entre services
- Démarrage parallèle (rapide)
- Commande principale : `systemctl`

```bash
systemctl status service       # État d'un service
systemctl enable service       # Activer au démarrage
systemctl start service        # Lancer un service
systemctl restart service      # Redémarrer
```

**Windows : Service Control Manager (SCM)**

- Lance les services selon la base de registre
- Moins parallélisé que systemd
- Contrôle via PowerShell ou Services.msc

### Processus d'Initialisation

Après le noyau, le système doit :

1. **Lancer un premier processus** (init/systemd sous Linux)
2. Cet processus a pour rôle de **démarrer tous les autres**
3. **Services/Démons** : Processus lancés au démarrage, toujours actifs
    - Ex : serveur web, base de données, SSH, etc.

---

## 📊 Résumé Graphique Global

```
╔════════════════════════════════════════════════════════════╗
║         GESTION DES PROCESSUS ET DE LA MÉMOIRE             ║
╚════════════════════════════════════════════════════════════╝

┌──────────────────────────────────────────────────────────┐
│  MATÉRIEL (Processeur + RAM)                             │
└──────────────────────────────────────────────────────────┘
                           ↓
┌──────────────────────────────────────────────────────────┐
│  SYSTÈME D'EXPLOITATION                                  │
├──────────────────────────────────────────────────────────┤
│  ┌─────────────────────────────────────────────────────┐ │
│  │ ORDONNANCEUR (Décide qui s'exécute)                │ │
│  │ • Politique : FIFO, Tourniquet, Priorités, CFS... │ │
│  └─────────────────────────────────────────────────────┘ │
│                        ↓                                  │
│  ┌─────────────────────────────────────────────────────┐ │
│  │ GESTIONNAIRE MÉMOIRE                               │ │
│  │ • Mémoire virtuelle (pages)                        │ │
│  │ • Swap (débordement disque)                        │ │
│  │ • Protection (droits d'accès)                      │ │
│  └─────────────────────────────────────────────────────┘ │
│                        ↓                                  │
│  ┌─────────────────────────────────────────────────────┐ │
│  │ TABLE DES PROCESSUS                                │ │
│  │ • ProcessA (PID 1000, priorité normal)             │ │
│  │ • ProcessB (PID 1001, priorité haute)              │ │
│  │ • Thread 1, Thread 2, ...                          │ │
│  └─────────────────────────────────────────────────────┘ │
└──────────────────────────────────────────────────────────┘
                           ↓
┌──────────────────────────────────────────────────────────┐
│  APPLICATIONS (Processus)                                │
│  • Navigateur, Éditeur, Serveur, etc.                   │
└──────────────────────────────────────────────────────────┘
```

---

## 🎯 Points Clés à Retenir

### ⭐ Concepts Fondamentaux

- **Programme** = code statique sur disque
- **Processus** = programme en cours d'exécution avec état
- **Multitâche préemptif** = OS bascule rapidement entre processus

### ⭐ Mémoire

- **Mémoire virtuelle** = chaque processus a son espace isolé
- **Pagination** = découpe la mémoire en petits blocs
- **Swap** = débordement sur disque quand RAM saturée
- **MMU** = traduit adresses virtuelles → physiques

### ⭐ Processus

- **Métadonnées** : PID, PPID, UID, priorité, état
- **Communication** : signaux entre processus
- **Threads** : processus légers partageant mémoire

### ⭐ Ordonnancement

- **Objectifs** : réactivité, efficacité, équité
- **CFS (Linux)** : ordonnanceur juste et moderne
- **Interruptions** : permettent de basculer entre processus

### ⭐ Démarrage

- **UEFI + Secure Boot** : démarrage moderne et sécurisé
- **GRUB (Linux)** : bootloader flexible
- **Systemd (Linux)** : démarrage moderne et parallélisé
- **Boot Manager (Windows)** : démarrage plus simple

---

## 📚 Sources

- Cours fourni : "Gestion processeur et mémoire.pdf"
- Notes personnelles fournies
- Documentation officielle GNU/Linux et Microsoft PowerShell

---

**Créé** : 2025 | **Niveau** : Bac+2 Informatique | **État** : Synthèse complète