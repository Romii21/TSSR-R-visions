
## Sommaire

1. [Introduction à Active Directory](https://claude.ai/chat/b818e34f-aca7-4b2c-b8aa-9a34343fc6af#1-introduction-%C3%A0-active-directory)
2. [Structure et Composants de Base](https://claude.ai/chat/b818e34f-aca7-4b2c-b8aa-9a34343fc6af#2-structure-et-composants-de-base)
3. [Arborescence Active Directory](https://claude.ai/chat/b818e34f-aca7-4b2c-b8aa-9a34343fc6af#3-arborescence-active-directory)
4. [Protocoles Réseaux Associés](https://claude.ai/chat/b818e34f-aca7-4b2c-b8aa-9a34343fc6af#4-protocoles-r%C3%A9seaux-associ%C3%A9s)
5. [Fonctionnalités Avancées](https://claude.ai/chat/b818e34f-aca7-4b2c-b8aa-9a34343fc6af#5-fonctionnalit%C3%A9s-avanc%C3%A9es)
6. [Objets Active Directory](https://claude.ai/chat/b818e34f-aca7-4b2c-b8aa-9a34343fc6af#6-objets-active-directory)
7. [Bonnes Pratiques d'Administration](https://claude.ai/chat/b818e34f-aca7-4b2c-b8aa-9a34343fc6af#7-bonnes-pratiques-dadministration)
8. [Sécurité Avancée](https://claude.ai/chat/b818e34f-aca7-4b2c-b8aa-9a34343fc6af#8-s%C3%A9curit%C3%A9-avanc%C3%A9e)

---

## 1. Introduction à Active Directory


## 2. Structure et Composants de Base


## 3. Arborescence Active Directory

### 3.1 Structure logique

L'arborescence AD est une **structure logique indépendante du réseau physique**.

⚠️ **Bonne pratique** : Ne jamais lier la structure AD à la topologie réseau pour maintenir la flexibilité.

### 3.2 Les objets AD

Tout dans AD est un **objet** représentant :

- Ressources physiques (postes, serveurs, imprimantes)
- Ressources logiques (utilisateurs, groupes, boîtes mail)
- Services (applications, bases de données)

**Commande PowerShell :**

```powershell
Get-ADObject -Filter {Name -like "*server*"}
```

### 3.3 Unité d'Organisation (OU)

L'**OU (Organizational Unit)** est le conteneur de base pour organiser les objets.

**Fonctions principales :**

- Organiser logiquement les objets de l'annuaire
- Déléguer des droits d'administration
- Appliquer des GPO de manière ciblée

**Commande PowerShell :**

```powershell
Get-ADOrganizationalUnit -Filter {Name -like "*serveurs*"}
```

**⚠️ Bonnes pratiques d'organisation :**

- ❌ **À ÉVITER** : Organisation par localisation géographique (Paris, Lyon, etc.)
- ✅ **RECOMMANDÉ** : Organisation par **fonction/métier** (Comptabilité, RH, Commercial, IT)

**Raison** : Les besoins en accès et GPO sont liés aux fonctions, pas à la géographie.

**Structure recommandée :**

```
Domaine Entreprise
├── OU Utilisateurs
│   ├── OU Direction Générale
│   ├── OU Ressources Humaines
│   ├── OU Commercial
│   └── OU Informatique
└── OU Ordinateurs
    ├── OU Clients
    │   ├── OU PC Portables
    │   └── OU PC Bureaux
    └── OU Serveurs
```

### 3.4 Le Domaine

Un **domaine AD** est une unité administrative et de sécurité.

**Caractéristiques :**

- Contrôle centralisé des politiques de sécurité
- Gestion unique des comptes utilisateurs et ordinateurs
- Périmètre de sécurité cohérent
- Authentification centralisée
- Réplication des données entre contrôleurs

**Commande PowerShell :**

```powershell
Get-ADDomain | Select-Object DomainControllersContainer, DomainMode, DomainSID, Name | Format-List
```

### 3.5 Workgroup vs Domaine

|Caractéristique|Workgroup|Domaine|
|---|---|---|
|Connexion|Compte local requis sur chaque machine|Compte domaine utilisable partout|
|Centralisation|Aucune|Complète via les DC|
|Limite machines|~35-40 machines|Illimitée|
|Réseau|Même réseau local obligatoire|Réseaux différents possibles|
|Rôles serveurs|Tous les PC ont le même rôle|DC(s) + clients|

### 3.6 Arbre et Forêt

**Arbre** : Collection de domaines partageant :

- Un schéma commun
- Une configuration commune
- Un espace de noms contigu (ex: `paris.masociete.fr`, `lyon.masociete.fr`)
- Des relations d'approbation

**Forêt** : Structure hiérarchique de plusieurs arbres comprenant :

- Un schéma d'annuaire commun
- Un catalogue global partagé
- Des relations d'approbation entre arbres
- Des domaines fonctionnant indépendamment

**Exemple de Forêt :**

```
masociete.fr (Arbre 1)          mafilliale.com (Arbre 2)
├── paris.masociete.fr          ├── nantes.mafilliale.com
└── lyon.masociete.fr           │   └── it.nantes.mafilliale.com
                                └── berlin.mafilliale.com
```

**Commande PowerShell :**

```powershell
Get-ADForest
```

**Avantage** : Un utilisateur de `lyon.masociete.fr` peut accéder aux ressources de `it.nantes.mafilliale.com` grâce aux relations d'approbation.

---

## 4. Protocoles Réseaux Associés

### 4.1 Tableau récapitulatif

|Protocole|Rôle|Importance|
|---|---|---|
|**DNS**|Résolution de noms et de services|⚠️ Obligatoire|
|**SNTP**|Synchronisation du temps|⚠️ Critique|
|**LDAP/LDIF**|Accès et modification de l'annuaire|⚠️ Fondamental|
|**Kerberos**|Authentification|⚠️ Central|
|**X509**|Certificats de sécurité|Important|
|**NTFS**|Gestion des permissions|Important|
|**SMB/CIFS**|Partage de fichiers|Important|

### 4.2 DNS - Domain Name System

**Rôle :**

- Résolution des noms d'hôtes
- Localisation des services AD (enregistrements SRV)

⚠️ **Point critique** : Le DNS est le **premier élément à vérifier** en cas de problème AD.

### 4.3 SNTP - Simple Network Time Protocol

**Rôle :** Synchronisation des horloges système (UTC).

**Importance critique :**

- Requis pour l'authentification **Kerberos**
- Nécessaire pour la réplication
- Impact sur les certificats et la journalisation

**Conséquences d'une mauvaise synchronisation :**

- Échec d'authentification Kerberos
- Problèmes de réplication
- Blocage de l'accès aux ressources
- Échec de communication sécurisée

### 4.4 Kerberos

**Protocole d'authentification** par défaut depuis Windows 2000.

**Caractéristiques :**

- Compatible Kerberos V5
- Authentification mutuelle (client ↔ serveur)
- Utilisation de tickets d'authentification à durée limitée
- Adresse du serveur Kerberos extraite du DNS

**Processus :**

1. Demande d'authentification au DC
2. Réception d'un ticket (TGT - Ticket Granting Ticket)
3. Utilisation du ticket pour accéder aux ressources

### 4.5 X509 et Certificats

**AD CS (Active Directory Certificate Services)** fournit :

- Génération et gestion de certificats X.509
- Infrastructure PKI interne (auto-certification)

**Usages :**

- Authentification par carte à puce
- Chiffrement de fichiers (EFS)
- Sécurisation IPSec

### 4.6 NTFS

**NTFS** gère :

- Les permissions sur les volumes et dossiers
- Les droits d'accès combinés avec les groupes AD
- La base de la méthode **AGDLP** (voir section 8.4)

---

## 5. Fonctionnalités Avancées

### 5.1 Niveau Fonctionnel

Le **niveau fonctionnel** détermine les fonctionnalités disponibles.

**Règles :**

- À la création d'un domaine, le niveau = version de l'OS serveur
- Le niveau de la forêt = niveau minimum des domaines
- ⚠️ **Impossible de revenir en arrière** après une montée de niveau

**Exemple :**

```
5 DC sous Windows Server 2012 R2
→ Niveau fonctionnel = Windows Server 2012 R2

3 DC sous 2012 R2 + 2 DC sous 2019
→ Niveau fonctionnel = Windows Server 2012 R2 (le plus ancien)
```

**Raisons de monter le niveau :**

- Accès aux dernières fonctionnalités
- Support des OS récents

### 5.2 Le Schéma

Le **schéma** définit :

- Les classes d'objets (utilisateur, ordinateur, groupe, etc.)
- Les attributs de chaque classe (nom, prénom, SID, etc.)

**Caractéristiques :**

- Partagé au niveau du domaine
- Les modifications sont **définitives**
- ⚠️ Aucun retour en arrière possible

**⚠️ Impact lors de fusions de domaines :**

- Le schéma du domaine parent écrase celui des domaines enfants
- Peut supprimer des attributs personnalisés critiques
- Nécessite une analyse approfondie (plusieurs mois)

### 5.3 Réplication

La **réplication** assure la cohérence des données entre DC.

**Éléments répliqués :**

- Base d'annuaire AD
- GPO (Group Policy Objects)
- Scripts
- Zones DNS

⚠️ **La réplication n'est PAS une sauvegarde**, c'est de la **synchronisation**.

### 5.4 Processus de Réplication

**Étapes :**

1. DC1 modifie un objet
2. DC1 interroge le **KCC (Knowledge Consistency Checker)** pour trouver les autres DC
3. DC1 demande au DNS où se trouve DC2
4. DC1 notifie DC2 des modifications
5. DC2 demande les détails à DC1
6. DC1 transmet les changements
7. DC2 met à jour sa base de données

### 5.5 KCC - Knowledge Consistency Checker

Le **KCC** est présent sur tous les DC et gère :

- La génération de la topologie de réplication
- La mise à jour automatique lors de changements (ajout/suppression de DC)
- Le fonctionnement intra-site et inter-site

### 5.6 Intervalle de Réplication

**Valeurs par défaut :**

|Type|Intervalle par défaut|Recommandation|
|---|---|---|
|Intra-site|5 minutes|❌ Trop fréquent (sature le réseau)|
|Inter-site|180 minutes|❌ Trop espacé (trop de données d'un coup)|

✅ **Recommandation** : **10 à 20 minutes** pour un bon équilibre.

⚠️ Si délai > 30-45 minutes : **problème probable**.

### 5.7 Rôles FSMO

**FSMO (Flexible Single Master Operation)** : 5 rôles critiques.

#### Au niveau de la Forêt (2 rôles)

|Rôle|Fonction|
|---|---|
|**Schema Master** (Maître de Schéma)|Gère les mises à jour du schéma|
|**Domain Naming Master** (Maître d'Attribution de Noms)|Gère l'ajout/suppression de domaines|

#### Au niveau du Domaine (3 rôles)

|Rôle|Fonction|
|---|---|
|**RID Master** (Maître RID)|Attribue les RID/SID uniques|
|**Infrastructure Master** (Maître d'Infrastructure)|Gère les relations inter-domaines|
|**PDC Emulator** (Émulateur PDC)|⚠️ **LE PLUS CRITIQUE** - Synchronisation du temps, verrouillage de comptes|

**⚠️ Importance du PDC Emulator :**

- Gère la synchronisation du temps pour tout le domaine
- Si le temps n'est pas synchronisé → **verrouillage complet de l'AD**

### 5.8 Bonnes Pratiques FSMO

✅ **Recommandations :**

- Ne jamais avoir tous les rôles sur le même DC
- Idéalement : **5 DC avec 1 rôle chacun**
- Au minimum : **2 DC** avec séparation forêt/domaine

⚠️ **Ne pas confondre :**

- Rôles serveur Windows (DHCP, DNS, etc.)
- Rôles AD (AD DS, AD FS, etc.)
- Rôles FSMO (Schema Master, RID Master, etc.)

---

## 6. Objets Active Directory

### 6.1 Définition

Chaque **objet AD** est :

- Une instance d'une **classe** définie dans le schéma
- Possède tous les **attributs** de sa classe
- Peut être de type **conteneur** ou **leaf object** (objet terminal)

**Types d'objets :**

1. **Ressources** : postes, serveurs, dossiers partagés, imprimantes
2. **Utilisateurs** : comptes individuels, comptes de service, groupes
3. **Services** : applications, bases de données

**Commande PowerShell :**

```powershell
Get-ADUser -Filter * -Properties * | Where-Object {$_.Name -eq "User1"}
```

### 6.2 Les Attributs

Les **attributs** définissent les informations contenues dans un objet.

**Exemples d'attributs :**

- Nom, prénom, email
- Date de dernière connexion
- Bureau, service, téléphone
- SID, GUID (identifiants uniques)

**Types d'attributs :**

- Numériques
- Textuels
- Booléens (vrai/faux)

### 6.3 Classes d'Objets

#### Conteneurs

- **Container** : Conteneur natif
- **Organizational Unit (OU)** : Unité d'organisation
- **Group** : Groupe

#### Leaf Objects (objets terminaux)

- **User** : Utilisateur
- **Computer** : Ordinateur (client ou serveur)
- **Printer** : Imprimante

### 6.4 Les Groupes

Les **groupes** ont deux propriétés principales :

#### Étendue (Scope)

|Type|Portée|Usage|
|---|---|---|
|**Domain Local**|Domaine uniquement|Droits sur ressources locales|
|**Global**|Tous les domaines approuvés|✅ Le plus courant - Groupes métiers|
|**Universal**|Toute la forêt|Groupes transverses|

#### Type

|Type|Usage|
|---|---|
|**Security**|✅ Majoritaire - Gestion des autorisations|
|**Distribution**|Listes de distribution (messagerie)|

### 6.5 Identifiants Uniques

#### SID - Security Identifier

- **Unique** au sein d'un domaine
- **Peut changer** si l'objet change de domaine
- Longueur max : 256 caractères
- Format : `S-1-5-21-156063872-1535639461-3779917529-1134`

**Commande PowerShell :**

```powershell
$Var1 = Get-ADUser -Filter * -Properties * | Where-Object {$_.Name -eq "User1"}
$Var1 | Select Name, ObjectSID
```

#### GUID - Globally Unique Identifier

- **Unique** au sein d'une forêt
- **Ne change JAMAIS**
- Codé sur 128 bits
- Format : `3F2504E0-4F89-11D3-9A0C-0305E82C3301`

**Commande PowerShell :**

```powershell
$Var1 | Select Name, ObjectGUID
```

#### DN - Distinguished Name

- **Chemin LDAP** complet dans l'annuaire
- Théoriquement unique dans la forêt
- **Peut changer** si l'objet est déplacé

**Format :**

```
CN=wilder,OU=RS,OU=utilisateurs,OU=Paris,OU=France,DC=masociete,DC=fr
```

**Composants :**

- `DC=` : Domain Component
- `OU=` : Organizational Unit
- `CN=` : Common Name

**Commande PowerShell :**

```powershell
$Var1 | Select Name, DistinguishedName
```

---

## 7. Bonnes Pratiques d'Administration

### 7.1 Gestion des Identités et Accès

#### Principe de Moindre Privilège (JEA - Just Enough Administration)

✅ Limiter les droits au **strict nécessaire** pour exécuter une tâche.

**Avantages :**

- Limite le périmètre de compromission
- Réduit les risques d'erreurs
- Améliore la traçabilité

#### Comptes d'Administration Séparés

✅ Avoir des comptes distincts pour :

- Usage quotidien (compte standard)
- Tâches administratives (compte admin)

**Modèle recommandé** : **Tiering Model** (voir section 8.1)

#### Gestion des Mots de Passe Locaux - LAPS

**LAPS (Local Administrator Password Solution)** :

- Génère des mots de passe aléatoires et complexes
- Change le mot de passe administrateur local de chaque machine
- Stocke les mots de passe dans l'AD de manière sécurisée

⚠️ **Problème sans LAPS** : Mots de passe génériques identiques sur tous les postes.

### 7.2 Politiques de Mot de Passe

**Recommandations ANSSI (2024) :**

|Type d'utilisateur|Longueur|Complexité|Expiration|
|---|---|---|---|
|Standard|17 caractères|Oui|Aucune|
|Administrateur|20+ caractères|Oui|~3 mois|

**Raison** : Changer fréquemment un mot de passe complexe amène les utilisateurs à simplement modifier 1-2 caractères, ce qui n'améliore pas la sécurité.

✅ **Bonne pratique** : Mot de passe long et complexe **sans** changement fréquent pour les utilisateurs standards.

### 7.3 Séparation Utilisateurs/Ressources - AGDLP

Voir section 8.4 pour le détail complet.

### 7.4 Sécurité Réseau et Serveurs

#### Protection des DC

✅ **Recommandations :**

- Placer les DC dans un **VLAN dédié** et sécurisé
- Isoler du réseau utilisateur (éventuellement DMZ)

#### Mises à Jour

✅ **Bonnes pratiques :**

- **Maintenance cyclique** régulière
- **Patch management** spécifique pour l'AD
- ❌ **Ne JAMAIS** activer les redémarrages automatiques sur les DC
- Tester les mises à jour majeures **hors production** d'abord

#### Communications LDAP

✅ Utiliser **LDAPS (LDAP over SSL/TLS)** pour sécuriser les transmissions.

### 7.5 Surveillance et Audit

#### Audits Réguliers

✅ Mettre en place :

- Audits internes et externes
- Examen régulier des logs
- Détection d'activités anormales

⚠️ **Constat** : Rarement mis en place dans les entreprises.

#### Stratégie de Sauvegarde

✅ Préparer :

- Plans de sauvegarde
- Plans de restauration pour les urgences
- Appliquer la **règle 3-2-1** :
    - **3** copies des données
    - Sur **2** supports différents
    - **1** copie hors site

⚠️ **Attention** : Restaurer un AD est une tâche **très complexe**.

---

## 8. Sécurité Avancée

### 8.1 Microsoft Tiering Model

Le **Tiering Model** (Modèle à Tiers) sépare l'infrastructure en **3 niveaux hermétiques**.

**Objectif** : Limiter la propagation des attaques en cloisonnant les niveaux d'administration.

#### Tier 0 - Infrastructure Critique

**Composants :**

- Contrôleurs de domaine (DC)
- Serveurs PKI
- Serveurs ADFS
- Tout système avec contrôle direct sur l'environnement AD

**Règles :**

- Un admin T0 **ne gère QUE** des composants T0
- ❌ **Interdit** de se connecter à des serveurs T1 ou T2 avec un compte T0

#### Tier 1 - Serveurs et Applications

**Composants :**

- Serveurs applicatifs
- Middlewares
- Services d'entreprise (SCCM, WSUS, SCOM, etc.)

**Règles :**

- Un admin T1 gère les serveurs applicatifs
- ❌ Ne doit pas accéder aux niveaux T0 ou T2

#### Tier 2 - Postes de Travail

**Composants :**

- Postes des utilisateurs finaux
- Postes des administrateurs (pour usage quotidien)
- Périphériques mobiles (smartphones, tablettes)

**Caractéristique :** Couche **la plus à risque** (phishing, ransomware, erreurs utilisateur).

**Règles :**

- Un admin T2 gère les postes utilisateurs
- ❌ Ne peut pas accéder aux niveaux supérieurs

#### Gestion des Comptes

Un administrateur doit avoir **plusieurs comptes** :

- 1 compte **utilisateur standard** (usage quotidien)
- 1 compte **admin T2** (gestion postes)
- 1 compte **admin T1** (gestion serveurs)
- 1 compte **admin T0** (infrastructure critique)

**⚠️ Complexité :**

- Gestion de multiples comptes
- Bascules fréquentes entre comptes
- Modifications réseau (VLANs)
- Révision complète des droits d'accès

### 8.2 JIT & JEA

#### JIT - Just-In-Time

**Principe** : Octroyer des privilèges **temporaires**.

**Fonctionnement :**

- L'administrateur demande l'accès pour une tâche spécifique
- Approbation requise (une ou plusieurs personnes)
- Droits accordés pour une **durée limitée** (heures, jours)
- Révocation automatique à l'expiration

**Avantages :**

- Limite la fenêtre d'exploitation en cas de compromission
- Traçabilité des actions

#### JEA - Just Enough Administration

**Principe** : Limiter les privilèges au **strict nécessaire**.

**Fonctionnement :**

- L'administrateur reçoit uniquement les droits requis pour sa tâche
- Pas d'accès global, même temporairement

**Exemple** : Un prestataire VPN accède uniquement aux ressources de son périmètre d'intervention.

**⚠️ Problème actuel :**

- Les comptes à privilèges élevés conservent ces privilèges en permanence
- En cas de compromission = **trou de sécurité majeur**

### 8.3 Solution Combinée JIT + JEA

✅ **Approche recommandée** :

1. Demande d'accès avec justification détaillée
2. Approbation par une/plusieurs personnes
3. Octroi des droits **minimaux nécessaires**
4. Pour une **durée limitée**
5. Révocation automatique

### 8.4 AGDLP - Méthode de Gestion des Permissions

**AGDLP** = **A**ccount → **G**lobal Groups → **D**omain **L**ocal Groups → **P**ermissions

#### Principe

✅ **Ne JAMAIS donner de droits directement à un utilisateur.**

**Flux recommandé :**

```
Utilisateurs (A) 
→ appartiennent à Groupes Métiers (G) 
→ qui appartiennent à Groupes de Droits (DL) 
→ qui ont des Permissions (P) sur les ressources
```

#### Exemple Pratique 1 - Comptables

**Besoin :**

- Bob et Alice sont comptables
- Ils doivent accéder à :
    - Dossier "Compta" en lecture/écriture
    - Dossier "Paye" en lecture seule

**❌ Mauvaise pratique :** Donner directement l'accès aux dossiers à Bob et Alice.

**✅ Bonne pratique AGDLP :**

1. **A** - Utilisateurs :
    
    - Bob
    - Alice
2. **G** - Groupe Métier :
    
    - `Grp_Compta` (contient Bob et Alice)
3. **DL** - Groupes de Droits :
    
    - `Grp_Compta_RW` → Permissions RW sur dossier Compta
    - `Grp_Paye_R` → Permissions R sur dossier Paye
4. **P** - Permissions :
    
    - `Grp_Compta` est membre de `Grp_Compta_RW` et `Grp_Paye_R`

**Résultat** : Bob et Alice ont les bons accès via leur appartenance au groupe métier.

#### Exemple Pratique 2 - Accès Spécifique

**Besoin :**

- Alice doit aussi accéder au dossier "Finance" en lecture
- Bob ne doit PAS y accéder

**❌ Mauvaise pratique :** Donner directement l'accès à Alice.

**✅ Bonne pratique AGDLP :**

1. Créer un **groupe métier transverse** : `Grp_Compta_Finance`
2. Ajouter Alice à `Grp_Compta_Finance`
3. `Grp_Compta_Finance` est membre de `Grp_Finance_R`
4. `Grp_Finance_R` a les permissions R sur dossier Finance

**Résultat** : Alice a l'accès, pas Bob. Au départ d'Alice, on la retire simplement des groupes.

#### Gestion des Comptes de Service

**❌ PIRE scénario** : Utiliser un compte nominatif pour une relation BDD/AD.

**Exemple de problème :**

- Compte utilisateur "Jean" utilisé pour connexion BDD
- Jean part à la retraite → compte désactivé
- Serveur perd l'accès à la BDD

**✅ Solution :**

1. Créer un **compte de service dédié**
2. Le placer dans un **groupe de droits** spécifique
3. L'intégrer dans une **OU Comptes de Service**
4. Donner les permissions au groupe, pas au compte directement

#### Hiérarchie de Groupes

✅ **Possible** : Imbriquer des groupes métiers.

**Exemple :**

```
Groupe Métier Principal
├── Groupe Métier 1
├── Groupe Métier 2
└── Groupe Métier Transverse
    ↓
Groupe de Droits
```

**⚠️ À ÉVITER** : Faire l'inverse (groupes de droits dans groupes métiers).

#### Automatisation

✅ **Recommandation** :

- Créer tous les groupes et accès NTFS **en amont**
- Utiliser des **scripts PowerShell** pour l'automatisation
- Simplifier la gestion lors des mouvements de personnel

**Avantages :**

- Gestion rapide des arrivées/départs
- Pas de manipulation directe des droits
- Traçabilité complète

---

## Tableau Récapitulatif des Concepts Clés

|Concept|Description|Commande PowerShell|
|---|---|---|
|**Objet AD**|Élément de base de l'annuaire|`Get-ADObject`|
|**OU**|Conteneur pour organiser les objets|`Get-ADOrganizationalUnit`|
|**Domaine**|Unité administrative et de sécurité|`Get-ADDomain`|
|**Forêt**|Collection d'arbres de domaines|`Get-ADForest`|
|**DC**|Serveur contrôleur de domaine|`Get-ADDomainController`|
|**FSMO**|Rôles maîtres uniques|(visible dans `Get-ADDomainController`)|
|**Groupe**|Conteneur d'utilisateurs/objets|`Get-ADGroup`|
|**Utilisateur**|Compte d'accès|`Get-ADUser`|

---

## Checklist des Bonnes Pratiques

### Structure

- [ ] Organiser les OU par fonction/métier (pas par géographie)
- [ ] Séparer clairement OU Utilisateurs et OU Ordinateurs
- [ ] Éviter une granularité excessive

### Sécurité de Base

- [ ] Appliquer le principe de moindre privilège
- [ ] Utiliser des comptes d'administration séparés
- [ ] Déployer LAPS sur tous les postes
- [ ] Politique de mots de passe conforme ANSSI

### Infrastructure

- [ ] Minimum 2 DC pour la redondance
- [ ] DC dans des VLANs sécurisés
- [ ] Pas de redémarrage automatique des DC
- [ ] Réplication configurée (10-20 min recommandé)
- [ ] Séparer les rôles FSMO sur différents DC

### Sécurité Avancée

- [ ] Implémenter le Tiering Model
- [ ] Mettre en place JIT et JEA
- [ ] Appliquer systématiquement AGDLP
- [ ] Utiliser LDAPS pour toutes les communications

### Surveillance

- [ ] Audits réguliers de l'AD
- [ ] Monitoring des logs
- [ ] Stratégie de sauvegarde 3-2-1
- [ ] Plan de restauration testé

---

## Glossaire des Acronymes

|Acronyme|Signification|Description|
|---|---|---|
|**AD**|Active Directory|Service d'annuaire Microsoft|
|**AD DS**|Active Directory Domain Services|Service principal d'annuaire et de domaine|
|**AD CS**|Active Directory Certificate Services|Service de gestion de certificats|
|**AD FS**|Active Directory Federation Services|Service de fédération (SSO)|
|**AD RMS**|Active Directory Rights Management Services|Gestion des droits sur fichiers|
|**AD LDS**|Active Directory Lightweight Directory Services|Service d'annuaire sans domaine|
|**LDAP**|Lightweight Directory Access Protocol|Protocole d'accès aux annuaires|
|**LDIF**|LDAP Data Interchange Format|Format de fichier pour import/export LDAP|
|**SAM**|Security Account Manager|Base de comptes locale (avant AD)|
|**NTDS**|NT Directory Services|Nom originel d'Active Directory|
|**DC**|Domain Controller|Contrôleur de domaine|
|**RODC**|Read-Only Domain Controller|Contrôleur de domaine en lecture seule|
|**OU**|Organizational Unit|Unité d'organisation|
|**GPO**|Group Policy Object|Stratégie de groupe|
|**DN**|Distinguished Name|Nom distinctif (chemin LDAP complet)|
|**SID**|Security Identifier|Identifiant de sécurité|
|**GUID**|Globally Unique Identifier|Identifiant unique global|
|**RID**|Relative Identifier|Identifiant relatif|
|**FSMO**|Flexible Single Master Operation|Rôles maîtres uniques|
|**PDC**|Primary Domain Controller|Contrôleur de domaine principal|
|**KCC**|Knowledge Consistency Checker|Vérificateur de cohérence (réplication)|
|**DNS**|Domain Name System|Système de noms de domaine|
|**SNTP**|Simple Network Time Protocol|Protocole de synchronisation du temps|
|**NTFS**|New Technology File System|Système de fichiers Windows|
|**SMB**|Server Message Block|Protocole de partage de fichiers|
|**CIFS**|Common Internet File System|Protocole de partage (évolution SMB)|
|**SSO**|Single Sign-On|Authentification unique|
|**PKI**|Public Key Infrastructure|Infrastructure à clés publiques|
|**LAPS**|Local Administrator Password Solution|Solution de gestion des mots de passe locaux|
|**ANSSI**|Agence Nationale de la Sécurité des Systèmes d'Information|Agence française de cybersécurité|
|**JIT**|Just-In-Time|Administration à la demande|
|**JEA**|Just Enough Administration|Administration au strict nécessaire|
|**AGDLP**|Account, Global, Domain Local, Permission|Méthode de gestion des permissions|
|**VLAN**|Virtual Local Area Network|Réseau local virtuel|
|**DMZ**|Demilitarized Zone|Zone démilitarisée (réseau)|
|**TGT**|Ticket Granting Ticket|Ticket Kerberos|
|**IPSec**|Internet Protocol Security|Protocole de sécurisation IP|
|**EFS**|Encrypting File System|Système de chiffrement de fichiers|

---

## Sources et Références

### Documents sources utilisés :

1. **Active Directory - partie 1.pdf** - Cours théorique sur les fondamentaux AD
2. **Active Directory - partie 2.pdf** - Cours avancé sur les protocoles et bonnes pratiques
3. **s10-cours-Active Directory - Notes par Gemini.docx** - Notes détaillées du cours en vidéo

### Documentation Microsoft :

- [Documentation officielle Active Directory](https://docs.microsoft.com/en-us/windows-server/identity/ad-ds/)
- [Bonnes pratiques de sécurité AD](https://docs.microsoft.com/en-us/windows-server/identity/ad-ds/plan/security-best-practices/)

### Ressources complémentaires :

- ANSSI - Recommandations de sécurité relatives à Active Directory
- Microsoft - Tiering Model (modèle d'administration à niveaux)
- RFC 4511 - Lightweight Directory Access Protocol (LDAP)

---

**Note** : Cette synthèse a été enrichie par les informations des notes Gemini qui ont apporté une perspective pratique et des exemples concrets issus du cours en vidéo, notamment sur :

- Les bonnes pratiques d'organisation des OU par fonction plutôt que géographie
- Les détails du processus de réplication
- L'importance critique du PDC Emulator
- Les exemples pratiques d'application d'AGDLP
- Les recommandations de l'ANSSI sur les mots de passe
- Les complexités de mise en œuvre du Tiering Model