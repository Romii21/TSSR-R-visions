# Correction – CP2 : Exploiter des serveurs Windows et un domaine Active Directory

**Niveau** : TSSR  
**Date de correction** : Mars 2026

---

## Récapitulatif des résultats

|Question|Résultat|Thème|
|---|---|---|
|Q1|❌ Non répondu|Définition AD, LDAP, SAM|
|Q2|✅ Partiel|Workgroup vs Domaine|
|Q3|⚠️ Incomplet|Hiérarchie logique AD|
|Q4|✅ Très bien|OU et bonnes pratiques|
|Q5|❌ Incomplet|Rôles FSMO|
|Q6|⚠️ Partiel|Réplication AD|
|Q7|❌ Non répondu|RODC|
|Q8|❌ Non répondu|Niveau fonctionnel|
|Q9|✅ Bonne réponse|GPO définition et usages|
|Q10|⚠️ Confusion|LSDOU vs LIFO|
|Q11|⚠️ Incomplet|Conditions d'application GPO|
|Q12|✅ Correct|Commandes GPO|
|Q13|❌ Non répondu|Méthode AGDLP|
|Q14|✅ Bonne réponse|LAPS|
|Q15|⚠️ Incomplet|Bonnes pratiques sécurité|

---

## Partie 1 – Fondamentaux d'Active Directory

### Question 1 — ❌ Non répondu

Tu n'as pas répondu à cette question.

**Ce qu'il fallait savoir :**

**Active Directory** est l'implémentation Microsoft d'un service d'annuaire basé sur le protocole **LDAP (Lightweight Directory Access Protocol)**. Ses objectifs principaux sont :

- Centraliser l'identification et l'authentification des utilisateurs
- Gérer les politiques de sécurité sur l'ensemble du réseau
- Répertorier et faciliter la gestion des ressources réseau

Avant AD, Microsoft utilisait **SAM (Security Account Manager)**, une base de comptes **locale** à chaque machine. SAM était limité à une gestion machine par machine, sans aucune centralisation, et devenait ingérable au-delà d'une vingtaine de machines. Active Directory a remplacé SAM en permettant une gestion centralisée à l'échelle d'un domaine entier via une base de données hiérarchique LDAP.

---

### Question 2 — ✅ Partiellement juste

Tu as bien identifié les grandes idées : absence de centralisation en Workgroup, gestion complexe dès que le parc grossit.

**Ce qui est à corriger ou compléter :**

Le **Workgroup** n'est pas un "domaine local". C'est un groupe de travail **pair-à-pair** où chaque machine gère ses propres comptes localement, sans serveur central ni DC. Un utilisateur qui veut accéder à une autre machine doit avoir un compte créé **sur cette machine spécifiquement**.

La limite indiquée dans les cours est **~35-40 machines** avant que le Workgroup devienne véritablement ingérable (tu avais dit 10).

|Caractéristique|Workgroup|Domaine|
|---|---|---|
|Connexion|Compte local requis sur chaque machine|Compte domaine utilisable partout|
|Centralisation|Aucune|Complète via les DC|
|Limite machines|~35-40|Illimitée|
|Réseau|Même réseau local obligatoire|Réseaux différents possibles|

---

### Question 3 — ⚠️ Incomplet et sens partiellement inversé

Tu as évoqué la notion de DN et de hiérarchie, mais tu es parti du domaine vers l'objet alors que la question demandait du plus petit au plus grand.

**La hiérarchie logique complète, du plus petit au plus grand :**

|Niveau|Définition|
|---|---|
|**Objet**|Élément de base : utilisateur, ordinateur, groupe, imprimante...|
|**OU (Organizational Unit)**|Conteneur logique pour organiser les objets|
|**Domaine**|Unité administrative et de sécurité, socle de l'AD|
|**Arbre**|Ensemble de domaines avec un espace de noms contigu (`paris.lab.lan`, `lyon.lab.lan`)|
|**Forêt**|Ensemble d'arbres reliés par des relations d'approbation — niveau le plus élevé|

Le **DN (Distinguished Name)** que tu as cité est correct : `CN=Utilisateur, OU=RH, DC=lab, DC=lan`. Bonne connaissance sur ce point précis.

---

### Question 4 — ✅ Très bonne réponse

