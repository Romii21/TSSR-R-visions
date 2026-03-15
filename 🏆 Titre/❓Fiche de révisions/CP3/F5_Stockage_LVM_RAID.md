# 📘 Fiche 5 — Stockage, Partitions & LVM
### Formation TSSR | CP3 — Exploiter des serveurs Linux

---

## 1. Disques et partitions

### Commandes d'exploration

```bash
lsblk                          # Lister les disques et partitions (arborescente)
lsblk -f                       # Avec les systèmes de fichiers et UUID
fdisk -l                       # Liste détaillée des partitions
blkid                          # UUID et type de toutes les partitions
df -h                          # Espace disque utilisé/disponible par partition
du -sh /var/log/               # Taille d'un répertoire
```

### MBR vs GPT

| Critère | MBR | GPT |
|---------|-----|-----|
| Origine | 1983 (ancien) | Moderne (UEFI) |
| Partitions max | 4 primaires | 128 partitions |
| Taille disque max | **2 To** | ~9,4 Zo (illimité en pratique) |
| Redondance table | ❌ Non | ✅ Oui (dupliquée) |
| Compatible | BIOS | UEFI (BIOS via CSM) |

> **Recommandation** : **GPT** sur tous les systèmes modernes. Obligatoire pour les disques > 2 To.

---

## 2. Systèmes de fichiers

### Les principaux FS

| FS | OS natif | Points forts |
|----|----------|-------------|
| **ext4** | Linux | Journalisation, stable, très répandu |
| **xfs** | Linux (RHEL) | Haute performance, gros volumes |
| **NTFS** | Windows | Droits ACL, chiffrement, gros fichiers |
| **FAT32** | Universel | Compatible partout, max fichier 4 Go |
| **swap** | Linux | Espace d'échange (extension RAM sur disque) |

```bash
# Créer un système de fichiers
mkfs.ext4 /dev/sdb1            # Formater en ext4
mkfs.xfs /dev/sdb1             # Formater en xfs

# Monter/démonter
mount /dev/sdb1 /mnt/data      # Montage manuel temporaire
umount /mnt/data               # Démontage

# Montage permanent → /etc/fstab
UUID=xxxx   /mnt/data   ext4   defaults   0   2
```

### Pourquoi utiliser l'UUID plutôt que /dev/sdb1 ?

```
/dev/sdb1 peut CHANGER selon l'ordre de détection au démarrage
(ajout d'un disque, changement de port SATA...)

UUID = identifiant unique IMMUABLE, attribué à la création de la partition
→ Toujours préférer l'UUID dans /etc/fstab
```

---

## 3. RAID

Le RAID (**R**edundant **A**rray of **I**ndependent **D**isks) combine plusieurs disques pour la performance et/ou la redondance.

### Les niveaux principaux

```
RAID 0 — Agrégation (Striping)
┌──────┬──────┐
│Disk 1│Disk 2│   Données réparties sur les 2 disques
│ A1   │ A2   │   ✅ Performance maximale (lecture+écriture)
│ B1   │ B2   │   ❌ Aucune redondance : 1 panne = tout perdu
└──────┴──────┘   Minimum : 2 disques

RAID 1 — Miroir (Mirroring)
┌──────┬──────┐
│Disk 1│Disk 2│   Copie identique sur les 2 disques
│  A   │  A   │   ✅ Survie à 1 panne
│  B   │  B   │   ✅ Lecture rapide (lecture sur les 2)
└──────┴──────┘   Capacité utile = 50%
                  Minimum : 2 disques

RAID 5 — Striping + Parité répartie
┌──────┬──────┬──────┐
│Disk 1│Disk 2│Disk 3│   Données + parité réparties
│  A1  │  A2  │  Ap  │   ✅ Survie à 1 panne
│  B1  │  Bp  │  B2  │   ✅ Bonne performance lecture
│  Cp  │  C1  │  C2  │   Capacité utile = (N-1)/N disques
└──────┴──────┴──────┘   Minimum : 3 disques

RAID 6 — Striping + Double parité
┌──────┬──────┬──────┬──────┐
│Disk 1│Disk 2│Disk 3│Disk 4│   Deux blocs de parité
│  A1  │  A2  │  Ap  │  Aq  │   ✅ Survie à 2 pannes simultanées
└──────┴──────┴──────┴──────┘   Minimum : 4 disques
```

