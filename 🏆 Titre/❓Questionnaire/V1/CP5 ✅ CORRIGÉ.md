# Correction — CP5 : Maintenir des serveurs dans une infrastructure virtualisée

> **Formation TSSR** | Correction détaillée avec explications pédagogiques

---

## NIVEAU 1 — Fondamentaux de la virtualisation

---

### Q1. Différence entre hyperviseur Type 1 et Type 2

**Ta réponse :** ✅ **Bonne base — À compléter**

Tu as bien identifié les deux types et donné des exemples corrects. Il manque quelques précisions importantes.

**Correction complète :**

| | Type 1 (Bare Metal) | Type 2 (Hosted) |
|---|---|---|
| **Installation** | Directement sur le matériel, sans OS hôte | Sur un OS existant (Windows, Linux, macOS) |
| **Performance** | Excellente (accès direct au matériel) | Réduite (overhead de l'OS hôte) |
| **Exemples** | Proxmox VE, VMware ESXi, Hyper-V, KVM | VirtualBox, VMware Workstation, Parallels |
| **Usage** | Serveurs de production, datacenters | Tests, développement, usage personnel |

> ⚠️ **Point clé :** Proxmox utilise **KVM** (Type 1) pour les machines virtuelles et **LXC** pour les conteneurs. Ce n'est pas juste "un hyperviseur de type 1", il combine les deux technologies.

---

### Q2. Trois avantages majeurs de la virtualisation

**Ta réponse :** ⚠️ **Partielle — 2 avantages mentionnés sur 3 minimum demandés**

Tu cites "moins de machines à entretenir" et "gestion rapide des ressources". C'est juste mais insuffisant.

**Correction complète — 5 avantages principaux :**

1. **Consolidation des serveurs** → Réduction du nombre de machines physiques, donc moins de coûts matériels et énergétiques
2. **Isolation des services** → Chaque VM est indépendante ; une panne d'une VM n'affecte pas les autres
3. **Flexibilité et rapidité de déploiement** → Créer, cloner, déployer une VM en quelques minutes
4. **Snapshots et sauvegardes simplifiées** → Retour arrière rapide en cas de problème
5. **Haute disponibilité** → Migration à chaud possible, clustering, redondance

---

### Q3. Qu'est-ce qu'une machine virtuelle (VM) ?

**Ta réponse :** ✅ **Correcte dans l'esprit — À préciser**

**Correction complète :**

Une machine virtuelle est une **émulation logicielle d'un ordinateur physique complet**, qui partage les ressources matérielles de la machine hôte (CPU, RAM, stockage) via un hyperviseur.

**Différences avec un serveur physique dédié :**

| | VM | Serveur physique |
|---|---|---|
| Matériel | Virtuel (émulé) | Réel et dédié |
| Ressources | Partagées avec d'autres VMs | Exclusives |
| Déploiement | Minutes | Jours/semaines |
| Isolation | Logicielle | Physique totale |
| Coût | Faible | Élevé |

---

### Q4. Qu'est-ce que Proxmox VE ?

**Ta réponse :** ⚠️ **Incomplète — La question demandait les deux technologies principales**

**Correction complète :**

**Proxmox VE** est une plateforme de virtualisation open-source de **Type 1 (bare-metal)**, gérée via une interface web. Il s'appuie sur **deux technologies principales** :

1. **KVM** (Kernel-based Virtual Machine) → Pour les **machines virtuelles** complètes (Windows, Linux...). Le kernel Linux lui-même joue le rôle d'hyperviseur.
2. **LXC** (Linux Containers) → Pour les **conteneurs légers**. Partage le kernel de l'hôte, plus léger qu'une VM KVM.

> Proxmox est gratuit et open-source, ce qui en fait une solution populaire en formation et en PME.

---

### Q5. Risque principal d'un seul serveur physique avec plusieurs VMs

**Ta réponse :** ✅ **Correcte et bien formulée**

**Compléments :**

Ce risque s'appelle **SPOF** (Single Point of Failure = Point de défaillance unique).

**Solutions pour atténuer ce risque :**
- **Clustering** : Plusieurs nœuds Proxmox synchronisés
- **Haute disponibilité (HA)** : Redémarrage automatique des VMs sur un autre nœud
- **Stockage partagé** : Les VMs ne sont pas liées à un seul disque local
- **Alimentation redondante** : Double alimentation sur le serveur physique

---

### Q6. Qu'est-ce que l'"overhead" en virtualisation ?

**Ta réponse :** ❌ **Définition incorrecte**

L'overhead **ne signifie pas** que les VMs utilisent plus de ressources que la machine ne peut en fournir (ça, c'est de la surcharge ou "overcommit").

