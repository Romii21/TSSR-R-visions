# CP9 — Fiche de Révision 2 : Les Outils de Déploiement

> **Niveau** : TSSR | **Compétence** : CP9 — Déploiement automatisé des postes de travail

---

## 1. Vue d'ensemble des outils

Le déploiement automatisé Windows repose sur une suite d'outils complémentaires :

```
┌─────────────────────────────────────────────────────┐
│                   SCCM (payant)                     │
│         Solution complète pour grands parcs         │
│                  Intègre WDS + MDT                  │
└──────────────────┬──────────────────────────────────┘
                   │
        ┌──────────┴──────────┐
        ▼                     ▼
┌──────────────┐    ┌──────────────────┐
│     WDS      │    │       MDT        │
│ (Boot PXE)   │◄───┤ (Séquences de   │
│              │    │   tâches)        │
└──────────────┘    └────────┬─────────┘
                             │ requiert
                             ▼
                    ┌──────────────────┐
                    │      WADK        │
                    │ (Suite d'outils) │
                    │ DISM, USMT,      │
                    │ WinPE, Sysprep…  │
                    └──────────────────┘
```

---

## 2. WDS — Windows Deployment Services

### Définition

**WDS** est un **rôle serveur Microsoft** permettant d'installer Windows via le réseau. C'est le successeur de **RIS** (Remote Installation Service).

- Disponible en tant que rôle sur **Windows Server 2008 et suivants**
- Supporte Windows Vista → 10 et Windows Server 2008 → 2016
- Nécessite une **partition dédiée** sur le serveur

### Ce que fait WDS

| Fonctionnalité | Description |
|---|---|
| **Déploiement d'images WIM** | Envoie l'image Windows aux postes clients |
| **Boot PXE via TFTP** | Fournit une image de démarrage par le réseau |
| **Fichier de réponse XML** | Automatise les réponses à l'assistant d'installation |

> 💡 WDS seul est suffisant pour un déploiement **simple**, mais il est **moins personnalisable** que MDT pour les configurations complexes.

### Limitations de WDS

- Pas de séquences de tâches avancées
- Pas de nommage automatique des machines
- Pas de jonction au domaine automatisée
- Pas de migration de profils utilisateurs

---

## 3. MDT — Microsoft Deployment Toolkit

### Définition

**MDT** est une solution **gratuite** de Microsoft qui automatise la création et l'installation d'images Windows à l'aide de **séquences de tâches**.

> ⚠️ **MDT doit être utilisé avec WADK** pour fonctionner.
> ⚠️ **MDT n'inclut pas de service TFTP** → pas de boot PXE natif.
> Solution : combiner **MDT + WDS** pour obtenir boot PXE ET séquences de tâches.

### Les deux modes de fonctionnement de MDT

| Mode | Description | Intervention humaine | Prérequis |
|---|---|:---:|---|
| **LiteTouch** | MDT seul | ✅ Minimale (choix au démarrage) | MDT + WADK |
| **ZeroTouch** | MDT + SCCM | ❌ Aucune (100% automatique) | MDT + WADK + SCCM |

### Fonctionnalités avancées de MDT

| Fonctionnalité | Détail |
|---|---|
| **Nommage automatique** | Nom de machine généré automatiquement (ex : depuis le numéro de série) |
| **Jonction au domaine** | Intégration automatique à l'Active Directory |
| **Installation d'applications** | Installation silencieuse via séquences de tâches |
| **Exécution de scripts** | VBScript, CMD, **PowerShell** |
| **Migration des profils** | Sauvegarde et restauration des données utilisateur |
| **Activation BitLocker** | Chiffrement du disque pendant le déploiement |
| **Intégration de pilotes** | Drivers intégrés à l'image ou injectés dynamiquement |

---

## 4. WADK — Windows Assessment and Deployment Kit

### Définition

**WADK** (anciennement **WAIK** — Windows Automated Installation Kit) est une **suite d'outils Microsoft** pour le déploiement d'OS. Il est **indispensable au fonctionnement de MDT**.

### Les outils inclus dans WADK

| Outil | Nom complet | Rôle |
|---|---|---|
| **USMT** | User State Migration Tool | Migration des données et profils utilisateurs lors d'un remplacement de poste |
| **DISM** | Deployment Image Servicing and Management | Gérer, modifier et capturer les images WIM (pilotes, MAJ, montage…) |
| **WinPE** | Windows Preinstallation Environment | Mini-OS pour le démarrage et le déploiement |
| **Sysprep** | System Preparation Tool | Généraliser une image Windows avant capture |
| **ACT** | Application Compatibility Toolkit | Tester la compatibilité des applications après une migration OS |
| **VAMT** | Volume Activation Management Tool | Gérer l'activation de licences (clé MAK ou serveur KMS) |

