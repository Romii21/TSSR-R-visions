# Correction – CP5 : Maintenir des serveurs dans une infrastructure virtualisée

> **Légende** : ✅ Correct | ⚠️ Incomplet ou imprécis | ❌ Incorrect ou manquant

---

## Question 1 – Hyperviseur Type 1 vs Type 2

**Résultat : ⚠️ Globalement correct, mais quelques imprécisions**

### Ce que tu as dit

Tu as bien compris la distinction essentielle : le Type 1 tourne directement sur le matériel, le Type 2 est un logiciel installé sur un OS existant. Les exemples sont globalement bons.

### Ce qui pose problème

- Le mot **"émule"** pour le Type 2 est techniquement inexact. Un hyperviseur Type 2 **virtualise**, il ne fait pas d'émulation au sens strict (l'émulation simule un matériel différent, par exemple jouer à une ROM GameBoy sur PC). On dit plutôt qu'il **virtualise les VMs au-dessus d'un OS hôte**.
- Tu as cité **VMware ESXI** : la bonne orthographe est **VMware ESXi** (pas d'accent, pas de majuscule sur le X).
- Il manquait le contexte d'utilisation demandé : le Type 1 est utilisé en **production en entreprise / datacenter** car il est plus performant (accès direct au matériel, pas d'overhead de l'OS hôte). Le Type 2 est utilisé pour le **développement, les tests, et les postes de travail personnels**.

### La réponse attendue

| |Type 1 (Bare Metal)|Type 2 (Hosted)|
|---|---|---|
|**Fonctionnement**|S'exécute directement sur le matériel physique, sans OS hôte|S'installe sur un OS hôte existant (Windows, Linux)|
|**Performances**|Meilleures (accès direct au matériel)|Moins bonnes (overhead de l'OS hôte)|
|**Exemples**|VMware ESXi, Microsoft Hyper-V, Proxmox VE, Citrix XenServer|VirtualBox, VMware Workstation, Parallels Desktop|
|**Usage en entreprise**|Serveurs de production, datacenters, cloud|Développement, tests, postes de travail|

---

## Question 2 – La consolidation de serveurs

**Résultat : ❌ Non répondu**

### La réponse attendue

La **consolidation de serveurs** consiste à regrouper plusieurs services (qui tournaient chacun sur un serveur physique dédié) sur un nombre réduit de serveurs physiques, en s'appuyant sur la virtualisation.

**Intérêt économique :**

- Réduction du nombre de machines à acheter et à maintenir.
- Réduction des coûts d'électricité et de refroidissement.
- Diminution de l'espace occupé en salle serveur (rack).

**Intérêt technique :**

- Les serveurs physiques étaient souvent utilisés à 30-40% de leur capacité. La virtualisation permet d'atteindre 70-90% d'utilisation des ressources.
- Déploiement rapide de nouvelles VMs à partir de templates.
- Simplification de la sauvegarde et de la restauration (snapshot de VM).

**Limite importante à citer :** Le **point de défaillance unique** (Single Point of Failure). Si le serveur physique hôte tombe en panne, toutes les VMs qu'il héberge sont impactées simultanément. Il faut donc coupler la consolidation avec une solution de haute disponibilité (cluster HA, serveur de secours).

---

## Question 3 – VM vs Conteneur

**Résultat : ⚠️ Partiellement correct, mais une erreur importante**

### Ce que tu as bien dit

Tu as bien compris qu'une VM embarque son propre OS complet (RAM, stockage, environnement isolé), et qu'un conteneur est plus léger car il partage le noyau avec l'hôte.

### Ce qui est incorrect

> _"les conteneurs ne fonctionnent qu'avec Linux"_

C'est **faux**. Les conteneurs Docker fonctionnent nativement sur Linux, mais ils fonctionnent aussi sur **Windows** (Docker Desktop utilise une couche de compatibilité, et Windows possède ses propres conteneurs Windows natifs depuis Windows Server 2016). En entreprise, il est très courant de faire tourner Docker sur Windows Server.

Il manque aussi le contexte de choix entre les deux :

### La réponse attendue

| |Machine Virtuelle (VM)|Conteneur (Docker / LXC)|
|---|---|---|
|**OS**|OS complet dédié (noyau inclus)|Partage le noyau de l'OS hôte|
|**Isolation**|Forte (matériel virtualisé)|Plus légère (isolation au niveau processus)|
|**Démarrage**|Lent (minutes)|Rapide (secondes)|
|**Ressources**|Lourdes (RAM, disque élevés)|Légères|
|**Cas d'usage**|Quand on a besoin d'un OS différent, d'une isolation forte, ou pour des services système complets (AD, serveur de fichiers)|Pour déployer des applications microservices, des APIs, des applications web portables|

**Exemple concret** : on préférera une VM pour héberger un contrôleur de domaine Active Directory sous Windows Server, et un conteneur Docker pour déployer une application web Node.js avec ses dépendances.

---

## Question 4 – RAID 5

**Résultat : ⚠️ La logique est bonne mais la description technique est imprécise**

### Ce que tu as bien dit

- Minimum 3 disques ✅
- Il y a un mécanisme qui permet de reconstruire un disque en cas de panne ✅

### Ce qui est imprécis

Ta description ("les données s'inscrivent sur le premier, puis le second, puis le troisième") décrit plutôt le **RAID 0** (striping simple). Le RAID 5 c'est du **striping avec parité répartie**, ce n'est pas exactement la même chose.

Il manquait aussi les **avantages, limites et la capacité utile**, qui étaient explicitement demandés.

### La réponse attendue

**Fonctionnement du RAID 5 :** Les données sont découpées en blocs et réparties sur tous les disques (striping). En plus, une **parité** (calculée par XOR) est répartie en rotation sur tous les disques. Cette parité permet de recalculer les données d'un disque manquant.

```
Disque 1 : [Data A1] [Data B1] [Parité C]
Disque 2 : [Data A2] [Parité B] [Data C1]
Disque 3 : [Parité A] [Data B2] [Data C2]
```

**Avantages :**

- Tolère la panne d'**1 disque** sans perte de données.
- Bonne capacité utile : **(n-1) × capacité d'un disque**.
- Bonnes performances en lecture.

**Limites :**

- Ne tolère qu'**1 panne**. Si un 2e disque lâche pendant la reconstruction, toutes les données sont perdues.
- Les **performances en écriture** sont réduites (calcul de parité à chaque écriture).
- La **reconstruction** est longue et sollicite fortement les disques restants.

**Minimum : 3 disques.** Capacité utile avec 3 disques de 1 To = **2 To**.

---

## Question 5 – RAID ≠ Sauvegarde

**Résultat : ⚠️ Tu as la bonne idée mais tu n'as pas cité les deux situations demandées**

### Ce que tu as bien dit

Tu as correctement expliqué le principe du RAID 1 (mirroring) et le fait qu'il ne remplace pas une sauvegarde. ✅

### Ce qui manque

La question demandait **au moins deux situations** concrètes que le RAID ne couvre pas. Tu ne les as pas citées explicitement.

### La réponse attendue

**Le RAID protège uniquement contre la défaillance matérielle d'un disque. Il ne peut pas couvrir :**

1. **La suppression accidentelle de fichiers** : si tu effaces un fichier par erreur, cette suppression est immédiatement répliquée sur le miroir. Les données sont perdues des deux côtés.
2. **Les ransomwares / malwares** : si un virus chiffre tes données, les fichiers chiffrés sont synchronisés en temps réel sur le disque miroir. Le RAID est inutile.
3. **La corruption de données logicielle** : une base de données corrompue par un bug sera corrompue sur les deux disques.
4. **L'incendie / inondation / vol** : les deux disques sont au même endroit physiquement.

**Conclusion à retenir : RAID ≠ Sauvegarde. Les deux sont complémentaires, pas substituables.**

---

## Question 6 – Architecture LVM

**Résultat : ❌ Non répondu**

### La réponse attendue

LVM (Logical Volume Manager) est organisé en **3 couches** :

```
Disques physiques (PV) → Pool de stockage (VG) → Partitions logiques (LV) → Systèmes de fichiers
```

**1. PV – Physical Volume (Volume Physique)** Un PV est un disque ou une partition préparé pour LVM (ex : `/dev/sda1`). C'est la brique de base.

**2. VG – Volume Group (Groupe de Volumes)** Un VG regroupe un ou plusieurs PV en un seul **pool de stockage**. C'est comme une réserve d'espace depuis laquelle on va découper des partitions logiques.

**3. LV – Logical Volume (Volume Logique)** Un LV est l'équivalent d'une partition. On y formate un système de fichiers (ext4, XFS…) et on le monte dans l'arborescence.

**Avantage majeur vs partitionnement traditionnel :** Le redimensionnement **à chaud** (sans démonter ni redémarrer). Avec le partitionnement classique, agrandir une partition nécessite souvent de démarrer sur un live CD. Avec LVM, on peut agrandir un LV en production en quelques secondes avec `lvextend -r`.

---

## Question 7 – Snapshot LVM

**Résultat : ❌ Non répondu**

### La réponse attendue

Un **snapshot LVM** est une copie figée d'un volume logique à un instant T. Il permet de "photographier" l'état d'un LV sans interrompre le service.

**Mécanisme technique : COW (Copy-On-Write / Copie à l'écriture)**

- À la création du snapshot : il référence les mêmes blocs que l'original, sans copie physique (quasi-instantané).
- Quand l'original est modifié : les **anciennes données** sont copiées dans le snapshot _avant_ l'écriture des nouvelles données.
- Résultat : le snapshot garde toujours l'image de l'état initial, même si l'original évolue.

**Cas d'usage concret en maintenance de VM :** Avant d'effectuer une mise à jour majeure d'un serveur (ex : mise à jour d'un serveur web), on crée un snapshot LVM du disque de la VM. Si la mise à jour casse quelque chose, on restaure le snapshot en quelques secondes avec `lvconvert --merge`, sans avoir à réinstaller.

```bash
# Créer le snapshot
lvcreate -L 5G -s -n snap_avant_maj /dev/vg_vms/lv_web

# En cas de problème, restaurer
lvconvert --merge /dev/vg_vms/snap_avant_maj
```

---

## Question 8 – RTO et RPO

**Résultat : ❌ Les définitions sont inversées et imprécises**

### Ce qui est incorrect

> _"Le RTO est le temps acceptable pour la récupération des données"_

C'est la définition du **RPO**, pas du RTO. Et le RPO n'est **pas un pourcentage**, c'est une **durée**.

### La réponse attendue

**RTO – Recovery Time Objective (Objectif de temps de reprise)** Durée maximale acceptable d'interruption de service après un incident. C'est la réponse à : _"Dans combien de temps au maximum nos services doivent-ils être de nouveau opérationnels ?"_

Exemple : RTO = 4h → le service doit être rétabli dans les 4 heures suivant la panne.

**RPO – Recovery Point Objective (Objectif de point de reprise)** Perte de données maximale acceptable, exprimée en **durée**. C'est la réponse à : _"Jusqu'à quand en arrière peut-on remonter pour restaurer les données ?"_

Exemple : RPO = 24h → on accepte de perdre au maximum les données des dernières 24h. Cela implique de faire une sauvegarde au minimum toutes les 24h.

**Importance dans un PRA sur infrastructure virtualisée :** Ces deux indicateurs dictent directement les choix techniques :

- Un RTO court → nécessite un cluster HA avec basculement automatique des VMs.
- Un RPO court → nécessite des sauvegardes fréquentes des VMs (snapshots planifiés, réplication).

---

## Question 9 – Migration à chaud (Live Migration)

**Résultat : ⚠️ Correct dans l'idée, mais incomplet sur le fonctionnement**

### Ce que tu as bien dit

- Déplacer une VM d'un nœud à un autre sans l'éteindre ✅
- Utile pour le partage de ressources ✅

### Ce qui manque

La question demandait d'**expliquer le fonctionnement général**. Tu ne l'as pas fait.

### La réponse attendue

**Fonctionnement général de la live migration (ex : Proxmox, VMware vMotion) :**

1. La mémoire RAM de la VM est **copiée progressivement** vers le nœud de destination, pendant que la VM continue de tourner.
2. Les pages mémoire qui changent pendant la copie (dirty pages) sont recopiées.
3. Quand la quantité de données restantes à copier est négligeable, la VM est **suspendue très brièvement** (quelques millisecondes à quelques secondes).
4. Le reste de l'état (CPU, mémoire finale) est transféré, et la VM **reprend sur le nœud de destination**.

**Prérequis :** un stockage partagé (NAS/SAN) accessible par les deux nœuds, ou une réplication du stockage.

**Cas d'usage utiles :**

- Effectuer une maintenance matérielle sur un nœud (remplacement RAM, disque) **sans interruption de service**.
- Rééquilibrer la charge entre hyperviseurs.
- Vider un nœud avant de le mettre hors tension.

---

## Question 10 – Haute Disponibilité (HA) en cluster

**Résultat : ✅ Bonne compréhension globale, quelques points à préciser**

### Ce que tu as bien dit

- En cas de défaillance d'un nœud, les VMs redémarrent sur les autres nœuds ✅
- Le stockage partagé empêche la perte de données ✅

### Ce qui manque

- La HA n'est pas de la **migration à chaud** (live migration). La VM est redémarrée, pas transférée sans interruption. Il y a un court temps d'interruption (quelques secondes à quelques minutes selon la configuration).
- Le **rôle précis du stockage partagé** : les disques virtuels des VMs sont stockés sur ce stockage partagé (NAS/SAN), ce qui permet à n'importe quel nœud du cluster de les démarrer. Sans stockage partagé, le nœud de secours n'aurait pas accès aux données de la VM.

---

## Question 11 – NAS vs SAN

**Résultat : ❌ Confusion importante entre NAS et SAN**

### Ce qui est incorrect

Ta réponse décrit en réalité un **SAN** ("besoin de beaucoup de ressources spécifiques") quand tu parles du NAS, et tu inverses les usages. Le NAS n'est **pas** adapté à un "plus petit réseau" en opposition au SAN, la distinction est de nature technique, pas de taille.

### La réponse attendue

| |NAS (Network Attached Storage)|SAN (Storage Area Network)|
|---|---|---|
|**Niveau d'accès**|**Fichiers** (haut niveau : NFS, SMB)|**Blocs** (bas niveau : iSCSI, Fibre Channel)|
|**Analogie**|Serveur de fichiers distant|Disque dur distant|
|**Complexité**|Simple à mettre en place|Complexe, réseau dédié souvent requis|
|**Coût**|Abordable|Élevé (Fibre Channel notamment)|
|**Performance**|Correcte|Excellente (très faible latence)|
|**Usage typique**|Partage de documents, bureautique, sauvegardes|Disques virtuels de VMs, bases de données critiques|

**Pour héberger des disques virtuels de VMs**, le **SAN est plus adapté** car il offre un accès au niveau bloc (comme un disque local), ce qui donne de meilleures performances pour les I/O intensifs des VMs. Le protocole **iSCSI** est le plus courant en PME car il fonctionne sur Ethernet standard. Le NAS via NFS peut aussi être utilisé pour des VMs avec Proxmox, mais avec des performances inférieures.

---

## Question 12 – Proxmox VE et pools de stockage

**Résultat : ❌ Non répondu**

### La réponse attendue

Dans **Proxmox VE**, les pools de stockage sont configurés depuis l'interface web dans **Datacenter > Storage**. Chaque pool de stockage est identifié par un nom (ex : `local`, `local-lvm`, `nfs-vms`) et est associé à un type de backend.

**Deux types de stockage supportés nativement :**

1. **LVM-Thin (local-lvm)** : stockage local basé sur LVM en mode thin provisioning. Utilisé pour stocker les disques virtuels des VMs et conteneurs LXC en local. Supporte les snapshots natifs. C'est le stockage par défaut lors de l'installation de Proxmox.
    
2. **NFS** : stockage réseau de type fichiers. Utilisé pour stocker des images ISO, des backups de VMs, ou des disques virtuels partagés entre plusieurs nœuds (utile pour la HA et la live migration). Nécessite un serveur NFS sur le réseau.
    

On peut aussi citer **ZFS**, **CIFS/SMB**, **iSCSI**, **Ceph** (pour du cluster distribué), tous supportés nativement.

---

## Question 13 – Précautions avant mise à jour de l'hyperviseur

**Résultat : ⚠️ La bonne idée mais réponse très incomplète**

### Ce que tu as bien dit

Faire un snapshot/sauvegarde avant la mise à jour ✅ (c'est effectivement l'étape la plus importante).

### Ce qui manque

La procédure complète attendue à niveau TSSR :

### La réponse attendue (procédure complète)

1. **Consulter les notes de version** de la mise à jour (release notes) pour identifier les changements majeurs, les incompatibilités ou les procédures de migration spécifiques.
2. **Planifier une fenêtre de maintenance** et en informer les utilisateurs concernés.
3. **Effectuer un snapshot ou une sauvegarde complète** de toutes les VMs critiques hébergées sur le serveur.
4. Si le serveur fait partie d'un cluster HA : **migrer les VMs à chaud** (live migration) vers un autre nœud du cluster pour vider le serveur à mettre à jour.
5. **Vérifier l'intégrité des sauvegardes** (tester une restauration si possible).
6. Appliquer la mise à jour **en dehors des heures de production** si les VMs ne peuvent pas être migrées.
7. **Surveiller les logs et le comportement** du système après la mise à jour.
8. En cas de problème : restaurer depuis le snapshot ou la sauvegarde.

---

## Question 14 – Principe du moindre privilège

**Résultat : ⚠️ Correct dans l'esprit, mais définition incomplète**

### Ce que tu as bien dit

La notion d'attribution des droits via les groupes est juste ✅.

### Ce qui manque

La définition formelle du **moindre privilège** et son application concrète dans Proxmox.

### La réponse attendue

**Définition :** Le principe du moindre privilège consiste à donner à chaque utilisateur, compte de service ou processus **uniquement les droits strictement nécessaires** à l'accomplissement de sa tâche, et rien de plus.

**Application concrète dans Proxmox VE :** Proxmox dispose d'un système de permissions basé sur des **rôles** (RBAC : Role-Based Access Control) :

- On crée des **groupes** (ex : `admins`, `helpdesk`, `monitoring`).
- On assigne à chaque groupe un **rôle** sur une ressource précise (ex : le groupe `helpdesk` a le rôle `PVEVMUser` uniquement sur les VMs de production, sans accès au stockage ni à la configuration réseau).
- Un technicien helpdesk peut ainsi démarrer/arrêter une VM sans pouvoir modifier la configuration de l'hyperviseur ou accéder aux autres VMs.

**Dans VMware**, cela se fait via les **vCenter Roles and Permissions** avec le même principe.

---

## Question 15 – Reconstruction d'un RAID 5 après panne

**Résultat : ⚠️ Tu as la logique mais sans les commandes ni les détails de vérification**

### Ce que tu as bien dit

La logique de la procédure est correcte : vérifier, supprimer le défaillant, remplacer physiquement, ajouter dans la grappe ✅.

### Ce qui manque

Les commandes `mdadm`, et surtout les **vérifications** demandées dans la question.

### La réponse attendue (procédure complète avec commandes)

**Contexte : RAID 5 Linux géré avec `mdadm`, grappe `/dev/md0`, disque défaillant `/dev/sdb1`**

**Étape 1 : Vérifier l'état de la grappe et identifier le disque défaillant**

```bash
cat /proc/mdstat
mdadm --detail /dev/md0
```

**Étape 2 : Marquer le disque comme défaillant (si pas déjà fait automatiquement)**

```bash
mdadm --fail /dev/md0 /dev/sdb1
```

**Étape 3 : Retirer le disque défaillant de la grappe**

```bash
mdadm --remove /dev/md0 /dev/sdb1
```

**Étape 4 : Remplacer le disque physiquement** (hot-swap si le serveur le supporte, sinon extinction programmée).

**Étape 5 : Ajouter le nouveau disque dans la grappe**

```bash
mdadm --add /dev/md0 /dev/sdb1
```

La reconstruction démarre automatiquement.

**Étape 6 : Surveiller la progression de la reconstruction**

```bash
watch cat /proc/mdstat
```

La reconstruction peut prendre plusieurs heures selon la taille des disques.

**Étape 7 : Vérifications finales**

```bash
# Vérifier que la grappe est revenue à l'état "clean"
mdadm --detail /dev/md0

# Vérifier que les VMs fonctionnent correctement (tester un accès fichier, ping, services)

# Vérifier les logs système pour détecter d'autres anomalies
journalctl -xe | grep md
```

**⚠️ Attention critique :** Pendant la reconstruction, si un 2e disque tombe en panne, **toutes les données sont perdues** (le RAID 5 ne tolère qu'1 panne). Il faut surveiller attentivement l'état des disques restants pendant cette phase.

---

## Bilan global

|Question|Sujet|Résultat|
|---|---|---|
|Q1|Hyperviseurs Type 1 / Type 2|⚠️ Incomplet|
|Q2|Consolidation de serveurs|❌ Non répondu|
|Q3|VM vs Conteneur|⚠️ Erreur sur les conteneurs Windows|
|Q4|RAID 5|⚠️ Imprécis|
|Q5|RAID ≠ Sauvegarde|⚠️ Incomplet|
|Q6|Architecture LVM|❌ Non répondu|
|Q7|Snapshot LVM|❌ Non répondu|
|Q8|RTO et RPO|❌ Définitions inversées|
|Q9|Live Migration|⚠️ Incomplet|
|Q10|Haute Disponibilité (HA)|✅ Bonne compréhension|
|Q11|NAS vs SAN|❌ Confusion|
|Q12|Proxmox – pools de stockage|❌ Non répondu|
|Q13|Procédure avant mise à jour|⚠️ Incomplet|
|Q14|Moindre privilège|⚠️ Incomplet|
|Q15|Reconstruction RAID 5|⚠️ Sans commandes ni vérifications|

### Points forts

- Tu as de bonnes intuitions globales, notamment sur la HA, la live migration et le principe du moindre privilège.
- Tu comprends les concepts de base (RAID, VM, hyperviseurs).

### Points à retravailler en priorité

1. **LVM** (Q6 et Q7) : architecture PV/VG/LV et snapshots COW — à relire en détail.
2. **RTO / RPO** (Q8) : les définitions sont inversées, à mémoriser avec un exemple concret.
3. **NAS vs SAN** (Q11) : la différence fichiers/blocs est fondamentale dans un contexte de VMs.
4. **Les commandes mdadm** (Q15) : à pratiquer en TP.
5. **Consolidation de serveurs** (Q2) : concept clé du CP5, à ne pas passer à côté.