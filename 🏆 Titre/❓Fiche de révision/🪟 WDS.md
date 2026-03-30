# 📚 Fiche de révision — Déploiement d'images & WDS

---

## 🔑 Concepts fondamentaux

| Terme | Définition |
|---|---|
| **WDS** | Windows Deployment Services — rôle Windows Server pour déployer des OS via le réseau |
| **MDT** | Microsoft Deployment Toolkit — outil complémentaire pour automatiser les déploiements |
| **PXE** | Pre-boot eXecution Environment — démarrage d'une machine via le réseau sans OS |
| **WIM** | Windows Imaging Format — format de fichier image utilisé par Microsoft (`.wim`) |
| **Sysprep** | System Preparation Tool — prépare une image Windows pour le déploiement (généralisation) |
| **DISM** | Deployment Image Servicing and Management — outil CLI pour créer/modifier des images |

---

## 🖼️ Les deux types d'images

### 📀 Image d'installation (Install Image)

> C'est l'**OS à installer** sur les machines cibles.

| Propriété | Détail |
|---|---|
| **Fichier** | `install.wim` (ou `install.esd`) |
| **Source** | ISO Windows officielle (`sources\install.wim`) |
| **Contenu** | Le système d'exploitation complet |
| **Rôle** | Déployé sur le disque de la machine cible |
| **Personnalisation** | Possible (ajout de drivers, mises à jour, logiciels) |
| **Emplacement WDS** | Dossier `Install Images` |

---

### 🥾 Image de démarrage (Boot Image)

> C'est l'**environnement minimal** qui permet de démarrer la machine avant l'installation.

| Propriété | Détail |
|---|---|
| **Fichier** | `boot.wim` |
| **Source** | ISO Windows (`sources\boot.wim`) |
| **Contenu** | Windows PE (WinPE) — environnement léger pré-installation |
| **Rôle** | Démarre via PXE, lance l'assistant d'installation ou les scripts MDT |
| **Emplacement WDS** | Dossier `Boot Images` |

---

### ⚡ Image de capture (Capture Image)

> Variante de la boot image pour **capturer** une image d'un PC référence.

```
PC référence (Sysprep) ──▶ Boot sur image de capture ──▶ Génère un .wim ──▶ Stocké sur WDS
```

---

## 🗂️ Résumé des images

| Type | Fichier | Rôle | Chargée par |
|---|---|---|---|
| **Boot Image** | `boot.wim` | Démarrer l'environnement WinPE | PXE / WDS |
| **Install Image** | `install.wim` | Installer l'OS sur le disque | WinPE / WDS |
| **Capture Image** | `boot.wim` modifié | Capturer un PC référence | PXE / WDS |
| **Discover Image** | `boot.wim` modifié | Rediriger vers WDS sans config PXE | USB / ISO |

---

## 🌐 WDS — Windows Deployment Services

### C'est quoi ?
Rôle Windows Server permettant de **déployer des images Windows via le réseau** (PXE), sans média physique (clé USB, DVD).

### Prérequis

```
✅ Windows Server avec rôle WDS installé
✅ Active Directory (domaine)
✅ Serveur DHCP (options 66 et 67 configurées)
✅ DNS
✅ Partage réseau pour stocker les images
✅ BIOS/UEFI des clients configuré pour démarrer sur réseau (PXE)
```

### Options DHCP pour PXE

| Option DHCP | Valeur | Rôle |
|---|---|---|
| **Option 66** | IP du serveur WDS | Adresse du serveur de boot TFTP |
| **Option 67** | `boot\x64\wdsnbp.com` (BIOS) ou `boot\x64\wdsmgfw.efi` (UEFI) | Nom du fichier de démarrage |

---

### Architecture WDS

```
┌─────────────┐   1. DHCP Request + PXE   ┌─────────────┐
│   Client    │ ─────────────────────────▶ │  Serveur    │
│  (PXE boot) │                           │    DHCP     │
│             │ ◀───── IP + options 66/67 ─┤             │
│             │                           └─────────────┘
│             │   2. Télécharge boot.wim   ┌─────────────┐
│             │ ─────────────────────────▶ │  Serveur    │
│             │ ◀───── WinPE démarre ────── │    WDS      │
│             │                           │             │
│             │   3. Choisit install.wim  │             │
│             │ ─────────────────────────▶ │             │
│             │ ◀───── OS installé ──────── │             │
└─────────────┘                           └─────────────┘
```

---

### Ports utilisés par WDS

| Port | Protocole | Service |
|---|---|---|
| **67 / 68** | UDP | DHCP |
| **69** | UDP | TFTP (transfert du fichier de boot) |
| **4011** | UDP | PXE (si DHCP et WDS sur le même serveur) |
| **135** | TCP | RPC (gestion WDS) |
| **5040** | UDP | WDS Transport Server (multidiffusion) |
| **139 / 445** | TCP | SMB (transfert des images WIM) |