### Zoom sur USMT

USMT est utilisé dans deux phases distinctes :

```
Phase 1 — AVANT remplacement du poste (ScanState)
  Capture les profils utilisateurs, fichiers, paramètres
        ↓
Phase 2 — APRÈS déploiement du nouvel OS (LoadState)
  Restaure les données capturées sur le nouveau poste
```

---

## 5. SCCM — System Center Configuration Manager

### Définition

**SCCM** est un logiciel Microsoft de **gestion de parc informatique à grande échelle**. C'est une solution **payante** (licence propriétaire).

> 💡 SCCM est dimensionné pour les **très grands parcs**. Il intègre WDS et MDT.

### Fonctionnalités SCCM

| Fonctionnalité | Description |
|---|---|
| **Déploiement OS** | Déploiement complet automatisé en mode ZeroTouch |
| **Prise de main à distance** | Administration des postes clients |
| **Gestion des correctifs** | Déploiement des mises à jour Windows et applicatives |
| **Télédistribution** | Installation à distance de logiciels |
| **Inventaire** | Inventaire matériel et logiciel du parc |
| **Conformité** | Vérification des configurations contre une baseline |
| **Sécurité** | Administration des politiques de sécurité |

### Architecture SCCM

```
Site Central d'Administration (CAS)
  └── Gère jusqu'à 25 sites primaires
      Site Primaire → jusqu'à 100 000 clients
        └── Sites Secondaires → jusqu'à 5 000 clients
              └── Points de distribution (DP) → jusqu'à 4 000 clients
                    └── Ordinateurs clients
```

### Quand utiliser SCCM ?

| Contexte | Outil recommandé |
|---|---|
| Parc < 50 postes | WDS + MDT (gratuit, suffisant) |
| Parc 50–500 postes | MDT + WDS (gratuit, personnalisable) |
| Parc > 500 postes | SCCM (payant, mais gestion centralisée complète) |

---

## 6. Tableau comparatif des outils

| Critère | WDS | MDT | SCCM |
|---|:---:|:---:|:---:|
| **Gratuit** | ✅ | ✅ | ❌ |
| **Boot PXE natif** | ✅ | ❌ | ✅ |
| **Personnalisation avancée** | ➖ | ✅ | ✅ |
| **Adapté aux grands parcs** | ➖ | ➖ | ✅ |
| **Nécessite WADK** | ❌ | ✅ | ✅ |
| **Mode ZeroTouch (0 intervention)** | ❌ | ❌ | ✅ |
| **Séquences de tâches** | ❌ | ✅ | ✅ |
| **Migration profils (USMT)** | ❌ | ✅ | ✅ |

---

## 7. Associations d'outils en pratique

### Scénario 1 — Déploiement simple (images identiques, < 50 postes)
```
WDS seul
→ Boot PXE + image WIM → OK
```

### Scénario 2 — Déploiement personnalisé (nommage, domaine, logiciels)
```
WDS + MDT + WADK
→ Boot PXE (WDS) + Séquences de tâches (MDT) → OK
```

### Scénario 3 — Déploiement ZeroTouch (grand parc, 0 intervention)
```
WDS + MDT + WADK + SCCM
→ Déploiement 100% automatique → OK
```

---

## 8. Glossaire des acronymes outils

| Acronyme | Signification |
|---|---|
| **WDS** | Windows Deployment Services |
| **MDT** | Microsoft Deployment Toolkit |
| **WADK** | Windows Assessment and Deployment Kit |
| **WAIK** | Windows Automated Installation Kit (ancien nom de WADK) |
| **SCCM** | System Center Configuration Manager |
| **USMT** | User State Migration Tool |
| **DISM** | Deployment Image Servicing and Management |
| **ACT** | Application Compatibility Toolkit |
| **VAMT** | Volume Activation Management Tool |
| **RIS** | Remote Installation Service (prédécesseur de WDS) |
| **CAS** | Central Administration Site (SCCM) |
| **DP** | Distribution Point (SCCM) |

---

*Fiche 2/3 — CP9 Déploiement automatisé | Niveau TSSR*
