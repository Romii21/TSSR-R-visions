# 🪟 FICHE 3 – Outils de Sauvegarde sous Windows Server
### CP8 – Mettre en place, assurer et tester les sauvegardes et restaurations

---

## 1. VSS – Volume Shadow Copy Service

### Rôle et principe

Le **VSS (Volume Shadow Copy Service)** est un service Windows qui coordonne la création de **clichés instantanés cohérents** de volumes, même lorsque des fichiers sont ouverts et en cours d'utilisation par des applications.

### Comment ça fonctionne

```
Processus VSS lors d'une sauvegarde :

1. L'outil de sauvegarde demande un cliché au service VSS
2. VSS contacte les "Writers" (SQL Server, Exchange, AD, etc.)
3. Les Writers mettent en pause leurs écritures / vident leurs tampons
4. VSS prend le snapshot du volume (opération quasi-instantanée)
5. Les Writers reprennent leurs activités normalement
6. L'outil de sauvegarde lit les données depuis le snapshot cohérent
```

### Les acteurs VSS

| Acteur | Rôle |
|--------|------|
| **VSS Requestor** | L'outil qui demande la sauvegarde (Windows Server Backup, Veeam, etc.) |
| **VSS Provider** | Gère la création du cliché (Software Provider ou matériel) |
| **VSS Writer** | Application qui se coordonne pour assurer la cohérence (SQL Writer, Exchange Writer, NTDS Writer pour AD) |

### Cas où VSS est indispensable

- Sauvegarde d'un **serveur SQL Server** en production (les fichiers .mdf/.ldf sont toujours ouverts)
- Sauvegarde d'**Exchange Server** (base de données de messagerie ouverte en permanence)
- Sauvegarde d'un **contrôleur de domaine Active Directory** (NTDS.dit ouvert)

> ⚠️ **Sans VSS :** Les fichiers ouverts seraient copiés dans un état incohérent → restauration impossible ou base corrompue.

---

## 2. Windows Server Backup

### Installation du rôle

```powershell
# Via PowerShell
Install-WindowsFeature Windows-Server-Backup

# Vérifier l'installation
Get-WindowsFeature Windows-Server-Backup
```

### Configuration d'une sauvegarde planifiée (GUI)

1. Ouvrir **Gestionnaire de serveur** → Outils → **Windows Server Backup**
2. Cliquer sur **Planification de sauvegarde** (volet Actions)
3. Choisir **Configuration personnalisée** → **Sauvegarde complète du serveur** (ou volumes sélectionnés)
4. Définir la fréquence : quotidienne, et l'heure (ex : 23h00)
5. Sélectionner la destination :
   - **Volumes dédiés** (disque dédié, recommandé pour perfs)
   - **Partage réseau distant** → entrer le chemin UNC (`\\backup-srv\sauvegardes`)
6. Entrer les identifiants d'accès au partage réseau
7. Valider et terminer l'assistant

### Commandes wbadmin essentielles

```cmd
:: Lancer une sauvegarde complète manuelle vers un partage réseau
wbadmin start backup -backupTarget:\\backup-srv\sauvegardes -include:C: -allCritical -quiet

:: Sauvegarder uniquement le System State (état du système)
wbadmin start systemstatebackup -backupTarget:E:

:: Lister les sauvegardes disponibles
wbadmin get versions

:: Obtenir les détails d'une sauvegarde
wbadmin get items -version:MM/JJ/AAAA-HH:MM

:: Restaurer un fichier spécifique
wbadmin start recovery -version:MM/JJ/AAAA-HH:MM -itemType:File -items:C:\dossier\fichier.txt -recursive -recoveryTarget:C:\restauration

:: Restaurer le System State
wbadmin start systemstaterecovery -version:MM/JJ/AAAA-HH:MM
```

### Ce qu'inclut le System State

Le **System State** est un ensemble de composants système critiques sauvegardés ensemble :

| Composant | Description |
|-----------|-------------|
| **NTDS.dit** | Base de données Active Directory (DC uniquement) |
| **SYSVOL** | Scripts de connexion et templates GPO (DC uniquement) |
| **Registre Windows** | HKLM, HKCU et toutes les ruches |
| **Fichiers de démarrage** | BCD, bootmgr, hal.dll |
| **Base COM+** | Informations d'inscription des composants COM+ |
| **Certificats** | Si le rôle ADCS est installé |

> 💡 **À retenir :** La sauvegarde du System State est l'opération minimale indispensable sur un contrôleur de domaine.

---

## 3. Clichés instantanés (Previous Versions / Shadow Copies)

### Principe

Les **clichés instantanés** (Shadow Copies) sont des versions précédentes de fichiers et dossiers stockés sur un volume NTFS. Ils permettent une **restauration rapide et autonome** sans passer par un outil de sauvegarde externe.

### Activation côté serveur

