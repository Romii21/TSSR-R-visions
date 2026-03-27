# 📁 FICHE 1 – Concepts Fondamentaux de la Sauvegarde
### CP8 – Mettre en place, assurer et tester les sauvegardes et restaurations

---

## 1. Qu'est-ce qu'une sauvegarde ?

Une **sauvegarde** est une copie structurée, cohérente et datée de données, réalisée dans le but explicite de pouvoir les **restaurer** en cas de perte, corruption ou sinistre.

### Différence entre sauvegarde et simple copie

| Critère | Sauvegarde | Copie simple |
|---------|-----------|--------------|
| Versionnage | ✅ Oui (historique) | ❌ Non (écrase l'ancienne) |
| Cohérence des données | ✅ Garantie (fichiers ouverts gérés) | ❌ Pas garantie |
| Métadonnées | ✅ Permissions, horodatages conservés | ⚠️ Pas toujours |
| Restaurabilité | ✅ Testée et documentée | ❌ Non garantie |
| Planification | ✅ Automatique et régulière | ❌ Manuelle et ponctuelle |

> 💡 **À retenir :** Une copie de fichier n'est pas une sauvegarde. Une sauvegarde est une démarche complète, planifiée et testable.

---

## 2. La règle 3-2-1

La règle **3-2-1** est la règle d'or de toute politique de sauvegarde sérieuse.

```
3 copies des données
  ├── 2 sur des supports/technologies différents
  └── 1 copie hors site (hors du bâtiment)
```

### Exemple concret en entreprise

| Copie | Support | Emplacement |
|-------|---------|-------------|
| Copie 1 | Serveur de production | Salle serveur principale |
| Copie 2 | NAS interne | Même bâtiment, réseau local |
| Copie 3 | Cloud Azure Backup ou bande | Site distant / hors bâtiment |

> ⚠️ **Piège fréquent :** Un NAS connecté en permanence au réseau local ne constitue PAS une copie hors site efficace contre un ransomware. Il faut une copie **hors ligne ou immuable**.

---

## 3. Les trois types de sauvegardes

### 3.1 Sauvegarde complète (Full)

- **Principe :** Copie 100 % des données sélectionnées à chaque exécution
- ✅ Restauration simple : un seul jeu de données à manipuler
- ❌ Longue à réaliser, consomme beaucoup d'espace disque

### 3.2 Sauvegarde incrémentale

- **Principe :** Copie uniquement les données modifiées **depuis la dernière sauvegarde, quelle qu'elle soit** (complète ou incrémentale précédente)
- ✅ Très rapide, espace minimal
- ❌ Restauration lente : il faut appliquer toutes les incrémentales dans l'ordre depuis la dernière complète

### 3.3 Sauvegarde différentielle

- **Principe :** Copie les données modifiées **depuis la dernière sauvegarde complète uniquement**
- ✅ Restauration plus simple : Full + 1 différentielle seulement
- ❌ Grossit avec le temps (plus volumineuse qu'une incrémentale après plusieurs jours)

### Tableau comparatif

| Type | Par rapport à | Espace disque | Vitesse de sauvegarde | Vitesse de restauration |
|------|---------------|---------------|-----------------------|-------------------------|
| **Complète** | Rien | ❌ Très lourd | ❌ Lente | ✅ Très rapide |
| **Différentielle** | Dernière complète | ⚠️ Moyen | ✅ Rapide | ✅ Assez rapide |
| **Incrémentale** | Dernière sauvegarde | ✅ Léger | ✅ Très rapide | ❌ Lente (chaîne) |

### Schéma d'une semaine type

```
Lundi    : [COMPLÈTE]          → base de référence
Mardi    : [INCRÉMENTALE 1]    → modifs depuis lundi
Mercredi : [INCRÉMENTALE 2]    → modifs depuis mardi
Jeudi    : [INCRÉMENTALE 3]    → modifs depuis mercredi

Restauration jeudi soir :
→ Complète + Incrémentale 1 + Incrémentale 2 + Incrémentale 3

Avec différentielle :
Mardi    : [DIFFÉRENTIELLE 1]  → modifs depuis lundi
Mercredi : [DIFFÉRENTIELLE 2]  → modifs depuis lundi (cumule)
Jeudi    : [DIFFÉRENTIELLE 3]  → modifs depuis lundi (cumule encore)

Restauration jeudi soir :
→ Complète + Différentielle 3 seulement ✅
```

---

## 4. RPO et RTO

Ces deux indicateurs définissent les objectifs de reprise d'activité. Ils sont fixés par la direction ou la DSI selon les enjeux métier.

### RPO – Recovery Point Objective (Point de Reprise Objectif)

> **Définition :** Perte de données maximale acceptable, exprimée en temps.

- RPO = 0 → Aucune perte de données tolérée (réplication synchrone)
- RPO = 1h → On accepte de perdre au maximum 1h de données → sauvegardes toutes les heures

**Exemple :** Un ERP financier critique a un RPO de 15 minutes. Les transactions sont répliquées en temps quasi-réel.

### RTO – Recovery Time Objective (Délai de Reprise Objectif)

> **Définition :** Durée maximale acceptable d'interruption du service après un incident.

- RTO = 4h → Le service doit être restauré dans les 4 heures
- RTO = 0 → Basculement automatique instantané (haute disponibilité)

**Exemple :** Un serveur de messagerie a un RTO de 2h. Au-delà, l'activité commerciale est bloquée.

### Tableau de synthèse

| Indicateur | Question | Impact si mal calibré |
|-----------|----------|-----------------------|
| **RPO** | "Jusqu'à quand peut-on remonter ?" | Perte de données trop importante |
| **RTO** | "Combien de temps peut-on être en panne ?" | Service indisponible trop longtemps |

> 💡 **Mémo visuel :**
> ```
> ──────────────[Dernière sauvegarde]────────[Incident]────────[Retour service]──►
>               |←────── RPO ───────►|       |←──── RTO ─────►|
> ```

---

## 5. Le RAID n'est PAS une sauvegarde

Le **RAID** (Redundant Array of Independent Disks) est un mécanisme de **tolérance aux pannes matérielles** uniquement.

### Pourquoi le RAID ne remplace pas la sauvegarde

Les données sont accessibles **en temps réel** sur tous les disques du RAID. Toute modification (bonne ou mauvaise) est immédiatement répercutée sur l'ensemble de la grappe.

### Scénarios contre lesquels le RAID ne protège pas

| Scénario | Pourquoi le RAID échoue |
|----------|------------------------|
| **Suppression accidentelle** | La suppression est immédiatement répliquée sur tous les disques |
| **Ransomware** | Le chiffrement malveillant s'applique à tous les disques simultanément |
| **Erreur humaine** | Une fausse manipulation est répercutée instantanément |
| **Corruption logicielle** | Un bug ou une mise à jour défaillante corrompt les données sur tous les disques |
| **Panne du contrôleur RAID** | Les disques peuvent devenir illisibles sans le contrôleur d'origine |
| **Incendie / sinistre** | Tous les disques sont au même endroit physique |

> ⚠️ **À retenir absolument :** RAID ≠ Sauvegarde. Le RAID protège contre la panne d'un disque. La sauvegarde protège contre tout le reste.

---

## 6. Le Snapshot (Instantané)

### Définition

Un **snapshot** est une photo de l'état d'un volume ou d'une machine virtuelle à un instant T précis.

### Mécanisme COW (Copy-On-Write)

```
1. Création du snapshot :
   [Volume original] ←── référence ──→ [Snapshot]
   (aucune copie physique, très rapide)

2. Une donnée est modifiée sur l'original :
   → L'ancienne version est d'abord copiée dans le snapshot
   → La nouvelle version est écrite sur l'original

   [Volume original - nouvelle donnée]
   [Snapshot - ancienne donnée sauvegardée]
```

### Cas d'utilisation

- Avant une mise à jour système risquée
- Avant une modification de configuration critique
- Lors d'une sauvegarde LVM à chaud (cohérence garantie)
- Sur des VMs (VMware, Hyper-V, Proxmox) avant une intervention

### Limites du snapshot comme solution de sauvegarde

| Limite | Explication |
|--------|-------------|
| **Même support physique** | Si le disque tombe en panne, on perd le snapshot ET les données |
| **Pas externalisé** | Le snapshot reste sur le serveur, pas de protection contre le sinistre |
| **Croissance imprévisible** | Si beaucoup de données changent, le snapshot peut occuper autant d'espace que l'original |
| **Non autonome** | Ce n'est pas une sauvegarde complète et transportable |

> 💡 **Utilisation correcte :** Le snapshot est un **complément** à la sauvegarde, pas un remplacement.

---

## 🎯 Récapitulatif à mémoriser

| Concept | Définition courte |
|---------|-------------------|
| **Sauvegarde** | Copie structurée, versionnée, testable en vue d'une restauration |
| **Règle 3-2-1** | 3 copies, 2 supports différents, 1 hors site |
| **Full** | 100% des données, restauration rapide, espace élevé |
| **Incrémentale** | Modifs depuis dernière sauvegarde, espace faible, restauration lente |
| **Différentielle** | Modifs depuis dernière Full, compromis espace/restauration |
| **RPO** | Perte de données maximale acceptable (en temps) |
| **RTO** | Durée maximale d'indisponibilité acceptable |
| **RAID** | Tolérance aux pannes disque uniquement, PAS une sauvegarde |
| **Snapshot** | Photo d'un volume à un instant T (COW), complément à la sauvegarde |
