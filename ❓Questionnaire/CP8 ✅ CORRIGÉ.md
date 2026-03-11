# 📋 CORRECTION – CP8 : Sauvegardes et Restaurations de l'Infrastructure

> **Légende :**
> - ✅ Réponse correcte ou bonne piste
> - ⚠️ Partiellement correct, à compléter
> - ❌ Incorrect ou manquant
> - 💡 Point clé à retenir

---

## PARTIE 1 – Concepts fondamentaux

---

### Question 1 — Sauvegarde vs copie de fichiers

**Ta réponse :** ⚠️ Partiellement correcte

> "Une sauvegarde est la copie complète et intégrale d'une entité, elle permet de restaurer les données en cas de problème."

**Ce qui est juste :** L'idée générale de copie et de restauration est bonne.

**Ce qui manque :**

Une **sauvegarde** est une copie structurée, cohérente et datée de données, réalisée dans le but explicite de **pouvoir les restaurer**. Elle inclut généralement des **métadonnées** (dates, permissions, attributs), une **organisation par versions**, et elle est stockée sur un **support distinct** de l'original.

Une **simple copie de fichiers** (ex : Ctrl+C/Ctrl+V, ou `cp`) ne préserve pas toujours :
- les permissions et attributs système,
- la cohérence des données (notamment pour les bases de données ou fichiers ouverts),
- l'historique des versions (si on copie à nouveau, l'ancienne version est écrasée).

💡 **La différence fondamentale :** une sauvegarde est une **démarche planifiée, versionnée et testable**, tandis qu'une copie est une opération ponctuelle sans garantie de restaurabilité.

---

### Question 2 — Règle 3-2-1

**Ta réponse :** ⚠️ Partiellement correcte

> "3 sauvegardes sur deux serveurs différents dans un autre lieu."

**Ce qui est juste :** Tu as retenu les chiffres 3 et 2, et la notion de lieu différent.

**Ce qui est incomplet/incorrect :**

