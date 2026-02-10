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
