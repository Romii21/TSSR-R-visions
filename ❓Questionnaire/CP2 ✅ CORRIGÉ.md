# CORRECTION — CP2 Active Directory

---

## Partie 1 — Connaissances générales sur Windows Server

---

### Q1 — Mode Core vs GUI

**Ta réponse :** Un serveur Core est un serveur Windows en ligne de commande, moins gourmand en ressources et moins sensible aux pannes.

**Verdict : ✅ Bonne base, mais à compléter**

Tu as la bonne idée. Voici la réponse complète attendue :

|Critère|Mode GUI|Mode Core|
|---|---|---|
|Interface|Graphique (bureau Windows)|Ligne de commande uniquement|
|Ressources|Élevées|Réduites (RAM, CPU, disque)|
|Surface d'attaque|Large|Réduite|
|Gestion|Locale + distante|Principalement distante (PowerShell, RSAT)|

**Avantages du mode Core en production :**

- Moins de ressources consommées → serveur plus performant
- Moins de composants installés → moins de vulnérabilités potentielles
- Moins de redémarrages nécessaires (pas d'interface à mettre à jour)
- Recommandé par Microsoft pour les DC en production

---

### Q2 — Server Manager

**Ta réponse :** (non répondue)

**Verdict : 🔵 Non répondu**

Le **Server Manager** est la console d'administration centrale de Windows Server (interface graphique). Il permet de :

1. **Ajouter/Supprimer des rôles et fonctionnalités** (ex : AD DS, DHCP, DNS)
2. **Surveiller l'état des serveurs** (événements, performances, alertes)
3. **Gérer plusieurs serveurs à distance** depuis une seule console
4. **Accéder aux outils d'administration** (ADUC, DNS Manager, etc.)
5. **Configurer le serveur local** (nom, IP, fuseau horaire, Windows Update)

---

### Q3 — Gestion des rôles en ligne de commande

**Ta réponse :** L'invite de commande et PowerShell.

**Verdict : ⚠️ Incomplet**

L'outil principal est **PowerShell** avec le module `ServerManager`. La commande pour installer AD DS est :

```powershell
Install-WindowsFeature -Name AD-Domain-Services -IncludeManagementTools
```

L'invite de commande classique (`cmd.exe`) ne permet pas d'installer des rôles directement. On peut aussi utiliser **DISM** mais PowerShell est la référence.

---

### Q4 — Rôle vs Fonctionnalité

**Ta réponse :** (non répondue)

**Verdict : 🔵 Non répondu**

|Concept|Définition|Exemples|
|---|---|---|
|**Rôle**|Service principal que le serveur va fournir au réseau|AD DS, DHCP, DNS, IIS, Hyper-V|
|**Fonctionnalité**|Composant complémentaire qui enrichit un rôle ou le serveur|.NET Framework, RSAT, Sauvegarde Windows Server|

**Résumé simple :** Un rôle = ce que le serveur "fait". Une fonctionnalité = un outil qui aide à le faire.

---

### Q5 — Workgroup vs Domaine AD

**Ta réponse :** Le Workgroup est le groupe par défaut, la machine est isolée, utilisé en privé. Les machines du domaine profitent des GPO, DHCP, DNS, etc.

**Verdict : ✅ Très bonne réponse**

Petite précision à ajouter : le Workgroup atteint ses limites autour de **35-40 machines**. Au-delà, la gestion devient ingérable (chaque machine a sa propre base de comptes locaux, aucune centralisation).

---

## Partie 2 — Fondamentaux d'Active Directory

---

### Q6 — Qu'est-ce qu'Active Directory ?

**Ta réponse :** Un ensemble de rôles qui permet aux machines du domaine de bénéficier de plusieurs fonctionnalités.

**Verdict : ⚠️ Trop vague**

La définition attendue : **Active Directory est l'implémentation Microsoft des services d'annuaire basés sur le protocole LDAP.** Il centralise la gestion des identités et des ressources d'un réseau.

**Ses trois objectifs principaux :**

1. **Centraliser l'identification et l'authentification** des utilisateurs
2. **Gérer les politiques de sécurité** (GPO) sur l'ensemble du réseau
3. **Répertorier et faciliter la gestion des ressources réseau** (serveurs, imprimantes, partages)

---

### Q7 — Protocole d'Active Directory + BDD hiérarchique vs relationnelle

**Ta réponse :** Le protocole LDAP ou LDAPS.

**Verdict : ✅ Correct pour le protocole**

Pour la deuxième partie de la question (différence BDD), tu n'as pas répondu :

|Base Hiérarchique (LDAP/AD)|Base Relationnelle (MySQL, Oracle)|
|---|---|
|Structure en arborescence|Tables avec relations|
|Recherche par chemin hiérarchique|Recherche par clés et jointures|
|Optimisée pour la **lecture**|Optimisée pour lectures **et** écritures|
|Exemple : AD, OpenLDAP|Exemple : MySQL, Oracle, SQL Server|

---

### Q8 — Le SAM avant Active Directory

**Ta réponse :** (non répondue)

**Verdict : 🔵 Non répondu**

**SAM (Security Account Manager)** était la base de données utilisée avant AD pour stocker les comptes **localement** sur chaque machine.

**Ses limites :**

- Adapté uniquement aux petits réseaux (moins de ~20 machines)
- **Aucune centralisation** : chaque machine gère ses propres comptes
- Gestion très complexe au-delà de quelques machines
- Pas de notion de domaine ni de politiques globales

Il a été remplacé par AD car les entreprises grandissaient et avaient besoin de gérer des centaines voire des milliers de comptes de manière centralisée.

---

### Q9 — Contrôleur de domaine (DC)

**Ta réponse :** Il contrôle le domaine dont il fait partie.

**Verdict : ❌ Trop imprécis**

Un **contrôleur de domaine (DC)** est un serveur Windows sur lequel est installé le rôle **AD DS**. Son rôle concret :

- Stocker et gérer la **base de données AD** (le fichier `NTDS.dit`)
- Gérer l'**authentification** des utilisateurs (via Kerberos)
- Appliquer les **GPO** aux machines et utilisateurs du domaine
- Assurer la **réplication** avec les autres DC
- Fournir les services **DNS** et **LDAP** du domaine

**Point important :** Pour la redondance, une infrastructure doit avoir **au minimum 2 DC**. Si le seul DC tombe, plus aucune authentification n'est possible.

---

### Q10 — Les cinq rôles Active Directory

**Ta réponse :** AD DS (gestion domaine), AD CS (certificats). Les 3 autres non répondus.

**Verdict : ⚠️ Incomplet**

|Rôle|Nom complet|Description|
|---|---|---|
|**AD DS**|Active Directory Domain Services|Rôle principal : gestion du domaine, authentification, GPO, annuaire|
|**AD CS**|Active Directory Certificate Services|Gestion des certificats numériques et de la PKI (clés de chiffrement)|
|**AD FS**|Active Directory Federation Services|Portail SSO (Single Sign-On) : authentification unique entre domaines/applications|
|**AD RMS**|Active Directory Rights Management Services|Gestion des droits d'accès sur les fichiers (qui peut lire, modifier, imprimer)|
|**AD LDS**|Active Directory Lightweight Directory Services|Service d'annuaire LDAP **sans** domaine (pour applications tierces)|

---

## Partie 3 — Structure et arborescence AD

---

### Q11 — L'OU (Organizational Unit)

**Ta réponse :** Une OU permet d'organiser et de trier les objets d'un domaine comme des dossiers.

**Verdict : ✅ Correct**

Les **trois fonctions principales** d'une OU :

1. **Organiser logiquement** les objets de l'annuaire (utilisateurs, ordinateurs, groupes)
2. **Déléguer des droits d'administration** (ex : un technicien peut gérer son OU sans toucher au reste)
3. **Appliquer des GPO de manière ciblée** (une GPO liée à une OU ne s'applique qu'à ses objets)

---

### Q12 — Organisation des OU : géographie ou fonction ?

**Ta réponse :** Par fonction et métier, c'est plus simple à gérer.

**Verdict : ✅ Bonne réponse**

La justification complète : les GPO sont liées aux **besoins fonctionnels**, pas à la localisation. Un commercial à Paris et un commercial à Lyon ont les mêmes besoins logiciels et de sécurité. Organiser par géographie créerait des doublons de GPO inutiles.

---

### Q13 — Arbre vs Forêt

**Ta réponse :** Un arbre représente un domaine et une forêt plusieurs. L'AD peut administrer plusieurs forêts et les relier.

**Verdict : ⚠️ Légèrement imprécis**

- **Arbre** = collection de domaines partageant un **espace de noms contigu** (ex : `paris.masociete.fr` et `lyon.masociete.fr` appartiennent à l'arbre `masociete.fr`). Ce n'est pas forcément un seul domaine.
- **Forêt** = ensemble de plusieurs arbres reliés par des **relations d'approbation**, partageant un schéma commun.

**Exemple de forêt multi-domaines :**

```
masociete.fr (Arbre 1)          mafilliale.com (Arbre 2)
├── paris.masociete.fr          ├── nantes.mafilliale.com
└── lyon.masociete.fr           └── berlin.mafilliale.com
```

Les utilisateurs de `masociete.fr` peuvent accéder aux ressources de `mafilliale.com` grâce aux relations d'approbation.

---

### Q14 — Domaine / Arbre / Forêt + lien entre domaines

**Ta réponse :** (non répondue dans le détail)

**Verdict : 🔵 Non répondu**

|Concept|Définition|
|---|---|
|**Domaine**|Unité administrative de base. Périmètre de sécurité avec un DC, des utilisateurs, des GPO.|
|**Arbre**|Ensemble de domaines partageant un espace de noms contigu, reliés par des approbations parentales.|
|**Forêt**|Ensemble d'arbres partageant un schéma commun et un catalogue global.|

**Le lien qui unit les domaines d'un même arbre :** des **relations d'approbation bidirectionnelles et transitives**. Si A approuve B et B approuve C, alors A approuve automatiquement C.

---

### Q15 — Distinguished Name (DN)

**Ta réponse :** Le CN est le Common Name (nom de l'objet), dans l'OU Informatique, dans l'OU Utilisateurs, dans le domaine entreprise.local.

**Verdict : ✅ Très bien**

Décomposition complète :

```
CN=jdupont,OU=Informatique,OU=Utilisateurs,DC=entreprise,DC=local
```

|Composant|Signification|Valeur|
|---|---|---|
|`CN=`|Common Name — nom de l'objet|jdupont|
|`OU=`|Organizational Unit — conteneur|Informatique, puis Utilisateurs|
|`DC=`|Domain Component — composant du domaine|entreprise.local|

**Important :** Le DN se lit de gauche (le plus spécifique) vers la droite (le plus général). Le DN peut changer si l'objet est déplacé dans une autre OU.

---

## Partie 4 — Objets Active Directory

---

### Q16 — Types d'objets AD

**Ta réponse :** Les utilisateurs et les groupes.

**Verdict : ❌ Incomplet**

Il existe deux grandes catégories d'objets AD :

1. **Objets conteneurs** (peuvent contenir d'autres objets) : OU, Group, Container
2. **Objets terminaux / Leaf objects** (ne contiennent pas d'autres objets) : User, Computer, Printer

Exemples concrets :

- **Conteneurs** : OU "Informatique", Groupe "GRP_Admins"
- **Terminaux** : compte utilisateur "jdupont", ordinateur "PC-BUREAU-01", imprimante "HP-RDC"

---

### Q17 — Étendues des groupes (Domain Local / Global / Universal)

**Ta réponse :** (non répondue)

**Verdict : 🔵 Non répondu**

|Étendue|Portée|Usage typique|
|---|---|---|
|**Domain Local**|Domaine local uniquement|Attribuer des droits sur des ressources locales (ex : accès à un partage)|
|**Global**|Tous les domaines approuvés|Regrouper des utilisateurs d'un même domaine par métier/fonction ✅ Le plus courant|
|**Universal**|Toute la forêt|Groupes transverses à plusieurs domaines (ex : groupe "Direction Générale" multi-sites)|

**Méthode recommandée AGDLP :** Account → Global → Domain Local → Permission

---

### Q18 — Groupe de sécurité vs groupe de distribution

**Ta réponse :** (non répondue)

**Verdict : 🔵 Non répondu**

|Type|Usage|Permission sur ressources|
|---|---|---|
|**Sécurité**|Gestion des droits d'accès (NTFS, partages)|✅ Oui — c'est son rôle principal|
|**Distribution**|Listes de diffusion e-mail (Exchange/Outlook)|❌ Non — il ne peut pas gérer des permissions|

**Réponse à la question :** C'est le groupe de **sécurité** qui gère les permissions sur les ressources.

---

### Q19 — SID (Security Identifier)

**Ta réponse :** Code unique donné à chaque objet AD, peut changer si l'objet est modifié.

**Verdict : ⚠️ Partiellement inexact**

Le SID **ne change pas si l'objet est modifié** (changement de nom, de mot de passe, etc.). Il change uniquement si l'objet **change de domaine**.

**Propriétés exactes du SID :**

- Unique au sein d'un **domaine**
- Format : `S-1-5-21-156063872-1535639461-3779917529-1134`
- **Peut changer** si l'objet est migré vers un autre domaine
- C'est le SID qui est utilisé par Windows pour vérifier les permissions sur les fichiers NTFS

---

### Q20 — GUID (Globally Unique Identifier)

**Ta réponse :** (non répondue)

**Verdict : 🔵 Non répondu**

|Caractéristique|SID|GUID|
|---|---|---|
|Unicité|Dans un domaine|Dans **toute la forêt**|
|Peut changer ?|✅ Oui (changement de domaine)|❌ **Jamais**|
|Usage|Permissions NTFS, ACL|Référencement interne AD (réplication, journaux)|
|Format|`S-1-5-21-...`|`3F2504E0-4F89-11D3-9A0C-0305E82C3301`|
|Taille|Variable|128 bits|

**Le GUID ne change jamais**, même si l'objet est renommé, déplacé ou migré. C'est l'identifiant absolu d'un objet dans AD.

---

## Partie 5 — Protocoles associés à AD

---

### Q21 — DNS indispensable à AD

**Ta réponse :** Il permet la résolution de noms du domaine. Sans lui, les communications dans le domaine bloquent.

**Verdict : ✅ Correct**

Précision importante : sans DNS, les machines ne peuvent pas **localiser les contrôleurs de domaine**. AD utilise des **enregistrements SRV** dans le DNS pour publier ses services (Kerberos, LDAP). Si le DNS est mal configuré : aucune authentification possible, aucune GPO appliquée, aucune jonction au domaine possible.

---

### Q22 — SNTP/NTP dans AD

**Ta réponse :** Maintient la synchronisation horaire. Sans synchronisation, des dysfonctionnements apparaissent sur les communications.

**Verdict : ✅ Très bien**

Précision critique à connaître pour l'examen : **Kerberos tolère un écart d'horloge de 5 minutes maximum** entre les machines. Au-delà, l'authentification échoue et les utilisateurs ne peuvent plus se connecter. C'est le rôle **PDC Emulator** (FSMO) qui est la source de temps de référence du domaine.

---

### Q23 — Kerberos et TGT

**Ta réponse :** (non répondue)

**Verdict : 🔵 Non répondu**

**Les 3 étapes de Kerberos :**

1. **AS-REQ / AS-REP** — L'utilisateur envoie son identifiant au KDC (Key Distribution Center, hébergé sur le DC). Le KDC vérifie les credentials et retourne un **TGT (Ticket Granting Ticket)** chiffré.
    
2. **TGS-REQ / TGS-REP** — L'utilisateur présente son TGT au KDC pour demander l'accès à une ressource (ex : un partage). Le KDC retourne un **ticket de service (ST)** pour cette ressource spécifique.
    
3. **AP-REQ** — L'utilisateur présente le ticket de service au serveur cible. Le serveur valide le ticket et autorise l'accès.
    

**TGT (Ticket Granting Ticket) :** C'est le "laissez-passer" initial délivré après la première authentification. Sa durée de vie par défaut est de **10 heures**. Il évite de retaper le mot de passe à chaque accès à une ressource.

---

### Q24 — LDAP vs LDAPS

**Ta réponse :** LDAPS est la version sécurisée et chiffrée de LDAP avec certificat.

**Verdict : ✅ Correct**

Complément :

|Protocole|Port|Chiffrement|Usage|
|---|---|---|---|
|**LDAP**|389|❌ Aucun — données en clair|À éviter en production|
|**LDAPS**|636|✅ TLS/SSL|Recommandé en production|

En production, on recommande LDAPS car LDAP transmet les identifiants et données de l'annuaire en clair sur le réseau, ce qui expose à des attaques de type "man-in-the-middle".

---

### Q25 — Authentification mutuelle dans Kerberos vs NTLM

**Ta réponse :** (non répondue)

**Verdict : 🔵 Non répondu**

**Authentification mutuelle :** Dans Kerberos, **les deux parties s'authentifient mutuellement**. Non seulement le serveur vérifie l'identité du client, mais le client peut aussi vérifier l'identité du serveur (grâce au ticket de service signé par le KDC).

**Avantage vs NTLM :**

|Critère|NTLM|Kerberos|
|---|---|---|
|Authentification mutuelle|❌ Non|✅ Oui|
|Résistance au pass-the-hash|❌ Vulnérable|✅ Plus résistant|
|Performance|Moins bonne (échanges multiples)|Meilleure (ticket réutilisable)|
|Usage actuel|Compatibilité legacy|Standard recommandé|

---

## Partie 6 — Gestion des utilisateurs et des groupes

---

### Q26 — Créer un utilisateur en PowerShell

**Ta réponse :** (non répondue)

**Verdict : 🔵 Non répondu**

```powershell
New-ADUser `
  -Name "Jean Dupont" `
  -GivenName "Jean" `
  -Surname "Dupont" `
  -SamAccountName "jdupont" `
  -UserPrincipalName "jdupont@entreprise.local" `
  -Path "OU=Informatique,OU=Utilisateurs,DC=entreprise,DC=local" `
  -AccountPassword (ConvertTo-SecureString "P@ssw0rd!" -AsPlainText -Force) `
  -Enabled $true
```

**Paramètres essentiels à connaître :**

- `-Name` : Nom affiché dans AD
- `-SamAccountName` : Identifiant de connexion (login)
- `-UserPrincipalName` : Identifiant UPN (format email)
- `-Path` : OU de destination
- `-AccountPassword` : Mot de passe (doit être un SecureString)
- `-Enabled $true` : Active le compte

---

### Q27 — Lister les utilisateurs d'une OU

**Ta réponse :** (non répondue)

**Verdict : 🔵 Non répondu**

```powershell
Get-ADUser -Filter * -SearchBase "OU=Informatique,OU=Utilisateurs,DC=entreprise,DC=local" `
  -Properties Enabled | Select-Object Name, Enabled
```

---

### Q28 — Méthode AGDLP

**Ta réponse :** (non répondue)

**Verdict : 🔵 Non répondu**

**AGDLP = Account → Global → Domain Local → Permission**

**Fonctionnement :**

1. Les **comptes utilisateurs (A)** sont placés dans des **groupes Globaux (G)** (ex : GG_Comptables)
2. Ces groupes Globaux sont ajoutés à des **groupes Domain Local (DL)** (ex : DL_Partage_Compta_Lecture)
3. Les groupes Domain Local reçoivent les **permissions (P)** sur les ressources

**Pourquoi c'est recommandé :**

- Séparation claire entre "qui sont les gens" (Global) et "qui accède à quoi" (Domain Local)
- Facilite la gestion dans les environnements multi-domaines
- Réduction du nombre de modifications nécessaires lors des changements d'organisation

---

### Q29 — Désactiver ou supprimer un compte ?

**Ta réponse :** Je le désactive avant de le supprimer pour archiver les données liées au compte.

**Verdict : ✅ Très bonne réponse**

La procédure recommandée complète :

1. **Désactiver** le compte immédiatement (l'utilisateur ne peut plus se connecter)
2. **Révoquer** les accès sensibles (VPN, boîte mail partagée, etc.)
3. **Transférer** les données/fichiers à un responsable
4. **Archiver** la boîte mail si nécessaire
5. **Attendre** la période légale (souvent 1 mois à 1 an selon les politiques internes)
6. **Supprimer** le compte définitivement

> ⚠️ Ne jamais supprimer immédiatement : le SID est perdu et les accès aux ressources partagées ne peuvent plus être audités.

---

### Q30 — LAPS

**Ta réponse :** (non répondue)

**Verdict : 🔵 Non répondu**

**LAPS (Local Administrator Password Solution)** résout le problème des **mots de passe administrateur locaux identiques** sur tous les postes.

**Problème sans LAPS :** Si tous les PC ont le même mot de passe administrateur local (`Admin123` par exemple), un attaquant qui compromet un seul poste peut se connecter en admin local sur tous les postes du parc.

**Fonctionnement de LAPS :**

1. LAPS génère automatiquement un **mot de passe aléatoire unique** pour chaque poste
2. Ce mot de passe est **stocké dans l'attribut AD** de l'objet ordinateur (chiffré)
3. Il est **renouvelé automatiquement** selon une politique définie
4. Seuls les administrateurs autorisés peuvent le consulter via AD ou PowerShell

---

## Partie 7 — Les GPO

---

### Q31 — Qu'est-ce qu'une GPO ?

**Ta réponse :** Une GPO est une règle appliquée sur le domaine qui autorise ou refuse des accès.

**Verdict : ⚠️ Incomplet**

Une **GPO (Group Policy Object)** est bien plus large. C'est un objet AD qui contient un ensemble de **politiques de configuration** appliquées aux ordinateurs et utilisateurs du domaine.

**4 domaines de configuration :**

1. **Politiques de sécurité** (longueur mot de passe, verrouillage de compte)
2. **Configuration logicielle** (déploiement d'applications)
3. **Configuration de l'environnement utilisateur** (fond d'écran, accès au panneau de configuration)
4. **Scripts** (scripts de connexion/déconnexion, démarrage/arrêt)

---

### Q32 — LSDOU et ordre d'application

**Ta réponse :** (non répondue)

**Verdict : 🔵 Non répondu**

**LSDOU = Local → Site → Domain → OU**

Ordre d'application (du moins prioritaire au plus prioritaire) :

1. **Local** — stratégies locales de la machine
2. **Site** — GPO du site AD
3. **Domain** — GPO du domaine
4. **OU** — GPO des unités d'organisation (de la plus haute à la plus spécifique)

**La priorité la plus élevée** est la GPO de l'**OU la plus proche de l'objet** (la plus spécifique). Exception : une GPO avec l'état **Enforced** a une priorité absolue, quel que soit son niveau.

---

### Q33 — Blocage d'héritage

**Ta réponse :** Le blocage d'héritage fait que la règle ne se répercute pas sur les OU en dessous.

**Verdict : ❌ Sens inversé**

C'est l'inverse : le **blocage d'héritage empêche les GPO des niveaux SUPÉRIEURS de descendre** dans cette OU.

**Exemple :** Une GPO appliquée au domaine ne s'appliquera pas à une OU où l'héritage est bloqué.

**Quand l'utiliser :** Pour une OU qui a des besoins très spécifiques et ne doit pas recevoir les politiques générales du domaine (ex : OU des administrateurs).

**Comment contourner le blocage :** En configurant la GPO supérieure en mode **Enforced** — elle s'applique alors même si l'héritage est bloqué.

---

### Q34 — LIFO et priorité des GPO

**Ta réponse :** LIFO = Last In First Out, la dernière règle mise en place part en premier. Celle avec le chiffre le plus haut.

**Verdict : ⚠️ Partiellement inexact**

La logique LIFO dans les GPO est **inversée dans la numérotation** :

```
GPO 1 (liée en premier)  → Ordre de lien : 3  → Traitée en dernier
GPO 3 (liée en dernier)  → Ordre de lien : 1  → Traitée en premier (PRIORITAIRE)
```

**La GPO prioritaire est celle avec l'ordre de lien numéro 1** (pas le numéro le plus haut). Le numéro 1 est le lien le plus récent, donc le plus prioritaire en cas de conflit.

---

### Q35 — Conditions pour qu'une GPO s'applique

**Ta réponse :** L'associer à l'objet et au bon emplacement. Supprimer "Authenticated Users" et la cibler sur le bon groupe.

**Verdict : ✅ Correct mais imprécis sur les droits**

Les **deux conditions obligatoires** sont :

1. **Lien actif** (Link Enabled) : la GPO est bien liée à la cible (site, domaine, OU)
2. **Droits suffisants** : l'objet doit avoir les permissions **Read** ET **Apply Group Policy** sur la GPO

**Pour restreindre à un groupe spécifique :**

- Retirer "Authenticated Users" du filtrage de sécurité
- Ajouter le groupe cible avec les droits **Read + Apply Group Policy**

---

### Q36 — Commandes GPO

**Ta réponse :** `gpupdate /force` et `gpresult`

**Verdict : ✅ Correct**

Complément :

```cmd
gpupdate /force          # Force la mise à jour immédiate des GPO
gpresult /r              # Affiche les GPO appliquées sur la machine locale
gpresult /h rapport.html # Génère un rapport HTML complet des GPO
```

---

### Q37 — gpedit.msc vs gpmc.msc

**Ta réponse :** (non répondue)

**Verdict : 🔵 Non répondu**

|Console|Nom complet|Contexte d'utilisation|
|---|---|---|
|`gpedit.msc`|Éditeur de stratégie de groupe locale|Machine standalone ou Workgroup — gère uniquement les stratégies **locales**|
|`gpmc.msc`|Console de gestion des stratégies de groupe|**Serveur AD uniquement** — gère toutes les GPO du domaine, les liaisons, les filtres|

---

## Partie 8 — Fonctionnalités avancées d'AD

---

### Q38 — Niveau fonctionnel

**Ta réponse :** (non répondue)

**Verdict : 🔵 Non répondu**

Le **niveau fonctionnel** détermine les fonctionnalités AD disponibles. Il est limité par la version du Windows Server le plus ancien présent parmi les DC.

**Règle :** Niveau fonctionnel = version du DC le plus ancien.

**Exemple de ta question :** Si le domaine contient des DC sous Windows Server 2012 R2 **et** 2019, le niveau fonctionnel reste bloqué à **Windows Server 2012 R2**. Il faut d'abord migrer ou retirer les DC 2012 R2 pour pouvoir monter le niveau fonctionnel.

> ⚠️ La montée de niveau fonctionnel est **irréversible**.

---

### Q39 — Les 5 rôles FSMO

**Ta réponse :** (non répondue)

**Verdict : 🔵 Non répondu**

**Au niveau de la Forêt (2 rôles) :**

|Rôle|Fonction|
|---|---|
|**Schema Master**|Gère les modifications du schéma AD (structure des objets)|
|**Domain Naming Master**|Gère l'ajout/suppression de domaines dans la forêt|

**Au niveau du Domaine (3 rôles) :**

|Rôle|Fonction|
|---|---|
|**RID Master**|Attribue les blocs de RID (Relative Identifier) pour créer des SID uniques|
|**Infrastructure Master**|Gère les références d'objets entre domaines|
|**PDC Emulator**|⚠️ **LE PLUS CRITIQUE**|

**Pourquoi le PDC Emulator est le plus critique :**

- Gère la **synchronisation du temps** (NTP) pour tout le domaine
- Gère les **verrouillages de comptes** en temps réel
- Si le temps dépasse 5 min de décalage → **Kerberos cesse de fonctionner** → plus aucune authentification possible

---

### Q40 — Réplication entre DC

**Ta réponse :** (non répondue)

**Verdict : 🔵 Non répondu**

**Processus de réplication :**

1. DC1 modifie un objet dans sa base AD
2. DC1 consulte le **KCC** pour identifier les autres DC
3. DC1 interroge le **DNS** pour localiser DC2
4. DC1 **notifie** DC2 qu'il y a des modifications
5. DC2 **demande** les détails des modifications à DC1
6. DC1 **transfère** les changements
7. DC2 met à jour sa base de données

**Le composant qui orchestre la réplication :** le **KCC (Knowledge Consistency Checker)**. Il est présent sur chaque DC et génère automatiquement la topologie de réplication.

---

### Q41 — Réplication intra-site vs inter-site

**Ta réponse :** (non répondue)

**Verdict : 🔵 Non répondu**

|Critère|Intra-site|Inter-site|
|---|---|---|
|Définition|Entre DC du même site AD|Entre DC de sites AD différents|
|Intervalle par défaut|5 minutes|180 minutes|
|Connexion réseau|LAN rapide|WAN (lien potentiellement lent)|
|Compression|Non|Oui (pour économiser la bande passante)|

**Recommandation :** Un intervalle de **10 à 20 minutes** offre le meilleur équilibre entre cohérence des données et charge réseau.

---

### Q42 — RODC

**Ta réponse :** (non répondue)

**Verdict : 🔵 Non répondu**

Un **RODC (Read-Only Domain Controller)** est un DC qui possède une copie **en lecture seule** de la base AD. Il ne peut pas écrire dans l'annuaire.

**Contextes d'utilisation recommandés :**

- **Sites distants** avec connexion WAN peu fiable (agence, filiale)
- **Zones à faible sécurité physique** (le serveur n'est pas dans une salle sécurisée)
- Si le RODC est compromis, l'attaquant ne peut pas modifier l'annuaire ni accéder à tous les mots de passe (seuls les comptes autorisés sont mis en cache)

---

## Partie 9 — Sécurité Windows et AD

---

### Q43 — Principe de moindre privilège

**Ta réponse :** Ne donner aucun droit par défaut aux utilisateurs, les droits viennent des groupes. Si un utilisateur est compromis, ses droits sont limités.

**Verdict : ✅ Excellente réponse**

C'est exactement ça. Le principe de moindre privilège (Least Privilege) = **chaque utilisateur ou système ne dispose que des permissions strictement nécessaires à ses tâches**. En pratique dans AD : droits via les groupes, jamais en direct sur l'utilisateur, et comptes d'administration séparés des comptes utilisateurs.

---

### Q44 — Tiering Model

**Ta réponse :** T0 accède aux serveurs critiques, T2 aux postes. T1 intermédiaire.

**Verdict : ✅ Correct dans les grandes lignes**

**Les 3 niveaux :**

|Niveau|Ressources protégées|Exemples de comptes|
|---|---|---|
|**Tier 0**|Infrastructure critique AD (DC, PKI, ADFS)|Compte administrateur du domaine|
|**Tier 1**|Serveurs applicatifs et de données|Compte admin serveur (ERP, fichiers)|
|**Tier 2**|Postes de travail et appareils utilisateurs|Compte helpdesk, admin poste|

**Règle d'or :** Un compte Tier 0 ne doit **jamais** se connecter sur une machine Tier 1 ou Tier 2 (risque d'extraction de credentials).

---

### Q45 — JIT et JEA

**Ta réponse :** (non répondue)

**Verdict : 🔵 Non répondu**

**JIT (Just-In-Time) :** L'administrateur reçoit des droits élevés **uniquement pour la durée nécessaire** à son intervention, puis les droits sont automatiquement révoqués. Cela limite la fenêtre d'exposition des comptes privilégiés.

**JEA (Just Enough Administration) :** L'administrateur dispose uniquement des **commandes PowerShell nécessaires** à sa tâche. Un technicien helpdesk peut réinitialiser des mots de passe sans avoir accès au reste de l'AD.

**Bénéfice commun :** Réduction drastique de la surface d'attaque. Même si un compte privilégié est compromis, l'attaquant n'a accès qu'à un périmètre limité et temporaire.

---

### Q46 — Event ID à surveiller

**Ta réponse :** Pings incessants, tentatives de connexion en boucle.

**Verdict : ❌ Hors sujet**

Les Event ID sont des **identifiants numériques dans les journaux d'événements Windows** (Observateur d'événements), pas des comportements réseau. Voici les 5 essentiels à connaître :

|Event ID|Description|Criticité|
|---|---|---|
|**4625**|Échec de connexion (mauvais mot de passe)|⚠️ Haute — tentative de brute force|
|**4624**|Connexion réussie|Info — à surveiller pour les comptes sensibles|
|**4720**|Création d'un compte utilisateur|⚠️ Haute — création non autorisée ?|
|**4728/4732**|Ajout à un groupe privilégié (Domain Admins, etc.)|🚨 Critique|
|**4740**|Compte utilisateur verrouillé|⚠️ Haute — tentative de bruteforce ?|
|**4672**|Connexion avec privilèges spéciaux (admin)|🚨 À auditer systématiquement|

---

### Q47 — WEF (Windows Event Forwarding)

**Ta réponse :** (non répondue)

**Verdict : 🔵 Non répondu**

**WEF** est un mécanisme Windows qui permet de **centraliser les journaux d'événements** de plusieurs machines sur un serveur collecteur (Windows Event Collector — WEC).

**Protocole utilisé :** WS-Management (WS-Man), basé sur **HTTP (port 5985)** ou **HTTPS (port 5986)**.

**Utilité :** Permet d'avoir tous les logs de sécurité au même endroit pour l'analyse (SIEM, supervision), sans avoir à se connecter sur chaque machine individuellement.

---

## Partie 10 — Cas pratiques et scénarios

---

### Q48 — Diagnostic de connexion impossible au domaine

**Ta réponse :** Vérifier le câble réseau, ping le serveur, vérifier les identifiants.

**Verdict : ⚠️ Bonne base mais trop rapide**

**Démarche complète en 5+ étapes :**

1. **Vérification physique** — câble réseau branché, voyant réseau actif, connexion Wi-Fi établie ✅ (tu l'as)
2. **Ping du DC** — `ping nomduserveur` pour tester la connectivité réseau ✅ (tu l'as)
3. **Vérification DNS** — `nslookup entreprise.local` pour s'assurer que le DNS répond et résout le domaine
4. **Vérification de l'appartenance au domaine** — la machine est-elle bien jointe au domaine ? (`Système > Paramètres avancés`)
5. **Vérification du compte dans AD** — le compte est-il activé ? N'est-il pas expiré ? (`dsa.msc`)
6. **Vérification des credentials** — l'utilisateur entre-t-il bien `DOMAINE\login` ou `login@domaine.local` ?
7. **Vérification de la synchronisation horaire** — `w32tm /query /status` (écart > 5 min → Kerberos bloque)

---

### Q49 — Politique de mot de passe différente par groupe

**Ta réponse :** (non répondue)

**Verdict : 🔵 Non répondu**

La fonctionnalité utilisée est la **PSO (Password Settings Object)** ou **stratégie de mot de passe affinée (Fine-Grained Password Policy)**.

**Fonctionnement :**

- Disponible depuis Windows Server 2008
- Permet de définir des politiques de mot de passe **différentes** pour des utilisateurs ou groupes spécifiques dans le **même domaine**
- S'applique directement à des **utilisateurs ou groupes** (pas à des OU)
- Configurée via `ADAC (Active Directory Administrative Center)` → `Conteneur System > Password Settings Container`

**Exemple :** PSO "Admins" avec 20 caractères minimum, complexité maximale, appliquée au groupe "Domain Admins".

---

### Q50 — Fusion de domaines / Relations d'approbation

**Ta réponse :** (non répondue)

**Verdict : 🔵 Non répondu**

Le mécanisme utilisé est une **relation d'approbation (Trust)** entre les deux domaines.

**Fonctionnement :**

- On crée une **approbation externe** entre `entreprise.local` et `filiale.com`
- L'approbation peut être **unidirectionnelle** (filiale accède à entreprise) ou **bidirectionnelle**
- Elle peut être **transitive** (se propage aux domaines enfants) ou **non transitive**

**Types d'approbation :**

- **Approbation de forêt** : entre deux forêts entières (la plus large)
- **Approbation externe** : entre deux domaines de forêts différentes
- **Approbation de raccourci** : entre deux domaines d'une même forêt pour accélérer l'authentification

---

### Q51 — Diagnostic de réplication en échec

**Ta réponse :** (non répondue)

**Verdict : 🔵 Non répondu**

**Causes possibles d'un échec de réplication :**

- Problème réseau entre les deux DC (VLAN, firewall)
- Service AD DS arrêté sur un DC
- Désynchronisation horaire > 5 minutes
- Problème DNS (DC ne trouve pas l'autre DC)
- Corruption de la base AD

**Commandes de diagnostic :**

```cmd
repadmin /showrepl          # État de la réplication (erreurs, délais)
repadmin /replsummary       # Résumé rapide de la santé de la réplication
dcdiag /test:replications   # Diagnostic complet des DC
netdiag                     # Diagnostic réseau général
```

---

### Q52 — Déployer une application via GPO

**Ta réponse :** (non répondue)

**Verdict : 🔵 Non répondu**

**Procédure :**

1. **Prérequis :** L'application doit être au format **MSI** (Windows Installer)
2. **Partage réseau :** Placer le fichier MSI dans un **partage réseau accessible** par tous les postes du domaine (ex : `\\srv-ad\Deploy$\AppName\app.msi`)
3. **Créer une nouvelle GPO** liée à l'OU cible
4. Dans la GPO : `Configuration ordinateur > Paramètres logiciel > Installation de logiciels`
5. Faire un clic droit → Nouveau → Package → sélectionner le MSI via le chemin UNC (`\\srv\Deploy$\...`)
6. Choisir le mode **Affecté** (installé automatiquement au démarrage) ou **Publié** (disponible à la demande)
7. **Forcer la mise à jour** avec `gpupdate /force` ou attendre le prochain démarrage des postes

---

## Récapitulatif des performances

|Partie|Questions|Correctes ✅|Incomplètes ⚠️|Inexactes ❌|Non répondues 🔵|
|---|---|---|---|---|---|
|Windows Server|Q1–Q5|Q1, Q5|Q3|—|Q2, Q4|
|Fondamentaux AD|Q6–Q10|Q7 (partiel)|Q6, Q9|—|Q8, Q10 (partiel)|
|Structure AD|Q11–Q15|Q11, Q12, Q15|Q13|—|Q14|
|Objets AD|Q16–Q20|—|Q19|Q16|Q17, Q18, Q20|
|Protocoles|Q21–Q25|Q21, Q22, Q24|—|—|Q23, Q25|
|PowerShell / Gestion|Q26–Q30|Q29|—|—|Q26, Q27, Q28, Q30|
|GPO|Q31–Q37|Q35, Q36|Q31|Q33, Q34|Q32, Q37|
|Fonctionnalités avancées|Q38–Q42|—|—|—|Q38, Q39, Q40, Q41, Q42|
|Sécurité|Q43–Q47|Q43, Q44|—|Q46|Q45, Q47|
|Cas pratiques|Q48–Q52|—|Q48|—|Q49, Q50, Q51, Q52|

---

## Points prioritaires à travailler

1. **Kerberos** (Q23, Q25) — mécanisme fondamental pour l'authentification AD
2. **FSMO** (Q39) — les 5 rôles, leur niveau et leur criticité
3. **Réplication** (Q40, Q41) — processus et KCC
4. **Commandes PowerShell AD** (Q26, Q27) — à pratiquer en lab
5. **GPO avancées** (Q32, Q33, Q34, Q37) — LSDOU et LIFO à bien mémoriser
6. **Sécurité AD** (Q45, Q46, Q47) — JIT/JEA et Event ID à connaître
7. **AGDLP** (Q28) — méthode standard à maîtriser

---

_Correction réalisée dans le cadre de la préparation au titre TSSR — CP2_