# Fiche 05 — Fonctionnalités avancées : FSMO, Réplication, Niveaux Fonctionnels

> **Formation TSSR — CP2**
> Révision : Niveaux fonctionnels, Schéma AD, Rôles FSMO, Réplication, KCC, RODC

---

## 1. Niveau Fonctionnel

### Définition

Le **niveau fonctionnel** détermine les fonctionnalités AD disponibles dans un domaine ou une forêt. Il est **limité par la version du Windows Server la plus ancienne** présente parmi les DC.

**Règle :** Niveau fonctionnel = version du DC le plus ancien.

**Exemple :**
```
3 DC sous Windows Server 2019
+ 2 DC sous Windows Server 2012 R2
──────────────────────────────────
→ Niveau fonctionnel = Windows Server 2012 R2
   (bloqué par les DC les plus anciens)
```

> ⚠️ **Irréversible :** La montée de niveau fonctionnel **ne peut pas être annulée**. Il faut d'abord migrer ou retirer les DC anciens avant de monter le niveau.

---

## 2. Le Schéma AD

Le **schéma** définit la structure de l'annuaire :
- Les **classes d'objets** (utilisateur, ordinateur, groupe, imprimante…)
- Les **attributs** de chaque classe (nom, prénom, SID, email…)

**Caractéristiques critiques :**
- Partagé au niveau de **toute la forêt** (un seul schéma pour toute la forêt)
- Les modifications sont **définitives** — aucun retour arrière possible
- Géré par le rôle FSMO **Schema Master**

> ⚠️ **Impact lors de fusions de domaines :** Le schéma du domaine parent peut écraser celui des domaines enfants. Une fusion de forêts nécessite une analyse approfondie (plusieurs mois de préparation).

---

## 3. Les 5 Rôles FSMO

**FSMO = Flexible Single Master Operation**

Ce sont des rôles uniques dans AD : une seule machine peut détenir chaque rôle à la fois. Ils gèrent des opérations critiques qui ne peuvent pas être distribuées.

---

### Niveau Forêt (2 rôles — uniques dans toute la forêt)

