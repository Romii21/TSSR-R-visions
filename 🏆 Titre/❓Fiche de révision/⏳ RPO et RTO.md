# 📚 Fiche de révision — RTO & RPO (Plan de reprise d'activité)

---

## 🔑 Définitions fondamentales

| Acronyme | Nom complet | Définition |
|---|---|---|
| **RPO** | Recovery Point Objective | **Jusqu'où peut-on remonter dans le temps ?** Perte de données maximale acceptable (en temps) |
| **RTO** | Recovery Time Objective | **Combien de temps peut-on rester hors ligne ?** Durée maximale acceptable pour restaurer le service |
| **PRA** | Plan de Reprise d'Activité | Ensemble des procédures pour reprendre l'activité après un sinistre |
| **PCA** | Plan de Continuité d'Activité | Maintien de l'activité **pendant** un sinistre (avant même la reprise) |

---

## 🎯 RPO — Recovery Point Objective

> **"Quelle quantité de données peut-on se permettre de perdre ?"**

### Schéma

```
Dernière          Incident
sauvegarde           │
    │                │
────●────────────────●────▶ temps
    │◄──── RPO ─────►│
         (ex: 4h)

→ On peut perdre au maximum 4h de données
```

### Ce que ça implique

| RPO court (ex: 15 min) | RPO long (ex: 24h) |
|---|---|
| Sauvegardes très fréquentes | Sauvegardes quotidiennes suffisantes |
| Coût élevé (stockage, réplication) | Coût réduit |
| Données critiques (base financière, ERP) | Données moins sensibles |

---

## ⏱️ RTO — Recovery Time Objective

> **"Combien de temps peut-on tolérer une interruption de service ?"**

### Schéma

```
Incident        Reprise du
   │              service
   │                │
───●────────────────●────▶ temps
   │◄──── RTO ─────►│
        (ex: 2h)

→ Le service doit être restauré en moins de 2h
```

### Ce que ça implique

| RTO court (ex: 5 min) | RTO long (ex: 48h) |
|---|---|
| Infrastructure redondante (HA, cluster) | Restauration manuelle acceptable |
| Basculement automatique (failover) | Coût très réduit |
| Coût très élevé | Services non critiques |

---

## 📊 Visualisation combinée RPO + RTO

```
Dernière      Incident                    Reprise
sauvegarde       │                           │
    │            │                           │
────●────────────●───────────────────────────●────▶ temps
    │◄── RPO ───►│◄──────── RTO ────────────►│
    │  (données  │      (interruption        │
    │   perdues) │       de service)         │
```

> 💡 **RPO** = fenêtre de **perte de données** | **RTO** = fenêtre de **temps d'arrêt**

---

## 🏗️ Stratégies selon les objectifs

### Classification des niveaux de criticité

| Niveau | RTO cible | RPO cible | Exemples de services |
|---|---|---|---|
| **Critique** | < 15 min | < 15 min | Paiement, urgences, trading |
| **Élevé** | < 4h | < 1h | ERP, messagerie, Active Directory |
| **Moyen** | < 24h | < 4h | Intranet, outils internes |
| **Faible** | < 72h | < 24h | Archives, reporting, outils annexes |

---

### Solutions techniques selon RTO/RPO

| Solution | RTO | RPO | Coût | Description |
|---|---|---|---|---|
| **Cluster HA** | Secondes | ~0 | 💰💰💰💰 | Basculement automatique en temps réel |
| **Réplication synchrone** | Minutes | ~0 | 💰💰💰💰 | Données écrites simultanément sur 2 sites |
| **Réplication asynchrone** | Minutes | Minutes | 💰💰💰 | Données répliquées avec léger décalage |
| **Snapshots fréquents** | 1-4h | 15 min - 1h | 💰💰 | Instantanés réguliers de la VM/disque |
| **Sauvegarde quotidienne** | 4-24h | 24h | 💰 | Backup classique (bande, NAS, cloud) |
| **Sauvegarde externe (tape)** | 24-72h | 24-48h | 💰 | Stockage hors site, restauration lente |

---

## 🔁 Lien avec la virtualisation

La virtualisation facilite grandement l'atteinte de RTO/RPO courts :

| Fonctionnalité | Impact sur RTO/RPO |
|---|---|
| **Snapshots de VM** | RPO très court (instantané en quelques secondes) |
| **Migration à chaud (Live Migration)** | RTO ≈ 0 (pas d'interruption) |
| **Haute disponibilité (HA)** | RTO très court (VM redémarrée automatiquement) |
| **Clonage / Templates** | RTO réduit (redéploiement rapide) |
| **Réplication de VM** (Proxmox, VMware) | RPO court selon fréquence |

---

## 📋 Règle des sauvegardes : 3-2-1

```
3 copies des données
  │
  ├── 2 supports différents (ex: disque local + NAS)
  │
  └── 1 copie hors site (cloud, site distant, bande externe)
```

> ✅ Règle minimale recommandée pour toute stratégie de sauvegarde sérieuse.  
> ✅ Variante moderne : **3-2-1-1-0** (+ 1 copie offline/immuable, 0 erreur vérifiée)

---

## ⚠️ Erreurs courantes à éviter

| Erreur | Risque |
|---|---|
| Sauvegardes non testées | La restauration échoue au moment critique |
| RPO/RTO définis mais jamais vérifiés | Les objectifs ne sont pas réellement atteignables |
| Sauvegarde sur le même site physique | Sinistre (incendie, inondation) = tout perdu |
| Pas de documentation des procédures | Perte de temps en cas d'urgence |
| Oublier les dépendances entre services | Restaurer un serveur inutilisable sans sa base de données |

---

## 🧪 Tester son PRA

Un PRA non testé **n'existe pas**. Les tests doivent être :

| Type de test | Description |
|---|---|
| **Test sur table** | Simulation théorique, revue des procédures |
| **Test de restauration** | Restaurer une sauvegarde sur un environnement isolé |
| **Test de basculement** | Simuler une panne et vérifier le failover |
| **Test grandeur nature** | Simulation complète avec coupure réelle |

> 🗓️ Fréquence recommandée : au minimum **1 fois par an**, idéalement après chaque changement majeur d'infrastructure.

---

## 📝 À retenir absolument

1. **RPO** = perte de données max tolérée (dans le passé) | **RTO** = temps d'arrêt max toléré (vers le futur)
2. Plus les objectifs sont courts → plus la solution est **coûteuse**
3. **PRA** = reprendre après le sinistre | **PCA** = continuer **pendant** le sinistre
4. La **virtualisation** (HA, snapshots, réplication) est un levier majeur pour réduire RTO et RPO
5. Règle **3-2-1** = base minimale de toute stratégie de sauvegarde
6. Un PRA **non testé = non fiable** → tester régulièrement les restaurations
7. Toujours cartographier les **dépendances entre services** avant de définir les priorités de reprise
