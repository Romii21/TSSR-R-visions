# Fiche 04 — GPO — Group Policy Objects

> **Formation TSSR — CP2**
> Révision : Définition, architecture, LSDOU, LIFO, héritage, filtrage, commandes

---

## 1. Définition

Les **GPO (Group Policy Objects)** sont des collections de politiques de configuration permettant la **gestion centralisée** d'un parc informatique via Active Directory.

**Ce qu'une GPO peut configurer :**
- **Politiques de sécurité** — longueur des mots de passe, verrouillage de compte, audit
- **Déploiement logiciel** — installation automatique d'applications (format MSI)
- **Environnement utilisateur** — fond d'écran, accès au panneau de configuration, navigateur par défaut
- **Scripts** — scripts de connexion/déconnexion, démarrage/arrêt
- **Redirection de dossiers** — centralisation des documents sur un serveur

> ⚠️ Les GPO fonctionnent uniquement dans un environnement **domaine AD**. Sur un Workgroup, on utilise des stratégies locales uniquement.

---

## 2. Architecture d'une GPO

Une GPO est constituée de **trois composantes** :

| Composante | Emplacement | Contenu |
|---|---|---|
| **Entrée LDAP** | `CN=Policies,CN=System,DC=xx,DC=xx` | Nom, GUID, droits d'édition |
| **Contenu** | `\\serveur\SYSVOL\domaine\Policies\{GUID}` | Fichiers d'instructions (les règles) |
| **Attribut gPLink** | Sur chaque OU / Site / Domaine | GUID, ordre de traitement, lien actif ou non |

> ⚠️ **SYSVOL** est un dossier partagé sur tous les DC. Il est répliqué entre eux. C'est là que vivent les GPO physiquement.

---

## 3. Ordre d'application — LSDOU

**LSDOU = Local → Site → Domain → OU**

```
Ordre d'application (du moins prioritaire au plus prioritaire) :

1. LOCAL      → Stratégies locales de la machine
2. SITE       → GPO du site AD
3. DOMAIN     → GPO du domaine
4. OU         → GPO des OU (de la plus haute à la plus spécifique)

Plus une GPO est appliquée tard, plus elle est prioritaire.
L'OU la plus proche de l'objet a la priorité la plus haute.
```

> ⚠️ **Exception absolue :** Une GPO avec l'état **Enforced** a la priorité sur tout, quel que soit son niveau dans LSDOU.

**Schéma de priorité :**
```
Local < Site < Domain < OU parent < OU enfant
  ↑                                     ↑
Faible priorité               Forte priorité
```

---

## 4. Priorité au sein d'une même OU — LIFO

Quand **plusieurs GPO sont liées à la même OU**, c'est le principe **LIFO** qui s'applique.

**LIFO = Last In, First Out** — La dernière GPO liée est la première traitée (donc prioritaire).

> ⚠️ **Piège fréquent :** La GPO avec le **numéro de lien 1** est la **plus prioritaire**, pas celle avec le numéro le plus élevé.

**Exemple :**
```
GPO "Fond d'écran"    → liée en 1er  → Ordre de lien : 3  → Traitée en dernier
GPO "Mot de passe"    → liée en 2ème → Ordre de lien : 2  → Traitée en 2ème
GPO "Sécurité renforcée" → liée en dernier → Ordre de lien : 1 → PRIORITAIRE ✅
```

**La GPO prioritaire = ordre de lien numéro 1.**

---

## 5. Blocage d'héritage et Enforced

### Blocage d'héritage

Le **blocage d'héritage** sur une OU **empêche les GPO des niveaux supérieurs** (Domaine, Site) de s'appliquer aux objets de cette OU.

> ⚠️ **Sens à bien retenir :** Le blocage empêche les GPO de *descendre du dessus*, pas de *monter depuis le dessous*.

**Quand l'utiliser :** Pour une OU avec des besoins très spécifiques qui ne doit pas recevoir les politiques générales (ex : OU des administrateurs du domaine).

---

### État Enforced

L'état **Enforced** sur une GPO lui donne une **priorité absolue**.

| Comportement | Description |
|---|---|
| Écrase toutes les GPO inférieures | S'applique même si une OU a l'héritage bloqué |
| Priorité absolue | Aucune autre GPO ne peut la contredire |
| Usage | Très rare — réservé aux politiques de sécurité critiques (Tier 0) |