C'est l'une des meilleures réponses du questionnaire. Tu as bien compris les trois fonctions d'une OU : organiser, déléguer, appliquer des GPO. Tu as aussi correctement expliqué pourquoi organiser par **services/fonctions** plutôt que par géographie (les GPO suivent les fonctions métier, pas les localisations), et pourquoi séparer les **utilisateurs et les ordinateurs** (pour ne pas leur appliquer les mêmes règles). Il n'y a rien à corriger ici.

---

## Partie 2 – Infrastructure et rôles

### Question 5 — ❌ Incomplet + erreur de nom

Tu avais la bonne structure (2 rôles forêt / 3 rôles domaine) mais tu t'es arrêté. De plus, tu as écrit **"RIP"** au lieu de **"RID"** — attention à la confusion avec le protocole de routage RIP.

**Les 5 rôles FSMO complets :**

**Au niveau de la Forêt (2 rôles) :**

|Rôle|Fonction|
|---|---|
|**Schema Master** (Maître de Schéma)|Gère les modifications du schéma AD (structure des classes d'objets et de leurs attributs). Toute modification du schéma passe obligatoirement par lui.|
|**Domain Naming Master** (Maître d'Attribution de Noms)|Gère l'ajout et la suppression de domaines dans la forêt.|

**Au niveau du Domaine (3 rôles) :**

|Rôle|Fonction|
|---|---|
|**RID Master** (Maître RID)|Distribue des blocs de RID (Relative Identifier) aux autres DC pour qu'ils puissent créer des objets avec des SID uniques.|
|**Infrastructure Master** (Maître d'Infrastructure)|Gère les références inter-domaines (quand un objet d'un domaine est membre d'un groupe d'un autre domaine).|
|**PDC Emulator** ⚠️ **LE PLUS CRITIQUE**|Synchronise l'heure sur tout le domaine, gère le verrouillage des comptes, assure la compatibilité avec les anciens clients.|

**Pourquoi le PDC Emulator est le plus critique :**  
Il gère la synchronisation du temps via SNTP. Kerberos (le protocole d'authentification AD) exige que les horloges des machines ne décalent pas de plus de 5 minutes. Si le PDC Emulator tombe, la synchronisation s'arrête → Kerberos échoue → **verrouillage complet de l'AD**.

**Bonne pratique :** ne jamais concentrer tous les rôles sur un seul DC. Idéalement 1 rôle par DC (5 DC), au minimum 2 DC avec séparation forêt/domaine.

---

### Question 6 — ⚠️ Partiellement juste, erreur sur l'intervalle

Tu as bien compris le but de la réplication (synchronisation entre DC pour la redondance) et le bon intervalle par défaut intra-site (5 min).

**Ce qui est incorrect :**

Tu indiques que la bonne pratique est **30 minutes**. C'est faux — au-delà de 30-45 min, le cours précise que c'est considéré comme un **problème**. La recommandation est **10 à 20 minutes**.

**Ce qui manque :**

- Le composant responsable : le **KCC (Knowledge Consistency Checker)**, présent sur tous les DC. Il génère automatiquement la topologie de réplication et se met à jour lors de changements (ajout/suppression de DC).
- ⚠️ **La réplication n'est PAS une sauvegarde**, c'est de la synchronisation. Si tu supprimes un objet par erreur, la suppression se réplique aussi sur tous les DC.

**Intervalles par défaut :**

|Type|Intervalle par défaut|Recommandation|
|---|---|---|
|Intra-site|5 minutes|❌ Trop fréquent|
|Inter-site|180 minutes|❌ Trop espacé|
|✅ Recommandé|—|**10 à 20 minutes**|

**Ce qui est répliqué :** base AD, GPO, scripts, zones DNS.

---

### Question 7 — ❌ Non répondu

**Ce qu'il fallait savoir :**

Un **RODC (Read-Only Domain Controller)** est un contrôleur de domaine dont la base AD est en **lecture seule**. Aucune modification ne peut être écrite directement dessus.

**Contexte d'utilisation :** sites distants ou peu sécurisés (agences, filiales, datacenters externalisés) où la sécurité physique du serveur ne peut pas être garantie.

**Limitations par rapport à un DC classique :**

- Ne peut pas écrire de modifications dans l'AD
- Réplication dans un seul sens : il reçoit les mises à jour du DC principal, il n'en envoie pas
- En cas de compromission physique, les dégâts sont limités car le serveur ne peut pas altérer l'annuaire

---

### Question 8 — ❌ Non répondu

**Ce qu'il fallait savoir :**

Le **niveau fonctionnel** détermine les fonctionnalités disponibles dans un domaine ou une forêt. Il est automatiquement défini par la version du Windows Server **la plus ancienne** parmi les DC présents.

**Règle clé :** si tu as 3 DC sous Windows Server 2019 et 2 sous 2012 R2, le niveau fonctionnel reste bloqué à **2012 R2** jusqu'à ce que tous les DC 2012 R2 soient retirés.

**Exemple :**

```
5 DC sous Windows Server 2012 R2
→ Niveau fonctionnel = Windows Server 2012 R2

3 DC sous 2012 R2 + 2 DC sous 2019
→ Niveau fonctionnel = Windows Server 2012 R2 (bloqué par le plus ancien)
```

⚠️ **Impossible de revenir en arrière** après une montée de niveau. C'est une décision **irréversible** — il faut être certain avant de procéder.

---

## Partie 3 – GPO (Group Policy Objects)

### Question 9 — ✅ Bonne réponse

Tu as bien cerné la définition et donné de bons exemples concrets : droits de connexion, horaires, mots de passe, fonds d'écran, restriction d'accès aux logiciels/CMD, mappage de lecteurs réseau. C'est exactement ce qui est attendu.

**Petite précision :** une GPO s'applique via les **OU** (qui contiennent les utilisateurs, ordinateurs ou groupes), pas directement sur des "groupes" AD au sens strict. Mais la logique que tu décris est correcte.

---

### Question 10 — ⚠️ Partiellement juste, confusion entre LSDOU et LIFO

Tu as mélangé deux concepts distincts qu'il faut bien dissocier.

**LSDOU** est l'**ordre d'application** des GPO entre les différents niveaux, du moins prioritaire au plus prioritaire :

```
Local → Site → Domain → OU
(faible priorité)          (forte priorité)
```

Une GPO liée à l'OU écrase une GPO liée au domaine si elles ont des paramètres contradictoires.

**LIFO (Last In, First Out)** est un principe **interne à une même OU** : si plusieurs GPO sont liées à la même OU, la **dernière liée** est prioritaire.

Ce sont deux mécanismes complémentaires, pas le même. LSDOU = ordre entre niveaux. LIFO = ordre entre GPO au même niveau.

**L'option Enforced :** tu as bien dit qu'elle rend la GPO prioritaire sur toutes les autres. C'est exact — elle ignore même le blocage d'héritage. C'est la **priorité absolue**.

---

### Question 11 — ⚠️ Incomplet

Tu as mentionné la liaison à une OU, ce qui est juste, mais les deux conditions exactes selon le cours sont :

1. ✅ La GPO est **liée** à un site, domaine ou OU (lien actif)
2. ✅ L'objet cible possède les droits **Read (Lecture)** + **Apply Group Policy** sur la GPO

**Pour le filtrage de sécurité :**  
Par défaut, le groupe **Authenticated Users** (tous les utilisateurs et ordinateurs authentifiés) possède ces droits. Pour restreindre une GPO à un groupe spécifique :

1. ❌ Retirer le droit "Apply Group Policy" du groupe **Authenticated Users**
2. ➕ Ajouter le groupe de sécurité AD souhaité
3. ✅ Lui accorder les droits **Lecture + Apply Group Policy**

**Résultat :** seuls les membres du groupe spécifié reçoivent la GPO.

---

### Question 12 — ✅ Correct avec une petite erreur de syntaxe

Les bonnes commandes ont été citées et leur rôle correctement expliqué.

**Petite erreur :** tu as écrit `gpresult -r` avec un tiret `-`. Sous Windows, la syntaxe correcte est avec un slash : `gpresult /r`.

**Commandes à connaître :**

|Commande|Rôle|
|---|---|
|`gpupdate /force`|Force la mise à jour immédiate des GPO ordinateur et utilisateur|
|`gpupdate /target:computer /force`|Force uniquement les stratégies ordinateur|
|`gpupdate /target:user /force`|Force uniquement les stratégies utilisateur|
|`gpresult /r`|Affiche un résumé des GPO appliquées|
|`gpresult /h rapport.html`|Génère un rapport HTML détaillé|
|`gpmc.msc`|Console de gestion des GPO (côté serveur)|
|`gpedit.msc`|Éditeur de stratégies locales (côté client)|

---

## Partie 4 – Sécurité et bonnes pratiques

### Question 13 — ❌ Non répondu

C'est un point fondamental à maîtriser absolument.

**AGDLP = Account → Global Group → Domain Local Group → Permission**

**Principe de base :** on ne donne **jamais** de droits directement à un utilisateur.

**Flux recommandé :**

```
Utilisateurs (A) 
→ appartiennent à des Groupes Métiers/G (ex: Grp_Compta)
→ qui appartiennent à des Groupes de Droits/DL (ex: Grp_Compta_RW)
→ qui ont des Permissions (P) sur les ressources (dossiers NTFS, etc.)
```

**Exemple concret :**

Bob et Alice sont comptables et doivent accéder au dossier "Compta" en lecture/écriture.

- ❌ **Mauvaise pratique :** donner directement l'accès NTFS à Bob et Alice
- ✅ **Bonne pratique AGDLP :**
    1. Créer le groupe métier `Grp_Compta` contenant Bob et Alice
    2. Créer le groupe de droits `Grp_Compta_RW`
    3. Donner les permissions RW sur le dossier à `Grp_Compta_RW`
    4. Ajouter `Grp_Compta` dans `Grp_Compta_RW`

**Avantage :** quand Bob quitte l'entreprise, on le retire de `Grp_Compta`. Il perd tous ses accès instantanément, sans toucher aux permissions NTFS. Gestion simplifiée et traçabilité complète.

---

### Question 14 — ✅ Bonne réponse

Tu as bien compris l'essentiel : LAPS génère des mots de passe aléatoires pour les comptes administrateurs locaux de chaque machine, stockés de façon sécurisée dans l'AD, ce qui évite un mot de passe commun à toutes les machines.

**Complément utile :**  
Le problème précis que LAPS résout est le risque de **mouvement latéral** (lateral movement) : si un attaquant compromet une machine et récupère le hash du mot de passe administrateur local, sans LAPS ce mot de passe est **identique sur toutes les machines** du domaine. Il peut donc se propager facilement à l'ensemble du parc. Avec LAPS, chaque machine a un mot de passe unique — la compromission reste isolée.

---

### Question 15 — ⚠️ Incomplet — 3 bonnes pratiques sur 5 minimum demandées

Tes 3 points sont pertinents (désactivation du compte admin par défaut, nomenclature non explicite, redondance des DC), mais la question en demandait au minimum 5.

**Bonnes pratiques complètes à connaître :**

|Bonne pratique|Justification|
|---|---|
|Désactiver le compte Administrateur par défaut|Compte à privilèges élevés connu de tous, cible privilégiée des attaques|
|Utiliser des **comptes d'administration séparés**|Un admin ne doit pas utiliser son compte admin pour ses tâches quotidiennes|
|Déployer **LAPS** sur tous les postes|Évite un mot de passe local commun à toutes les machines|
|Placer les **DC dans des VLANs dédiés**|Isole le trafic d'annuaire et protège les DC des attaques latérales|
|**Séparer les rôles FSMO** sur plusieurs DC|Évite un SPOF (Single Point of Failure) sur l'infrastructure AD|
|Appliquer **AGDLP** pour toutes les permissions|Gestion propre et traçable des droits d'accès|
|**Ne jamais redémarrer automatiquement les DC**|Risque de coupure de service d'annuaire|
|**Politique de mots de passe conforme ANSSI**|Complexité, durée de vie et historique des mots de passe|
|**Monitoring des logs et audits réguliers**|Détection des anomalies et activités suspectes|
|**Stratégie de sauvegarde 3-2-1**|3 copies, 2 supports différents, 1 copie hors site|

---

## Points forts à conserver

- **Q4** — Excellente maîtrise des OU et des bonnes pratiques d'organisation
- **Q9** — Bonne compréhension des GPO et de leurs usages concrets
- **Q14** — LAPS bien compris et bien expliqué
- **Q12** — Commandes GPO maîtrisées (juste la syntaxe `/` à corriger)

## Points à réviser en priorité

- **FSMO (Q5)** — Les 5 rôles, leur portée et le rôle le plus critique
- **RODC (Q7)** — Définition, contexte et limitations
- **Niveau fonctionnel (Q8)** — Règle du DC le plus ancien, irréversibilité
- **AGDLP (Q13)** — Méthode incontournable pour la gestion des permissions
- **Réplication (Q6)** — Intervalle recommandé (10-20 min), rôle du KCC