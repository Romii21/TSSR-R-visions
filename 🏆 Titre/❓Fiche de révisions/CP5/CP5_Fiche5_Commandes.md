# FICHE 5 — Commandes Essentielles & Aide-Mémoire
### Formation TSSR — CP5 : Maintenir des serveurs dans une infrastructure virtualisée

---

## 1. Commandes RAID (mdadm)

### Création et configuration

```bash
# Créer un RAID 5 avec 3 disques
mdadm --create /dev/md0 --level=5 --raid-devices=3 /dev/sda1 /dev/sdb1 /dev/sdc1

# Créer un RAID 1 avec 2 disques
mdadm --create /dev/md0 --level=1 --raid-devices=2 /dev/sda1 /dev/sdb1

# Créer un RAID 6 avec 4 disques
mdadm --create /dev/md0 --level=6 --raid-devices=4 /dev/sda1 /dev/sdb1 /dev/sdc1 /dev/sdd1
```

### Surveillance et état

```bash
# Afficher l'état en temps réel de toutes les grappes
cat /proc/mdstat

# Détails complets d'une grappe
mdadm --detail /dev/md0

# Surveiller la reconstruction
watch -n2 cat /proc/mdstat
```

### Gestion des disques défaillants

```bash
# Marquer un disque comme défaillant
mdadm --fail /dev/md0 /dev/sdb1

# Retirer un disque défaillant de la grappe
mdadm --remove /dev/md0 /dev/sdb1

# Ajouter un nouveau disque (remplacement ou spare)
mdadm --add /dev/md0 /dev/sdd1

# Ajouter un disque de spare (rechange automatique)
mdadm --add /dev/md0 /dev/sde1
```

### Configuration persistante

```bash
# Sauvegarder la configuration RAID
mdadm --detail --scan >> /etc/mdadm/mdadm.conf

# Assembler une grappe au démarrage
mdadm --assemble /dev/md0 /dev/sda1 /dev/sdb1 /dev/sdc1
```

---

## 2. Commandes LVM

### Physical Volumes (PV)

```bash
pvcreate /dev/sdb          # Initialiser un disque pour LVM
pvdisplay                  # Détails des PV
pvs                        # Résumé des PV
pvremove /dev/sdb          # Supprimer un PV
```

### Volume Groups (VG)

```bash
vgcreate vg_data /dev/sdb /dev/sdc   # Créer un VG avec 2 PV
vgextend vg_data /dev/sdd            # Ajouter un PV à un VG
vgdisplay                            # Détails des VG
vgs                                  # Résumé des VG
vgs -o +vg_free                      # Voir l'espace libre dans les VG
vgreduce vg_data /dev/sdd            # Retirer un PV d'un VG
vgremove vg_data                     # Supprimer un VG
```

### Logical Volumes (LV)

```bash
# Création
lvcreate -L 50G -n lv_home vg_data          # LV de 50 Go
lvcreate -l 100%FREE -n lv_data vg_data     # LV utilisant tout l'espace libre
lvcreate -l 50%FREE -n lv_tmp vg_data       # LV utilisant 50% de l'espace libre

# Formatage et montage
mkfs.ext4 /dev/vg_data/lv_home             # Formater en ext4
mkfs.xfs /dev/vg_data/lv_data              # Formater en XFS
mount /dev/vg_data/lv_home /mnt/home       # Monter le LV
umount /mnt/home                            # Démonter le LV

# Agrandissement (sûr, à chaud)
lvextend -L +10G /dev/vg_data/lv_home      # Étendre de 10 Go
lvextend -L +10G -r /dev/vg_data/lv_home  # Étendre + FS en même temps (-r)
resize2fs /dev/vg_data/lv_home             # Étendre le FS ext4 seul

# Réduction (risqué, hors ligne uniquement, ext4 seulement)
umount /dev/vg_data/lv_home
e2fsck -f /dev/vg_data/lv_home
resize2fs /dev/vg_data/lv_home 9G
lvreduce -L 10G /dev/vg_data/lv_home
resize2fs /dev/vg_data/lv_home
mount /dev/vg_data/lv_home /mnt/home

# Affichage
lvdisplay                                   # Détails des LV
lvs                                         # Résumé des LV

# Suppression
lvremove /dev/vg_data/lv_home              # Supprimer un LV
```

### Snapshots LVM

```bash
# Créer un snapshot de 5 Go
lvcreate -L 5G -s -n snap_home /dev/vg_data/lv_home

# Monter le snapshot en lecture seule
mount -o ro /dev/vg_data/snap_home /mnt/backup

# Restaurer depuis un snapshot (VM arrêtée)
lvconvert --merge /dev/vg_data/snap_home

# Supprimer un snapshot
lvremove /dev/vg_data/snap_home
```

---

## 3. Commandes réseau utiles (depuis une VM)

```bash
# Interfaces réseau
ip a                        # Afficher toutes les interfaces et IPs
ip link show                # État des interfaces (UP/DOWN)
ip route show               # Table de routage

# Connectivité
ping <IP>                   # Test ICMP de base
traceroute <IP>             # Chemin vers une destination
curl -I http://<IP>         # Test HTTP (entêtes)

# Ports et services
ss -tulnp                   # Ports en écoute (TCP/UDP)
ss -tnp                     # Connexions TCP établies
netstat -tulnp              # Alternatif à ss (ancienne commande)

# DNS
nslookup <nom>              # Résolution DNS simple
dig <nom>                   # Résolution DNS détaillée
host <nom>                  # Résolution DNS rapide
```