> ⚠️ **Bonne pratique :** N'utiliser Enforced qu'en dernier recours. Un usage excessif complique la gestion et le diagnostic.

---

### Tableau récapitulatif

| Configuration | Effet | Contournement |
|---|---|---|
| **Blocage d'héritage sur OU** | Bloque les GPO des niveaux supérieurs | GPO Enforced ignore le blocage |
| **GPO Enforced** | Force l'application quoi qu'il arrive | Aucun |
| **Link Disabled** | La GPO ne s'applique plus à cette OU | Réactiver le lien |
| **GPO Disabled** | La GPO est désactivée partout | Réactiver la GPO |

---

## 6. Conditions pour qu'une GPO s'applique

**Deux conditions obligatoires :**
1. **Lien actif (Link Enabled)** — la GPO est bien liée à la cible (OU, domaine, site)
2. **Droits suffisants** — l'objet doit avoir les permissions **Read** ET **Apply Group Policy**

**Pour restreindre une GPO à un groupe spécifique :**
1. Retirer **"Authenticated Users"** du filtrage de sécurité
2. Ajouter le groupe cible avec les droits **Read + Apply Group Policy**

**Timing d'application :**
| Type de GPO | Moment d'application |
|---|---|
| **GPO Ordinateurs** | Pendant le démarrage (avant l'écran de connexion) |
| **GPO Utilisateurs** | Pendant la connexion de l'utilisateur (logon) |

---

## 7. Consoles de gestion

| Console | Commande | Contexte |
|---|---|---|
| **Éditeur de stratégie locale** | `gpedit.msc` | Machine standalone ou Workgroup — stratégies locales uniquement |
| **Console de gestion des GPO** | `gpmc.msc` | Serveur AD uniquement — gestion complète de toutes les GPO du domaine |

---

## 8. Commandes essentielles

```cmd
gpupdate /force               Forcer la mise à jour immédiate des GPO
gpresult /r                   Afficher les GPO appliquées sur la machine locale
gpresult /h rapport.html      Générer un rapport HTML complet des GPO appliquées
gpmc.msc                      Ouvrir la console de gestion des GPO (serveur)
gpedit.msc                    Ouvrir l'éditeur de stratégie locale (client)
```

---

## 9. Déployer une application via GPO

**Prérequis :** L'application doit être au format **MSI** (Windows Installer).

**Procédure :**
1. Placer le fichier MSI dans un **partage réseau accessible** par tous les postes
   - Exemple : `\\srv-ad\Deploy$\AppName\application.msi`
2. Créer une **nouvelle GPO** liée à l'OU cible
3. Naviguer dans la GPO : `Configuration ordinateur > Paramètres logiciel > Installation de logiciels`
4. Clic droit → **Nouveau** → **Package** → sélectionner le MSI via le **chemin UNC**
5. Choisir le mode :
   - **Affecté** = installation automatique au prochain démarrage du poste
   - **Publié** = disponible à la demande dans "Programmes et fonctionnalités"
6. Forcer la mise à jour avec `gpupdate /force` ou attendre le prochain démarrage

> ⚠️ **Important :** Toujours utiliser le chemin UNC (`\\serveur\partage\...`) et jamais un chemin local (`C:\...`). Le poste client doit pouvoir accéder au partage pour installer l'application.

---

## 10. Bonnes pratiques GPO

- **Une action = Une GPO** — ne pas mélanger plusieurs configurations dans une seule GPO
- **Ne jamais modifier** la `Default Domain Policy` ni la `Default Domain Controllers Policy`
- **Ne jamais modifier une GPO active en production** — créer une nouvelle GPO et tester d'abord
- **Ne pas bloquer l'héritage** sauf cas exceptionnel et justifié
- **Nommer clairement** les GPO (ex : `GPO-Securite-MotDePasse-Admins`)

---

## Points clés à retenir

- **LSDOU** : Local → Site → Domain → OU — l'OU la plus proche de l'objet est la plus prioritaire
- **LIFO** : ordre de lien **numéro 1 = le plus prioritaire** (pas le plus élevé)
- **Blocage d'héritage** = bloque les GPO qui *descendent du dessus*
- **Enforced** = priorité absolue, ignore le blocage d'héritage
- Pour qu'une GPO s'applique : lien actif **ET** droits Read + Apply Group Policy
- `gpupdate /force` = forcer la mise à jour | `gpresult /r` = voir les GPO appliquées