**Correction :**

L'**overhead** désigne les **ressources supplémentaires consommées par l'hyperviseur lui-même** pour gérer la virtualisation, qui ne sont donc pas disponibles pour les VMs.

**Exemple :** Un serveur avec 32 Go de RAM et Proxmox installé : Proxmox lui-même consomme ~1-2 Go pour fonctionner → il ne reste que ~30 Go disponibles pour les VMs.

L'overhead impacte le **CPU** et la **RAM** principalement. Il est généralement faible sur les hyperviseurs modernes (Type 1).

> ⚠️ L'**overcommit** (ou surallocation) est différent : c'est le fait d'allouer plus de ressources que ce que la machine possède réellement, en pariant sur le fait que toutes les VMs ne les utilisent pas en même temps.

---

### Q7. Formats de disques virtuels

**Ta réponse :** ❌ **Non répondu**

**Correction complète :**

| Format | Nom complet | Hyperviseur associé |
|---|---|---|
| **VMDK** | Virtual Machine Disk | VMware (ESXi, Workstation) |
| **QCOW2** | QEMU Copy-On-Write v2 | KVM / Proxmox / QEMU |
| **VHD/VHDX** | Virtual Hard Disk | Microsoft Hyper-V |
| **VDI** | VirtualBox Disk Image | Oracle VirtualBox |

> **QCOW2** est le format natif de Proxmox/KVM. Il supporte les snapshots, la compression et le thin provisioning (allocation à la demande).

---

### Q8. Snapshot vs Sauvegarde complète

**Ta réponse :** ✅ **Correcte dans l'esprit — À approfondir**

**Correction complète :**

