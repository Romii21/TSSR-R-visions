# 🐧 FICHE 2 – Outils de Sauvegarde sous Linux
### CP8 – Mettre en place, assurer et tester les sauvegardes et restaurations

---

## 1. rsync – Synchronisation et sauvegarde incrémentale

### Principe

`rsync` est l'outil de référence sous Linux pour les sauvegardes. Il transfère uniquement les **blocs modifiés** (algorithme delta), ce qui le rend très efficace pour des sauvegardes répétées.

### Options essentielles

| Option | Signification |
|--------|--------------|
| `-a` | Archive : récursif + conservation des permissions, horodatages, liens symboliques |
| `-v` | Verbose : affiche les fichiers traités |
| `-z` | Compression pendant le transfert (réseau) |
| `--delete` | Supprime sur la destination les fichiers absents de la source |
| `-n` ou `--dry-run` | Simulation sans rien modifier (toujours tester d'abord) |
| `--exclude` | Exclure des fichiers ou dossiers |
| `-e ssh` | Utilise SSH comme tunnel de transport |

### Exemples de commandes

```bash
# Sauvegarde locale simple
rsync -av /var/www/ /backup/www/

# Sauvegarde vers un serveur distant via SSH
rsync -avz --delete /var/www/ backup-srv:/sauvegardes/web/

# Sauvegarde avec exclusions
rsync -av --exclude='*.tmp' --exclude='cache/' /données/ /backup/données/

# Simulation d'abord (dry-run)
rsync -av --dry-run /var/www/ backup-srv:/sauvegardes/web/
```

> ⚠️ **Attention au slash final :**
> - `rsync -a /var/www/ /backup/` → copie le **contenu** de www dans /backup/
> - `rsync -a /var/www /backup/` → copie le **dossier** www dans /backup/ (crée /backup/www/)

---

## 2. tar – Archivage et compression

### Principe

`tar` (Tape ARchive) crée des archives compressées d'un répertoire. Contrairement à rsync, il produit un **fichier archive unique** (`.tar.gz`, `.tar.bz2`).

### Options essentielles

| Option | Signification |
|--------|--------------|
| `-c` | Créer une archive |
| `-x` | Extraire une archive |
| `-z` | Compression gzip (`.gz`) |
| `-j` | Compression bzip2 (`.bz2`, meilleure compression) |
| `-f` | Spécifier le fichier d'archive |
| `-v` | Verbose |
| `-C` | Dossier de destination à l'extraction |
| `-t` | Lister le contenu sans extraire |

### Exemples de commandes

```bash
# Créer une archive compressée de /etc
tar -czf /backup/etc_backup.tar.gz /etc

# Lister le contenu d'une archive sans extraire
tar -tzf /backup/etc_backup.tar.gz

# Extraire dans /tmp/restauration
mkdir -p /tmp/restauration
tar -xzf /backup/etc_backup.tar.gz -C /tmp/restauration

# Archiver avec bzip2 (meilleure compression)
tar -cjf /backup/etc_backup.tar.bz2 /etc

# Archiver en excluant un sous-dossier
tar -czf /backup/home.tar.gz --exclude='/home/user/cache' /home
```

> 💡 **Bonne pratique :** Ajouter la date dans le nom de l'archive pour versionner :
> ```bash
> tar -czf /backup/etc_$(date +%Y%m%d).tar.gz /etc
> ```

---

## 3. Snapshots LVM et mécanisme COW

### Rappel du mécanisme COW (Copy-On-Write)

```
À la création du snapshot :
  [LV Original] ←──── référence ────→ [Snapshot LV]
  (aucune donnée copiée physiquement)

Lors d'une modification sur l'original :
  1. L'ancienne donnée est copiée dans le snapshot
  2. La nouvelle donnée est écrite sur l'original

  [LV Original] → nouvelles données
  [Snapshot LV] → anciennes données conservées
```

### Séquence complète de sauvegarde via snapshot LVM

```bash
# Étape 1 : Créer un snapshot de 5 Go du LV à sauvegarder
lvcreate -L 5G -s -n snap_data /dev/vg_data/lv_data
#           │   │ └─ nom du snapshot
#           │   └─ -s = snapshot
#           └─ taille réservée pour les blocs modifiés

# Étape 2 : Monter le snapshot en lecture seule
mkdir -p /mnt/snap
mount -o ro /dev/vg_data/snap_data /mnt/snap

# Étape 3 : Effectuer la sauvegarde depuis le snapshot (données cohérentes)
tar -czf /backup/data_$(date +%Y%m%d).tar.gz /mnt/snap/

# Étape 4 : Démonter et supprimer le snapshot
umount /mnt/snap
lvremove -f /dev/vg_data/snap_data
```

### Points importants sur les snapshots LVM

| Point | Détail |
|-------|--------|
| **Taille du snapshot** | Doit être suffisante pour contenir toutes les modifications pendant la sauvegarde |
| **Si snapshot plein** | Il devient invalide → la sauvegarde échoue |
| **Surveillance** | Vérifier avec `lvs` ou `lvdisplay` le taux de remplissage |
| **Lecture seule** | Toujours monter en `-o ro` pour éviter toute modification accidentelle |

---

## 4. Automatisation avec cron

### Principe de cron

`cron` est le planificateur de tâches Linux. Chaque ligne d'un fichier crontab définit une tâche planifiée.

### Syntaxe crontab

```
┌───────── minute (0-59)
│ ┌─────── heure (0-23)
│ │ ┌───── jour du mois (1-31)
│ │ │ ┌─── mois (1-12)
│ │ │ │ ┌─ jour de la semaine (0-7, 0 et 7 = dimanche)
│ │ │ │ │
* * * * *  commande à exécuter
```

### Exemples pratiques

```bash
# Ouvrir l'éditeur crontab de l'utilisateur courant
crontab -e

# Sauvegarde rsync tous les jours à 23h00
0 23 * * * rsync -az --delete /données/ /backup/données/

# Sauvegarde complète tar chaque dimanche à 2h du matin
0 2 * * 0 tar -czf /backup/full_$(date +\%Y\%m\%d).tar.gz /données/

# Sauvegarde incrémentale rsync avec horodatage (lundi à samedi à 23h)
0 23 * * 1-6 rsync -az /données/ /backup/$(date +\%Y\%m\%d)/

# Afficher les tâches cron en cours
crontab -l
```

> ⚠️ **Attention dans crontab :** Le `%` doit être échappé avec `\%` dans les commandes cron.

### Outil avancé : rsnapshot

`rsnapshot` est un outil basé sur rsync qui gère automatiquement les rotations de sauvegardes (daily, weekly, monthly) avec des liens matériels pour économiser l'espace.

```bash
# Installation
apt install rsnapshot

# Configuration dans /etc/rsnapshot.conf
snapshot_root   /backup/
retain  daily   7      # 7 sauvegardes quotidiennes
retain  weekly  4      # 4 sauvegardes hebdomadaires
backup  /données/      localhost/données/

# Exécution manuelle
rsnapshot daily
rsnapshot weekly
```

---

## 5. Sauvegarde à chaud vs à froid

### Définitions

| | Sauvegarde à chaud | Sauvegarde à froid |
|--|-------------------|-------------------|
| **Service** | Actif pendant la sauvegarde | Arrêté pendant la sauvegarde |
| **Cohérence** | ⚠️ Risque d'incohérence (fichiers ouverts) | ✅ Cohérence garantie à 100% |
| **Continuité** | ✅ Aucune interruption de service | ❌ Interruption nécessaire |
| **Complexité** | Plus complexe (gestion des fichiers ouverts) | Simple |

### Exemples d'utilisation justifiée

**Sauvegarde à chaud :**
- Un serveur web Apache : les fichiers `/var/www` ne sont généralement pas modifiés pendant la sauvegarde → `rsync` pendant que le service tourne est acceptable
- Un serveur de fichiers NFS avec snapshot LVM : le snapshot garantit la cohérence même si des fichiers sont ouverts

**Sauvegarde à froid :**
```bash
# Exemple : sauvegarde à froid d'une base MySQL
systemctl stop mysql
tar -czf /backup/mysql_data_$(date +%Y%m%d).tar.gz /var/lib/mysql/
systemctl start mysql
```
- Cas justifié : base de données petite, tolérance à une courte interruption, besoin de cohérence absolue

### Gestion des fichiers ouverts (sauvegarde à chaud)

Pour les bases de données, ne jamais copier directement les fichiers à chaud. Utiliser les outils dédiés :

```bash
# MySQL/MariaDB : dump cohérent à chaud (InnoDB)
mysqldump -u root -p --single-transaction --all-databases > /backup/mysql_$(date +%Y%m%d).sql

# PostgreSQL : dump cohérent
pg_dumpall > /backup/pgsql_$(date +%Y%m%d).sql
```

---

## 🎯 Récapitulatif des commandes clés

| Outil                  | Usage principal                            | Commande type                                             |
| ---------------------- | ------------------------------------------ | --------------------------------------------------------- |
| **rsync**              | Sauvegarde incrémentale locale ou distante | `rsync -avz --delete /src/ dest:/path/`                   |
| **tar**                | Archive compressée d'un répertoire         | `tar -czf /backup/archive.tar.gz /src/`                   |
| **tar** (restauration) | Extraction d'une archive                   | `tar -xzf /backup/archive.tar.gz -C /dest/`               |
| **lvcreate -s**        | Snapshot LVM                               | `lvcreate -L 5G -s -n snap /dev/vg/lv`                    |
| **lvremove**           | Suppression snapshot                       | `lvremove -f /dev/vg/snap`                                |
| **crontab -e**         | Planifier une sauvegarde                   | `0 23 * * * rsync ...`                                    |
| **mysqldump**          | Dump cohérent MySQL                        | `mysqldump -u root -p --single-transaction db > dump.sql` |
