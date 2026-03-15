# 🔴 FICHE 4 – Sauvegarde des Services Critiques
### CP8 – Mettre en place, assurer et tester les sauvegardes et restaurations

---

## 1. Active Directory – Sauvegarde et restauration

### Pourquoi c'est critique

L'Active Directory est le **cœur du système d'information** Windows. Il gère :
- L'authentification de tous les utilisateurs et ordinateurs
- Les GPO appliquées à l'ensemble du parc
- Les droits d'accès aux ressources

Une corruption ou perte de l'AD rend **l'ensemble du SI inopérant**. De plus, l'AD se réplique entre plusieurs contrôleurs de domaine (DC) : une erreur répliquée sur tous les DC peut devenir irréversible sans sauvegarde préalable.

### Ce qu'il faut sauvegarder

| Composant | Emplacement | Méthode |
|-----------|-------------|---------|
| **NTDS.dit** | `C:\Windows\NTDS\` | wbadmin System State |
| **SYSVOL** | `C:\Windows\SYSVOL\` | wbadmin System State |
| **Registre** | `C:\Windows\System32\config\` | wbadmin System State |

> ⚠️ **Le fichier NTDS.dit est toujours ouvert et verrouillé** quand le DC tourne. Il est impossible de le copier directement. Il faut obligatoirement passer par le **System State** ou un outil compatible VSS.

### Sauvegarder le System State d'un DC

```cmd
:: Sauvegarde vers un disque local (E:)
wbadmin start systemstatebackup -backupTarget:E: -quiet

:: Vérifier les sauvegardes disponibles
wbadmin get versions -backupTarget:E:
```

### Les deux types de restauration AD

#### Restauration Non-Authoritative (par défaut)

```
Quand utiliser : DC tombé en panne ou corrompu, les autres DC sont intacts.

Processus :
1. Démarrer en mode de restauration des services d'annuaire (DSRM)
   → Au démarrage : F8 → "Mode de restauration des services d'annuaire"
2. Restaurer le System State
   wbadmin start systemstaterecovery -version:MM/JJ/AAAA-HH:MM
3. Redémarrer normalement
4. Le DC restauré se synchronise automatiquement avec les autres DC (réplication)
   → Les données restaurées sont écrasées par les données récentes des autres DC
```

#### Restauration Authoritative

```
Quand utiliser : Suppression accidentelle d'objets AD (OU, utilisateurs, GPO)
répliquée sur TOUS les DC. On veut forcer la restauration d'une version antérieure.

