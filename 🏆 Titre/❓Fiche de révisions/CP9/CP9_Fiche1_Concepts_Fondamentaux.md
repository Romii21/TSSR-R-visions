# CP9 — Fiche de Révision 1 : Concepts Fondamentaux

> **Niveau** : TSSR | **Compétence** : CP9 — Déploiement automatisé des postes de travail

---

## 1. Pourquoi automatiser le déploiement ?

Installer Windows manuellement poste par poste est **long, coûteux et source d'erreurs**. Le déploiement automatisé permet de :

- Préparer une **image de référence** une seule fois
- La déployer sur **des dizaines ou centaines de postes** via le réseau
- Inclure l'OS, les logiciels, les personnalisations et les pilotes **en une seule opération**

### Pourquoi ne pas simplement copier un poste existant ?

Dupliquer un poste sans précaution crée des **conflits d'identifiants** :

| Problème | Explication |
|---|---|
| **SID dupliqué** | Le Security Identifier (SID) Windows est unique à chaque machine. Deux postes avec le même SID créent des conflits d'authentification dans un domaine |
| **Nom de machine identique** | Deux machines avec le même nom sur un réseau créent des conflits réseau |
| **Paramètres matériels figés** | Une copie secteur (type Ghost) est liée au matériel d'origine |

> ⚠️ **Solution obligatoire** : utiliser **Sysprep** pour généraliser l'image avant toute duplication.

---

## 2. Prérequis réseau pour le déploiement

Trois éléments réseau sont nécessaires pour un déploiement automatisé via le réseau :

| Élément                      | Obligatoire ? | Rôle                                                     |
| ---------------------------- | :-----------: | -------------------------------------------------------- |
| **Serveur DHCP**             |     ✅ Oui     | Attribue une adresse IP au poste client au démarrage     |
| **Serveur TFTP**             |  ✅ Pour PXE   | Transfère le fichier d'amorçage réseau (bootloader)      |
| **AD-DS (Active Directory)** |     ❌ Non     | Optionnel — nécessaire uniquement si jonction au domaine |

---

## 3. PXE — Preboot Execution Environment

### Définition

**PXE** permet à un poste de **démarrer depuis le réseau** sans support physique (USB/CD) pour récupérer et installer un OS.

> 💡 PXE n'est pas exclusif à Windows : le projet **Syslinux** utilise PXE pour démarrer des OS Linux.

### Les 3 étapes du boot PXE

```
Étape 1 — DHCP/BOOTP
  Le client diffuse une requête pour obtenir une IP
  + l'emplacement du fichier d'amorçage réseau
        ↓
Étape 2 — TFTP
  Le client télécharge le fichier d'amorçage
  depuis le serveur TFTP
        ↓
Étape 3 — Exécution
  Le fichier d'amorçage est exécuté
  (ex : WinPE, PXELinux)
```

### Détail des échanges réseau

| Étape | Protocole / Port | Description |
|---|---|---|
| **DHCP-DISCOVER** | UDP port 67 | Le client demande une IP + options PXE |
| **DHCP-OFFER** | UDP port 68 | Le serveur répond avec IP + options 60, 66, 67 |
| **DHCP-REQUEST** | UDP | Le client confirme l'adresse IP proposée |
| **DHCP-ACK** | UDP | Le serveur valide → début du bail IP |
| **NBP** | UDP | Demande de téléchargement du fichier d'amorçage |
| **Transfert fichier** | TFTP | Envoi du bootloader au client |
| **Requête installateur** | HTTP | Téléchargement de l'installateur Windows |

### Options DHCP spécifiques au PXE

| Option DHCP | Rôle |
|---|---|
| **Option 60** | Identifie le client comme client PXE (valeur : `PXEClient`) |
| **Option 66** | Adresse IP du serveur TFTP (serveur de boot) |
| **Option 67** | Nom du fichier d'amorçage à télécharger (ex : `boot\x64\wdsnbp.com`) |

---

## 4. WinPE — Windows Preinstallation Environment

### Définition

**WinPE** est un **mini-OS Windows** minimaliste utilisé pour démarrer un poste sans OS installé, afin de préparer le disque et lancer l'installation de Windows.

