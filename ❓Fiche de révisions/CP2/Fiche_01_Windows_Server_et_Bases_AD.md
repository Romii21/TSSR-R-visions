# Fiche 01 — Windows Server & Bases d'Active Directory

> **Formation TSSR — CP2**
> Révision : Windows Server, introduction AD, historique, composants fondamentaux

---

## 1. Windows Server — Les essentiels

### Modes d'installation

| Critère | Mode GUI | Mode Core |
|---|---|---|
| Interface | Bureau graphique complet | Ligne de commande uniquement |
| Ressources consommées | Élevées | Réduites |
| Surface d'attaque | Large | Réduite |
| Gestion | Locale + distante | Principalement distante (PowerShell, RSAT) |
| **Usage recommandé** | Tests, formation | **Production** ✅ |

> ⚠️ **À retenir** : En production, le mode Core est préféré car il consomme moins de ressources et expose moins de vulnérabilités. Moins de composants = moins de redémarrages pour les mises à jour.

---

### Server Manager

Console d'administration centrale de Windows Server. Permet de :

- Ajouter / Supprimer des **rôles et fonctionnalités** (AD DS, DNS, DHCP…)
- **Surveiller** l'état du serveur (événements, alertes, performances)
- Gérer **plusieurs serveurs à distance** depuis une seule console
- Accéder aux outils d'administration (ADUC, DNS Manager…)

---

### Rôles vs Fonctionnalités

| Concept | Définition | Exemples |
|---|---|---|
| **Rôle** | Service principal fourni au réseau | AD DS, DHCP, DNS, IIS, Hyper-V |
| **Fonctionnalité** | Composant complémentaire | .NET Framework, RSAT, Sauvegarde Windows |

**Résumé :** Un rôle = ce que le serveur "fait". Une fonctionnalité = un outil qui l'aide à le faire.

**Installer le rôle AD DS en PowerShell :**
```powershell
Install-WindowsFeature -Name AD-Domain-Services -IncludeManagementTools
```

---

### Workgroup vs Domaine Active Directory

| Caractéristique | Workgroup | Domaine AD |
|---|---|---|
| Connexion | Compte local sur chaque machine | Compte domaine utilisable partout |
| Centralisation | Aucune | Complète via les DC |
| Limite machines | ~35–40 machines | Illimitée |
| GPO | ❌ Impossible | ✅ Oui |
| Gestion des comptes | Locale (base SAM par machine) | Centralisée (AD) |

> ⚠️ Au-delà de ~35 machines, le Workgroup devient ingérable.

---

## 2. Active Directory — Introduction

### Définition

**Active Directory** est l'implémentation Microsoft des services d'annuaire basés sur le protocole **LDAP**. C'est le cœur de l'infrastructure Windows en entreprise.

**Ses trois objectifs principaux :**
1. **Centraliser l'identification et l'authentification** des utilisateurs
2. **Gérer les politiques de sécurité** (GPO) sur l'ensemble du réseau
3. **Répertorier et faciliter la gestion** des ressources réseau

**Chiffres clés :**
- Plus de **2 milliards d'objets** créables dans une forêt
- Maximum de **999 GPO** applicables à un objet
- ~25% du marché officiel (potentiellement 75% en réalité)

---

### Historique rapide

| Année | Événement |
|---|---|
| 1996 | Création sous le nom **NTDS (NT Directory Services)** |
| 1999 | Première version majeure avec Windows 2000 Server |
| Évolution | Passage de la base **SAM** à **LDAP** |
| Évolution | Nouveau moteur de stockage : **ESENT** |

---

### SAM vs Active Directory

**SAM (Security Account Manager)** = base de comptes utilisée AVANT AD, stockée localement sur chaque machine.

| Critère | SAM | Active Directory |
|---|---|---|
| Stockage | Local sur chaque machine | Centralisé sur les DC |
| Portée | Machine isolée | Tout le domaine |
| Limite | ~20 machines max | Illimitée |
| Centralisation | ❌ Aucune | ✅ Complète |

**Pourquoi SAM a été remplacé :** impossible de gérer efficacement des centaines de comptes répartis sur des dizaines de machines sans centralisation.

---

### Base de données : Hiérarchique vs Relationnelle

| Base Hiérarchique (LDAP / AD) | Base Relationnelle (MySQL, SQL Server) |
|---|---|
| Structure en **arborescence** | Tables avec relations |
| Recherche par **chemin hiérarchique** | Recherche par clés et jointures |
| Optimisée pour la **lecture** | Optimisée pour lectures **et** écritures |
| Exemple : AD, OpenLDAP | Exemple : MySQL, Oracle |

---

## 3. Les cinq rôles Active Directory

| Rôle | Nom complet | Description |
|---|---|---|
| **AD DS** | Active Directory Domain Services | Rôle principal : domaine, authentification, GPO, annuaire |
| **AD CS** | Active Directory Certificate Services | Gestion des certificats numériques et PKI |
| **AD FS** | Active Directory Federation Services | Portail SSO (Single Sign-On) inter-domaines |
| **AD RMS** | Active Directory Rights Management Services | Droits d'accès sur les fichiers (lire, modifier, imprimer) |
| **AD LDS** | Active Directory Lightweight Directory Services | Annuaire LDAP **sans** domaine (pour applis tierces) |

> ⚠️ **AD DS** est le rôle fondamental. Sans lui, pas de domaine. Les autres sont optionnels et viennent enrichir l'infrastructure.

---

## 4. Le Contrôleur de Domaine (DC)

Un **DC (Domain Controller)** est un serveur Windows sur lequel est installé **AD DS**. C'est le pivot de l'infrastructure.

**Ses rôles concrets :**
- Stocker et gérer la **base de données AD** (fichier `NTDS.dit`)
- Gérer l'**authentification** via Kerberos
- Appliquer les **GPO** aux machines et utilisateurs
- Assurer la **réplication** avec les autres DC
- Fournir les services **DNS** et **LDAP** du domaine

> ⚠️ **Bonne pratique absolue :** Minimum **2 DC** par domaine pour la redondance. Si le seul DC tombe, plus aucune authentification n'est possible sur le réseau.

---

## Points clés à retenir

- Mode Core recommandé en production (moins de ressources, moins de vulnérabilités)
- AD repose sur **LDAP** (protocole d'annuaire hiérarchique)
- SAM = avant AD, limité à ~20 machines, sans centralisation
- AD DS = rôle fondamental, les 4 autres sont complémentaires
- Minimum 2 DC en production — toujours
