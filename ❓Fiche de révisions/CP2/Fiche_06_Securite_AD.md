# Fiche 06 — Sécurité Active Directory

> **Formation TSSR — CP2**
> Révision : Moindre privilège, Tiering Model, LAPS, JIT/JEA, Event ID, WEF, PSO

---

## 1. Principe de Moindre Privilège

### Définition

**Chaque utilisateur ou système ne dispose que des permissions strictement nécessaires à ses tâches.** Rien de plus.

### Application concrète dans AD

- Les droits sont attribués **via des groupes**, jamais directement sur un compte utilisateur
- Les comptes d'administration sont **séparés** des comptes utilisateurs du quotidien
- Un administrateur a deux comptes : un compte normal pour le travail quotidien, un compte admin pour les tâches d'administration
- Les droits sont révoqués dès qu'un utilisateur change de poste ou quitte l'entreprise

**Avantages :**
- Si un compte est compromis, le rayon d'action de l'attaquant est limité
- Facilite l'audit (on sait exactement qui a accès à quoi)
- Réduit les erreurs accidentelles (un utilisateur ne peut pas supprimer ce qu'il ne devrait pas toucher)

---

## 2. Tiering Model (Modèle en niveaux)

### Principe

Le Tiering Model divise l'administration en **3 niveaux d'isolation**, chacun protégeant des ressources spécifiques. Un compte d'un niveau ne doit **jamais** se connecter sur une machine d'un niveau inférieur.

```
┌─────────────────────────────────────────────────────┐
│  TIER 0 — Infrastructure critique AD                │
│  DC, PKI, AD FS, systèmes de sauvegarde AD          │
│  Comptes : Administrateurs du domaine               │
├─────────────────────────────────────────────────────┤
│  TIER 1 — Serveurs applicatifs et de données        │
│  Serveurs ERP, fichiers, bases de données, messagerie│
│  Comptes : Admins serveurs applicatifs              │
├─────────────────────────────────────────────────────┤
│  TIER 2 — Postes de travail et appareils utilisateurs│
│  PC, laptops, smartphones                           │
│  Comptes : Helpdesk, admins postes                  │
└─────────────────────────────────────────────────────┘
```

### Règle d'or du Tiering

> **Un compte Tier 0 ne doit JAMAIS se connecter sur un poste Tier 2.**

**Pourquoi :** Si un poste Tier 2 est compromis et qu'un admin Tier 0 s'y connecte, ses credentials (hash NTLM, ticket Kerberos) peuvent être extraits de la mémoire. L'attaquant obtient alors un accès Tier 0 à toute l'infrastructure.

---

## 3. LAPS — Local Administrator Password Solution

### Problème résolu

Sans LAPS, tous les postes d'un parc ont souvent le **même mot de passe administrateur local** (ex : `Admin@2023`). Si un seul poste est compromis → l'attaquant peut se connecter en admin local sur **tous** les postes du parc.

### Fonctionnement

1. LAPS génère automatiquement un **mot de passe aléatoire unique** pour chaque poste
2. Ce mot de passe est **stocké dans l'attribut AD** de l'objet ordinateur (chiffré)
3. Il est **renouvelé automatiquement** selon la politique définie (ex : tous les 30 jours)
4. Seuls les administrateurs autorisés peuvent le consulter via AD ou PowerShell

**Commande PowerShell pour lire un mot de passe LAPS :**
```powershell
Get-ADComputer -Identity "NomDuPoste" -Properties ms-Mcs-AdmPwd | Select ms-Mcs-AdmPwd
```

---

## 4. JIT et JEA

### JIT — Just-In-Time Administration

**Principe :** Les droits élevés sont accordés **uniquement pendant la durée nécessaire** à l'intervention, puis automatiquement révoqués.

**Exemple :** Un administrateur demande des droits Domain Admin pour une intervention. Il les reçoit pour 2 heures. Après 2 heures, les droits sont automatiquement supprimés, même s'il n'a pas fait la demande de révocation.

**Avantage :** La fenêtre d'exposition des comptes privilégiés est minimisée. Même si un compte est compromis pendant une session, les droits disparaissent automatiquement.

---

### JEA — Just Enough Administration

**Principe :** Chaque administrateur n'a accès qu'aux **cmdlets PowerShell strictement nécessaires** à sa fonction.

**Exemple :** Un technicien helpdesk ne peut exécuter que `Reset-ADAccountPassword` et `Unlock-ADAccount`. Il ne peut pas créer d'utilisateurs, modifier des GPO ou accéder aux DC.

**Avantage :** Même un compte admin compromis ne peut faire que ce qui lui est autorisé dans son profil JEA.

---

### JIT vs JEA en résumé

| Concept | Limite | Dimension |
|---|---|---|
| **JIT** | La durée d'exposition des droits | Temporelle |
| **JEA** | L'étendue des actions possibles | Fonctionnelle |

---

## 5. Politique de mot de passe affinée — PSO

### Problème

Par défaut, un domaine AD n'a **qu'une seule politique de mot de passe** (Default Domain Policy). Comment appliquer des règles différentes pour les admins et les utilisateurs standard dans le **même domaine** ?

### Solution : Fine-Grained Password Policy (PSO)

**PSO = Password Settings Object**

- Disponible depuis Windows Server 2008
- Permet plusieurs politiques de mot de passe dans le même domaine
- S'applique directement à des **utilisateurs ou groupes** (pas à des OU)
- Configurée dans **Active Directory Administrative Center (ADAC)** → `System > Password Settings Container`

**Exemple :**
- PSO "Standard" : 12 caractères, complexité activée → appliquée à tous les utilisateurs
- PSO "Admins" : 20 caractères, changement tous les 30 jours → appliquée au groupe "Domain Admins"

---

## 6. Gestion des comptes utilisateurs — Bonnes pratiques

### Quand un utilisateur quitte l'entreprise

| Étape | Action | Pourquoi |
|---|---|---|
| 1 | **Désactiver** le compte immédiatement | L'utilisateur ne peut plus se connecter |
| 2 | **Révoquer** les accès sensibles (VPN, mail partagé) | Couper les accès critiques |
| 3 | **Transférer** les fichiers et données | Ne pas perdre le travail |
| 4 | **Archiver** la boîte mail | Obligations légales |
| 5 | **Attendre** (1 mois à 1 an selon politique) | S'assurer qu'aucune donnée n'est oubliée |
| 6 | **Supprimer** le compte | Nettoyage final |

> ⚠️ Ne jamais supprimer directement : le SID est perdu définitivement et les accès aux ressources partagées ne peuvent plus être audités.

---

## 7. Event ID — Journaux à surveiller

Les **Event ID** sont des identifiants numériques dans l'**Observateur d'événements Windows** (eventvwr.msc). En cas d'incident AD, ce sont les premiers journaux à consulter.

### Les Event ID prioritaires

| Event ID | Description | Criticité |
|---|---|---|
| **4625** | Échec d'authentification (mauvais mot de passe) | ⚠️ Haute — tentative de brute force ? |
| **4624** | Connexion réussie | Info — à surveiller sur les comptes sensibles |
| **4720** | Création d'un compte utilisateur | ⚠️ Haute — création non autorisée ? |
| **4726** | Suppression d'un compte utilisateur | 🚨 Critique |
| **4728 / 4732** | Ajout à un groupe privilégié (Domain Admins, Admins locaux) | 🚨 Critique |
| **4740** | Verrouillage d'un compte utilisateur | ⚠️ Haute — brute force en cours ? |
| **4672** | Connexion avec des privilèges spéciaux (admin) | 🚨 À auditer systématiquement |
| **4771** | Échec de pré-authentification Kerberos | ⚠️ Tentative sur compte AD |

---

## 8. WEF — Windows Event Forwarding

### Définition

**WEF** est un mécanisme Windows permettant de **centraliser les journaux d'événements** de plusieurs machines sur un serveur collecteur unique (**WEC — Windows Event Collector**).

**Protocole :** WS-Management (WS-Man)
- HTTP : port **5985**
- HTTPS : port **5986**

**Utilité :** Avoir tous les logs de sécurité au même endroit pour :
- Analyse centralisée (SIEM, supervision)
- Détection d'intrusion
- Audit et conformité
- Sans avoir à se connecter sur chaque machine individuellement

---

## 9. Relations d'approbation (Trusts)

### Quand en a-t-on besoin ?

Lorsqu'une entreprise rachète une autre avec son propre domaine AD, ou pour permettre aux utilisateurs d'un domaine d'accéder aux ressources d'un autre.

### Types d'approbations

| Type | Portée | Usage |
|---|---|---|
| **Approbation de forêt** | Entre deux forêts entières | Fusion de sociétés |
| **Approbation externe** | Entre deux domaines de forêts différentes | Accès ciblé inter-sociétés |
| **Approbation de raccourci** | Entre deux domaines d'une même forêt | Accélérer l'authentification |

**Directions :**
- **Unidirectionnelle** : A fait confiance à B mais B ne fait pas confiance à A
- **Bidirectionnelle** : A et B se font mutuellement confiance

**Transitivité :** Si A approuve B et B approuve C → A approuve C automatiquement.

---

## Points clés à retenir

- **Moindre privilège** = droits via les groupes, jamais en direct, comptes admin séparés
- **Tiering** = Tier 0 (DC) / Tier 1 (serveurs) / Tier 2 (postes) — jamais croiser les comptes
- **LAPS** = mot de passe admin local unique par poste, stocké dans AD, renouvelé automatiquement
- **JIT** = droits limités dans le temps / **JEA** = droits limités en périmètre
- **PSO** = plusieurs politiques de mot de passe dans un même domaine
- **Event ID** critiques : 4625 (échec login), 4728/4732 (ajout groupe admin), 4740 (verrouillage), 4672 (connexion admin)
- **WEF** = centralisation des logs Windows, ports 5985/5986
