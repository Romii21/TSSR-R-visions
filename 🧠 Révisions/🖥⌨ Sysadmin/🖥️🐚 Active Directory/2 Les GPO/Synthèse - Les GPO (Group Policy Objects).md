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

Une GPO est constituée de **trois composantes principales** :

### 2.1 L'entrée LDAP

|Élément|Emplacement|Contenu|
|---|---|---|
|**Localisation**|`CN=Policies,CN=System,DC=xx,DC=xx`|Partition principale de l'AD|
|**Informations**|Entrée LDAP|Nom, GUID, droits d'édition|
|**Fonction**|Administrative|Gestion des permissions et délégations|

### 2.2 Le contenu de la GPO

```
Emplacement : \\serveur\SYSVOL\domaine\Policies\{GUID}
```

|Élément|Description|
|---|---|
|**Stockage**|Partage SYSVOL sur le serveur AD|
|**Structure**|Répertoire nommé avec le GUID|
|**Contenu**|Fichiers d'instructions définissant les actions|

> **Important** : Le dossier SYSVOL contient l'ensemble des configurations qui seront appliquées.

### 2.3 L'attribut gPLink

Cet attribut est affecté à une OU ou à un site AD et contient :

- 🔑 L'identifiant de la GPO (GUID)
- 📍 Le chemin LDAP de la GPO
- 🔢 L'ordre de traitement
- ✓/✗ L'application ou non de la GPO

---

## 3. États et configurations

### 3.1 État forcé (Enforced)

L'état **Enforced** donne la priorité absolue à une GPO.

|Caractéristique|Description|
|---|---|
|**Priorité**|Écrase toutes les GPO de niveaux inférieurs|
|**Héritage**|Ignore le blocage d'héritage|
|**Usage**|Très rare, réservé au Tier 0 (infrastructure critique)|
|**Exemple**|GPO domaine > GPO OU même si blocage activé|

> ⚠️ **Attention** : L'état forcé doit être utilisé avec parcimonie car il ignore toutes les autres configurations.

### 3.2 État actif (Enabled/Disabled)

|État|Description|
|---|---|
|**Enabled**|La GPO est active et s'applique|
|**Disabled**|La GPO est désactivée, aucune application|

### 3.3 Link Enabled vs GPO Status

Différence importante entre deux concepts :

|Type|Portée|Effet|
|---|---|---|
|**Link Enabled**|Contrôle le lien GPO ↔ OU|Si désactivé : GPO non appliquée à cette OU spécifique|
|**GPO Status Enabled**|État global de la GPO|Si désactivé : GPO non appliquée nulle part|

**Règle** : Une GPO doit être Enabled ET son lien doit être Link Enabled pour s'appliquer.

---

## 4. Règles de priorité (LSDOU)

### 4.1 Stratégies locales vs GPO

|Type|Gestion|Console|Environnement|Priorité|
|---|---|---|---|---|
|**Stratégies locales**|Locale (par machine)|`gpedit.msc`|Workgroup ou Domaine|Basse|
|**GPO**|Centralisée via AD|`gpmc` (serveur)|Domaine uniquement|Haute|

> **Principe** : Les GPO écrasent toujours les stratégies locales.

### 4.2 Ordre d'application LSDOU

Les GPO s'appliquent dans cet ordre précis :

```
1. Local      → Stratégies locales de la machine
2. Site       → GPO du site AD
3. Domain     → GPO du domaine
4. OU         → GPO des unités d'organisation (de la plus haute à la plus spécifique)
```

**Principe d'héritage** : Chaque niveau hérite du précédent et peut le compléter ou l'écraser.

### 4.3 Ordre au sein d'une OU (LIFO)

Quand plusieurs GPO sont liées à une même OU :

|Principe|Description|
|---|---|
|**LIFO**|Last In, First Out = Dernière liée, première traitée|
|**Ordre manuel**|Peut être défini manuellement dans la console|
|**Chevauchement**|La dernière GPO traitée prévaut en cas de conflit|

**Exemple** :

```
GPO 1 (liée en premier)   → Ordre de lien : 3
GPO 2 (liée en deuxième)  → Ordre de lien : 2
GPO 3 (liée en dernier)   → Ordre de lien : 1 (prioritaire)
```

### 4.4 Blocage d'héritage et Enforced

|Configuration|Effet|Contre-mesure|
|---|---|---|
|**Blocage d'héritage sur OU**|Bloque les GPO des niveaux supérieurs|GPO Enforced ignore le blocage|
|**GPO Enforced**|Force l'application même si héritage bloqué|Aucune (priorité absolue)|

> ⚠️ **Bonne pratique** : Ne pas bloquer l'héritage sauf cas exceptionnel.

### 4.5 Timing d'application

|Type de GPO|Moment d'application|
|---|---|
|**GPO Ordinateurs**|Entre le démarrage et le boot (avant l'écran de connexion)|
|**GPO Utilisateurs**|Entre le début du login et la fin du login|

---

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