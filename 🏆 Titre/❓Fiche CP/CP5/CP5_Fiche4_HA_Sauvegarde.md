# FICHE 4 — Haute Disponibilité, Sauvegarde et Maintenance
### Formation TSSR — CP5 : Maintenir des serveurs dans une infrastructure virtualisée

---

## 1. Le Cluster Proxmox

### Définition
Un **cluster Proxmox** est un ensemble de plusieurs hyperviseurs (nœuds) qui fonctionnent ensemble, partagent une configuration commune et permettent la migration des VMs entre eux.

### Nombre de nœuds
- **Minimum technique :** 2 nœuds
- **Minimum recommandé :** **3 nœuds** (pour le quorum)
- **Règle :** toujours un nombre impair de nœuds pour éviter les situations d'égalité lors des votes

### Le Quorum
Mécanisme de vote qui détermine si le cluster peut continuer à fonctionner.

**Règle du quorum :** il faut que **plus de la moitié des nœuds (n/2 + 1)** soient accessibles.

| Nœuds total | Nœuds pour le quorum | Pannes tolérées |
|---|---|---|
| 2 | 2 | 0 (⚠️ déconseillé) |
| 3 | 2 | 1 |
| 5 | 3 | 2 |

**Pourquoi le quorum est-il essentiel ?**
Sans quorum, le cluster se met en pause pour éviter le **split-brain** : situation où deux parties du cluster pensent chacune être la seule active et écrivent des données contradictoires, causant de la corruption.

```
Cluster 3 nœuds — Panne du nœud 1 :
┌──────────┐    ✅ Nœud 2  ─┐
│  Nœud 1  │    ✅ Nœud 3  ─┤ → 2/3 nœuds = quorum ✅ → Cluster actif
│ ❌ Panne │                └→ Les VMs du nœud 1 sont redémarrées ici
└──────────┘
```

---

## 2. Haute Disponibilité (HA)

### Définition
La **HA (High Availability)** dans Proxmox permet le **redémarrage automatique des VMs** sur un autre nœud du cluster en cas de panne d'un nœud.

### Ce qui se passe lors d'une panne

```
État normal :                    Panne du nœud 1 :
┌──────────┐  ┌──────────┐      ┌──────────┐  ┌──────────┐
│  Nœud 1  │  │  Nœud 2  │      │  Nœud 1  │  │  Nœud 2  │
│  VM-AD   │  │  (libre) │  →   │  ❌ Down │  │  VM-AD   │
│  VM-Web  │  │          │      │          │  │  VM-Web  │
└──────────┘  └──────────┘      └──────────┘  └──────────┘
                                     ↑                ↑
                              HA Manager détecte  Redémarre auto
                              la panne            les VMs critiques
```

### Conditions requises pour la HA
1. **Cluster Proxmox** configuré (minimum 3 nœuds recommandé)
2. **Stockage partagé** entre les nœuds (NFS, Ceph, SAN/iSCSI) — les VMs ne doivent PAS être sur un disque local
3. **Quorum** actif
4. VM configurée dans le groupe HA de Proxmox

### Fencing (Isolation)
Mécanisme qui "isole" (éteint de force) un nœud défaillant avant de redémarrer ses VMs ailleurs, pour éviter que deux nœuds lancent la même VM simultanément.

---

## 3. RTO et RPO — Indicateurs de reprise

Ces deux indicateurs définissent les objectifs de reprise après incident et guident la stratégie de sauvegarde.

| Indicateur | Nom complet | Définition | Exemple |
|---|---|---|---|
| **RTO** | Recovery Time Objective | Durée maximale acceptable d'interruption de service | RTO = 4h → service rétabli dans les 4h |
| **RPO** | Recovery Point Objective | Perte de données maximale acceptable (exprimée en durée) | RPO = 1h → on peut perdre au max 1h de données |

### Impact sur la stratégie de sauvegarde

```
RPO = 24h  → Sauvegarde quotidienne suffisante
RPO = 1h   → Sauvegardes toutes les heures ou réplication continue
RPO = 0    → Réplication synchrone en temps réel (coût très élevé)

RTO = 24h  → Restauration manuelle acceptable
RTO = 1h   → Infrastructure redondante nécessaire (HA, cluster)
RTO = 0    → Redondance totale en temps réel (actif/actif)
```

**Plus le RTO et le RPO sont faibles, plus l'infrastructure coûte cher à mettre en place.**

---

## 4. La Règle 3-2-1

Standard incontournable de la sauvegarde :

```
3 copies des données
  2 supports différents
    1 copie hors site
```

**Application dans un environnement Proxmox :**

| Copie | Où | Support |
|---|---|---|
| **1** — Original | VM active sur le nœud Proxmox | Disque local ou SAN |
| **2** — Sauvegarde locale | Proxmox Backup Server (PBS) sur le LAN | NAS ou serveur dédié |
| **3** — Sauvegarde distante | PBS répliqué vers un site distant ou cloud | Cloud ou datacenter secondaire |

> ⚠️ Le RAID ne remplace pas la règle 3-2-1. Il protège contre les pannes matérielles, pas contre les erreurs humaines ou les ransomwares.

---

## 5. Proxmox Backup Server (PBS)

Solution de sauvegarde open-source dédiée aux environnements Proxmox.

### PBS vs Copie de fichier classique

