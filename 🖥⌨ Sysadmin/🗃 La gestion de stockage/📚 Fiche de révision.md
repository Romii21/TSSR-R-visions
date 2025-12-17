# 📚 Synthèse : Gestion du Stockage

## 📋 Sommaire

1. [Les concepts fondamentaux](https://claude.ai/chat/f8b27620-5a74-438c-a9fe-e35045614585#concepts)
2. [L'arborescence de fichiers](https://claude.ai/chat/f8b27620-5a74-438c-a9fe-e35045614585#arborescence)
3. [Les chemins d'accès](https://claude.ai/chat/f8b27620-5a74-438c-a9fe-e35045614585#chemins)
4. [Les systèmes de fichiers](https://claude.ai/chat/f8b27620-5a74-438c-a9fe-e35045614585#systemes)
5. [Approche GNU/Linux](https://claude.ai/chat/f8b27620-5a74-438c-a9fe-e35045614585#linux)
6. [Approche Windows](https://claude.ai/chat/f8b27620-5a74-438c-a9fe-e35045614585#windows)
7. [Schéma récapitulatif](https://claude.ai/chat/f8b27620-5a74-438c-a9fe-e35045614585#recap)

---

## <a name="concepts"></a>1️⃣ Les Concepts Fondamentaux

### 🗂️ Qu'est-ce qu'un fichier ?

Un **fichier** est une **unité de stockage élémentaire** qui contient des données. Il s'agit de l'objet fondamental où les informations sont rangées. Chaque fichier possède un nom et se trouve à un emplacement spécifique dans le système.

### 📁 Qu'est-ce qu'un dossier ?

Un **dossier** (ou répertoire) est un **conteneur logique** qui regroupe d'autres fichiers et d'autres dossiers. C'est comme une enveloppe qui organise le rangement des fichiers.

### 🌳 Qu'est-ce qu'une arborescence ?

Une **arborescence de fichiers** est une **structure hiérarchique** en forme d'arbre qui organise tous les fichiers et dossiers du système. Elle part d'une racine et se divise en branches (dossiers) et feuilles (fichiers).

```
         🌲 RACINE
         /    |    \
        /     |     \
    📁 D1   📁 D2   📁 D3
    /  \          /   \
  📄F1 📁D4    📄F2  📄F3
```

---

## <a name="arborescence"></a>2️⃣ L'Arborescence de Fichiers

### 🎯 Structure générale

```
        Racine (/)
        ├── Branche 1 (Dossier)
        │   ├── Feuille 1 (Fichier)
        │   └── Feuille 2 (Fichier)
        ├── Branche 2 (Dossier)
        │   └── Sous-branche (Dossier)
        │       └── Feuille 3 (Fichier)
        └── Branche 3 (Dossier)
```

### ⚫ Les pseudo-dossiers . et ..

Chaque dossier contient **deux pseudo-dossiers spéciaux** :

|Symbole|Signification|Utilité|
|---|---|---|
|**`.`**|Le dossier lui-même|Référencer le répertoire courant|
|**`..`**|Le dossier parent|Remonter d'un niveau|

**Exemple en ligne de commande Linux :**

```bash
$ cd ..      # Remonter au dossier parent
$ ls -la     # Afficher . et .. dans la liste
```

---

## <a name="chemins"></a>3️⃣ Les Chemins d'Accès

### 📍 Types de chemins

#### **Chemin absolu**

Part de la **racine** du système et décrit le chemin complet jusqu'au fichier/dossier.

|OS|Exemple|
|---|---|
|**GNU/Linux**|`/home/wilder/documents/rapport.pdf`|
|**Windows**|`C:\Users\wilder\Documents\rapport.pdf`|

#### **Chemin relatif**

Part du **dossier courant** (le répertoire où vous êtes actuellement).

|OS|Exemple|
|---|---|
|**GNU/Linux**|`documents/rapport.pdf` ou `.ssh/id_ed25519.pub`|
|**Windows**|`Documents\rapport.pdf` ou `.\Pictures\photo.jpg`|

### 🔄 Notion de dossier courant

Chaque processus (notamment le shell) possède un **répertoire courant** (working directory) qui :

- Sert de point de départ pour les chemins relatifs
- S'affiche dans le **prompt** (invité de commande)
- Peut être changé avec la commande `cd`

```bash
$ pwd                      # Affiche le répertoire courant
/home/wilder
$ cd documents/            # Change le répertoire courant
$ pwd
/home/wilder/documents
```

### 🔍 Variable d'environnement PATH

La variable **PATH** est une liste de répertoires où le système cherche les commandes à exécuter. Quand vous tapez une commande, le shell la recherche dans tous les chemins listés dans PATH.

```bash
$ echo $PATH
/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin
```

**🎯 Importance :** Sans PATH, vous devriez taper le chemin complet de chaque programme (`/usr/bin/ls` au lieu de `ls`).

---

## <a name="systemes"></a>4️⃣ Les Systèmes de Fichiers

### 🏗️ Qu'est-ce qu'un système de fichiers (FS) ?

Un **système de fichiers** est une **couche logicielle** qui :

- Fournit une arborescence abstraite (fichiers et dossiers)
- Organise les données sur un périphérique de stockage
- Travaille par **blocs** (unités de stockage de base)

```
Disque dur physique
        ↓
    [PARTITIONS]
        ↓
[SYSTÈME DE FICHIERS]
        ↓
[ARBORESCENCE ABSTRAITE]
```

### 📊 Approche classique : Disques → Partitions → FS

1. **1 disque** est divisé en **1 ou plusieurs partitions**
2. **1 partition** = **1 système de fichiers**
3. Taille du FS = Taille de la partition

### 📝 Métadonnées du FS

Les métadonnées sont des **données sur les données** :

**Métadonnées globales :**

- Configuration générale du système
- Blocs disponibles/occupés
- Blocs défectueux
- Localisation de la racine
- Secteur de démarrage (boot)

**Métadonnées par fichier/dossier :**

- Nom
- Taille
- Propriétaire (Utilisateur/Groupe)
- Droits d'accès
- Dates (création, accès, modification)
- Compteur de liens

### 🔗 Liens physiques (Hard links) et symboliques

**Objectif :** Accéder au même fichier depuis différents répertoires

|Type|Fonctionnement|Avantages|Inconvénients|
|---|---|---|---|
|**Hard link**|Crée une référence supplémentaire au même fichier physique|Suppression sûre (fichier existe tant qu'il y a des liens)|Ne fonctionne qu'au sein d'une même partition|
|**Lien symbolique**|Créé un fichier spécial qui pointe vers un chemin|Fonctionne entre partitions|Devient "cassé" si le fichier original est supprimé|

### 📀 Les principaux systèmes de fichiers

#### **FAT (File Allocation Table)**

- **Créé par :** Microsoft (1977, MS-DOS)
- **Limitation :** Fichiers max 4 Go, pas de droits d'accès
- **Utilisation :** Clés USB, cartes SD, compatibilité universelle
- **Version actuelle :** FAT32

#### **NTFS (New Technology File System)**

- **Système natif de :** Windows NT/2000 et ultérieurs
- **Avantages :** Grands fichiers, droits ACL avancés, journalisation, chiffrement
- **Utilisation :** Disques système Windows, stockage externe avancé

#### **EXT4 (Extended File System 4)**

- **Système natif de :** GNU/Linux
- **Avantages :** Limite la fragmentation, défragmentation en ligne, journalisation, chiffrement
- **Version actuelle :** Version 4 (la plus stabile et performante)

---

## <a name="linux"></a>5️⃣ Approche GNU/Linux

### 🐧 Spécificités de Linux

#### **Une arborescence unique et globale**

```
Tous les systèmes de fichiers (disques, partitions, clés USB...)
            ↓
    Convergent à la RACINE : /
            ↓
    Arborescence unique et cohérente
```

- **Une seule racine `/`** indépendante des périphériques
- Les autres systèmes de fichiers s'y "montent" (mounting)
- Avantage : Navigation transparente entre disques

#### **Philosophie : "Tout est fichier"**

- Fichiers réguliers : données
- Dossiers : fichiers spéciaux
- Périphériques : fichiers dans `/dev`
- Processus : fichiers dans `/proc`

### 📂 Arborescence Linux classique (à retenir)

|Dossier|Contenu|
|---|---|
|**`/bin`**|Commandes essentielles pour tous les utilisateurs|
|**`/sbin`**|Commandes pour l'administration système|
|**`/boot`**|Fichiers de démarrage et noyaux Linux|
|**`/dev`**|Pseudo-fichiers des périphériques (disques, USB, etc.)|
|**`/etc`**|Fichiers de configuration du système|
|**`/home`**|Répertoires personnels des utilisateurs|
|**`/lib`**|Bibliothèques partagées essentielles|
|**`/mnt` / `/media`**|Points de montage pour disques temporaires|
|**`/opt`**|Programmes non standard (hors dépôt officiel)|
|**`/proc`**|Pseudo-fichiers représentant les processus en cours|
|**`/root`**|Répertoire personnel du superutilisateur (root)|
|**`/sys`**|Configuration actuelle du système|
|**`/tmp`**|Données temporaires (nettoyées au redémarrage)|
|**`/usr`**|Programmes et ressources standards|
|**`/var`**|Données variables (journaux, fichiers temporaires)|

### 🛠️ Outils de partitionnement Linux

**CLI (Ligne de commande) :**

- `fdisk`
- `cfdisk`
- `parted`

**GUI (Interface graphique) :**

- `gparted` (Partition Editor)
- `gnome-disks`
- `partitionmanager` (KDE)

### 🏷️ Nomenclature des périphériques

Les disques et partitions sont accessibles via des fichiers spéciaux dans `/dev/` :

```
Disques IDE (anciens)       → /dev/hdX (X = a, b, c...)
Partitions IDE              → /dev/hdXP (P = numéro)

Disques SATA (courant)      → /dev/sdX (X = a, b, c...)
Partitions SATA             → /dev/sdXP (P = numéro)

Disques NVMe (rapides)      → /dev/nvmeYnZ (Y = 0,1..., Z = 1,2...)
Partitions NVMe             → /dev/nvmeYnZpP (P = numéro)
```

**⚠️ Important :** La numérotation dépend de l'ordre de détection. Un disque peut changer de nom après redémarrage !

### 💾 Formatage d'une partition

**Commande générale :**

```bash
mkfs.ext4 /dev/sda1    # Formate en ext4
mkfs.ntfs /dev/sda2    # Formate en NTFS
```

**Options courantes :**

- `-L` : Assigner une étiquette (label)
- `-U` : Assigner un UUID personnalisé

**Autres outils utiles :**

- `fsck` : Vérifier l'intégrité du FS
- `resize2fs` : Redimensionner une partition ext4
- `tune2fs` : Modifier les paramètres
- `e2label` : Changer l'étiquette

### 🔑 UUID (Identifiant Unique Universel)

Chaque partition reçoit un **UUID unique et stable** qui l'identifie de manière fiable, indépendamment du nom du périphérique.

```bash
$ lsblk                    # Liste les périphériques et UUIDs
$ ls /dev/disk/by-uuid/   # Liens symboliques vers les partitions par UUID
$ blkid                    # Affiche les UUIDs bas niveau
```

**Avantage :** Dans `/etc/fstab`, utiliser l'UUID plutôt que le nom du disque garantit la cohérence au démarrage.

### 🔌 Montage/Démontage d'un FS

**Montage :**

```bash
mount /dev/sda1 /mnt/data     # Monte la partition à /mnt/data
```

**Démontage :**

```bash
umount /mnt/data              # Démonte la partition
```

**Montages automatiques au démarrage :**

Le fichier `/etc/fstab` configure les montages automatiques :

```
UUID=9e35d3c3... / ext4 errors=remount-ro 0 1
UUID=df91b5ef... /boot ext4 defaults 0 2
UUID=2e80388d... none swap sw 0 0
```

### 📋 Commandes Linux essentielles

#### Navigation et consultation

|Commande|Fonction|
|---|---|
|`pwd`|Affiche le répertoire courant|
|`ls`|Liste le contenu d'un répertoire|
|`cat [fichier]`|Affiche le contenu d'un fichier|
|`more` / `less`|Affichage paginé d'un fichier|
|`head` / `tail`|Début/fin d'un fichier|

#### Gestion des fichiers et dossiers

|Commande|Fonction|
|---|---|
|`cp [source] [dest]`|Copie un fichier/dossier|
|`mv [source] [dest]`|Déplace/renomme un fichier/dossier|
|`rm [fichier]`|Supprime un fichier|
|`mkdir [dossier]`|Crée un dossier|
|`rmdir [dossier]`|Supprime un dossier (vide)|
|`touch [fichier]`|Crée un fichier vide / met à jour la date|

#### Recherche et traitement

|Commande|Fonction|
|---|---|
|`find`|Recherche des fichiers avancée|
|`which` / `whereis` / `locate`|Localise un fichier/programme|
|`diff [f1] [f2]`|Affiche les différences entre fichiers|
|`grep [motif]`|Filtre les lignes correspondant au motif|
|`wc`|Compte lignes, mots, caractères|
|`sed`|Édition de fichier texte en ligne|
|`awk`|Traitement de fichier par champs|

---

## <a name="windows"></a>6️⃣ Approche Windows

### 🪟 Spécificités de Windows

#### **Plusieurs racines : Les lettres de disque**

```
Chaque partition = Une racine indépendante

C:\ ← Habituellement le disque système
D:\ ← Disque de données
E:\ ← Clé USB ou disque externe
```

- Chaque partition/FS = **sa propre arborescence**
- Identifiée par une **lettre** (C:, D:, E:...)
- Pas de notion d'arborescence globale unique

### 📂 Arborescence Windows classique

|Dossier|Contenu|
|---|---|
|**`C:\$Recycle.Bin`**|Corbeille du système de fichiers|
|**`C:\PerfLogs`**|Journaux de performance|
|**`C:\Program Files`**|Programmes tiers 64 bits|
|**`C:\Program Files (x86)`**|Programmes tiers 32 bits|
|**`C:\ProgramData`**|Données globales partagées|
|**`C:\Recovery`**|Fichiers de récupération|
|**`C:\System Volume Information`**|Métadonnées NTFS, points de restauration|
|**`C:\Users`**|Répertoires personnels des utilisateurs|
|**`C:\Windows`**|Installation du système d'exploitation|

### 🛠️ Outils de gestion du stockage Windows

**CLI (Ligne de commande) :**

- `Diskpart` : Gestion complète disques/partitions
- `Format` : Création FS sur une partition

**GUI (Interface graphique) :**

- `diskmgmt.msc` : Gestion de disques (plus intuitive)
- **Note :** Les disques sont automatiquement disponibles (pas besoin de montage manuel)

### 📋 Commandes PowerShell essentielles

#### Navigation et consultation

|Commande|Alias|Fonction|
|---|---|---|
|`Get-Location`|`pwd`|Affiche le répertoire courant|
|`Get-ChildItem`|`ls`|Liste le contenu d'un répertoire|
|`Get-Content`|`cat`|Affiche un fichier|

#### Gestion des fichiers et dossiers

|Commande|Alias|Fonction|
|---|---|---|
|`Copy-Item`|`cp`|Copie un fichier/dossier|
|`Move-Item`|`mv`|Déplace/renomme|
|`Remove-Item`|`rm`, `rmdir`|Supprime un fichier/dossier|
|`mkdir`|—|Crée un dossier|

#### Recherche et traitement

|Commande|Fonction|
|---|---|
|`Compare-Object`|Affiche les différences|
|`Where-Object`|Filtre les objets|
|`Select-Object`|Sélectionne des colonnes|

---

## <a name="recap"></a>7️⃣ Schéma Récapitulatif

### 🔄 Vue globale du système de stockage

```
┌─────────────────────────────────────────────────────────┐
│              DISQUES DURS / STOCKAGE PHYSIQUE            │
└────────────────────┬────────────────────────────────────┘
                     │
         ┌───────────┴────────────┐
         │                        │
    ┌────▼──────┐         ┌──────▼─────┐
    │ Partition │         │ Partition  │
    │    1 (/)  │         │    2 (/home)
    └────┬──────┘         └──────┬─────┘
         │                       │
    ┌────▼──────────┐    ┌──────▼─────────┐
    │ EXT4 FS       │    │ EXT4 FS        │
    │ Métadonnées   │    │ Métadonnées    │
    │ Blocs         │    │ Blocs          │
    └────┬──────────┘    └──────┬─────────┘
         │                       │
    ┌────▼──────────────────────▼──────┐
    │   ARBORESCENCE ABSTRAITE UNIQUE   │
    │  (Linux : unique, Windows : par   │
    │           lettre de disque)       │
    │                                   │
    │  /                                │
    │  ├── bin/                         │
    │  ├── home/                        │
    │  │   └── user/                    │
    │  │       └── documents/           │
    │  │           └── rapport.pdf      │
    │  └── etc/                         │
    │                                   │
    └───────────────────────────────────┘
```

### 📊 Comparaison Linux vs Windows

|Aspect|**Linux**|**Windows**|
|---|---|---|
|**Racine**|Une seule : `/`|Plusieurs : `C:\`, `D:\`, ...|
|**Arborescence**|Unique, unifiée|Séparée par disque|
|**Montage**|Manuel (mount)|Automatique|
|**Séparateur**|`/`|`\`|
|**FS principal**|ext4|NTFS|
|**Path**|`$PATH`|`$env:path`|
|**Disques**|`/dev/sdX`|Lettres|
|**Philosophie**|Tout est fichier|Abstraction par lettres|

---

## 💡 Points clés à retenir

✅ **Fichiers et dossiers** : Unités élémentaires du stockage  
✅ **Arborescence** : Organisation hiérarchique des données  
✅ **Chemins** : Absolus (depuis racine) ou relatifs (depuis dossier courant)  
✅ **Système de fichiers** : Couche logicielle organisant les données par blocs  
✅ **Linux** : Une racine `/`, UUIDs stables, montage flexible  
✅ **Windows** : Racines multiples (C:, D:...), montage automatique  
✅ **ext4 vs NTFS** : Les systèmes natifs de Linux et Windows  
✅ **PATH** : Variable d'environnement définissant où chercher les commandes

---

## 📚 Sources

- **Cours fourni** : "Gestion de stockage.pdf"
- **Notes personnelles** : Compilation des concepts clés
- Philosophie Unix/Linux : Principes standardisés POSIX
- [Documentation ext4](https://ext4.wiki.kernel.org/) : Système de fichiers Linux
- [Références NTFS](https://docs.microsoft.com/) : Documentation officielle Microsoft

---

**Dernière mise à jour** : Novembre 2025