---

## 🛠️ Sysprep — Préparer une image

Sysprep **généralise** une image Windows avant capture : supprime les identifiants uniques (SID, nom machine, licences) pour qu'elle soit déployable sur n'importe quelle machine.

```
PC Référence
    │
    ├── Installer Windows
    ├── Configurer (drivers, logiciels, paramètres)
    ├── Lancer Sysprep
    │     └── %WINDIR%\System32\Sysprep\sysprep.exe
    │         Options : /generalize /oobe /shutdown
    │
    └── Machine éteinte → prête pour capture
```

### Options Sysprep

| Option | Rôle |
|---|---|
| `/generalize` | Supprime les infos uniques (SID, nom machine…) |
| `/oobe` | Au prochain démarrage, lance l'assistant de configuration |
| `/shutdown` | Éteint la machine après Sysprep |
| `/reboot` | Redémarre après Sysprep |
| `/unattend:fichier.xml` | Utilise un fichier de réponses automatique |

---

## 📄 Fichier de réponses — `unattend.xml`

Fichier XML qui **automatise** toutes les étapes de l'installation (langue, partition, compte admin, clé de licence…).

```xml
<!-- Exemple simplifié -->
<unattend>
  <settings pass="windowsPE">
    <component name="Microsoft-Windows-International-Core-WinPE">
      <UILanguage>fr-FR</UILanguage>
    </component>
  </settings>
  <settings pass="oobeSystem">
    <component name="Microsoft-Windows-Shell-Setup">
      <ComputerName>PC-DEPLOY</ComputerName>
      <TimeZone>Romance Standard Time</TimeZone>
    </component>
  </settings>
</unattend>
```

**Créé avec :** Windows System Image Manager (WSIM), inclus dans le Windows ADK.

---

## 🔧 DISM — Commandes essentielles

```powershell
# Monter une image WIM
dism /Mount-Image /ImageFile:install.wim /Index:1 /MountDir:C:\mount

# Voir les index disponibles dans un WIM
dism /Get-ImageInfo /ImageFile:install.wim

# Ajouter un driver
dism /Image:C:\mount /Add-Driver /Driver:C:\drivers\pilote.inf

# Ajouter une mise à jour (.msu)
dism /Image:C:\mount /Add-Package /PackagePath:C:\updates\update.msu

# Démonter et valider les changements
dism /Unmount-Image /MountDir:C:\mount /Commit

# Capturer une image
dism /Capture-Image /ImageFile:D:\image.wim /CaptureDir:C:\ /Name:"Windows 11 Pro"

# Appliquer une image sur un disque
dism /Apply-Image /ImageFile:D:\image.wim /Index:1 /ApplyDir:C:\
```

---

## 🔄 Flux de déploiement complet

```
1. PRÉPARATION
   └── PC référence → Install Windows → Config → Sysprep /generalize /oobe /shutdown

2. CAPTURE
   └── Boot PXE (image de capture) → DISM capture → install.wim stocké sur WDS

3. DÉPLOIEMENT
   └── Client → Boot PXE → WinPE → Sélection image → unattend.xml → OS installé

4. POST-DÉPLOIEMENT (MDT / scripts)
   └── Jonction domaine → GPO → Logiciels → Mises à jour
```

---

## 🆚 WDS vs MDT vs SCCM

| Outil | Usage | Complexité | Coût |
|---|---|---|---|
| **WDS** | Déploiement simple d'images via PXE | ⭐⭐ | Gratuit (rôle Windows Server) |
| **MDT** | Déploiement automatisé, séquences de tâches | ⭐⭐⭐ | Gratuit |
| **SCCM / Intune** | Gestion complète (déploiement + mises à jour + inventaire) | ⭐⭐⭐⭐⭐ | Payant (licence Microsoft) |

> 💡 **MDT + WDS** = combinaison gratuite très utilisée en entreprise pour automatiser entièrement le déploiement.

---

## 📝 À retenir absolument

1. **Boot image** (`boot.wim`) = environnement WinPE pour démarrer | **Install image** (`install.wim`) = l'OS à déployer
2. **PXE** permet de booter sur le réseau sans média — nécessite **DHCP options 66 & 67**
3. **Sysprep `/generalize`** est obligatoire avant de capturer une image (supprime le SID unique)
4. **DISM** est l'outil CLI pour monter, modifier et capturer des images WIM
5. Le fichier **`unattend.xml`** automatise toute l'installation (langue, compte, partition…)
6. Ports clés : **TFTP = 69 (UDP)**, **DHCP = 67/68 (UDP)**, **SMB = 445 (TCP)**
7. **WDS seul** = simple | **WDS + MDT** = déploiement automatisé complet