### Tableau récapitulatif

| RAID | Disques min | Tolère pannes | Capacité utile | Cas d'usage |
|------|------------|---------------|---------------|-------------|
| **0** | 2 | 0 | 100% | Performances (vidéo, scratch) |
| **1** | 2 | 1 | 50% | Serveurs critiques peu de données |
| **5** | 3 | 1 | (N-1)/N | Usage général serveur |
| **6** | 4 | 2 | (N-2)/N | Stockage critique (> 4 disques) |
| **10** | 4 | 1 par paire | 50% | Base de données haute perf |

---

## 4. LVM — Logical Volume Manager

Le LVM ajoute une **couche d'abstraction** entre les disques physiques et les partitions logiques, permettant de les gérer dynamiquement (redimensionnement à chaud, ajout de disques…).

### Les 3 niveaux

```
┌─────────────────────────────────────────────────────────┐
│                                                         │
│  NIVEAU 1 — PV (Physical Volume)                       │
│  Disques ou partitions initialisés pour LVM            │
│  /dev/sda1    /dev/sdb    /dev/sdc1                    │
│                                                         │
│  NIVEAU 2 — VG (Volume Group)                          │
│  Pool de stockage regroupant les PV                    │
│  ┌─────────────────────────────────────┐               │
│  │         vg_data (500 Go)            │               │
│  │  /dev/sda1 (200G) + /dev/sdb (300G) │               │
│  └─────────────────────────────────────┘               │
│                                                         │
│  NIVEAU 3 — LV (Logical Volume)                        │
│  "Partitions virtuelles" créées dans un VG             │
│  ┌──────────────┬──────────────┬──────────┐            │
│  │  lv_web      │  lv_base     │  lv_logs │            │
│  │  100 Go      │  200 Go      │  50 Go   │            │
│  └──────────────┴──────────────┴──────────┘            │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

### Commandes LVM

```bash
# Initialiser un disque en PV
pvcreate /dev/sdb
pvs                                # Lister les PV

# Créer un VG
vgcreate vg_data /dev/sdb
vgs                                # Lister les VG

# Créer un LV
lvcreate -n lv_web -L 100G vg_data
lvs                                # Lister les LV

# Formater et monter le LV
mkfs.ext4 /dev/vg_data/lv_web
mount /dev/vg_data/lv_web /var/www/

# Étendre un LV (à chaud !)
lvextend -L +50G /dev/vg_data/lv_web
resize2fs /dev/vg_data/lv_web       # Étendre le FS ext4

# Ajouter un disque au VG
pvcreate /dev/sdc
vgextend vg_data /dev/sdc
```

---

## 5. Points clés à retenir

```
✅ df -h           →  espace disque disponible par partition
✅ du -sh /dossier →  taille d'un dossier
✅ blkid           →  afficher les UUID
✅ GPT > MBR       →  plus de 4 partitions, plus de 2 To
✅ UUID dans fstab →  stable (≠ /dev/sdb1 qui peut changer)
✅ RAID 0          →  performance, zéro redondance
✅ RAID 1          →  miroir, survie 1 panne
✅ RAID 5          →  compromis perf/redondance, min 3 disques
✅ RAID 6          →  survie 2 pannes, min 4 disques
✅ LVM PV          →  disque/partition physique initialisé
✅ LVM VG          →  pool de stockage (regroupe les PV)
✅ LVM LV          →  partition virtuelle dans le VG
✅ LVM avantage    →  redimensionnement à chaud, ajout de disque
```
