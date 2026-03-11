# CP9 — Fiche de Révision 3 : Processus de Déploiement

> **Niveau** : TSSR | **Compétence** : CP9 — Déploiement automatisé des postes de travail

---

## 1. Le processus de déploiement de bout en bout

```
┌─────────────────────────────────────────────────────────────────────┐
│                    PHASE 1 — PRÉPARATION                            │
│                                                                     │
│  Poste de référence → Installation Windows + logiciels              │
│       ↓                                                             │
│  Exécution de Sysprep (/oobe /generalize /shutdown)                 │
│       ↓                                                             │
│  Capture de l'image avec DISM → fichier .WIM                        │
└─────────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────────┐
│                    PHASE 2 — DÉPLOIEMENT                            │
│                                                                     │
│  Poste client sans OS → Boot PXE → WinPE se charge                  │
│       ↓                                                             │
│  diskpart → préparation du disque                                   │
│       ↓                                                             │
│  DISM applique l'image WIM sur le disque                            │
│       ↓                                                             │
│  Premier démarrage → Séquence OOBE ou fichier de réponse            │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 2. Les types de Master (image de référence)

Le **master** est l'image disque de référence capturée sur un poste modèle, puis déployée sur les autres machines.

### Les 3 types de master

| Type | Contenu | Fréquence de mise à jour | Usage idéal |
|---|---|:---:|---|
| **Thin image** (image légère) | OS uniquement | Tous les 1–2 ans | Les logiciels sont déployés après via GPO, MDT ou scripts |
| **Thick image** (image complète) | OS + **tous** les logiciels | Maximum tous les 6 mois | Environnement stable, mêmes logiciels sur tous les postes |
| **Hybrid image** (image hybride) | OS + logiciels de base (antivirus, navigateur, bureautique) | Intermédiaire | Bon équilibre entre légèreté et praticité |

### Comment choisir son type de master ?

```
Logiciels identiques sur tous les postes ET environnement stable ?
  → Thick image

Logiciels très variables selon les utilisateurs ?
  → Thin image (déploiement logiciels post-installation)

Situation intermédiaire (quelques logiciels communs + personnalisation) ?
  → Hybrid image (recommandé dans la plupart des cas)
```

---

## 3. Sysprep — Généraliser l'image

### Définition

**Sysprep** (System Preparation Tool) est l'outil qui **prépare un poste Windows** pour être capturé et déployé sur d'autres machines, en supprimant tous les identifiants uniques.

### Ce que fait Sysprep

| Action | Description |
|---|---|
| Supprime le **SID** unique | Chaque nouveau déploiement génère un SID différent |
| Supprime le **nom de machine** | Le poste sera renommé lors du premier démarrage |
| Supprime les paramètres spécifiques au matériel | L'image devient indépendante du hardware |
| Réinitialise l'activation Windows | Le compteur d'activations est remis à zéro |
| Prépare la séquence **OOBE** | Premier démarrage comme un PC neuf |

### La commande Sysprep

```cmd
C:\Windows\System32\sysprep\sysprep.exe /oobe /generalize /shutdown
```

### Détail des options

| Option | Signification | Effet |
|---|---|---|
| `/oobe` | Out-Of-Box Experience | Au premier démarrage, lance la séquence de personnalisation (langue, compte utilisateur…) |
| `/generalize` | Généralisation | Supprime toutes les informations spécifiques à la machine (SID, pilotes, activation…) |
| `/shutdown` | Extinction | Éteint le PC après le Sysprep — prêt pour capture |

### Règle absolue après Sysprep

> ⚠️ **IMPORTANT** : Après un Sysprep, **ne JAMAIS rallumer le poste** avant d'avoir capturé l'image WIM.
>
> Si on redémarre avant la capture, le Sysprep est annulé : Windows régénère un SID et démarre la séquence OOBE. Il faudra recommencer entièrement.

### Comment éviter la séquence OOBE interactive ?

Pour que le premier démarrage soit entièrement automatique (sans interaction utilisateur), on peut fournir un **fichier de réponse XML** (`unattend.xml`) qui pré-renseigne toutes les informations (langue, mot de passe, nom de machine, fuseau horaire, etc.).

Ce fichier peut être créé avec l'outil **SIM** (System Image Manager) inclus dans le WADK.

---

## 4. DISM — Gestion des images WIM

### Définition

**DISM** (Deployment Image Servicing and Management) est l'outil de référence pour **manipuler les images WIM**. Il remplace l'ancien outil **ImageX**.

### Actions disponibles

| Action | Description |
|---|---|
| **Mount / Unmount** | Monter/démonter l'image pour l'éditer sans recapturer |
| **Capture** | Copier une structure de fichiers vers un nouveau WIM |
| **Append** | Ajouter une image dans un WIM existant |
| **Export** | Extraire une image d'un WIM vers un nouveau WIM |
| **Delete** | Supprimer une image dans un WIM |
| **Get-WimInfo** | Afficher les métadonnées XML de l'image |
| **Add-Driver** | Intégrer un pilote dans une image montée |
| **Add-Package** | Intégrer une mise à jour (.cab / .msu) dans l'image |

### Commandes DISM essentielles

**Afficher les informations d'un WIM :**
```cmd
DISM /Get-WimInfo /WimFile:Sources\install.wim
```

**Monter une image WIM pour la modifier :**
```cmd
DISM /Mount-Wim /WimFile:Sources\install.wim /index:1 /MountDir:C:\Mount
```

**Intégrer un pilote dans l'image montée :**
```cmd
DISM /Image:C:\Mount /Add-Driver /Driver:C:\Drivers\mon_pilote.inf
```

**Intégrer tous les pilotes d'un dossier (récursif) :**
```cmd
DISM /Image:C:\Mount /Add-Driver /Driver:C:\Drivers /Recurse
```

**Enregistrer les modifications et démonter :**
```cmd
DISM /Unmount-Wim /MountDir:C:\Mount /commit
```

**Démonter sans enregistrer les modifications :**
```cmd
DISM /Unmount-Wim /MountDir:C:\Mount /discard
```

### Workflow complet : intégration d'un pilote dans une image WIM

```
1. DISM /Get-WimInfo → identifier le numéro d'index de l'image
       ↓
