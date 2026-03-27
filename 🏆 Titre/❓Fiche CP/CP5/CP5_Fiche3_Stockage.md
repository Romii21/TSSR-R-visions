# FICHE 3 — Gestion du Stockage (RAID, LVM, NAS, SAN)
### Formation TSSR — CP5 : Maintenir des serveurs dans une infrastructure virtualisée

---

## 1. Le RAID — Redondance et performance

**RAID** = Redundant Array of Independent Disks

**Objectifs :** agrandir la capacité, améliorer les performances, améliorer la fiabilité.

> ⚠️ **Le RAID n'est PAS une sauvegarde.** Il protège contre les pannes matérielles, pas contre les erreurs humaines, les ransomwares ou les corruptions logiques.

---

### Niveaux RAID à connaître

#### RAID 0 — Striping (Entrelaçage)
```
Données → [Part 1] → Disque 1
        → [Part 2] → Disque 2
```
- Capacité utile : **n × taille d'un disque**
- Tolérance pannes : **aucune** (1 disque perdu = tout perdu)
- Performance : excellente (lecture/écriture multipliées)
- Usage : cache, édition vidéo — jamais pour des données importantes

#### RAID 1 — Mirroring (Miroir)
```
Données → [Copie complète] → Disque 1
        → [Copie complète] → Disque 2
```
- Capacité utile : **taille d'un disque** (50% de la capacité totale)
- Tolérance pannes : **n-1** (peut perdre tous les disques sauf 1)
- Performance : légère amélioration en lecture
- Usage : données critiques (OS, bases de données)

