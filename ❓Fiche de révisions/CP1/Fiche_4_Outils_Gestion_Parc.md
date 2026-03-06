# 📕 Fiche 4 — Outils & Gestion de Parc Informatique
**Formation TSSR | CP1 — Support utilisateur en centre de services**

---

## 🗄️ 1. La CMDB — Configuration Management Database

### Définition
Base de données centralisant **toutes les informations de configuration** du parc informatique.

### Ce qu'on y stocke (pour chaque ordinateur)

| Catégorie | Données |
|---|---|
| **Identification** | Nom, code-barre, numéro de série (S/N) |
| **Référence matérielle** | Marque, modèle, constructeur |
| **Référence réseau** | Adresse IP, adresse MAC |
| **Composants** | CPU, RAM, disque dur, carte mère |
| **Logiciels** | OS, pilotes, applications installées |
| **Statut** | En production / en stock / en réparation |
| **Budget** | Prix, date d'achat, date de livraison |
| **Utilisateur** | Nom de l'utilisateur assigné |

---

## 🖥️ 2. GLPI

### Définition
**GLPI** = Logiciel Libre de Gestion de Parc Informatique

Solution **open source** qui combine :
- Une **CMDB** (inventaire du parc)
- Un **Helpdesk** (gestion des tickets d'incidents)

### Type de gestion : **PASSIVE (Statique)**

| Caractéristique | Détail |
|---|---|
| **Nature** | Statique |
| **Orientation** | Suivi et documentation |
| **Temps réel** | ❌ Non |
| **Intervention** | Documentation, pas d'action directe sur les postes |
| **Usage** | Gestion de parc + support |

---

## 📱 3. MDM — Mobile Device Management

### Définition
Outil de **gestion active** des appareils mobiles (smartphones, tablettes, laptops).

### Type de gestion : **ACTIVE (Dynamique)**

| Caractéristique | Détail |
|---|---|
| **Nature** | Dynamique |
| **Orientation** | Intervention directe sur les appareils |
| **Temps réel** | ✅ Oui |
| **Intervention** | Directe (push de config, effacement à distance…) |
| **Usage** | Gestion des appareils mobiles |

---

## ⚖️ 4. Comparatif GLPI vs MDM

| Critère | GLPI | MDM |
|---|---|---|
| **Type de gestion** | Passive | Active |
| **Nature** | Statique | Dynamique |
| **Temps réel** | ❌ Non | ✅ Oui |
| **Type d'intervention** | Documentation et suivi | Directe sur les appareils |
| **Cible principale** | Tout le parc informatique | Appareils mobiles |
| **Exemples** | GLPI (libre), Ivanti LanDesk, SCCM | IBM MaaS360, MobileIron |

---

## 🔍 5. Logiciels de Découverte Automatique

Ces outils **scannent le réseau** pour alimenter automatiquement la CMDB :

| Logiciel | Type |
|---|---|
| **Fusion Inventory** | Open source (plugin GLPI) |
| **NextThink** | Commercial |
| **SCCM** | Microsoft (System Center Configuration Manager) |

---

## 🏛️ 6. Les 3 Piliers de la Gestion de Parc

### Pilier 1 — ENTRETENIR
> Maintenir le parc en état de fonctionnement

| Action | Détail |
|---|---|
| Recensement | Inventaire via Fusion Inventory, SCCM… |
| Maintenance préventive | Mensuelle ou annuelle, matérielle ou fonctionnelle |
| Dépannage | Classifié par criticité via le Helpdesk |

### Pilier 2 — DÉVELOPPER
> Maîtriser l'évolution du parc

| Action | Détail |
|---|---|
| Renouvellement | Cycle de vie prédéfini (voir ci-dessous) |
| Prévision budgétaire | Budget établi en sept-nov, prévision sur N+3 ans |
| Gestion de l'expansion | Mise en conformité lors d'acquisitions |

### Pilier 3 — OPTIMISER
> Rendre chaque élément plus efficace

| Action | Détail |
|---|---|
| Protection | Outils de sécurité, veille technologique |
| Formation utilisateurs | Sensibilisation sécurité SI, RGPD, charte |
| Conformité | Matérielle et logicielle (méthode 5M) |

---

## ⏱️ 7. Cycles de Vie Recommandés

| Équipement | Durée recommandée |
|---|---|
| **PC fixe** | **5 ans** |
| **PC portable** | **3 ans** |
| **Serveur** | **5 ans** |
| **Écran** | **3 à 5 ans** |

### Risques si non respectés ⚠️
- Perte de performance générale
- Augmentation des pannes et des coûts de réparation
- **Fin de garantie constructeur** → coûts de support élevés
- Incompatibilité avec les logiciels récents
- **Risques de sécurité** (OS non supporté = pas de mises à jour)

---

## ✅ À retenir absolument

- [ ] CMDB = définition + 6 catégories de données
- [ ] GLPI = gestion **passive/statique**, open source, CMDB + helpdesk
- [ ] MDM = gestion **active/dynamique**, temps réel, appareils mobiles
- [ ] Tableau comparatif GLPI vs MDM (5 critères)
- [ ] 3 outils de découverte automatique
- [ ] Les 3 piliers : Entretenir / Développer / Optimiser
- [ ] Cycles de vie : PC fixe 5 ans, PC portable 3 ans, Serveur 5 ans, Écran 3-5 ans

---
*Fiche 4/5 — CP1 TSSR | Révision*