| | Snapshot | Sauvegarde complète |
|---|---|---|
| **Définition** | Photo instantanée de l'état d'une VM à un instant T | Copie intégrale des données de la VM |
| **Espace disque** | Faible au départ (stocke uniquement les différences) | Important (copie complète) |
| **Vitesse de création** | Quasi-instantanée | Longue |
| **Restauration** | Rapide (retour à l'état exact du snapshot) | Plus lente |
| **Indépendance** | Dépend du disque original | Indépendante |
| **Usage** | Avant une mise à jour ou un test | Sauvegarde régulière de production |

> ⚠️ **Ils ne sont PAS interchangeables !** Un snapshot n'est pas une sauvegarde. Si le disque physique tombe en panne, le snapshot est perdu avec lui. La sauvegarde complète peut être stockée sur un autre support.

---

## NIVEAU 2 — Administration d'une infrastructure virtualisée

---

### Q9. Cinq paramètres à définir lors de la création d'une VM Proxmox

**Ta réponse :** ✅ **5 paramètres corrects**

**Correction et complément — Liste exhaustive :**

Tu as cité : Nom, ID, Taille du disque, RAM, Carte réseau → **Correct !**

**Paramètres supplémentaires importants :**
- **OS (type et version)** → aide Proxmox à optimiser les pilotes virtuels
- **CPU** (nombre de cœurs, type de socket)
- **Type de disque** (virtio, SCSI, IDE) et **stockage** (local, NFS, SAN)
- **ISO d'installation** → image d'installation du système
- **Type de réseau** (bridge, VirtIO, E1000)
- **Options de démarrage** (boot order, BIOS/UEFI)

---

### Q10. Template (modèle) de VM dans Proxmox

**Ta réponse :** ⚠️ **Approximative**

**Correction complète :**

Un **template** est une VM convertie en modèle **non démarrable** qui sert de base pour créer rapidement de nouvelles VMs identiques.

**Intérêt :**
- ✅ Gain de temps : on clone le template au lieu d'installer un OS from scratch
- ✅ Standardisation : toutes les VMs issues du même template ont la même configuration de base
- ✅ Cohérence : même version d'OS, mêmes paquets de base, même configuration réseau

**Workflow Proxmox :**
1. Créer une VM → Installer l'OS → Configurer les outils de base (agent QEMU, sudo, etc.)
2. Éteindre la VM → Clic droit → "Convertir en template"
3. Ensuite : clic droit sur le template → "Cloner" pour chaque nouvelle VM

---

### Q11. Clone complet vs Clone lié

**Ta réponse :** ❌ **Non répondu**

**Correction complète :**

| | Clone complet (Full clone) | Clone lié (Linked clone) |
|---|---|---|
| **Fonctionnement** | Copie intégrale et indépendante du disque | Partage le disque du template, ne stocke que les différences |
| **Espace disque** | Important (autant que la VM originale) | Faible au départ |
| **Indépendance** | Totale (peut fonctionner sans le template) | Dépend du template (si supprimé = problème) |
| **Performance** | Optimale | Légèrement réduite |
| **Usage** | Production, déploiement permanent | Tests rapides, environnements temporaires |

> **Règle de base :** En production → clone complet. Pour des tests ou du dev → clone lié.

---

### Q12. Bridge Linux vs vSwitch dans Proxmox

**Ta réponse :** ❌ **Non répondu**

**Correction complète :**

**Bridge Linux (`vmbr`)** : C'est un switch virtuel logiciel intégré au kernel Linux. Il connecte les VMs entre elles et au réseau physique via une interface réseau réelle.

**vSwitch** : Terme plus générique (VMware notamment). Dans Proxmox, on parle de bridges Linux, qui jouent le même rôle.

**Rôle de `vmbr0` :**
```
Interface physique (eth0/eno1)
         |
       vmbr0  ←── bridge par défaut
      /      \
   VM1       VM2       → accès réseau vers l'extérieur
```

`vmbr0` est le **bridge principal** qui relie les VMs à l'interface réseau physique de l'hyperviseur. C'est par lui que les VMs obtiennent un accès au réseau local et à Internet.

---

### Q13. VLAN dans un contexte de virtualisation

**Ta réponse :** ❌ **Non répondu**

**Correction complète :**

Un **VLAN** (Virtual LAN) est une segmentation logique du réseau permettant de créer plusieurs réseaux virtuels isolés sur la même infrastructure physique.

**Dans Proxmox :** On peut tagguer les VMs avec un VLAN ID. Deux VMs sur le même hyperviseur mais sur des VLANs différents ne peuvent pas communiquer directement, même si elles partagent le même bridge physique.

**Exemple :**
- VM Serveur Web → VLAN 10 (DMZ)
- VM Active Directory → VLAN 20 (Interne)
- VM de test → VLAN 30 (Test)

> Le trafic est isolé par les **tags 802.1Q** dans les trames Ethernet. Le switch physique doit être configuré en **mode trunk** pour laisser passer les trames taguées.

---

### Q14. Diagnostic d'une VM qui consomme trop de RAM

**Ta réponse :** ❌ **Non répondu**

**Correction complète :**

**Outils de diagnostic :**

Depuis l'**interface Proxmox** :
- Onglet "Summary" de la VM → graphiques de consommation CPU/RAM en temps réel

Depuis la **VM elle-même (Linux)** :
```bash
free -h          # Affiche l'utilisation mémoire
top / htop       # Identifie les processus consommateurs
ps aux --sort=-%mem | head  # Top processus par RAM
```

**Actions correctives :**
1. **Court terme** : Identifier le processus gourmand et le redémarrer ou le tuer
2. **Moyen terme** : Augmenter la RAM allouée à la VM (possible à chaud sous Linux avec balloon driver)
3. **Long terme** : Limiter la RAM max de la VM via Proxmox, activer le **ballooning** (voir Q15)

---

### Q15. Ballooning mémoire

**Ta réponse :** ❌ **Non répondu**

**Correction complète :**

Le **memory ballooning** est un mécanisme qui permet à l'hyperviseur de **récupérer dynamiquement de la RAM inutilisée** dans les VMs pour la redistribuer à d'autres VMs.

**Fonctionnement :**
1. Un **driver balloon** est installé dans la VM
2. Quand une autre VM a besoin de RAM, l'hyperviseur "gonfle le ballon" dans la première VM → le driver balloon réserve de la mémoire, réduisant ce qui est disponible pour la VM
3. Quand la pression baisse, le ballon "se dégonfle" → la mémoire est rendue à la VM

**Intérêt :** Permet l'**overcommit mémoire** — allouer en théorie plus de RAM que la machine physique n'en possède, en pariant sur le fait que toutes les VMs ne l'utilisent pas simultanément.

---

### Q16. LVM — Architecture en trois couches

**Ta réponse :** ❌ **Non répondu**

**Correction complète :**

**LVM** (Logical Volume Manager) est un système de gestion du stockage flexible qui abstrait les disques physiques.

**Architecture en 3 couches :**

```
Disques physiques (sda, sdb...)
         ↓
PV - Physical Volumes  → Disques ou partitions préparées pour LVM
         ↓
VG - Volume Group      → Pool de stockage regroupant plusieurs PV
         ↓
LV - Logical Volumes   → "Partitions virtuelles" créées dans le VG
         ↓
Système de fichiers (ext4, XFS...)
```

**En détail :**
- **PV** : Un disque (`/dev/sdb`) ou une partition (`/dev/sda1`) initialisé pour LVM avec `pvcreate`
- **VG** : Regroupe 1 ou plusieurs PV en un "pool". Ex : VG "vg_data" = sdb + sdc = 4 To
- **LV** : Partition virtuelle taillée dans le VG. Ex : LV de 100 Go pour `/home`

---

### Q17. Réduction d'un volume logique (LV) — Risques et précautions

**Correction complète :**

**Risques :**
- ⚠️ **Perte de données** si le LV est réduit à une taille inférieure aux données qu'il contient
- ⚠️ **Corruption du système de fichiers** si la réduction n'est pas faite dans l'ordre correct
- ⚠️ **XFS ne supporte pas la réduction** (uniquement ext4)

**Précaution absolue : TOUJOURS sauvegarder avant de réduire**

**Procédure sécurisée :**
```bash
# 1. Démonter le volume
umount /dev/vg_data/lv_home

# 2. Vérifier le système de fichiers
e2fsck -f /dev/vg_data/lv_home

# 3. Réduire d'abord le FS (avant le LV !)
resize2fs /dev/vg_data/lv_home 9G

# 4. Réduire le LV ensuite
lvreduce -L 10G /dev/vg_data/lv_home

# 5. Remonter
mount /dev/vg_data/lv_home /mnt/home
```

> ⚠️ **Ordre impératif** : réduire le FS d'abord, puis le LV. L'inverse = corruption garantie.

---

### Q18. NAS vs SAN — Lequel pour les disques de VMs ?

**Correction complète :**

| Critère | NAS | SAN |
|---|---|---|
| **Niveau d'accès** | Fichiers (haut niveau) | Blocs (bas niveau, comme un disque local) |
| **Protocoles** | NFS, SMB/CIFS, FTP | iSCSI, Fibre Channel, FCoE |
| **Performance** | Correcte | Excellente |
| **Complexité** | Simple | Complexe |
| **Coût** | Abordable | Élevé |

**Pour les disques de VMs → Le SAN est plus adapté** car :
- L'accès bloc (bas niveau) est nécessaire pour obtenir les performances d'un disque local
- Les VMs nécessitent des accès fréquents et rapides en lecture/écriture
- **iSCSI** sur SAN permet de monter un stockage distant comme s'il était local

> Le NAS est suffisant pour du stockage de fichiers (partage de documents, backups), mais insuffisant pour héberger des disques de VMs de production exigeants.

---

### Q19. RAID 5 avec 4 disques de 2 To

**Ta réponse :** ❌ **Calcul incorrect**

**Correction :**

Formule RAID 5 : **Capacité utile = (n-1) × taille d'un disque**

Avec 4 disques de 2 To :
- **(4-1) × 2 To = 3 × 2 To = 6 To de capacité utile** (et non 8 To)
- 1 disque équivalent est "sacrifié" pour la parité distribuée

**Tolérance aux pannes :** RAID 5 tolère **1 panne disque** (et non plus). Si 2 disques tombent simultanément → perte totale des données.

> 💡 Pour tolérer 2 pannes avec 4 disques, il faudrait du **RAID 6** → capacité = (4-2) × 2 To = **4 To utiles**.

---

### Q20. Vérification de l'état d'une grappe RAID avec mdadm

**Correction complète :**

```bash
# Commande principale
cat /proc/mdstat

# Plus détaillé
mdadm --detail /dev/md0
```

**Contenu de `/proc/mdstat` :**
```
Personalities : [raid5]
md0 : active raid5 sda1[0] sdb1[1] sdc1[2]
      4194304 blocks super 1.2 level 5, 512k chunk, algorithm 2 [3/3] [UUU]
```
- `[3/3]` → 3 disques sur 3 actifs
- `[UUU]` → tous les disques sont **U**p (en fonctionnement)
- `[UU_]` → un disque est **_** = défaillant

---

## NIVEAU 3 — Haute disponibilité, maintenance et sécurité

---

### Q21. Migration à chaud vs Migration à froid

**Correction complète :**

| | Migration à chaud (Live Migration) | Migration à froid |
|---|---|---|
| **État de la VM** | En cours de fonctionnement | Éteinte ou suspendue |
| **Interruption de service** | Nulle (quelques millisecondes max) | Oui, service indisponible |
| **Complexité** | Élevée | Simple |

**Conditions pour la migration à chaud :**
1. **Stockage partagé** entre les deux nœuds (NFS, SAN, Ceph) — les VMs ne sont pas sur les disques locaux
2. **Même architecture CPU** sur les deux nœuds (ou CPU compatible)
3. **Réseau dédié** à la migration (bande passante suffisante)
4. **Cluster Proxmox** configuré avec communication entre nœuds

---

### Q22. Cluster Proxmox, nœuds minimum et quorum

**Correction complète :**

Un **cluster Proxmox** est un ensemble de plusieurs nœuds hyperviseurs qui fonctionnent ensemble et partagent une vue commune de l'infrastructure.

**Minimum : 3 nœuds** pour un cluster fiable (2 suffit techniquement mais est déconseillé).

**Le quorum :**
- Mécanisme de vote pour qu'un cluster reste opérationnel
- **Règle : il faut la majorité des nœuds (n/2 + 1) pour avoir le quorum**
- Exemple avec 3 nœuds : si 1 tombe, les 2 restants ont le quorum (2/3 > 50%) → le cluster continue
- Sans quorum, le cluster se met en pause pour éviter le **split-brain** (deux parties du cluster qui pensent être les seules actives et écrivent des données contradictoires)

---

### Q23. Fonctionnalité HA (High Availability) dans Proxmox

**Ta réponse :** ✅ **Correcte dans l'idée — À préciser techniquement**

**Correction complète :**

La **HA (Haute Disponibilité)** dans Proxmox permet le **redémarrage automatique des VMs** sur un autre nœud du cluster en cas de panne.

**Ce qui se passe concrètement :**
1. Un nœud du cluster tombe en panne
2. Le **HA Manager** détecte la panne (via watchdog et heartbeat entre nœuds)
3. Le quorum est vérifié (les nœuds restants doivent avoir la majorité)
4. Les VMs configurées en HA sont **redémarrées automatiquement** sur un autre nœud
5. Un **watchdog matériel** peut être utilisé pour "fencer" (isoler) le nœud défaillant

> ⚠️ Condition : le stockage doit être **partagé** (Ceph, NFS, SAN) pour que les VMs soient accessibles depuis n'importe quel nœud.

---

### Q24. RTO et RPO

**Correction complète :**

| Indicateur | Signification | Exemple |
|---|---|---|
| **RTO** (Recovery Time Objective) | **Durée maximale** acceptable d'interruption de service | RTO = 4h → le service doit être rétabli dans les 4h |
| **RPO** (Recovery Point Objective) | **Perte de données maximale** acceptable (exprimée en temps) | RPO = 1h → on peut perdre au maximum 1h de données |

**Impact sur la stratégie de sauvegarde :**
- **RPO faible (ex: 15 min)** → sauvegardes fréquentes, réplication en temps quasi-réel, snapshots réguliers
- **RTO faible (ex: 30 min)** → infrastructure redondante (HA, cluster), stockage partagé, procédures de bascule rapides
- **RPO = 0 / RTO = 0** → réplication synchrone en temps réel (coût très élevé)

---

### Q25. Règle 3-2-1 en matière de sauvegarde

**Correction complète :**

La **règle 3-2-1** est un standard de la sauvegarde :
- **3** copies des données (1 originale + 2 sauvegardes)
- **2** supports de stockage différents (ex : disque local + NAS)
- **1** copie hors site (ex : cloud, site distant)

**Application dans Proxmox :**
1. VM active sur le nœud Proxmox (copie 1)
2. Sauvegarde sur PBS (Proxmox Backup Server) local (copie 2 - support différent)
3. Réplication PBS vers un site distant ou cloud (copie 3 - hors site)

---

### Q26. PBS (Proxmox Backup Server)

**Correction complète :**

**PBS** est une solution de sauvegarde open-source dédiée aux environnements Proxmox.

**Avantages par rapport à une copie de fichier classique :**

| | Copie de fichier | PBS |
|---|---|---|
| **Déduplication** | ❌ Non | ✅ Oui (stocke une seule fois les blocs identiques) |
| **Incrémentaux** | ❌ Difficile | ✅ Natif (ne sauvegarde que ce qui a changé) |
| **Vérification d'intégrité** | ❌ Non | ✅ Vérification cryptographique (SHA-256) |
| **Restauration granulaire** | ❌ Non | ✅ Peut restaurer un seul fichier d'une VM |
| **Compression** | ❌ Non | ✅ Compression automatique |
| **Chiffrement** | ❌ Non | ✅ Chiffrement côté client |

---

### Q27. Mise à jour d'un nœud hyperviseur en production

**Ta réponse :** ⚠️ **Très incomplète**

**Correction complète :**

**Risques :**
- Redémarrage imposé après certains updates (kernel) → interruption de service
- Incompatibilité de version entre nœuds du cluster
- Régression ou bug post-mise à jour

**Procédure à suivre :**
1. **Planifier une fenêtre de maintenance** (hors heures de production)
2. **Snapshot de toutes les VMs critiques** avant la MAJ
3. En cluster : **migrer les VMs à chaud** vers un autre nœud (`Migrate` dans Proxmox)
4. **Mettre en mode maintenance** le nœud à mettre à jour
5. Appliquer les mises à jour (`apt update && apt dist-upgrade`)
6. **Redémarrer** si nécessaire (nouveau kernel)
7. Vérifier le bon fonctionnement, puis rapatrier les VMs
8. Procéder nœud par nœud (jamais tous simultanément)

---

### Q28. Reconstruction d'une grappe RAID 5 dégradée

**Ta réponse :** ✅ **Logique correcte — Commandes manquantes**

**Correction complète avec les commandes `mdadm` :**

```bash
# 1. Identifier l'état de la grappe et le disque défaillant
cat /proc/mdstat
mdadm --detail /dev/md0

# 2. Marquer le disque comme défaillant (si pas déjà fait)
mdadm --fail /dev/md0 /dev/sdb1

# 3. Retirer le disque défaillant de la grappe
mdadm --remove /dev/md0 /dev/sdb1

# 4. Remplacer physiquement le disque (ou utiliser un spare déjà connecté)

# 5. Partitionner le nouveau disque de façon identique
fdisk /dev/sdd  # créer une partition de type "Linux RAID" (fd)

# 6. Ajouter le nouveau disque à la grappe
mdadm --add /dev/md0 /dev/sdd1

# 7. Suivre la reconstruction
watch cat /proc/mdstat
```

> ⚠️ La reconstruction (rebuild) peut prendre plusieurs heures selon la taille des disques. Pendant ce temps, le RAID est **dégradé mais opérationnel**. Si un deuxième disque tombe pendant la reconstruction → perte totale des données !

---

### Q29. Isolation des VMs — Menaces contenues et limites

**Correction complète :**

**L'isolation des VMs** signifie que chaque VM est cloisonnée dans son propre environnement virtuel, sans accès direct aux ressources des autres VMs.

**Menaces contenues :**
- Une VM compromise ne peut pas accéder aux données des autres VMs
- Un malware dans une VM ne peut pas se propager directement aux autres
- Une VM crashée n'entraîne pas les autres (sauf si elle sature les ressources de l'hôte)