| Rôle | Fonction |
|---|---|
| **Schema Master** (Maître de Schéma) | Gère les modifications du schéma AD (structure des objets et attributs) |
| **Domain Naming Master** (Maître d'Attribution de Noms) | Autorise l'ajout et la suppression de domaines dans la forêt |

---

### Niveau Domaine (3 rôles — un par domaine)

| Rôle | Fonction | Criticité |
|---|---|---|
| **RID Master** | Attribue des blocs de RID (Relative Identifier) aux DC pour créer des SID uniques | ⚠️ Haute |
| **Infrastructure Master** | Gère les références d'objets entre domaines différents | Moyenne |
| **PDC Emulator** | Synchronisation du temps, gestion des verrouillages de compte, compatibilité downlevel | 🚨 **CRITIQUE** |

---

### PDC Emulator — Le rôle le plus critique

Le **PDC Emulator** est le rôle FSMO le plus important. Si il tombe, les conséquences sont immédiates :

**Ses responsabilités :**
- **Source de temps (NTP)** pour tout le domaine → si l'heure dérive > 5 min, Kerberos cesse de fonctionner
- **Gestion des verrouillages de comptes** en temps réel (c'est lui qui reçoit les alertes de mauvais mot de passe)
- **Compatibilité avec les anciens clients** (Windows NT, pre-2000)
- **Synchronisation des mots de passe** (les changements de mot de passe lui sont transmis en priorité)

> ⚠️ Si le PDC Emulator tombe sans remplaçant : les utilisateurs dont le mot de passe vient d'être changé peuvent être bloqués, et les verrouillages de comptes peuvent ne plus être gérés correctement.

---

### Bonnes pratiques FSMO

- Ne jamais concentrer tous les rôles sur un seul DC
- Idéalement : **5 DC avec 1 rôle FSMO chacun**
- Au minimum : **2 DC** avec séparation des rôles forêt/domaine
- Séparer physiquement les DC portant des rôles FSMO critiques

**Commande pour voir les rôles FSMO :**
```powershell
Get-ADDomainController -Filter * | Select Name, OperationMasterRoles
netdom query fsmo
```

---

## 4. La Réplication

### Définition

La **réplication** assure la **cohérence des données** entre tous les contrôleurs de domaine. Chaque DC possède une copie de la base AD et les modifications doivent se propager.

**Éléments répliqués :**
- Base d'annuaire AD
- GPO (Group Policy Objects)
- Scripts de connexion
- Zones DNS intégrées à AD

> ⚠️ **La réplication N'EST PAS une sauvegarde.** C'est de la synchronisation. Si une erreur est faite dans AD (suppression accidentelle), elle se réplique sur tous les DC.

---

### Processus de réplication (étape par étape)

```
1. DC1 modifie un objet dans sa base AD
      ↓
2. DC1 interroge le KCC pour identifier les autres DC
      ↓
3. DC1 consulte le DNS pour localiser DC2
      ↓
4. DC1 notifie DC2 qu'il y a des modifications disponibles
      ↓
5. DC2 demande les détails des modifications à DC1
      ↓
6. DC1 transmet les changements à DC2
      ↓
7. DC2 met à jour sa propre base de données
```

---

### KCC — Knowledge Consistency Checker

Le **KCC** est le composant qui orchestre la réplication. Il est présent sur **chaque DC**.

**Son rôle :**
- Génère automatiquement la **topologie de réplication** (qui réplique avec qui)
- Met à jour la topologie lors de changements (ajout/suppression d'un DC)
- Gère la réplication intra-site ET inter-site

---

### Réplication Intra-site vs Inter-site

| Critère | Intra-site | Inter-site |
|---|---|---|
| Définition | Entre DC du même site AD | Entre DC de sites différents |
| Intervalle par défaut | 5 minutes | 180 minutes |
| Type de réseau | LAN rapide | WAN (lien potentiellement lent) |
| Compression des données | Non | ✅ Oui (économise la bande passante) |

**Recommandation :** Un intervalle de **10 à 20 minutes** offre le meilleur équilibre.

> ⚠️ Si le délai de réplication dépasse 30 à 45 minutes → **problème probable à investiguer**.

---

### Commandes de diagnostic de la réplication

```cmd
repadmin /showrepl           État détaillé de la réplication (erreurs, horodatages)
repadmin /replsummary        Résumé rapide de la santé de la réplication
dcdiag /test:replications    Diagnostic complet des réplications entre DC
```

---

## 5. RODC — Read-Only Domain Controller

### Définition

Un **RODC** est un contrôleur de domaine qui possède une copie **en lecture seule** de la base AD. Il ne peut pas écrire dans l'annuaire.

### Contextes d'utilisation

| Situation | Raison |
|---|---|
| **Site distant** avec WAN peu fiable | Authentification locale même si le WAN tombe |
| **Zone à faible sécurité physique** | Si le serveur est compromis, l'attaquant ne peut pas modifier l'AD |
| **Filiale avec personnel non IT** | Risque réduit d'administration accidentelle |

**Avantage sécurité :** Par défaut, un RODC ne met en cache que les comptes autorisés. Si le RODC est volé ou compromis, seuls ces comptes sont exposés (pas tous les mots de passe du domaine).

---

## Synthèse — Les 5 rôles FSMO d'un coup d'œil

```
FORÊT (1 de chaque dans toute la forêt)
├── Schema Master          → Modifie la structure de l'AD
└── Domain Naming Master   → Ajoute/supprime des domaines

DOMAINE (1 de chaque par domaine)
├── RID Master             → Génère des SID uniques
├── Infrastructure Master  → Gère les liens inter-domaines
└── PDC Emulator           → 🚨 LE PLUS CRITIQUE
                              • Source NTP du domaine
                              • Verrouillages de comptes
                              • Synchronisation des mots de passe
```

---

## Points clés à retenir

- **Niveau fonctionnel** = limité par le DC le plus ancien — irréversible une fois monté
- **Schéma** = structure de l'annuaire, unique dans la forêt, modifications définitives
- **5 rôles FSMO** : 2 au niveau forêt (Schema Master, Domain Naming Master) + 3 au niveau domaine (RID, Infrastructure, PDC Emulator)
- **PDC Emulator = le plus critique** — gère l'heure et les verrouillages
- **Réplication** = synchronisation entre DC, orchestrée par le **KCC**
- **10 à 20 min** = intervalle de réplication recommandé
- **RODC** = DC en lecture seule, pour les sites distants ou peu sécurisés physiquement