#### RAID 5 — Striping avec parité distribuée
```
Disque 1 : [Data A1] [Data B1] [Parité C]
Disque 2 : [Data A2] [Parité B] [Data C1]
Disque 3 : [Parité A] [Data B2] [Data C2]
```
- Minimum : **3 disques**
- Capacité utile : **(n-1) × taille d'un disque**
- Tolérance pannes : **1 disque**
- Performance : bonne (pas de goulot d'étranglement)
- Usage : standard en entreprise — **le plus populaire**

> **Exemple calcul :** 4 disques × 2 To → (4-1) × 2 To = **6 To utiles**

#### RAID 6 — Double parité
- Minimum : **4 disques**
- Capacité utile : **(n-2) × taille d'un disque**
- Tolérance pannes : **2 disques simultanément**
- Usage : gros volumes, données très critiques

> **Exemple calcul :** 4 disques × 2 To → (4-2) × 2 To = **4 To utiles**

#### RAID 10 (1+0) — Miroir de grappes en striping
```
RAID 1 ──→ [Disque 1] ←miroir→ [Disque 2]
               ↓ RAID 0 ↓
RAID 1 ──→ [Disque 3] ←miroir→ [Disque 4]
```
- Minimum : **4 disques**
- Capacité utile : **50% de la capacité totale**
- Tolérance pannes : plusieurs pannes (si pas sur le même miroir)
- Performance : excellente (striping + mirroring)
- Usage : serveurs haute performance avec sécurité

---

### Tableau comparatif RAID

| Niveau | Disques min. | Capacité utile | Tolérance panne | Usage typique |
|---|---|---|---|---|
| RAID 0 | 2 | 100% | ❌ Aucune | Performance pure |
| RAID 1 | 2 | 50% | ✅ n-1 disques | Données critiques |
| RAID 5 | 3 | (n-1)/n | ✅ 1 disque | Standard entreprise |
| RAID 6 | 4 | (n-2)/n | ✅✅ 2 disques | Gros volumes critiques |
| RAID 10 | 4 | 50% | ✅✅ Plusieurs | Haute perf + sécurité |

---

### Gestion RAID logiciel sous Linux (mdadm)

```bash
# Créer une grappe RAID 5 avec 3 disques
mdadm --create /dev/md0 --level=5 --raid-devices=3 /dev/sda1 /dev/sdb1 /dev/sdc1

# Vérifier l'état de la grappe
cat /proc/mdstat
mdadm --detail /dev/md0

# Ajouter un disque de spare (rechange)
mdadm --add /dev/md0 /dev/sdd1

# Marquer un disque comme défaillant
mdadm --fail /dev/md0 /dev/sdb1

# Retirer un disque défaillant
mdadm --remove /dev/md0 /dev/sdb1

# Suivre la reconstruction
watch cat /proc/mdstat
```

**Lecture de `/proc/mdstat` :**
```
md0 : active raid5 sda1[0] sdb1[1] sdc1[2]
      [3/3] [UUU]   ← U = Up (fonctionnel), _ = down (défaillant)
      [2/3] [UU_]   ← Grappe dégradée, 1 disque défaillant
```

### Procédure de remplacement d'un disque RAID 5 défaillant

```bash
# Étape 1 : Identifier le disque défaillant
cat /proc/mdstat
mdadm --detail /dev/md0

# Étape 2 : Marquer le disque comme défaillant (si pas automatique)
mdadm --fail /dev/md0 /dev/sdb1

# Étape 3 : Retirer le disque de la grappe
mdadm --remove /dev/md0 /dev/sdb1

# Étape 4 : Remplacer physiquement le disque

# Étape 5 : Partitionner le nouveau disque (type fd = Linux RAID)
fdisk /dev/sdd   # → type partition : fd

# Étape 6 : Ajouter le nouveau disque à la grappe
mdadm --add /dev/md0 /dev/sdd1

# Étape 7 : Suivre la reconstruction
watch cat /proc/mdstat
```

---

## 2. LVM — Gestionnaire de Volumes Logiques

**LVM** = Logical Volume Manager — permet une gestion **flexible et dynamique** du stockage.

### Architecture en 3 couches

```
┌─────────────────────────────────────────────────┐
│  Systèmes de fichiers (ext4, XFS, btrfs...)     │  ← Couche applicative
├─────────────────────────────────────────────────┤
│  LV — Logical Volumes (Volumes logiques)        │  ← "Partitions virtuelles"
├─────────────────────────────────────────────────┤
│  VG — Volume Group (Groupe de volumes)          │  ← Pool de stockage
├─────────────────────────────────────────────────┤
│  PV — Physical Volumes (Volumes physiques)      │  ← Disques préparés LVM
├─────────────────────────────────────────────────┤
│  Disques physiques (/dev/sda, /dev/sdb...)      │
└─────────────────────────────────────────────────┘
```

### Les trois composants en détail

**PV (Physical Volume) :**
Un disque ou une partition initialisé pour LVM avec `pvcreate`.
```bash
pvcreate /dev/sdb       # Initialiser un disque entier
pvcreate /dev/sda1      # Initialiser une partition
pvdisplay               # Lister les PV
```

**VG (Volume Group) :**
Regroupe 1 ou plusieurs PV en un pool de stockage. Sa taille = somme des PV.
```bash
vgcreate vg_data /dev/sdb /dev/sdc    # Créer un VG avec 2 disques
vgextend vg_data /dev/sdd             # Ajouter un PV à un VG existant
vgdisplay                             # Lister les VG
```

**LV (Logical Volume) :**
Partition virtuelle créée dans un VG. Équivalent d'une partition traditionnelle.
```bash
lvcreate -L 50G -n lv_home vg_data    # Créer un LV de 50 Go
lvcreate -l 100%FREE -n lv_data vg_data  # Utiliser tout l'espace libre
mkfs.ext4 /dev/vg_data/lv_home        # Formater le LV
mount /dev/vg_data/lv_home /mnt/home  # Monter le LV
```

---

### Redimensionnement LVM

#### Agrandir un LV ✅ — Opération sûre, possible à chaud

```bash
# Étendre le LV de 10 Go et redimensionner le FS en même temps
lvextend -L +10G -r /dev/vg_data/lv_home

# Sans l'option -r (deux étapes séparées)
lvextend -L +10G /dev/vg_data/lv_home
resize2fs /dev/vg_data/lv_home    # Pour ext4
```

**Support agrandissement à chaud :** ext4 ✅ | XFS ✅ | btrfs ✅

#### Réduire un LV ⚠️ — Opération risquée, uniquement ext4

```bash
# TOUJOURS sauvegarder avant de réduire !

# Étape 1 : Démonter obligatoirement
umount /dev/vg_data/lv_home

# Étape 2 : Vérifier le système de fichiers
e2fsck -f /dev/vg_data/lv_home

# Étape 3 : Réduire le FS d'abord (légèrement en dessous de la cible)
resize2fs /dev/vg_data/lv_home 9500M   # Réduire à 9.5 Go

# Étape 4 : Réduire le LV ensuite
lvreduce -L 10G /dev/vg_data/lv_home

# Étape 5 : Ajuster le FS à la taille réelle du LV
resize2fs /dev/vg_data/lv_home

# Étape 6 : Remonter
mount /dev/vg_data/lv_home /mnt/home
```

> ⚠️ **XFS ne supporte PAS la réduction.** Ordre impératif : réduire le FS AVANT le LV.

---

### Snapshots LVM

Mécanisme **COW (Copy-On-Write)** : à la création, le snapshot référence les mêmes données. Lors d'une modification, les anciennes données sont copiées dans le snapshot.

```bash
# Créer un snapshot de 5 Go
lvcreate -L 5G -s -n snap_home /dev/vg_data/lv_home

# Monter le snapshot en lecture seule
mount -o ro /dev/vg_data/snap_home /mnt/backup

# Restaurer depuis un snapshot (la VM doit être arrêtée)
lvconvert --merge /dev/vg_data/snap_home

# Supprimer un snapshot devenu inutile
lvremove /dev/vg_data/snap_home
```

---

## 3. NAS vs SAN — Stockage réseau

### NAS (Network Attached Storage)
Serveur de stockage partageant des **fichiers** via le réseau.

**Protocoles :** NFS (Linux), SMB/CIFS (Windows), FTP/SFTP

| ✅ Avantages | ⚠️ Inconvénients |
|---|---|
| Simple à mettre en place | Performances limitées par le réseau |
| Abordable | Latence plus élevée qu'un disque local |
| Centralisation du stockage | Inadapté aux applications très exigeantes |

**Usage :** Partage de fichiers, sauvegardes, bureautique

---

### SAN (Storage Area Network)
Réseau spécialisé et dédié au stockage, partageant des **blocs** (bas niveau, comme un disque local).

**Protocoles :**

| Protocole | Transport | Caractéristiques |
|---|---|---|
| **Fibre Channel** | Réseau fibre optique dédié | Ultra-performant, très coûteux |
| **iSCSI** | IP/Ethernet standard | Performances élevées, coût raisonnable |
| **FCoE** | Ethernet 10 Gb+ | Fibre Channel sur Ethernet, coûteux |

| ✅ Avantages | ⚠️ Inconvénients |
|---|---|
| Performances excellentes | Infrastructure complexe |
| Accès bas niveau (bloc) | Coût élevé |
| Idéal pour les VMs | Compétences spécialisées requises |

---

### NAS vs SAN — Tableau de décision

| Critère | NAS | SAN |
|---|---|---|
| **Niveau d'accès** | Fichiers (haut niveau) | Blocs (bas niveau) |
| **Protocoles** | NFS, SMB, FTP | iSCSI, FC, FCoE |
| **Performance** | Correcte | Excellente |
| **Complexité** | Faible | Élevée |
| **Coût** | Abordable | Élevé |
| **VMs de production** | ⚠️ Possible mais limité | ✅ Recommandé |
| **Partage de fichiers** | ✅ Idéal | ❌ Surdimensionné |

> **Pour héberger des disques de VMs en production → SAN (iSCSI) est recommandé.** L'accès bloc offre des performances proches d'un disque local.

---

## 4. Aide-mémoire — Calculs RAID

| Configuration | Formule | Exemple (disques de 2 To) |
|---|---|---|
| RAID 0 (n disques) | n × taille | 4 × 2 To = **8 To** |
| RAID 1 (2 disques) | 1 × taille | 1 × 2 To = **2 To** |
| RAID 5 (n disques) | (n-1) × taille | (4-1) × 2 To = **6 To** |
| RAID 6 (n disques) | (n-2) × taille | (4-2) × 2 To = **4 To** |
| RAID 10 (n disques) | n/2 × taille | 4/2 × 2 To = **4 To** |

---
*CP5 — Fiche 3/5 — Gestion du Stockage (RAID, LVM, NAS, SAN)*
