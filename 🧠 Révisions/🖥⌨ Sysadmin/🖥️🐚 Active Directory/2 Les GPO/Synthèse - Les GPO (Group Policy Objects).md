## 📑 Sommaire

1. [Définition et concepts de base](https://claude.ai/chat/16e8577c-b74d-4e54-b17f-feacd447e1af#1-d%C3%A9finition-et-concepts-de-base)
2. [Architecture d'une GPO](https://claude.ai/chat/16e8577c-b74d-4e54-b17f-feacd447e1af#2-architecture-dune-gpo)
3. [États et configurations](https://claude.ai/chat/16e8577c-b74d-4e54-b17f-feacd447e1af#3-%C3%A9tats-et-configurations)
4. [Règles de priorité (LSDOU)](https://claude.ai/chat/16e8577c-b74d-4e54-b17f-feacd447e1af#4-r%C3%A8gles-de-priorit%C3%A9-lsdou)
5. [Filtrage de sécurité](https://claude.ai/chat/16e8577c-b74d-4e54-b17f-feacd447e1af#5-filtrage-de-s%C3%A9curit%C3%A9)
6. [Bonnes pratiques](https://claude.ai/chat/16e8577c-b74d-4e54-b17f-feacd447e1af#6-bonnes-pratiques)
7. [Points clés à retenir](https://claude.ai/chat/16e8577c-b74d-4e54-b17f-feacd447e1af#7-points-cl%C3%A9s-%C3%A0-retenir)

---

## 1. Définition et concepts de base


## 2. Architecture d'une GPO


## 3. États et configurations


## 4. Règles de priorité (LSDOU)


## 5. Filtrage de sécurité

### 5.1 Principe de base

Une GPO ne s'applique que si **DEUX conditions** sont remplies :

1. ✅ Elle est liée à un site/domaine/OU
2. ✅ L'objet AD possède les droits :
    - **Read** (Lecture)
    - **Apply Group Policy** (Appliquer la stratégie de groupe)

### 5.2 Groupe Authenticated Users

|Configuration par défaut|Effet|
|---|---|
|**Authenticated Users** a le droit "Apply Group Policy"|Tous les utilisateurs et ordinateurs authentifiés reçoivent la GPO|

### 5.3 Restreindre une GPO à un groupe spécifique

**Procédure** :

1. ❌ Retirer le droit "Apply Group Policy" du groupe **Authenticated Users**
2. ➕ Ajouter un groupe de sécurité AD (Utilisateur ou Ordinateur)
3. ✅ Lui donner les droits **Lecture + Apply Group Policy**

**Résultat** : Seuls les membres du groupe spécifique reçoivent la GPO.

### 5.4 Filtrage WMI

Le filtrage WMI (Windows Management Instrumentation) permet d'affiner l'application des GPO selon des critères techniques (version OS, type de matériel, etc.).

---

## 6. Bonnes pratiques

### 6.1 Règles à respecter

|✅ À FAIRE|❌ À ÉVITER|
|---|---|
|Se baser sur une hiérarchie d'OU claire|Modifier les GPO par défaut (Default Domain Policy)|
|Utiliser une nomenclature descriptive|Utiliser les dossiers de base "Users" et "Computers"|
|Une action = Une GPO (principe de séparation)|Bloquer l'héritage des GPO|
|Supprimer un lien plutôt que désactiver|Modifier une GPO déjà active en production|
|Utiliser de petites GPO spécialisées|Créer des GPO monolithiques complexes|
|Désactiver les configurations inutilisées (ordinateurs OU utilisateurs)|Laisser des configurations vides activées|

### 6.2 Recommandations techniques

- 🔐 **Sécurité** : Utiliser la gestion avancée des mots de passe
- 📋 **Organisation** : Créer une OU dédiée pour chaque type d'objet
- 🧪 **Tests** : Toujours tester les GPO dans un environnement de non-production
- 📝 **Documentation** : Documenter chaque GPO et son objectif
- 🔄 **MCO** : Maintenir les GPO à jour et vérifier leur pertinence régulièrement

### 6.3 Commandes CLI utiles

#### Actualiser les GPO sur un poste client

```cmd
gpupdate /force
```

Force la mise à jour immédiate des stratégies ordinateur et utilisateur.

```cmd
gpupdate /target:computer /force
```

Force uniquement les stratégies ordinateur.

```cmd
gpupdate /target:user /force
```

Force uniquement les stratégies utilisateur.

#### Consulter les GPO appliquées

```cmd
gpresult /r
```

Affiche un résumé des GPO appliquées à l'ordinateur et à l'utilisateur.

```cmd
gpresult /h rapport.html
```

Génère un rapport HTML détaillé des GPO appliquées.

#### Ouvrir les consoles de gestion

```cmd
gpmc.msc
```

Console de gestion des stratégies de groupe (serveur).

```cmd
gpedit.msc
```

Éditeur de stratégies locales (poste client).

---

## 7. 🎯 Points clés à retenir

### Architecture

|Composant|Emplacement|Rôle|
|---|---|---|
|**Entrée LDAP**|`CN=Policies,CN=System,DC=xx,DC=xx`|Métadonnées (nom, GUID, droits)|
|**Contenu**|`SYSVOL\Policies\{GUID}`|Instructions de la GPO|
|**Attribut gPLink**|Sur OU/Site/Domaine|Liaison et ordre de traitement|

### Priorité LSDOU

```
Local < Site < Domain < OU
    ↑                     ↑
  Faible              Forte
  priorité           priorité
```

> **Exception** : GPO Enforced = priorité absolue (ignore tout, même le blocage d'héritage)

### Règles d'or

1. **Une action = Une GPO** (principe de séparation des responsabilités)
2. **Ne jamais modifier** la Default Domain Policy
3. **Ne jamais modifier** une GPO active en production (créer une nouvelle version)
4. **Ne pas bloquer l'héritage** sauf cas exceptionnel
5. **LIFO** : La dernière GPO liée est prioritaire au sein d'une OU

### Filtrage de sécurité

Pour qu'une GPO s'applique, il faut :

- ✅ Lien actif (Link Enabled)
- ✅ GPO active (Enabled)
- ✅ Droits **Read** + **Apply Group Policy** sur l'objet cible

### Timing d'application

|GPO Ordinateurs|GPO Utilisateurs|
|---|---|
|Démarrage → Boot|Début login → Fin login|

### Commandes essentielles

```bash
gpupdate /force          # Forcer la mise à jour des GPO
gpresult /r              # Voir les GPO appliquées
gpmc.msc                 # Console de gestion (serveur)
gpedit.msc               # Éditeur local (client)
```

---

## 📚 Sources

- Document de cours : "Les GPO.pdf" (support pédagogique fourni)
- Notes de cours : "GPO.md" (notes personnelles fournies)
- Microsoft Documentation : [Group Policy Overview](https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2012-r2-and-2012/hh831791\(v=ws.11\)) (concepts Microsoft officiels)

---

_Synthèse réalisée le 08/01/2026_