| Fonctionnalité | Copie de fichier | PBS |
|---|---|---|
| **Déduplication** | ❌ Non | ✅ Oui (blocs identiques stockés une seule fois) |
| **Sauvegardes incrémentielles** | ❌ Difficile | ✅ Natif (seules les modifications sont sauvegardées) |
| **Vérification d'intégrité** | ❌ Non | ✅ Hash SHA-256 sur chaque bloc |
| **Restauration granulaire** | ❌ Non | ✅ Peut restaurer un seul fichier d'une VM |
| **Compression** | ❌ Non | ✅ Automatique |
| **Chiffrement** | ❌ Non | ✅ Côté client (AES-256) |
| **Interface web** | ❌ Non | ✅ Intégrée dans Proxmox |

---

## 6. Procédure de maintenance d'un nœud hyperviseur

### Ne jamais mettre à jour en production sans procédure !

**Risques d'une MAJ sans précaution :**
- Redémarrage imposé (nouveau kernel) → interruption de service
- Incompatibilité de version entre nœuds du cluster
- Régression ou bug post-mise à jour

### Procédure sécurisée (en cluster)

```
Étape 1 : Planifier une fenêtre de maintenance (hors heures de prod)
    ↓
Étape 2 : Snapshot de toutes les VMs critiques du nœud
    ↓
Étape 3 : Migrer les VMs à chaud vers les autres nœuds (Live Migration)
    ↓
Étape 4 : Mettre le nœud en mode "Maintenance" dans Proxmox
    ↓
Étape 5 : Appliquer les MAJ
    apt update && apt dist-upgrade
    ↓
Étape 6 : Redémarrer si nécessaire (kernel, drivers)
    ↓
Étape 7 : Vérifier le bon fonctionnement du nœud et son intégration au cluster
    ↓
Étape 8 : Rapatrier les VMs migrées
    ↓
Étape 9 : Répéter nœud par nœud (jamais tous simultanément)
```

---

## 7. Sécurité d'une infrastructure virtualisée

### Isolation des VMs
Chaque VM est cloisonnée dans son propre espace virtuel.

**Menaces contenues par l'isolation :**
- Une VM compromise ne peut pas accéder aux données des autres VMs
- Un malware ne se propage pas automatiquement aux autres VMs
- Un crash VM n'entraîne pas les autres (sauf saturation des ressources hôte)

**Limites de l'isolation :**
- **VM Escape** : Faille permettant à du code dans une VM de s'échapper vers l'hyperviseur (rare)
- **Side-channel attacks** : Exploitent les ressources partagées (Spectre/Meltdown, cache CPU)
- **Réseau partagé** : Sans VLAN, les VMs d'un même bridge peuvent se voir
- **Stockage partagé** : Une VM admin sur un NAS partagé peut impacter les autres

### Bonnes pratiques de sécurité

| Pratique | Objectif |
|---|---|
| Segmenter le réseau avec des VLANs | Isoler les flux entre VMs |
| Mettre à jour régulièrement l'hyperviseur | Corriger les failles de sécurité |
| Limiter l'accès à l'interface Proxmox | Réduire la surface d'attaque |
| Utiliser des mots de passe forts + MFA | Protéger l'accès administrateur |
| Activer le chiffrement PBS | Protéger les sauvegardes |
| Surveiller les logs | Détecter les comportements anormaux |

---

## 8. Plan de résilience — Ordre de priorité

Voici l'ordre logique pour améliorer la résilience d'une infrastructure avec un budget limité :

```
Priorité 1 — SAUVEGARDE (protection des données)
    → PBS, règle 3-2-1, tests de restauration réguliers
    → Coût : faible (serveur secondaire ou NAS)

Priorité 2 — REDONDANCE DU STOCKAGE
    → RAID 5/6 sur les disques locaux
    → NAS partagé pour préparer le cluster

Priorité 3 — CLUSTERING ET HA
    → Deuxième nœud Proxmox + stockage partagé
    → Activation de la HA sur les VMs critiques

Priorité 4 — SUPERVISION ET DOCUMENTATION
    → Zabbix, Grafana, alerting
    → Procédures de restauration testées et documentées
    → Définition des RTO/RPO par service
```

> 💡 **Règle d'or :** Commencer par la sauvegarde (aucune importance d'avoir de la HA si les données sont corrompues ou perdues).

---

## 9. Récapitulatif des concepts clés

| Concept | Définition en une phrase |
|---|---|
| **Cluster** | Ensemble de nœuds Proxmox travaillant ensemble |
| **Quorum** | Majorité de nœuds nécessaire pour que le cluster reste actif |
| **HA** | Redémarrage automatique des VMs en cas de panne d'un nœud |
| **Split-brain** | Situation où deux parties du cluster s'estiment maîtres simultanément |
| **Fencing** | Isolation forcée d'un nœud défaillant pour éviter le split-brain |
| **RTO** | Temps maximum d'interruption acceptable |
| **RPO** | Perte de données maximale acceptable |
| **Règle 3-2-1** | 3 copies, 2 supports différents, 1 hors site |
| **PBS** | Proxmox Backup Server — solution de sauvegarde dédiée |
| **Live Migration** | Déplacement d'une VM en cours de fonctionnement, sans interruption |

---
*CP5 — Fiche 4/5 — Haute Disponibilité, Sauvegarde et Maintenance*
