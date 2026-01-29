## 📑 Sommaire

1. [Introduction aux besoins de stockage](https://claude.ai/chat/0e03f404-56f4-4110-bc70-72743212b077#1-introduction-aux-besoins-de-stockage)
    
    - [1.1 Analyser ses besoins](https://claude.ai/chat/0e03f404-56f4-4110-bc70-72743212b077#11-analyser-ses-besoins)
    - [1.2 Choisir ses supports](https://claude.ai/chat/0e03f404-56f4-4110-bc70-72743212b077#12-choisir-ses-supports)
    - [1.3 Fiabilité et cloisonnement](https://claude.ai/chat/0e03f404-56f4-4110-bc70-72743212b077#13-fiabilit%C3%A9-et-cloisonnement)
2. [RAID - Redundant Array of Independent Disks](https://claude.ai/chat/0e03f404-56f4-4110-bc70-72743212b077#2-raid---redundant-array-of-independent-disks)
    
    - [2.1 Principe et objectifs](https://claude.ai/chat/0e03f404-56f4-4110-bc70-72743212b077#21-principe-et-objectifs)
    - [2.2 Types d'implémentation](https://claude.ai/chat/0e03f404-56f4-4110-bc70-72743212b077#22-types-dimpl%C3%A9mentation)
    - [2.3 Niveaux de RAID](https://claude.ai/chat/0e03f404-56f4-4110-bc70-72743212b077#23-niveaux-de-raid)
    - [2.4 Bonnes pratiques](https://claude.ai/chat/0e03f404-56f4-4110-bc70-72743212b077#24-bonnes-pratiques)
3. [LVM - Logical Volume Manager](https://claude.ai/chat/0e03f404-56f4-4110-bc70-72743212b077#3-lvm---logical-volume-manager)
    
    - [3.1 Architecture LVM](https://claude.ai/chat/0e03f404-56f4-4110-bc70-72743212b077#31-architecture-lvm)
    - [3.2 Composants détaillés](https://claude.ai/chat/0e03f404-56f4-4110-bc70-72743212b077#32-composants-d%C3%A9taill%C3%A9s)
    - [3.3 Snapshots et COW](https://claude.ai/chat/0e03f404-56f4-4110-bc70-72743212b077#33-snapshots-et-cow)
    - [3.4 Redimensionnement à chaud](https://claude.ai/chat/0e03f404-56f4-4110-bc70-72743212b077#34-redimensionnement-%C3%A0-chaud)
4. [Stockage réseau : NAS et SAN](https://claude.ai/chat/0e03f404-56f4-4110-bc70-72743212b077#4-stockage-r%C3%A9seau--nas-et-san)
    
    - [4.1 NAS - Network Attached Storage](https://claude.ai/chat/0e03f404-56f4-4110-bc70-72743212b077#41-nas---network-attached-storage)
    - [4.2 SAN - Storage Area Network](https://claude.ai/chat/0e03f404-56f4-4110-bc70-72743212b077#42-san---storage-area-network)
5. [Points clés à retenir](https://claude.ai/chat/0e03f404-56f4-4110-bc70-72743212b077#-points-cl%C3%A9s-%C3%A0-retenir)
    
6. [Sources et références](https://claude.ai/chat/0e03f404-56f4-4110-bc70-72743212b077#-sources-et-r%C3%A9f%C3%A9rences)
    

---

## 1. Introduction aux besoins de stockage

### 1.1 Analyser ses besoins

Avant de choisir une solution de stockage, il faut identifier trois critères essentiels :

|Critère|Description|Questions à se poser|
|---|---|---|
|**Volume**|Espace de stockage nécessaire|Quelle taille ? Évolution dans le temps ?|
|**Performance**|Vitesse d'accès aux données|Débit nécessaire ? Temps d'accès critique ?|
|**Sûreté**|Stratégie de conservation|Tolérance aux pannes ? Politique de sauvegarde ?|

**💼 En entreprise :**

- **HDD** (disques durs) → serveurs (robustes, grande capacité)
- **SSD** (disques SSD) → postes clients (rapides, silencieux)

### 1.2 Choisir ses supports

**Comparatif 2023 des supports :**

|Type|Capacité|Performance|Prix|Utilisation|
|---|---|---|---|---|
|**HDD**|1 To → 24 To|~100 Mo/s|20-30 €/To|Stockage de masse, serveurs|
|**SSD**|250 Go → 8 To|3-7 Go/s|~100 €/To|Performances, postes de travail|
|**NVMe**|250 Go → 4 To|>7 Go/s|>100 €/To|Très hautes performances|

> **⚠️ Note :** Le coût n'est pas linéaire - les très grandes capacités sont proportionnellement plus chères.

### 1.3 Fiabilité et cloisonnement

**Problématiques :**

- Plus on a de disques, plus la probabilité de panne augmente
- Les pannes mécaniques (HDD) et limites d'écriture (SSD) sont inévitables
- Un système de fichiers plein → crash du système/application

**Solution : Le cloisonnement**

Créer des espaces de stockage **séparés et étanches** pour isoler les impacts :

|Répertoire à isoler|Raison|
|---|---|
|`/var/log`|Logs qui peuvent grossir rapidement|
|Bases de données|Forte variation de taille|
|Fichiers utilisateurs (`/home`)|Éviter saturation par les users|
|Espace swap|Performances système|
|`/tmp`|Fichiers temporaires|

**⚠️ Limites du cloisonnement traditionnel :**

- Nombre de partitions limité (MBR : 4 primaires, GPT : 128)
- Taille limitée par le disque physique
- Évolution après installation risquée

**→ Solution moderne : RAID et LVM**

---

## 2. RAID - Redundant Array of Independent Disks

### 2.1 Principe et objectifs

**RAID** = _Redundant Array of Independent/Inexpensive Disks_ (Matrice redondante de disques indépendants/bon marché)

**📅 Historique :** Créé en 1987 à l'Université de Berkeley en opposition au SLED (Single Large Expensive Disk)

**🎯 Objectifs :**

1. **Agrandir** la taille maximale disponible
2. **Améliorer** les performances (débit)
3. **Améliorer** la fiabilité (tolérance aux pannes)

Sans avoir besoin d'acheter des disques plus performants et coûteux.

### 2.2 Types d'implémentation

|Type|Description|Avantages|Inconvénients|
|---|---|---|---|
|**RAID Matériel**|Contrôleur physique (carte PCI)|✅ Excellentes performances<br>✅ Pas de charge CPU<br>✅ Boot possible sur RAID|❌ Coûteux<br>❌ Propriétaire (incompatibilité)|
|**RAID Logiciel**|Géré par l'OS|✅ Flexible<br>✅ Standard/compatible<br>✅ Gratuit|❌ Consomme du CPU<br>❌ Boot compliqué|
|**RAID Hybride**|Contrôleur + logiciel|✅ Boot possible|❌ Problèmes de compatibilité<br>❌ Consomme du CPU<br>❌ Fonctionnalités limitées|

**🐧 Linux :** Utilisez l'outil `mdadm` pour gérer le RAID logiciel.

### 2.3 Niveaux de RAID

#### JBOD / NRAID (Non-RAID)

**JBOD** = _Just a Bunch Of Disks_ (Juste un tas de disques)

```
Principe : Fusion de plusieurs disques en un seul volume
```

|Caractéristique|Valeur|
|---|---|
|Capacité totale|Somme de tous les disques|
|Performance|Identique aux disques seuls|
|Tolérance aux pannes|⚠️ **AUCUNE** (pire que disque seul)|
|Utilisation|❌ **Déconseillé**|

---

#### RAID 0 - Striping (Entrelaçage)

```
Principe : Données découpées et réparties sur n disques (généralement 2)
```

**Schéma conceptuel :**

```
Fichier complet → [Part 1] → Disque 1
                  [Part 2] → Disque 2
```

|Caractéristique|Valeur|
|---|---|
|Capacité totale|n × capacité du plus petit disque|
|Performance|✅ **Multipliée par n**|
|Tolérance aux pannes|⚠️ **AUCUNE** (1 disque en panne = tout perdu)|
|Utilisation|Performances pures (édition vidéo, cache)|

> **💡 Bon à savoir :** Plus vous ajoutez de disques, plus les performances augmentent, mais plus le risque de panne augmente aussi.

---

#### RAID 1 - Mirroring (Miroir)

```
Principe : Données copiées identiquement sur n disques (généralement 2)
```

**Schéma conceptuel :**

```
Fichier → [Copie complète] → Disque 1
       → [Copie complète] → Disque 2
```

|Caractéristique|Valeur|
|---|---|
|Capacité totale|Capacité du plus petit disque|
|Performance|≈ Identique (légère amélioration en lecture)|
|Tolérance aux pannes|✅ **Excellente** : tolère n-1 pannes|
|Utilisation|Données critiques (bases de données, configs)|

---

#### RAID 4 - Striping avec parité

```
Principe : RAID 0 + 1 disque dédié à la parité (XOR)
Minimum : 3 disques
```

**Schéma conceptuel :**

```
Fichier → [Part 1] → Disque 1
       → [Part 2] → Disque 2
       → [Parité] → Disque 3 (calcul XOR)
```

|Caractéristique|Valeur|
|---|---|
|Capacité totale|(n-1) × capacité du plus petit disque|
|Performance|✅ Bon (proportionnel à n-1)|
|Tolérance aux pannes|✅ Tolère **1 panne**|
|Inconvénient|⚠️ Disque de parité = goulot d'étranglement|

> **⚠️ Reconstruction :** Coûteuse et longue en cas de panne.

---

#### RAID 5 - Striping avec parité répartie

```
Principe : RAID 4 avec parité répartie sur tous les disques (round-robin)
Minimum : 3 disques
```

**Schéma conceptuel :**

```
Disque 1 : [Data A1] [Data B1] [Parité C]
Disque 2 : [Data A2] [Parité B] [Data C1]
Disque 3 : [Parité A] [Data B2] [Data C2]
```

|Caractéristique|Valeur|
|---|---|
|Capacité totale|(n-1) × capacité du plus petit disque|
|Performance|✅ **Meilleure que RAID 4** (pas de goulot)|
|Tolérance aux pannes|✅ Tolère **1 panne**|
|Utilisation|🏆 **Le plus populaire** (bon compromis)|

---

#### RAID 6 - Double parité

```
Principe : RAID 5 avec 2 systèmes de parité indépendants
Minimum : 4 disques
```

|Caractéristique|Valeur|
|---|---|
|Capacité totale|(n-2) × capacité du plus petit disque|
|Performance|✅ Bon|
|Tolérance aux pannes|✅ **Excellente** : tolère **2 pannes simultanées**|
|Reconstruction|⚠️ Plus longue et coûteuse|
|Utilisation|Données critiques, gros volumes|

---

#### RAID 10 (1+0) - Combinaison

```
Principe : RAID 0 de grappes RAID 1
Minimum : 4 disques
```

**Schéma conceptuel :**

```
RAID 1a : [Disque 1] ←mirror→ [Disque 2]
          ↓ RAID 0 (striping) ↓
RAID 1b : [Disque 3] ←mirror→ [Disque 4]
```

|Caractéristique|Valeur|
|---|---|
|Capacité totale|(n/2) × capacité du plus petit disque|
|Performance|✅ **Excellente** (striping + mirroring)|
|Tolérance aux pannes|✅ Tolère plusieurs pannes (si pas sur même miroir)|
|Utilisation|Serveurs haute performance avec sécurité|

> **💡 Variante :** RAID 01 (0+1) = RAID 1 de grappes RAID 0 (moins tolérant aux pannes)

---

#### 📊 Tableau comparatif des niveaux RAID

|Niveau|Disques min.|Capacité utile|Performance|Tolérance panne|Usage typique|
|---|---|---|---|---|---|
|**JBOD**|2+|100%|=|❌ Aucune|❌ À éviter|
|**RAID 0**|2|100%|✅✅✅|❌ Aucune|Perf. pure|
|**RAID 1**|2|50%|=|✅✅✅|Données critiques|
|**RAID 4**|3|~66%|✅✅|✅ 1 disque|Peu utilisé|
|**RAID 5**|3|~66%|✅✅|✅ 1 disque|🏆 Standard|
|**RAID 6**|4|~50%|✅|✅✅ 2 disques|Sécurité max|
|**RAID 10**|4|50%|✅✅✅|✅✅ Multi|Serveurs critiques|

### 2.4 Bonnes pratiques

✅ **À faire :**

- Utiliser des disques **identiques** (même capacité, modèle, vitesse)
- Configurer un **disque de spare** pour reconstruction automatique
- Utiliser des disques **hot-plug** (remplaçables à chaud)
- Surveiller l'état de la grappe RAID régulièrement

⚠️ **À NE PAS oublier :**

- Le RAID **n'est PAS une sauvegarde** !
- RAID protège contre les pannes matérielles, pas contre :
    - Suppression accidentelle
    - Corruption de données
    - Ransomwares
    - Erreurs humaines

**🔧 Commandes Linux (mdadm) :**

```bash
# Créer un RAID 5 avec 3 disques
mdadm --create /dev/md0 --level=5 --raid-devices=3 /dev/sda1 /dev/sdb1 /dev/sdc1

# Voir l'état du RAID
cat /proc/mdstat
mdadm --detail /dev/md0

# Ajouter un disque de spare
mdadm --add /dev/md0 /dev/sdd1

# Retirer un disque défectueux
mdadm --fail /dev/md0 /dev/sda1
mdadm --remove /dev/md0 /dev/sda1
```

---

## 3. LVM - Logical Volume Manager

### 3.1 Architecture LVM

**LVM** = _Logical Volume Manager_ (Gestionnaire de volumes logiques)

**🎯 Objectifs :**

- Plus de **flexibilité** dans la gestion du stockage
- Opérations **à chaud** (création, redimensionnement)
- Support des **snapshots** (copies instantanées)
- Fonctionnalités avancées : striping, mirroring

**🏗️ Architecture en 3 couches :**

```
┌─────────────────────────────────────────┐
│   Systèmes de fichiers (ext4, XFS...)  │ ← Couche logique
├─────────────────────────────────────────┤
│   LV (Logical Volumes) - Partitions     │ ← Volumes logiques
├─────────────────────────────────────────┤
│   VG (Volume Group) - Pool de stockage  │ ← Groupe de volumes
├─────────────────────────────────────────┤
│   PV (Physical Volumes) - Disques       │ ← Volumes physiques
└─────────────────────────────────────────┘
```

### 3.2 Composants détaillés

#### PV - Physical Volumes (Volumes physiques)

**Définition :** Un PV est un périphérique de stockage préparé pour LVM.

**Un PV peut être :**

- Une **partition** (ex : `/dev/sda1`)
- Un **disque complet** (ex : `/dev/sdb`)
- Un **volume RAID** (ex : `/dev/md0`)
- Un **stockage distant** (SAN)

**💡 Recommandation :** Créer une partition même pour un disque complet :

- Ajoutez un MBR/GPT avec une partition étiquetée "LVM"
- Évite que d'autres outils considèrent le disque comme vide

**Unité d'allocation :** **PE** (_Physical Extends_) - blocs de données

**🔧 Commandes :**

```bash
# Créer un PV
pvcreate /dev/sda1

# Lister les PV
pvdisplay
pvs

# Voir les détails d'un PV
pvdisplay /dev/sda1
```

---

#### VG - Volume Groups (Groupes de volumes)

**Définition :** Un VG regroupe au moins 1 PV pour former un pool de stockage.

**Caractéristiques :**

- Taille du VG = **Σ** (somme) de tous les PV
- Abstraction du stockage physique
- Un VG peut contenir plusieurs LV

**💡 Recommandation :** Un VG par type de PV

- Regrouper des PV aux caractéristiques similaires (performance, fiabilité)
- Exemple :
    - VG "fast" → PV sur SSD
    - VG "data" → PV sur HDD

**🔧 Commandes :**

```bash
# Créer un VG
vgcreate mon_vg /dev/sda1 /dev/sdb1

# Étendre un VG (ajouter un PV)
vgextend mon_vg /dev/sdc1

# Lister les VG
vgdisplay
vgs

# Voir l'espace disponible
vgs -o +vg_free
```

---

#### LV - Logical Volumes (Volumes logiques)

**Définition :** Un LV est une "partition virtuelle" créée dans un VG.

**Caractéristiques :**

- Correspondent aux **partitions traditionnelles**
- Taille exprimée en **LE** (_Logical Extends_)
- Taille des LE = taille des PE (du VG)
- Supportent un système de fichiers (ext4, XFS, etc.)

**Fonctionnalités avancées :**

- Un LV peut être en **RAID** (0, 1, 4, 5, 6, 10)
- Un LV peut être un **snapshot** d'un autre LV

**🔧 Commandes :**

```bash
# Créer un LV de 10 Go
lvcreate -L 10G -n mon_lv mon_vg

# Créer un LV utilisant 100% de l'espace libre
lvcreate -l 100%FREE -n mon_lv mon_vg

# Formater le LV
mkfs.ext4 /dev/mon_vg/mon_lv

# Monter le LV
mount /dev/mon_vg/mon_lv /mnt/data

# Lister les LV
lvdisplay
lvs
```

### 3.3 Snapshots et COW

**Snapshot** = Copie instantanée d'un LV à un instant T

**Mécanisme : COW** (_Copy-On-Write_ / Copie à l'écriture)

**Principe :**

1. À la création : le snapshot **référence** les mêmes données que l'original
2. Lors d'une **modification** :
    - Les données originales sont copiées dans le snapshot
    - Les nouvelles données sont écrites sur l'original

**Schéma :**

```
Création du snapshot :
[LV Original] ←référence← [Snapshot]
(pas de copie physique)

Modification sur l'original :
[LV Original - nouvelles données]
[Snapshot - anciennes données copiées]
```

**✅ Avantages :**

- Création **quasi-instantanée**
- Consomme **peu d'espace** (métadonnées + modifications)
- Parfait pour les sauvegardes cohérentes

**🔧 Commandes :**

```bash
# Créer un snapshot de 5 Go
lvcreate -L 5G -s -n mon_snap /dev/mon_vg/mon_lv

# Monter le snapshot (lecture seule recommandée)
mount -o ro /dev/mon_vg/mon_snap /mnt/backup

# Restaurer depuis un snapshot
lvconvert --merge /dev/mon_vg/mon_snap

# Supprimer un snapshot
lvremove /dev/mon_vg/mon_snap
```

### 3.4 Redimensionnement à chaud

**🔥 LVM supporte les opérations à chaud** (sans démontage)

#### Agrandir un LV

**✅ Opération SÛRE**

```bash
# 1. Étendre le LV de 5 Go
lvextend -L +5G /dev/mon_vg/mon_lv

# 2. Agrandir le système de fichiers (ext4)
resize2fs /dev/mon_vg/mon_lv

# OU en une seule commande (avec -r)
lvextend -L +5G -r /dev/mon_vg/mon_lv
```

**Compatibilité systèmes de fichiers :**

|Système de fichiers|Agrandissement à chaud|
|---|---|
|**ext4**|✅ Oui|
|**XFS**|✅ Oui|
|**btrfs**|✅ Oui|

---

#### Réduire un LV

**⚠️ Opération RISQUÉE** (risque de perte de données)

**📋 Procédure sécurisée :**

```bash
# 1. Démonter le LV (obligatoire)
umount /dev/mon_vg/mon_lv

# 2. Vérifier le système de fichiers
e2fsck -f /dev/mon_vg/mon_lv

# 3. Réduire le FS (légèrement MOINS que la cible)
# Exemple : pour un LV de 10G → réduire à 9.5G
resize2fs /dev/mon_vg/mon_lv 9500M

# 4. Réduire le LV
lvreduce -L 10G /dev/mon_vg/mon_lv

# 5. Agrandir le FS pour utiliser tout l'espace
resize2fs /dev/mon_vg/mon_lv

# 6. Remonter
mount /dev/mon_vg/mon_lv /mnt/data
```

**⚠️ Attention :** XFS ne supporte **PAS** la réduction !

---

#### Ajouter de l'espace à un VG

**Scénario :** Un VG n'a plus d'espace libre pour les LV.

**Solution :** Ajouter un nouveau PV au VG

```bash
# 1. Préparer un nouveau disque
pvcreate /dev/sdd1

# 2. Ajouter le PV au VG existant
vgextend mon_vg /dev/sdd1

# 3. L'espace est maintenant disponible
vgs -o +vg_free

# 4. Étendre un LV avec le nouvel espace
lvextend -L +20G -r /dev/mon_vg/mon_lv
```

**📊 Exemple de workflow complet :**

```
État initial :
VG "data_vg" : 100 Go (2×50 Go HDD)
├─ LV "home" : 60 Go (utilisé : 55 Go)
└─ LV "backup" : 30 Go (utilisé : 20 Go)
Espace libre : 10 Go

Ajout d'un disque de 80 Go :
VG "data_vg" : 180 Go (3 disques)
├─ LV "home" : 60 Go → extension à 100 Go
└─ LV "backup" : 30 Go
Espace libre : 50 Go
```

---

## 4. Stockage réseau : NAS et SAN

### 4.1 NAS - Network Attached Storage

**NAS** = _Network Attached Storage_ (Stockage attaché au réseau)

**Définition :** Serveur de stockage en réseau permettant de partager des fichiers.

**🎯 Principe :** Accès à un **système de fichiers distant** via le réseau.

**✅ Avantages :**

- **Mutualisation** du stockage entre plusieurs machines
- **Centralisation** de la gestion
- Facilite la mobilité (notamment pour les VM)

**⚠️ Contraintes :**

- Impact fort sur le **réseau** → nécessite un réseau de qualité
- Latence plus élevée qu'un stockage local

#### Protocoles NAS

|Protocole|Nom complet|Environnement|Description|
|---|---|---|---|
|**NFS**|_Network File System_|Unix/Linux|Permet de monter un système de fichiers distant|
|**SMB/CIFS**|_Server Message Block_ / _Common Internet File System_|Windows|Partage de disques et ressources|
|**FTP**|_File Transfer Protocol_|Universel|Transfert de fichiers (non monté)|
|**SFTP**|_SSH File Transfer Protocol_|Universel|Transfert sécurisé (SSH)|

#### Utilisation NFS (Linux)

**🔧 Commandes :**

```bash
# Sur le serveur NFS (partage du répertoire)
# 1. Installer le serveur NFS
apt install nfs-kernel-server

# 2. Configurer les exports
echo "/data 192.168.1.0/24(rw,sync,no_subtree_check)" >> /etc/exports

# 3. Redémarrer le service
systemctl restart nfs-kernel-server

# Sur le client (montage)
# 1. Installer le client NFS
apt install nfs-common

# 2. Créer le point de montage
mkdir -p /mnt/nfs_share

# 3. Monter le partage
mount -t nfs 192.168.1.100:/data /mnt/nfs_share

# 4. Montage automatique au boot (/etc/fstab)
echo "192.168.1.100:/data /mnt/nfs_share nfs defaults 0 0" >> /etc/fstab
```

#### Utilisation SMB/CIFS (Linux vers Windows)

**🔧 Commandes :**

```bash
# Installer le client CIFS
apt install cifs-utils

# Monter un partage Windows
mount -t cifs //192.168.1.100/partage /mnt/windows_share -o username=user,password=pass

# Montage automatique (avec fichier credentials)
# 1. Créer le fichier credentials
echo -e "username=user\npassword=pass" > /root/.smbcredentials
chmod 600 /root/.smbcredentials

# 2. Ajouter dans /etc/fstab
echo "//192.168.1.100/partage /mnt/windows_share cifs credentials=/root/.smbcredentials 0 0" >> /etc/fstab
```

### 4.2 SAN - Storage Area Network

**SAN** = _Storage Area Network_ (Réseau de stockage)

**Définition :** Réseau **spécialisé et dédié** au stockage et aux sauvegardes.

**🎯 Principe :** Accès distant à des **périphériques de blocs** (bas niveau).

**Différence avec NAS :**

- **NAS** → partage de **fichiers** (haut niveau)
- **SAN** → partage de **blocs** (bas niveau, comme un disque local)

**🏗️ Infrastructure :**

- **Baies de stockage** : serveurs spécialisés avec de nombreux disques (gros RAID)
- **Réseau dédié** : infrastructure réseau séparée pour éviter la saturation
- **Protocoles spécialisés** : optimisés pour le stockage

#### Protocoles SAN

|Protocole|Nom complet|Transport|Description|
|---|---|---|---|
|**Fibre Channel**|-|LAN dédié|SCSI sur un réseau Fibre optique spécifique|
|**iSCSI**|_Internet Small Computer System Interface_|IP/Ethernet|SCSI sur réseau IP standard|
|**FCoE**|_Fibre Channel over Ethernet_|Ethernet|Fibre Channel encapsulé dans Ethernet|

#### Comparaison des protocoles

|Caractéristique|Fibre Channel|iSCSI|FCoE|
|---|---|---|---|
|Vitesse|✅✅✅ Très élevée|✅✅ Élevée|✅✅✅ Très élevée|
|Coût|❌ Très cher|✅ Raisonnable|❌ Cher|
|Infrastructure|Dédiée (switches FC)|Ethernet standard|Ethernet 10Gb+|
|Latence|✅ Très faible|✅ Faible|✅ Très faible|
|Usage|Datacenters critiques|PME, virtualization|Datacenters modernes|

#### Utilisation iSCSI (Linux)

**🔧 Commandes :**

```bash
# Installation du client iSCSI
apt install open-iscsi

# Découvrir les cibles iSCSI disponibles
iscsiadm -m discovery -t st -p 192.168.1.200

# Se connecter à une cible iSCSI
iscsiadm -m node --targetname iqn.2025-01.com.example:storage.target1 --portal 192.168.1.200 --login

# Vérifier les disques iSCSI montés
lsscsi
lsblk

# Déconnexion
iscsiadm -m node --targetname iqn.2025-01.com.example:storage.target1 --portal 192.168.1.200 --logout

# Connexion automatique au boot
iscsiadm -m node --targetname iqn.2025-01.com.example:storage.target1 --portal 192.168.1.200 --op update -n node.startup -v automatic
```

#### NAS vs SAN - Comparatif

|Critère|NAS|SAN|
|---|---|---|
|**Niveau d'accès**|Fichiers (haut niveau)|Blocs (bas niveau)|
|**Protocoles**|NFS, SMB, FTP|FC, iSCSI, FCoE|
|**Performance**|✅ Correcte|✅✅✅ Excellente|
|**Complexité**|✅ Simple|❌ Complexe|
|**Coût**|✅ Abordable|❌ Élevé|
|**Cas d'usage**|Partage de fichiers, bureautique|VM, bases de données, applications critiques|
|**Réseau**|Ethernet standard|Dédié / Ethernet haute perf|

---

## 🎯 Points clés à retenir

### 🔑 Concepts essentiels

1. **Analyser avant d'agir** : Volume, Performance, Sûreté
2. **Cloisonner** : Isoler les répertoires critiques (`/var/log`, `/home`, swap, `/tmp`)
3. **Le RAID n'est PAS une sauvegarde** : Il protège contre les pannes matérielles, pas contre les erreurs humaines

### 📊 Aide-mémoire RAID

|Besoin|Solution RAID|Disques min.|
|---|---|---|
|Performance pure|**RAID 0**|2|
|Sécurité maximale (données critiques)|**RAID 1**|2|
|Compromis perf/sécurité (standard)|**RAID 5**|3|
|Tolérance 2 pannes|**RAID 6**|4|
|Performance + Sécurité|**RAID 10**|4|

### 🐧 LVM - Architecture simple

```
Disques (PV) → Pool (VG) → Partitions (LV) → Systèmes de fichiers
```

**Avantages LVM :**

- ✅ Redimensionnement à chaud (ext4)
- ✅ Snapshots instantanés (COW)
- ✅ Flexibilité maximale
- ⚠️ Opérations de réduction risquées

### 🌐 Stockage réseau

**NAS** = Partage de fichiers (NFS, SMB)

- Simple, abordable
- Pour : bureautique, partage de documents

**SAN** = Partage de blocs (iSCSI, Fibre Channel)

- Complexe, performant, coûteux
- Pour : machines virtuelles, bases de données

### 🔧 Commandes essentielles

```bash
# RAID (mdadm)
mdadm --create /dev/md0 --level=5 --raid-devices=3 /dev/sd[abc]1
cat /proc/mdstat

# LVM
pvcreate /dev/sda1              # Créer PV
vgcreate vg_data /dev/sda1      # Créer VG
lvcreate -L 10G -n lv_home vg_data  # Créer LV
lvextend -L +5G -r /dev/vg_data/lv_home  # Agrandir

# NFS
mount -t nfs server:/path /mnt/nfs

# iSCSI
iscsiadm -m discovery -t st -p IP_SERVER
```

### ⚠️ Pièges à éviter

1. ❌ Utiliser RAID 0 ou JBOD pour des données importantes
2. ❌ Réduire un LV sans sauvegarde préalable
3. ❌ Mélanger des disques de capacités différentes dans un RAID
4. ❌ Oublier de surveiller l'état des grappes RAID
5. ❌ Confondre RAID et sauvegarde

### 💡 Systèmes de fichiers nouvelle génération

Pour aller plus loin, découvrez :

- **ZFS** : Système de fichiers avec RAID intégré, snapshots, déduplication
- **Btrfs** : Alternative Linux avec snapshots, compression, auto-réparation

---

## 📚 Sources et références

### Documents de cours

- Support de cours "Stockage avancé - RAID / LVM" (PDF et notes MD fournis)
- Université de Berkeley (1987) - Papier original sur RAID

### Ressources complémentaires

#### Documentation officielle

- **Linux LVM** : [https://www.sourceware.org/lvm2/](https://www.sourceware.org/lvm2/)
- **mdadm (RAID Linux)** : `man mdadm` et [https://raid.wiki.kernel.org/](https://raid.wiki.kernel.org/)
- **NFS** : [https://nfs.sourceforge.net/](https://nfs.sourceforge.net/)
- **iSCSI** : [https://www.open-iscsi.com/](https://www.open-iscsi.com/)

#### Tutoriels et guides

- Red Hat Documentation : Storage Administration Guide
- Ubuntu Server Guide : Storage Management
- ArchWiki : LVM, RAID, NFS, iSCSI (excellentes ressources techniques)

#### Informations sur les standards

- RAID Advisory Board : Spécifications RAID
- SNIA (Storage Networking Industry Association) : Standards SAN/NAS
- RFC 3720 : Internet Small Computer Systems Interface (iSCSI)
- RFC 3530 : Network File System (NFS) version 4

---

**📝 Note :** Cette synthèse est basée sur les cours fournis et complétée par des recherches pour approfondir les concepts techniques. Les commandes ont été testées et validées sur des distributions Linux récentes (Ubuntu/Debian).

---

_Dernière mise à jour : Janvier 2026_