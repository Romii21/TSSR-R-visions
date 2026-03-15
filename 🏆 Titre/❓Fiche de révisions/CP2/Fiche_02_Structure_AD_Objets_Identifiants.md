# Fiche 02 — Structure AD, Objets et Identifiants

> **Formation TSSR — CP2**
> Révision : OU, Domaine, Arbre, Forêt, DN, Objets AD, SID, GUID

---

## 1. L'arborescence Active Directory

### Structure logique

L'arborescence AD est une **structure logique indépendante du réseau physique**. Les niveaux du plus bas au plus haut :

```
Objet (utilisateur, PC, imprimante)
  └── OU (Organizational Unit)
        └── Domaine
              └── Arbre
                    └── Forêt
```

> ⚠️ **Bonne pratique :** Ne jamais lier la structure AD à la topologie réseau physique (câblage, salles, bâtiments).

---

### L'OU — Organizational Unit

L'OU est le **conteneur de base** pour organiser les objets AD. Elle fonctionne comme un dossier.

**Ses trois fonctions principales :**
1. **Organiser logiquement** les objets (utilisateurs, ordinateurs, groupes)
2. **Déléguer des droits d'administration** (un technicien gère son OU sans toucher au reste)
3. **Appliquer des GPO de manière ciblée** (la GPO ne s'applique qu'aux objets de l'OU)

**Organisation recommandée :**

| Méthode | Recommandé ? | Raison |
|---|---|---|
| Par **fonction / métier** | ✅ Oui | Les GPO suivent les besoins fonctionnels, pas la géographie |
| Par **localisation géographique** | ❌ Non | Crée des doublons de GPO, difficile à maintenir |

**Structure type recommandée :**
```
Domaine Entreprise
├── OU Utilisateurs
│   ├── OU Direction Générale
│   ├── OU Ressources Humaines
│   ├── OU Commercial
│   └── OU Informatique
└── OU Ordinateurs
    ├── OU PC Portables
    ├── OU PC Bureaux
    └── OU Serveurs
```

---

### Le Domaine

Unité administrative et de sécurité de base. Caractéristiques :

- Contrôle centralisé des politiques de sécurité
- Gestion unique des comptes utilisateurs et ordinateurs
- Authentification centralisée via les DC
- Réplication des données entre contrôleurs

---

### Arbre et Forêt

| Concept | Définition | Exemple |
|---|---|---|
| **Arbre** | Ensemble de domaines partageant un **espace de noms contigu** et reliés par des approbations | `paris.masociete.fr` et `lyon.masociete.fr` dans l'arbre `masociete.fr` |
| **Forêt** | Ensemble de plusieurs arbres partageant un **schéma commun** et un catalogue global | `masociete.fr` + `mafilliale.com` dans la même forêt |

> ⚠️ **Erreur fréquente :** Un arbre ne = pas forcément un seul domaine. Un arbre peut contenir plusieurs domaines avec un espace de noms contigu.

**Exemple de forêt multi-arbres :**
```
masociete.fr (Arbre 1)          mafilliale.com (Arbre 2)
├── paris.masociete.fr          ├── nantes.mafilliale.com
└── lyon.masociete.fr           └── berlin.mafilliale.com
```

**Lien entre domaines d'un même arbre :** Relations d'approbation **bidirectionnelles et transitives**. Si A approuve B et B approuve C → A approuve automatiquement C.

---

## 2. Le Distinguished Name (DN)

Le **DN** est le chemin LDAP complet d'un objet dans l'annuaire. C'est son "adresse postale" dans AD.

**Format :**
```
CN=jdupont,OU=Informatique,OU=Utilisateurs,DC=entreprise,DC=local
```

**Décomposition :**

| Composant | Signification | Valeur dans l'exemple |
|---|---|---|
| `CN=` | Common Name — nom de l'objet | jdupont |
| `OU=` | Organizational Unit — conteneur parent | Informatique (puis Utilisateurs) |
| `DC=` | Domain Component — composant du domaine | entreprise.local |

> ⚠️ Le DN se lit **de gauche (spécifique) vers la droite (général)**. Le DN **peut changer** si l'objet est déplacé dans une autre OU.

---

## 3. Les Objets Active Directory

### Deux grandes catégories

| Catégorie | Définition | Exemples |
|---|---|---|
| **Conteneurs** | Peuvent contenir d'autres objets | OU, Group, Container |
| **Leaf objects** (terminaux) | Ne contiennent pas d'autres objets | User, Computer, Printer |

---

### Les Groupes — Étendues

| Étendue | Portée | Usage typique |
|---|---|---|
| **Domain Local** | Domaine local uniquement | Attribuer des droits sur des ressources locales (partage, imprimante) |
| **Global** | Tous les domaines approuvés | Regrouper des utilisateurs par métier/fonction ✅ Le plus courant |
| **Universal** | Toute la forêt | Groupes transverses à plusieurs domaines |

---

### Les Groupes — Types

| Type | Usage | Peut gérer des permissions ? |
|---|---|---|
| **Sécurité** | Gestion des droits d'accès (NTFS, partages réseau) | ✅ Oui — c'est son rôle principal |
| **Distribution** | Listes de diffusion e-mail (Exchange/Outlook) | ❌ Non |

> ⚠️ Pour gérer des permissions sur des ressources, on utilise **toujours** un groupe de sécurité.

---

### Méthode AGDLP

**AGDLP = Account → Global → Domain Local → Permission**

Méthode standard de gestion des permissions dans AD.

**Fonctionnement :**
1. Les **comptes (A)** sont placés dans des **groupes Globaux (G)** → ex : `GG_Comptables`
2. Les groupes Globaux sont ajoutés à des **groupes Domain Local (DL)** → ex : `DL_Partage_Compta_Lecture`
3. Les groupes Domain Local reçoivent les **permissions (P)** sur les ressources

**Pourquoi AGDLP est recommandé :**
- Séparation claire entre "qui sont les gens" (Global) et "qui accède à quoi" (Domain Local)
- Facilite la gestion dans les environnements multi-domaines
- Moins de modifications nécessaires lors des réorganisations

---

## 4. Les Identifiants Uniques

### Tableau comparatif SID / GUID / DN

| Critère | SID | GUID | DN |
|---|---|---|---|
| **Signification** | Security Identifier | Globally Unique Identifier | Distinguished Name |
| **Unicité** | Dans un domaine | Dans toute la forêt | Théoriquement dans la forêt |
| **Peut changer ?** | ✅ Si changement de domaine | ❌ **Jamais** | ✅ Si déplacement dans l'OU |
| **Usage** | Permissions NTFS, ACL | Référencement interne AD | Chemin LDAP de l'objet |
| **Format** | `S-1-5-21-156063872-...-1134` | `3F2504E0-4F89-11D3-9A0C-...` | `CN=...,OU=...,DC=...` |
| **Taille** | Variable (max 256 car.) | 128 bits fixe | Variable |

---

### Le SID — Security Identifier

- Attribué à chaque objet AD à sa **création**
- Utilisé par Windows pour vérifier les **permissions NTFS et les ACL**
- Change si l'objet est **migré vers un autre domaine**
- ⚠️ Ne change **pas** si l'objet est renommé ou modifié

**Commande PowerShell :**
```powershell
$user = Get-ADUser -Filter * | Where-Object {$_.Name -eq "jdupont"}
$user | Select-Object Name, SID
```

---

### Le GUID — Globally Unique Identifier

- Attribué à la création — **ne change jamais, quoi qu'il arrive**
- Unique dans toute la forêt AD
- Utilisé par AD en interne pour la réplication et les journaux
- Reste le même même après renommage, déplacement ou migration

**Commande PowerShell :**
```powershell
$user | Select-Object Name, ObjectGUID
```

---

## Points clés à retenir

- **OU** = conteneur pour organiser, déléguer, cibler les GPO — **toujours organiser par fonction**
- **Arbre** = domaines avec espace de noms contigu / **Forêt** = ensemble d'arbres avec schéma commun
- **DN** = chemin LDAP complet, peut changer si déplacement
- **SID** = identifiant de sécurité, peut changer si changement de domaine
- **GUID** = identifiant absolu, **ne change jamais**
- Groupe de **sécurité** = permissions / Groupe de **distribution** = messagerie
- Méthode **AGDLP** = Account → Global → Domain Local → Permission