**Limites de l'isolation :**
- **VM Escape** : Faille de sécurité permettant à un code dans une VM de s'échapper vers l'hyperviseur (rare mais existent)
- **Side-channel attacks** : Attaques exploitant les ressources partagées (cache CPU, Spectre/Meltdown)
- **Réseau virtuel partagé** : Si les VMs sont sur le même bridge sans VLAN, elles peuvent "se voir"
- **Stockage partagé** : Une VM avec accès admin à un NAS partagé peut impacter les autres

---

### Q30. Conteneur LXC vs Machine Virtuelle KVM

**Correction complète :**

| | Conteneur LXC | VM KVM |
|---|---|---|
| **Kernel** | Partagé avec l'hôte | Propre kernel virtuel |
| **Isolation** | Logicielle (namespaces Linux) | Matérielle (hyperviseur) |
| **Performance** | Excellente (quasi-native) | Très bonne (légère pénalité) |
| **Empreinte mémoire** | Très faible | Plus importante |
| **Démarrage** | Quelques secondes | 30s à plusieurs minutes |
| **OS invité** | Linux uniquement | N'importe quel OS (Windows, BSD...) |
| **Sécurité** | Isolation moins forte | Isolation plus forte |

**Quand utiliser LXC :**
- Services Linux légers (serveur web, DNS, proxy...)
- Environnements multiples identiques (dev/preprod/prod)
- Quand les performances sont critiques