- Existe depuis **Windows XP**
- Remplace l'ancien **MS-DOS** de démarrage avec beaucoup plus de fonctionnalités
- Compatible avec **MDT** et **WDS**

### Modes de démarrage de WinPE

| Support | Description |
|---|---|
| **Clé USB** | Démarrage local, pratique pour interventions ponctuelles |
| **CD/DVD** | Démarrage local depuis un support optique |
| **Réseau (PXE)** | Démarrage centralisé depuis un serveur WDS — idéal en production |

### Commandes diskpart dans WinPE

```cmd
diskpart                    → Lancer l'utilitaire de gestion des disques
list disk                   → Lister les disques physiques disponibles
select disk 0               → Sélectionner le disque 0
clean                       → Supprimer toutes les partitions du disque sélectionné
create partition primary     → Créer une partition principale
active                      → Marquer la partition comme active (bootable)
format fs=ntfs quick        → Formater en NTFS (formatage rapide)
```

> ⚠️ L'ordre des commandes diskpart est impératif :
> `list disk` → `select disk` → `clean` → `create partition primary` → `active` → `format`

---

## 5. WIM — Windows Imaging Format

### Définition

**WIM** est le format de fichier image disque utilisé par Microsoft pour le déploiement Windows.

- Extension : `.wim` (ou `.swm` pour les images fractionnées)
- Existe depuis **Windows XP SP2**
- Compatible avec **WADK**, **MDT** et **WinPE**

### Caractéristiques clés

| Caractéristique | Explication |
|---|---|
| **Indépendant du matériel** | Fonctionne sur n'importe quelle configuration matérielle, contrairement aux images secteurs (Ghost) qui sont liées au disque physique |
| **Multi-images** | Un seul fichier WIM peut contenir plusieurs images (ex : Win 10 Pro, Éducation, Entreprise) |
| **Compression** | Les fichiers sont stockés une seule fois même s'ils apparaissent dans plusieurs images (déduplication) |
| **Modifiable** | On peut monter, modifier et remonter l'image sans recapturer depuis zéro |

### Comparaison WIM vs Image secteur (Ghost)

| Critère | WIM | Ghost / Image secteur |
|---|---|---|
| Dépendance matérielle | ❌ Indépendant | ✅ Lié au matériel source |
| Multi-images en 1 fichier | ✅ Oui | ❌ Non |
| Modifiable après capture | ✅ Oui (DISM) | ❌ Non sans recapturer |
| Copie des fichiers | À la prise de vue | Secteur par secteur |

### Fichiers WIM dans une ISO Windows

| Fichier | Emplacement | Contenu |
|---|---|---|
| `install.wim` | `Sources\` | L'image d'installation de Windows |
| `boot.wim` | `Sources\` | L'environnement WinPE de démarrage |

### Structure interne d'un fichier WIM

```
[ En-tête WIM ]
  ├── Ressources de fichiers (blocs compressés)
  ├── Métadonnées des ressources (descripteurs de dossiers, sécurité…)
  ├── Table d'adresses
  ├── Données XML (caractéristiques et infos sur l'image)
  └── Table d'intégrité
```

---

## 6. Glossaire des acronymes fondamentaux

| Acronyme | Signification | En bref |
|---|---|---|
| **PXE** | Preboot Execution Environment | Démarrage OS depuis le réseau |
| **WIM** | Windows Imaging Format | Format d'image disque Microsoft |
| **WinPE** | Windows Preinstallation Environment | Mini-OS pour déploiement |
| **TFTP** | Trivial File Transfer Protocol | Transfert du bootloader réseau |
| **DHCP** | Dynamic Host Configuration Protocol | Attribution d'adresses IP |
| **SID** | Security Identifier | Identifiant unique de machine Windows |
| **OOBE** | Out-Of-Box Experience | Séquence de premier démarrage Windows |
| **NBP** | Network Bootstrap Program | Fichier d'amorçage téléchargé via TFTP |

---

*Fiche 1/3 — CP9 Déploiement automatisé | Niveau TSSR*