Processus :
1. Démarrer en DSRM
2. Restaurer le System State (restauration non-authoritative d'abord)
3. Avant de redémarrer normalement, utiliser ntdsutil pour marquer
   les objets comme "authoritative" :

   ntdsutil
   activate instance ntds
   authoritative restore
   restore subtree "OU=Utilisateurs,DC=domaine,DC=local"
   quit
   quit

4. Redémarrer → les objets marqués sont répliqués sur tous les DC
   (ils "écrasent" les suppressions sur les autres DC)
```

| Type | Scénario | Résultat |
|------|----------|----------|
| **Non-Authoritative** | DC en panne, autres DC intacts | Resynchronisation depuis les autres DC |
| **Authoritative** | Suppression accidentelle répliquée | Restauration d'objets spécifiques, propagée sur tous les DC |

---

## 2. Bases de données MySQL / MariaDB

### Pourquoi la copie directe des fichiers est insuffisante

MySQL/MariaDB écrit ses données en continu dans des fichiers binaires. Si on copie ces fichiers pendant que la base tourne :
- Des **transactions sont en cours** (ni commitées ni rollbackées)
- Des **tampons mémoire** n'ont pas été vidés sur disque
- Les fichiers copiés seront dans un état **incohérent** → restauration impossible ou base corrompue

### mysqldump – La solution standard

```bash
# Sauvegarder toutes les bases de données
mysqldump -u root -p --all-databases > /backup/mysql_all_$(date +%Y%m%d).sql

# Sauvegarder une seule base
mysqldump -u root -p ma_base > /backup/ma_base_$(date +%Y%m%d).sql

# Sauvegarde cohérente à chaud pour InnoDB (RECOMMANDÉ en production)
# --single-transaction : démarre une transaction pour assurer la cohérence sans verrouiller
mysqldump -u root -p --single-transaction --all-databases > /backup/dump_$(date +%Y%m%d).sql

# Inclure les procédures stockées et triggers
mysqldump -u root -p --routines --triggers --all-databases > /backup/dump_complet.sql
```

> 💡 **`--single-transaction`** : Indispensable pour les tables **InnoDB** en production. La base reste accessible pendant la sauvegarde (pas de verrouillage).

### Restauration d'un dump SQL

```bash
# Restaurer toutes les bases
mysql -u root -p < /backup/mysql_all_20240115.sql

# Restaurer une base spécifique
mysql -u root -p ma_base < /backup/ma_base_20240115.sql

# Créer la base si elle n'existe pas, puis restaurer
mysql -u root -p -e "CREATE DATABASE ma_base;"
mysql -u root -p ma_base < /backup/ma_base_20240115.sql
```

### Outil avancé : Percona XtraBackup

Pour les très grandes bases (plusieurs Go) où `mysqldump` serait trop lent :
- Sauvegarde **à chaud et incrémentale** sans interruption
- Sauvegarde des fichiers binaires directement (plus rapide à restaurer)

```bash
# Installation
apt install percona-xtrabackup-80

# Sauvegarde complète
xtrabackup --backup --target-dir=/backup/mysql/full/

# Restauration
xtrabackup --prepare --target-dir=/backup/mysql/full/
xtrabackup --copy-back --target-dir=/backup/mysql/full/
```

---

## 3. Machines virtuelles VMware ESXi

### Méthodes de sauvegarde

#### Méthode 1 : Snapshot + Export manuel

```
1. Créer un snapshot de la VM dans vSphere
2. Exporter les fichiers VMDK vers un stockage externe
3. Supprimer le snapshot

Limites :
- Processus manuel, chronophage
- Le snapshot doit être supprimé rapidement (performances)
- Pas de gestion centralisée
```

#### Méthode 2 : API VADP (vSphere Data Protection API)

```
- API officielle VMware pour les sauvegardes de VMs
- Permet une sauvegarde cohérente avec quiescing (mise en pause des I/O du système invité)
- Utilisée par tous les outils de sauvegarde tiers sérieux
- Sauvegarde en mode "Changed Block Tracking (CBT)" pour l'incrémental
```

### Solution de référence en entreprise : Veeam Backup & Replication

**Veeam** est la solution professionnelle la plus utilisée pour les sauvegardes VMware (et Hyper-V).

| Fonctionnalité | Description |
|---------------|-------------|
| **Sauvegarde complète + incrémentale** | Via CBT (Changed Block Tracking) |
| **Restauration granulaire** | Restaurer un seul fichier depuis une sauvegarde de VM entière |
| **Instant VM Recovery** | Démarrer une VM directement depuis la sauvegarde en quelques minutes |
| **Veeam Agent** | Sauvegarde des machines physiques (Windows/Linux) |
| **Réplication** | Réplique les VMs vers un site de secours (DR) |

> 💡 **Alternatives connues :** Acronis Backup, Commvault, Zerto (spécialisé DR/réplication).

---

## 4. NAS vs SAN comme cible de sauvegarde

### NAS (Network Attached Storage)

```
Protocoles : NFS (Linux), SMB/CIFS (Windows)
Accès : Fichiers partagés sur le réseau
```

| Avantages | Inconvénients |
|-----------|---------------|
| ✅ Simple à déployer et administrer | ❌ Performances limitées (partage réseau = latence) |
| ✅ Coût abordable | ❌ Vulnérable aux ransomwares si toujours connecté |
| ✅ Compatible avec la plupart des outils de sauvegarde | ❌ Non adapté aux très fortes charges I/O |
| ✅ Accessible par plusieurs serveurs simultanément | ❌ Dépendant de la bande passante réseau |

### SAN (Storage Area Network)

```
Protocoles : iSCSI, Fibre Channel
Accès : Blocs (disque distant présenté comme disque local)
```

| Avantages | Inconvénients |
|-----------|---------------|
| ✅ Très hautes performances | ❌ Coût élevé (matériel spécialisé) |
| ✅ Faible latence | ❌ Administration complexe |
| ✅ Idéal pour les VMs et bases de données volumineuses | ❌ Nécessite des compétences spécifiques |
| ✅ Isolation réseau (protocole dédié) | ❌ Infrastructure réseau dédiée (Fibre Channel) |

### Quand utiliser quoi ?

| Besoin | Solution |
|--------|----------|
| PME, partage de fichiers, sauvegardes standard | **NAS** |
| Grandes infrastructures virtualisées, BDD critiques, RTO très court | **SAN** |
| Sauvegardes hors ligne (protection ransomware) | **Bandes magnétiques / disques débranchés** |
| Sauvegarde externalisée / hors site | **Cloud (Azure Backup, AWS S3, etc.)** |

---

## 🎯 Récapitulatif à mémoriser

| Service | Outil de sauvegarde | Points clés |
|---------|--------------------|----|
| **Active Directory** | `wbadmin start systemstatebackup` | NTDS.dit + SYSVOL, 2 types de restauration |
| **MySQL/MariaDB** | `mysqldump --single-transaction` | Jamais copier les fichiers à chaud directement |
| **VMs VMware** | Veeam / API VADP | CBT pour l'incrémental, restauration granulaire |
| **VMs Hyper-V** | Veeam / Windows Server Backup | Snapshots temporaires uniquement |
| **Stockage** | NAS (usage courant) / SAN (critique) | Toujours appliquer la règle 3-2-1 |