**Quand éviter LXC (préférer KVM) :**
- Besoin d'un OS non-Linux (Windows Server)
- Isolation de sécurité maximale requise
- Application nécessitant un kernel spécifique ou des modules personnalisés
- Environnement de test nécessitant un comportement identique à la production

---

## BONUS — Q31. Plan de résilience avec budget limité

**Correction / Proposition d'architecture :**

**Situation actuelle :** 1 serveur physique → SPOF total.

**Architecture améliorée proposée (par ordre de priorité) :**

**Priorité 1 — Sauvegarde (coût minimal)**
- Installer **PBS** sur un NAS ou un deuxième petit serveur
- Appliquer la règle **3-2-1** : sauvegardes locales + copie distante (cloud ou site distant)
- Planifier des sauvegardes **quotidiennes** avec rétention sur 30 jours

**Priorité 2 — Stockage redondant**
- Mettre en place du **RAID 5 ou RAID 6** sur le serveur existant pour les disques de VMs
- Envisager un **NAS partagé** (NFS/iSCSI) pour préparer une future migration vers un cluster

**Priorité 3 — Clustering (si budget permet)**
- Ajouter un **deuxième nœud Proxmox** et former un cluster
- Utiliser un **stockage partagé** (NFS ou Ceph) pour permettre la migration à chaud
- Activer la **HA** sur les VMs critiques (AD, messagerie, fichiers)