---

## 4. Commandes de surveillance système

```bash
# CPU et mémoire
top                         # Monitoring en temps réel
htop                        # Version améliorée de top (interactive)
free -h                     # Utilisation mémoire (human readable)
vmstat 1 5                  # Stats VM : CPU, mémoire, I/O (5 itérations, 1s)
uptime                      # Charge système et temps de fonctionnement

# Disques et stockage
df -h                       # Espace disque par partition
du -sh /chemin/*            # Taille des répertoires
lsblk                       # Arborescence des disques et partitions
fdisk -l                    # Liste des disques et partitions
iostat -x 1                 # Statistiques I/O disque

# Processus
ps aux                      # Tous les processus en cours
ps aux --sort=-%mem         # Triés par consommation mémoire
ps aux --sort=-%cpu         # Triés par consommation CPU
kill -9 <PID>               # Tuer un processus forcément
```

---

## 5. Commandes de journalisation (logs)

```bash
# Logs système
journalctl -xe              # Logs récents avec détails d'erreur
journalctl -u <service>     # Logs d'un service spécifique
journalctl -f               # Logs en temps réel (follow)
journalctl --since "1 hour ago"  # Logs de la dernière heure

# Fichiers de logs classiques
tail -f /var/log/syslog     # Logs système en temps réel
tail -f /var/log/auth.log   # Logs d'authentification
tail -100 /var/log/syslog   # 100 dernières lignes
grep "error" /var/log/syslog  # Filtrer les erreurs

# Services
systemctl status <service>  # État d'un service
systemctl restart <service> # Redémarrer un service
systemctl enable <service>  # Activer au démarrage
systemctl disable <service> # Désactiver au démarrage
systemctl list-units --failed  # Services en erreur
```

---

## 6. Commandes iSCSI (connexion à un SAN)

```bash
# Installation
apt install open-iscsi

# Découverte des cibles iSCSI disponibles
iscsiadm -m discovery -t st -p <IP_SAN>

# Connexion à une cible
iscsiadm -m node --targetname iqn.xxxx --portal <IP_SAN> --login

# Connexion automatique au boot
iscsiadm -m node --targetname iqn.xxxx --portal <IP_SAN> \
  --op update -n node.startup -v automatic

# Déconnexion
iscsiadm -m node --targetname iqn.xxxx --portal <IP_SAN> --logout

# Vérifier les disques iSCSI montés
lsscsi
lsblk
```

---

## 7. Aide-mémoire — Calculs RAID rapides

```
RAID 0  : n × taille_disque
RAID 1  : taille_disque
RAID 5  : (n - 1) × taille_disque      → tolère 1 panne
RAID 6  : (n - 2) × taille_disque      → tolère 2 pannes
RAID 10 : (n / 2) × taille_disque      → tolère plusieurs pannes

Exemples avec 4 disques × 2 To :
  RAID 0  → 4 × 2 = 8 To
  RAID 1  → 2 To  (2 disques seulement)
  RAID 5  → 3 × 2 = 6 To
  RAID 6  → 2 × 2 = 4 To
  RAID 10 → 2 × 2 = 4 To
```

---

## 8. Tableau récapitulatif — Quoi utiliser quand

| Besoin | Solution | Commande clé |
|---|---|---|
| Redondance disque basique | RAID 1 | `mdadm --create --level=1` |
| Compromis capacité/sécurité | RAID 5 | `mdadm --create --level=5` |
| Double tolérance panne | RAID 6 | `mdadm --create --level=6` |
| Haute perf + sécurité | RAID 10 | `mdadm --create --level=10` |
| Stockage flexible | LVM | `pvcreate → vgcreate → lvcreate` |
| Partage de fichiers réseau | NAS (NFS/SMB) | `mount -t nfs` |
| Disques VMs haute perf | SAN (iSCSI) | `iscsiadm -m discovery` |
| Sauvegarde VMs | PBS | Interface Proxmox / Web PBS |
| Vérifier RAID | mdadm | `cat /proc/mdstat` |
| Agrandir un LV | LVM | `lvextend -L +xG -r` |
| Surveiller une VM | Linux | `htop`, `free -h`, `df -h` |

---

## 9. Pièges classiques à éviter

| ⛔ À ne PAS faire | ✅ Ce qu'il faut faire |
|---|---|
| Utiliser un snapshot comme sauvegarde | Appliquer la règle 3-2-1 avec PBS |
| Réduire un LV sans sauvegarder | Toujours sauvegarder avant, puis réduire le FS avant le LV |
| Réduire un XFS | Impossible : XFS ne supporte pas la réduction |
| Mettre à jour un hôte Proxmox sans migrer les VMs | Migrer les VMs à chaud, puis mettre à jour en maintenance |
| Confondre RAID et sauvegarde | RAID = disponibilité, sauvegarde = protection des données |
| Cluster à 2 nœuds en production | Minimum 3 nœuds pour le quorum |
| Supprimer le template d'un clone lié | Le clone lié dépend du template, suppression = perte du clone |
| Mixer des disques de tailles différentes en RAID | Seule la taille du plus petit disque est utilisée |
| Mélanger nœuds de types CPU différents pour HA | Incompatibilités possibles lors de la migration à chaud |

---
*CP5 — Fiche 5/5 — Commandes Essentielles & Aide-Mémoire*