2. DISM /Mount-Wim → monter l'image dans un dossier
       ↓
3. DISM /Add-Driver → injecter le pilote
       ↓
4. DISM /Unmount-Wim /commit → valider et démonter
```

---

## 5. Diskpart — Préparation du disque

### Définition

**Diskpart** est l'utilitaire Windows (exécutable dans WinPE) pour **partitionner et formater un disque** avant l'application de l'image.

### Ordre logique des commandes pour préparer un disque vierge

```cmd
1. list disk               → Lister les disques physiques disponibles
2. select disk 0           → Sélectionner le disque cible (disk 0)
3. clean                   → Supprimer toutes les partitions existantes
4. create partition primary → Créer une partition principale
5. active                  → Marquer la partition comme active (bootable)
6. format fs=ntfs quick    → Formater en NTFS (formatage rapide)
```

### Rôle de chaque commande

| Commande | Rôle |
|---|---|
| `list disk` | Affiche tous les disques physiques détectés avec leur taille |
| `select disk 0` | Cible le disque 0 pour les opérations suivantes |
| `clean` | Efface entièrement la table de partitions (MBR/GPT) — **irréversible** |
| `create partition primary` | Crée une partition principale sur tout l'espace disponible |
| `active` | Indique au BIOS que cette partition est la partition de démarrage |
| `format fs=ntfs quick` | Formate en NTFS avec formatage rapide (ne recrée pas les secteurs) |

> ⚠️ `clean` est **irréversible**. Toutes les données du disque sont perdues.
> ⚠️ Sans `active`, le poste ne pourra pas démarrer depuis cette partition.

---

## 6. Processus complet WDS + MDT

### Étapes pour déployer Windows 10 sur un parc avec WDS + MDT

```
PRÉPARATION SERVEUR
─────────────────────────────────────────────────────
1. Installer le rôle WDS sur Windows Server
2. Installer WADK sur le serveur
3. Installer MDT sur le serveur
4. Configurer MDT : créer un "Deployment Share"
5. Importer l'image Windows (install.wim) dans MDT

PRÉPARATION DU MASTER
─────────────────────────────────────────────────────
6. Installer Windows sur un poste de référence
7. Installer les logiciels communs (thick/hybrid)
8. Exécuter Sysprep (/oobe /generalize /shutdown)
9. Capturer l'image avec DISM → fichier .wim
10. Importer l'image capturée dans MDT

CONFIGURATION MDT
─────────────────────────────────────────────────────
11. Créer une séquence de tâches (Task Sequence)
12. Configurer : nommage machines, jonction domaine, apps
13. Générer le fichier d'amorçage MDT (LiteTouch Boot ISO)
14. Importer le fichier boot.wim dans WDS

DÉPLOIEMENT CLIENT
─────────────────────────────────────────────────────
15. Démarrer le poste client en PXE (F12 au boot)
16. WinPE se charge depuis WDS
17. L'assistant MDT apparaît (mode LiteTouch)
18. diskpart → préparation du disque
19. DISM applique l'image WIM
20. Redémarrage → Windows s'installe et se configure
```

---

## 7. Récapitulatif des points critiques

| Point clé | Règle à retenir |
|---|---|
| **Sysprep avant capture** | Toujours obligatoire avant toute duplication de poste |
| **Ne pas rallumer après Sysprep** | Sinon le Sysprep est annulé, tout est à refaire |
| **MDT nécessite WADK** | WADK doit être installé avant MDT |
| **MDT sans TFTP** | MDT ne fait pas de boot PXE → combiner avec WDS |
| **ZeroTouch = SCCM obligatoire** | LiteTouch = MDT seul (interaction minimale) |
| **WIM multi-images** | Un .wim peut contenir Pro, Éducation, Entreprise |
| **DISM /commit** | Sans /commit, les modifications sont perdues |
| **diskpart clean** | Irréversible — efface toutes les données |

---

## 8. Glossaire des termes processus

| Terme | Définition |
|---|---|
| **Master** | Image de référence capturée sur un poste modèle |
| **Thin image** | Master contenant uniquement l'OS |
| **Thick image** | Master contenant l'OS + tous les logiciels |
| **Hybrid image** | Master contenant l'OS + logiciels de base |
| **Sysprep** | Outil de généralisation d'une image Windows |
| **OOBE** | Séquence de premier démarrage (langue, compte…) |
| **Unattend.xml** | Fichier de réponse pour automatiser l'OOBE |
| **Task Sequence** | Séquence de tâches MDT définissant le déroulement du déploiement |
| **LiteTouch** | Mode MDT avec intervention humaine minimale |
| **ZeroTouch** | Mode MDT+SCCM totalement automatique |
| **ScanState** | Phase USMT de capture des profils utilisateurs |
| **LoadState** | Phase USMT de restauration des profils |
| **Deployment Share** | Dossier partagé MDT contenant images, pilotes et séquences |

---

*Fiche 3/3 — CP9 Déploiement automatisé | Niveau TSSR*