La règle **3-2-1** signifie :
- **3** copies des données (l'original + 2 sauvegardes)
- **2** supports/technologies différents (ex : disque dur local + NAS)
- **1** copie hors site (hors du bâtiment : cloud, site distant, bande magnétique externalisée)

**Exemple concret en entreprise :**
- Données originales sur le serveur de production
- Sauvegarde sur un NAS interne (réseau local)
- Sauvegarde sur un service cloud distant (ex : Azure Backup, ou disques transportés hors site)

💡 L'objectif est de se protéger contre : panne matérielle, sinistre local (incendie, inondation), vol.

---

### Question 3 — Types de sauvegardes

**Ta réponse :** ⚠️ Globalement correcte mais incomplète

**Sauvegarde complète (Full) :**
✅ Correcte dans l'idée.
- Copie **l'intégralité** des données sélectionnées
- ✅ **Avantage :** restauration simple et rapide (un seul jeu de sauvegarde)
- ❌ **Inconvénient :** longue à réaliser, consomme beaucoup d'espace

**Sauvegarde incrémentale :**
❌ Ta définition est incorrecte. Ce n'est pas depuis la dernière incrémentale dans tous les cas.
- Copie uniquement les données **modifiées depuis la dernière sauvegarde, quelle qu'elle soit** (complète ou incrémentale précédente)
- ✅ **Avantage :** rapide, espace minimal
- ❌ **Inconvénient :** restauration longue (il faut appliquer toutes les incrémentales dans l'ordre depuis la dernière complète)

**Sauvegarde différentielle :**
✅ Correcte.
- Copie les données modifiées **depuis la dernière sauvegarde complète uniquement**
- ✅ **Avantage :** restauration plus rapide qu'une incrémentale (Full + 1 différentielle)
- ❌ **Inconvénient :** grossit avec le temps, plus volumineuse qu'une incrémentale

💡 **Mémo :**

| Type | Par rapport à | Espace | Restauration |
|------|---------------|--------|--------------|
| Complète | Rien | ❌ Lourd | ✅ Rapide |
| Différentielle | Dernière complète | ⚠️ Moyen | ✅ Assez rapide |
| Incrémentale | Dernière sauvegarde | ✅ Léger | ❌ Lente |

---

### Question 4 — RPO et RTO

**Ta réponse :** ❌ Non répondue

**RPO – Recovery Point Objective (Point de Reprise Objectif) :**
C'est la **perte de données maximale acceptable**, exprimée en temps.

> Exemple : RPO = 1h signifie que si un incident survient, on accepte de perdre au maximum 1h de données. On doit donc sauvegarder toutes les heures au minimum.

**RTO – Recovery Time Objective (Délai de Reprise Objectif) :**
C'est le **temps maximal acceptable pour remettre le service en fonctionnement** après un incident.

> Exemple : RTO = 4h signifie que le service doit être restauré dans les 4 heures suivant la panne. Au-delà, l'impact métier est trop important.

💡 **En résumé :**
- RPO = "Jusqu'où peut-on remonter dans le temps ?"
- RTO = "Combien de temps peut-on rester sans service ?"

---

### Question 5 — Le RAID n'est pas une sauvegarde

**Ta réponse :** ❌ Interrompue, non complétée

**Réponse complète :**

Le RAID est un mécanisme de **tolérance aux pannes matérielles** (disque dur défaillant), pas une sauvegarde. Les données du RAID sont accessibles **en temps réel et simultanément** sur tous les disques : si une donnée est détruite, elle est détruite partout.

**3 scénarios contre lesquels le RAID ne protège pas :**

1. **Suppression accidentelle** : Si un utilisateur supprime un fichier, la suppression est répercutée sur tous les disques du RAID immédiatement.
2. **Ransomware / chiffrement malveillant** : Le malware chiffre les fichiers sur le volume → tous les disques du RAID contiennent des données chiffrées.
3. **Corruption logicielle ou erreur humaine** : Une mauvaise manipulation, une mise à jour corrompue, une fausse manipulation d'un admin → répercutée instantanément sur tous les disques.
4. **Panne du contrôleur RAID** (bonus) : Si le contrôleur tombe en panne, les disques peuvent devenir illisibles.

---

### Question 6 — Le snapshot

**Ta réponse :** ❌ Non répondue

**Réponse :**

Un **snapshot** (ou instantané) est une **photo de l'état d'un volume ou d'une VM à un instant T**. Il est généralement réalisé via un mécanisme **COW (Copy-On-Write)** : à la création, le snapshot référence les données existantes. Quand les données originales sont modifiées, l'ancienne version est copiée dans le snapshot.

**Contextes d'utilisation :**
- Avant une mise à jour système ou applicative risquée
- Dans LVM (`lvcreate -s`) pour des sauvegardes cohérentes à chaud
- Sur des VMs (VMware, Hyper-V, Proxmox) avant une opération critique
- En environnement Cloud (snapshots disques Azure, AWS EBS)

**Limite principale en tant que solution de sauvegarde :**
- Le snapshot **reste sur le même support physique** que la donnée d'origine : si le disque tombe en panne, on perd le snapshot ET les données.
- Il n'est **pas une sauvegarde externalisée**, il n'est donc pas autonome.
- Il peut **grossir rapidement** si beaucoup de données sont modifiées.

---

## PARTIE 2 – Outils et mise en œuvre sous Linux

---

### Question 7 — rsync

**Ta réponse :** ❌ Non répondue

**Réponse :**

`rsync` est un outil de synchronisation et de transfert de fichiers qui ne transfère que les **blocs modifiés** (algorithme delta), ce qui le rend très efficace pour les sauvegardes incrémentales.

**Option clé :** `-a` (archive) préserve les permissions, horodatages, liens symboliques. Associée à `--delete` pour supprimer les fichiers supprimés à la source.

**Exemple de commande :**
```bash
rsync -avz --delete /var/www/ backup-srv:/sauvegardes/web/
```
- `-a` : mode archive (récursif + préservation des attributs)
- `-v` : verbeux
- `-z` : compression pendant le transfert
- `--delete` : supprime les fichiers sur la destination si absents de la source

---

### Question 8 — tar : archive et restauration

**Ta réponse :** ❌ Non répondue

**Créer l'archive :**
```bash
tar -czf /backup/etc_backup.tar.gz /etc
```
- `-c` : créer une archive
- `-z` : compresser avec gzip
- `-f` : spécifier le fichier de sortie

**Restaurer dans `/tmp/restauration` :**
```bash
mkdir -p /tmp/restauration
tar -xzf /backup/etc_backup.tar.gz -C /tmp/restauration
```
- `-x` : extraire
- `-C` : spécifier le répertoire de destination

---

### Question 9 — Snapshot LVM et mécanisme COW

**Ta réponse :** ❌ Non répondue

**Le mécanisme COW (Copy-On-Write) :**
À la création du snapshot, aucune copie physique n'est faite : le snapshot pointe vers les mêmes données que le volume original. Lorsqu'une donnée est **modifiée sur l'original**, l'ancienne version est d'abord **copiée dans le snapshot**, puis la nouvelle version est écrite sur l'original. Le snapshot garde ainsi l'état initial.

**Séquence de commandes :**

```bash
# 1. Créer un snapshot de 5 Go
lvcreate -L 5G -s -n snap_data /dev/vg_data/lv_data

# 2. Monter en lecture seule
mount -o ro /dev/vg_data/snap_data /mnt/snap

# 3. Supprimer le snapshot (après sauvegarde)
umount /mnt/snap
lvremove /dev/vg_data/snap_data
```

---

### Question 10 — Automatiser une sauvegarde avec rsync et cron

**Ta réponse :** ❌ Non répondue

**Outils à utiliser :** `rsync` + `cron`

**Planifier avec cron :**
```bash
crontab -e
```

**Exemple de ligne cron pour une sauvegarde quotidienne à 23h :**
```cron
0 23 * * * rsync -az --delete /données/ /backup/données/
```

Pour une sauvegarde différentielle avec horodatage :
```bash
0 23 * * * rsync -az /données/ /backup/données_$(date +\%Y\%m\%d)/
```

💡 On peut aussi utiliser **`rsnapshot`**, un outil basé sur rsync qui gère nativement les sauvegardes incrémentales/différentielles avec rotation automatique.

---

### Question 11 — Sauvegarde à chaud vs à froid

**Ta réponse :** ❌ Non répondue

| | Sauvegarde à chaud | Sauvegarde à froid |
|--|---|---|
| **Définition** | Réalisée pendant que le service fonctionne | Réalisée service arrêté |
| **Avantage** | Pas d'interruption de service | Cohérence garantie à 100% |
| **Inconvénient** | Risque d'incohérence (fichiers ouverts) | Nécessite une interruption |

**Exemple sauvegarde à chaud :** Un serveur web Apache en production → on sauvegarde `/var/www` avec rsync pendant que le serveur tourne. Le site reste accessible.

**Exemple sauvegarde à froid :** Une base de données MySQL critique → on arrête le service MySQL (`systemctl stop mysql`), on sauvegarde les fichiers, puis on redémarre. Cela garantit la cohérence des données.

---

## PARTIE 3 – Outils et mise en œuvre sous Windows Server

---

### Question 12 — Service VSS (Volume Shadow Copy Service)

**Ta réponse :** ❌ Non répondue

**Le VSS** est un service Windows qui coordonne la création de **clichés instantanés cohérents** de volumes, même lorsque des fichiers sont ouverts et en cours d'utilisation.

**Rôle dans la sauvegarde :**
Il permet de geler temporairement les écritures sur le volume (via des "writers" VSS propres à chaque application) pour prendre une photo cohérente, puis reprendre les écritures. Les outils de sauvegarde (Windows Server Backup, Veeam, etc.) s'appuient sur VSS.

**Exemple indispensable :** La sauvegarde d'un serveur **Exchange** ou **SQL Server** en production. Sans VSS, les fichiers de base de données sont ouverts et verrouillés → la copie serait incohérente. Avec VSS, le writer SQL/Exchange met en pause les transactions, le snapshot est pris, puis tout reprend.

---

### Question 13 — Windows Server Backup : planifier une sauvegarde

**Ta réponse :** ❌ Non répondue

**Étapes de configuration (via GUI) :**
1. Ouvrir le **Gestionnaire de serveur** → Outils → Windows Server Backup
2. Cliquer sur **Planification de sauvegarde**
3. Choisir **Configuration personnalisée** → **Sauvegarde complète du serveur** ou volumes sélectionnés
4. Définir la fréquence (quotidienne, avec horaire)
5. Sélectionner la destination : **Partage réseau distant** → entrer le chemin UNC (`\\serveur\partage`)
6. Entrer les identifiants d'accès au partage
7. Valider et terminer l'assistant

**Commande `wbadmin` pour une sauvegarde complète manuelle :**
```cmd
wbadmin start backup -backupTarget:\\backup-srv\sauvegardes -include:C: -allCritical -quiet
```

---

### Question 14 — Restauration rapide de fichier supprimé (Windows)

**Ta réponse :** ❌ Non répondue

**La fonctionnalité utilisée est les "Clichés instantanés" (Previous Versions / Shadow Copies).**

**Procédure :**
1. Faire un clic droit sur le dossier parent du fichier supprimé
2. Cliquer sur **"Propriétés"** → onglet **"Versions précédentes"**
3. Sélectionner la version antérieure souhaitée
4. Cliquer sur **"Restaurer"** ou **"Ouvrir"** pour accéder aux fichiers

Côté serveur, les clichés instantanés doivent être activés sur le volume : **Gestionnaire de serveur → Partages → Gérer les clichés instantanés**.

💡 Cette méthode est rapide et ne nécessite pas de restauration complète depuis une sauvegarde.

---

### Question 15 — Snapshots Hyper-V et risques en production

**Ta réponse :** ❌ Non répondue

**Fonctionnement :**
Un snapshot (ou **point de contrôle**) Hyper-V sauvegarde l'état de la VM (mémoire RAM + disques) à un instant T. Les modifications suivantes sont écrites dans un fichier différentiel (`.avhd`/`.avhdx`) alors que le disque original reste figé.

**Risques liés à la conservation prolongée en production :**
1. **Dégradation des performances** : Chaque lecture/écriture doit traverser la chaîne des fichiers différentiels, ce qui alourdit les I/O disques.
2. **Espace disque** : Le fichier différentiel peut grossir jusqu'à atteindre la taille du disque d'origine, voire le dépasser si beaucoup de données changent.
3. **Risque à la suppression** : Fusionner un ancien snapshot peut prendre des heures et risque de corrompre la VM si l'opération est interrompue.
4. **Sauvegarde complexifiée** : Les outils de sauvegarde doivent gérer la chaîne des différentiels.

---

## PARTIE 4 – Sauvegarde des services critiques

---

### Question 16 — Sauvegarde Active Directory : authoritative vs non-authoritative

**Ta réponse :** ❌ Non répondue

**Pourquoi c'est critique :**
L'AD centralise l'authentification, les GPO, les droits d'accès de toute l'infrastructure. Une corruption ou perte de l'AD rend l'ensemble du SI inopérant. De plus, l'AD se réplique entre plusieurs contrôleurs de domaine (DC) : une erreur répliquée sur tous les DC peut être irréversible sans sauvegarde.

**Restauration Non-Authoritative :**
- Par défaut lors d'une restauration d'un DC
- Les données restaurées sont **mises à jour par la réplication** depuis les autres DC
- Utilisée quand le DC était simplement tombé en panne → on restaure et il se resynchronise avec les autres DC

**Restauration Authoritative :**
- Force le DC restauré à être considéré comme **la source de vérité**
- Ses données **remplacent** celles des autres DC via la réplication
- Utilisée quand une **suppression accidentelle d'objets AD** a été répliquée sur tous les DC (ex : suppression d'une OU entière). On restaure depuis une sauvegarde antérieure et on marque les objets comme "authoritative" avec `ntdsutil`

---

### Question 17 — Sauvegarde System State sur un DC

**Ta réponse :** ❌ Non répondue

**Commande `wbadmin` :**
```cmd
wbadmin start systemstatebackup -backupTarget:E:
```

**Éléments inclus dans le System State d'un DC :**
- **NTDS.dit** : la base de données Active Directory
- **SYSVOL** : les scripts de connexion et les templates de GPO
- Le **Registre Windows**
- Les **fichiers de démarrage système** (BCD, bootmgr)
- La **base de données COM+**
- Les **certificats** (si rôle ADCS installé)

---

### Question 18 — Sauvegarde cohérente MySQL/MariaDB

**Ta réponse :** ❌ Non répondue

**Pourquoi une copie directe des fichiers est insuffisante :**
MySQL/MariaDB écrit ses données en continu. Si on copie les fichiers pendant que la base tourne, on risque de capturer des fichiers dans un état **inconsistant** (une transaction en cours, des tampons non vidés) → la restauration produira une base corrompue.

**Outil recommandé : `mysqldump`**
```bash
mysqldump -u root -p --all-databases > /backup/mysql_all_$(date +%Y%m%d).sql
```

Pour une base spécifique :
```bash
mysqldump -u root -p ma_base > /backup/ma_base_$(date +%Y%m%d).sql
```

**Alternative pour les très grandes bases :** `mysqldump --single-transaction` (pour InnoDB) permet une sauvegarde cohérente sans verrouiller les tables :
```bash
mysqldump -u root -p --single-transaction --all-databases > /backup/dump.sql
```

**Outil avancé :** `Percona XtraBackup` permet des sauvegardes à chaud cohérentes et incrémentales sans interruption.

---

### Question 19 — Sauvegarde de VMs VMware ESXi

**Ta réponse :** ❌ Non répondue

**2 méthodes de sauvegarde d'une VM sur VMware ESXi :**

1. **Snapshot + export** : Créer un snapshot de la VM via vSphere/ESXi, puis exporter les fichiers VMDK vers un stockage externe.
2. **Sauvegarde via API vSphere (VADP)** : VMware fournit une API (vSphere Data Protection API) qui permet aux outils tiers de sauvegarder les VMs de manière cohérente (avec quiescing du système invité).

**Solution professionnelle fréquemment utilisée :**
**Veeam Backup & Replication** est la solution de référence en entreprise pour la gestion centralisée des sauvegardes VMware (et Hyper-V). Elle s'appuie sur l'API VADP et permet : sauvegardes complètes/incrémentales, restauration granulaire de fichiers, réplication, etc.

---

### Question 20 — NAS vs SAN pour les sauvegardes

**Ta réponse :** ❌ Non répondue

**Avantages du NAS comme cible de sauvegarde :**
- ✅ Simple à déployer et administrer
- ✅ Coût abordable
- ✅ Compatible avec les protocoles NFS/SMB utilisés par la plupart des outils de sauvegarde
- ✅ Accessible par plusieurs serveurs simultanément

**Limites du NAS :**
- ❌ Performances limitées (partage réseau = latence)
- ❌ Pas adapté aux charges I/O très intenses (bases de données volumineuses)
- ❌ Si toujours connecté au réseau → vulnérable aux ransomwares (comme illustré en Q25)

**Quand préférer un SAN :**
- Pour des sauvegardes nécessitant des **débits très élevés** (grandes bases de données, hyperviseurs)
- Quand on a besoin d'un stockage **dédié et isolé**, accessible en mode bloc (iSCSI, Fibre Channel)
- Dans des infrastructures **critiques** où les RTO sont très courts (restauration rapide nécessaire)

---

## PARTIE 5 – Tests, validation et plan de reprise

---

### Question 21 — Importance des tests de restauration

**Ta réponse :** ❌ Non répondue

**Pourquoi tester régulièrement :**
Une sauvegarde non testée est une sauvegarde dont on ignore si elle fonctionne. Les causes d'échec sont nombreuses : corruption silencieuse des données, problème de support, procédure de restauration obsolète, changement d'infrastructure. **La seule preuve qu'une sauvegarde est valide, c'est de l'avoir restaurée avec succès.**

**Procédure de test pour un serveur de fichiers Windows :**
1. Identifier une sauvegarde récente à tester
2. Monter ou restaurer la sauvegarde dans un **environnement isolé** (VM de test, réseau isolé)
3. Vérifier l'intégrité des fichiers (présence, contenu, droits NTFS)
4. Tester l'accès aux partages depuis un poste client de test
5. Documenter le test (date, résultat, durée)

**Critères de validation :**
- ✅ Les données restaurées sont complètes et accessibles
- ✅ Les droits et permissions sont corrects
- ✅ La durée de restauration est compatible avec le RTO défini
- ✅ Aucun message d'erreur ou avertissement pendant le processus

---

### Question 22 — PCA vs PRA

**Ta réponse :** ❌ Non répondue

| | PCA | PRA |
|--|-----|-----|
| **Objectif** | Maintenir l'activité **sans interruption** | **Reprendre** l'activité après une interruption |
| **Interruption** | Nulle ou imperceptible | Interruption acceptée, durée limitée |
| **Coût** | Élevé (infrastructure redondante) | Moindre |
| **RTO typique** | Quelques secondes à minutes | Quelques heures |
| **RPO typique** | Quasi-zéro | 1h à 24h |

**Exemple PCA :** Un site e-commerce avec deux datacenters en **actif/actif**. Si le DC principal tombe, le trafic bascule automatiquement sur le second sans interruption perceptible. RTO ≈ 0, RPO ≈ 0.

**Exemple PRA :** Une PME avec un serveur de fichiers sauvegardé toutes les nuits. En cas de sinistre, on restaure depuis la dernière sauvegarde sur un nouveau serveur. RTO = 4h, RPO = 24h (on peut perdre jusqu'à une journée de données).

---

### Question 23 — Politique de sauvegarde : éléments essentiels

**Ta réponse :** ❌ Non répondue

Une politique de sauvegarde doit contenir :

1. **Périmètre** : Quelles données sont sauvegardées ? (serveurs, bases de données, postes utilisateurs, partages)
2. **Types et fréquences** : Complète hebdomadaire, différentielle ou incrémentale quotidienne, etc.
3. **Durée de rétention** : Combien de temps conserve-t-on les sauvegardes ? (ex : 30 jours, 1 an pour certaines données légales)
4. **Supports et destinations** : NAS, bandes, cloud, site distant → application de la règle 3-2-1
5. **RPO et RTO définis** par criticité des données
6. **Responsabilités** : Qui est responsable de lancer, vérifier, tester les sauvegardes ?
7. **Procédure de test de restauration** : Fréquence et méthode des tests (mensuel, trimestriel)
8. **Sécurité des sauvegardes** : Chiffrement des sauvegardes, accès restreint aux supports
9. **Procédure de restauration documentée** : Étape par étape, pour chaque type de serveur
10. **Journalisation et alertes** : Surveillance des jobs de sauvegarde, notification en cas d'échec

---

### Question 24 — Ordre de restauration avec sauvegardes incrémentales

**Ta réponse :** ❌ Non répondue

**Contexte :**
- Sauvegarde complète : J-1 à 23h00
- Incrémentale 1 : J à 6h00 (changes depuis la complète)
- Incrémentale 2 : J à 12h00 (changes depuis l'incrémentale 1)
- Panne : J à 14h00

**Ordre de restauration :**
1. **Restaurer la sauvegarde complète** de la veille à 23h00 (base de départ)
2. **Appliquer l'incrémentale de 6h00** (ajoute les changements entre 23h et 6h)
3. **Appliquer l'incrémentale de 12h00** (ajoute les changements entre 6h et 12h)

**État récupéré :** Les données telles qu'elles existaient à **12h00**, soit une perte de données de 2h (de 12h à 14h) → c'est le RPO effectif dans ce scénario.

💡 C'est pourquoi les sauvegardes incrémentales doivent être restaurées **dans l'ordre chronologique strict**.

---

### Question 25 — Mise en situation : ransomware et NAS chiffré

**Ta réponse :** ❌ Non répondue

**Conclusions sur la politique de sauvegarde en place :**
- Le NAS était **connecté en permanence** au réseau de production → il était accessible au ransomware comme n'importe quel partage réseau
- Il n'y avait **pas de copie hors ligne ou hors site** (violation de la règle 3-2-1)
- **Absence de sauvegarde immuable** (non modifiable, non effaçable)
- La politique de sauvegarde était **insuffisante** voire inexistante

**Mesures qui auraient pu éviter la situation :**
1. Appliquer la **règle 3-2-1** avec une copie hors ligne (disque débranché, bande, cloud avec verrouillage WORM)
2. Utiliser des **sauvegardes immuables** (option "Object Lock" sur S3, ou supports WORM)
3. **Isoler le NAS** du réseau de production (accès restreint, VLAN dédié, authentification forte)
4. Mettre en place une **rétention suffisante** pour détecter le ransomware avant que toutes les versions soient chiffrées
5. Déployer un **antivirus/EDR** et surveiller les comportements anormaux (chiffrement massif de fichiers)

**Organisation de la reprise d'activité :**
1. **Isoler immédiatement** les systèmes infectés du réseau
2. **Identifier la souche** du ransomware (des outils comme NoMoreRansom.org peuvent avoir une clé de déchiffrement)
3. Si pas de clé disponible : **reconstruire les serveurs** depuis les images système (OS propre)
4. **Contacter le prestataire cloud** ou vérifier si une copie hors site non affectée existe
5. Restaurer depuis la **sauvegarde la plus ancienne non chiffrée** si disponible
6. Documenter l'incident et **mettre à jour la politique de sauvegarde** pour éviter la récidive
7. Déposer une **plainte** (ransomware = infraction pénale)

---

## 📊 Bilan global

| Partie | Questions | Réponses fournies | Évaluation |
|--------|-----------|-------------------|------------|
| Partie 1 – Concepts | Q1 à Q6 | Q1, Q2, Q3 partielles | ⚠️ Bases présentes, à consolider |
| Partie 2 – Linux | Q7 à Q11 | Aucune | ❌ À travailler |
| Partie 3 – Windows | Q12 à Q15 | Aucune | ❌ À travailler |
| Partie 4 – Services critiques | Q16 à Q20 | Aucune | ❌ À travailler |
| Partie 5 – Tests/PRA | Q21 à Q25 | Aucune | ❌ À travailler |

**Points forts :** Tu as une bonne intuition sur les concepts de base (sauvegarde complète, règle 3-2-1, LVM).

**Axes de progression prioritaires :**
- Les commandes Linux (`rsync`, `tar`, `lvcreate -s`, `mysqldump`, `cron`)
- Les outils Windows Server (`wbadmin`, VSS, Previous Versions)
- Les notions RPO/RTO et PCA/PRA (fondamentaux pour un TSSR)
- La mise en situation sécurité (ransomware, politique de sauvegarde)

---

*Correction basée sur les cours TSSR fournis et les référentiels techniques Linux/Windows Server.*