**Priorité 4 — Documentation et procédures**
- Documenter les **procédures de restauration** (et les tester !)
- Définir les **RTO/RPO** pour chaque service
- Mettre en place une **supervision** (Zabbix, Grafana) pour détecter les problèmes avant la panne

> 💡 **Ordre logique :** Commencer par la sauvegarde (protection des données) avant d'investir dans la haute disponibilité (continuité de service). Un cluster sans sauvegarde ne protège pas contre les erreurs humaines ou les ransomwares.

---

## Récapitulatif de tes résultats

| Niveau | Questions | Résultat |
|---|---|---|
| Niveau 1 | Q1 à Q8 | Q1✅ Q2⚠️ Q3✅ Q4⚠️ Q5✅ Q6❌ Q7❌ Q8✅ |
| Niveau 2 | Q9 à Q20 | Q9✅ Q10⚠️ Q11❌ Q12❌ Q13❌ Q14❌ Q15❌ Q16❌ Q17— Q18— Q19❌ Q20— |
| Niveau 3 | Q21 à Q30 | Q21— Q22— Q23✅ Q24— Q25— Q26— Q27⚠️ Q28✅ Q29— Q30— |

**Points forts :** Bonne compréhension des concepts de base (types d'hyperviseurs, VMs, snapshots, HA).

**Points à travailler :**
- Les formats de disques virtuels (VMDK, QCOW2, VHD)
- LVM (architecture PV/VG/LV et commandes)
- Le réseau virtuel dans Proxmox (bridge, VLAN)
- Les commandes `mdadm` pour RAID
- RTO/RPO et stratégies de sauvegarde
- Différence LXC vs KVM en détail

---

*Correction réalisée dans le cadre de la formation TSSR — CP5*