1. **Gestionnaire de serveur** → Fichiers et services de stockage → **Partages**
2. Clic droit sur le volume → **Configurer les clichés instantanés**
3. Sélectionner le volume → **Activer**
4. Configurer la planification (ex : 2 fois par jour) et la limite d'espace

```powershell
# Activation via PowerShell
Enable-ShadowCopy -Volume C:

# Créer un cliché manuel
vssadmin create shadow /for=C:

# Lister les clichés existants
vssadmin list shadows
```

### Restauration d'un fichier supprimé (côté utilisateur)

1. Naviguer jusqu'au **dossier parent** du fichier supprimé
2. Clic droit sur le dossier → **Propriétés**
3. Onglet **"Versions précédentes"**
4. Sélectionner la version antérieure souhaitée dans la liste
5. Cliquer sur **"Restaurer"** (remet le fichier en place) ou **"Ouvrir"** (accès sans modifier l'original)

> ✅ **Avantage :** L'utilisateur peut souvent restaurer lui-même son fichier sans intervention du technicien.
> ⚠️ **Limite :** Les clichés restent sur le même volume → pas de protection contre la panne disque.

---

## 4. Snapshots Hyper-V (Points de contrôle)

### Fonctionnement

Un **point de contrôle Hyper-V** (anciennement "snapshot") sauvegarde l'état complet d'une VM à un instant T :
- **État de la mémoire RAM** (si VM en fonctionnement)
- **Configuration de la VM** (fichier XML)
- **Disque différentiel** (`.avhd` / `.avhdx`) : les modifications suivantes sont écrites dans ce fichier, le disque d'origine reste figé

```
Avant snapshot :
  [Disque VM] ← toutes les écritures

Après snapshot :
  [Disque VM original] ← figé
  [Fichier différentiel .avhdx] ← nouvelles écritures
```

### Deux types de points de contrôle (Windows Server 2016+)

| Type | Description | Quand l'utiliser |
|------|-------------|-----------------|
| **Standard** | Capture mémoire + disque + état applicatif | Environnements de test/dev |
| **Production** | Utilise VSS pour une cohérence applicative, sans capture mémoire | Environnements de production |

### Risques de la conservation prolongée en production

| Risque | Explication |
|--------|-------------|
| **Dégradation des performances** | Chaque I/O traverse la chaîne des fichiers différentiels |
| **Consommation disque** | Le fichier `.avhdx` grossit proportionnellement aux modifications |
| **Fusion longue et risquée** | Supprimer un vieux snapshot peut prendre des heures et bloquer la VM |
| **Corruption potentielle** | Si la fusion est interrompue (panne, coupure), la VM peut devenir inutilisable |

> ⚠️ **Règle d'or :** Un snapshot Hyper-V en production doit être **temporaire** (quelques heures à 24h maximum). Il n'est pas une solution de sauvegarde à long terme.

### Commandes PowerShell Hyper-V

```powershell
# Créer un point de contrôle
Checkpoint-VM -Name "NomVM" -SnapshotName "Avant_MàJ_$(Get-Date -Format 'yyyyMMdd')"

# Lister les points de contrôle d'une VM
Get-VMSnapshot -VMName "NomVM"

# Restaurer un point de contrôle
Restore-VMSnapshot -VMName "NomVM" -Name "Avant_MàJ_20240115" -Confirm:$false

# Supprimer un point de contrôle (déclenche la fusion)
Remove-VMSnapshot -VMName "NomVM" -Name "Avant_MàJ_20240115"
```

---

## 5. Tableau de synthèse des outils Windows Server

| Outil | Usage | Commande / Emplacement |
|-------|-------|------------------------|
| **Windows Server Backup** | Sauvegarde complète/planifiée | `wbadmin` / Gestionnaire de serveur |
| **wbadmin start backup** | Sauvegarde manuelle | `wbadmin start backup -backupTarget:...` |
| **wbadmin start systemstatebackup** | System State (AD, registre) | `wbadmin start systemstatebackup -backupTarget:E:` |
| **VSS (vssadmin)** | Gestion des clichés instantanés | `vssadmin list shadows` |
| **Previous Versions** | Restauration fichier par utilisateur | Clic droit → Propriétés → Versions précédentes |
| **Checkpoint-VM** | Snapshot Hyper-V | PowerShell Hyper-V |

---

## 🎯 Récapitulatif à mémoriser

- **VSS** = le chef d'orchestre qui assure la cohérence des sauvegardes à chaud sous Windows
- **wbadmin** = outil CLI pour toutes les opérations de sauvegarde/restauration Windows Server Backup
- **System State** = sauvegarde minimale obligatoire sur un DC (contient NTDS.dit + SYSVOL)
- **Clichés instantanés** = restauration rapide de fichiers supprimés, sans outil tiers
- **Snapshot Hyper-V** = temporaire, jamais en production prolongée
