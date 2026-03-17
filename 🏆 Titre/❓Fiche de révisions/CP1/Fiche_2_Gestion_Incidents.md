# 📗 Fiche 2 — Gestion des Incidents
**Formation TSSR | CP1 — Support utilisateur en centre de services**

---

## 🔄 1. Les 8 Étapes de Gestion d'un Incident

> **À connaître dans l'ordre !**

```
1. IDENTIFICATION / DÉTECTION
        ↓
2. NOTIFICATION
        ↓
3. ENREGISTREMENT (création du ticket)
        ↓
4. CATÉGORISATION & PRIORISATION
        ↓
5. DIAGNOSTIC & INVESTIGATION
        ↓
6. SUIVI (ou ESCALADE)
        ↓
7. RÉSOLUTION & DOCUMENTATION
        ↓
8. CLÔTURE
```

### Détail de chaque étape

| Étape | Objectif | Moyens / Actions |
|---|---|---|
| **1. Identification** | Détecter l'anomalie | Utilisateurs, supervision, monitoring |
| **2. Notification** | Alerter le support | Téléphone, email, ticketing, SMS, face à face |
| **3. Enregistrement** | Créer le ticket | Formulaire logiciel, formulaire web, BDD |
| **4. Catégorisation** | Classer et prioriser | Gravité × Impact → priorité |
| **5. Diagnostic** | Analyser la cause | Logs, CMDB, outils de supervision |
| **6. Suivi** | Suivre les incidents longs | Relance intervenant, escalade si besoin |
| **7. Résolution** | Corriger + documenter | Actions correctives + wiki/FAQ/base de connaissance |
| **8. Clôture** | Fermer officiellement | Confirmation utilisateur + validation satisfaction |

---

## 🎯 2. Catégorisation — Gravité × Impact

### Niveaux de Gravité
| Gravité | Description |
|---|---|
| **Faible** | L'utilisateur peut travailler, peu gênant |
| **Normal** | L'utilisateur peut continuer mais c'est gênant |
| **Urgent** | L'utilisateur est bloqué |
| **Critique** | L'utilisateur ne peut plus du tout travailler |

### Niveaux d'Impact
| Impact | Périmètre touché |
|---|---|
| **Utilisateur** | 1 personne |
| **Service** | 1 service / département |
| **Site** | 1 site géographique |
| **Entreprise** | Toute l'entreprise |

---

## 🧮 3. Matrice de Priorisation

| Gravité ↓ / Impact → | Utilisateur | Service     | Site        | Entreprise  |
| -------------------- | ----------- | ----------- | ----------- | ----------- |
| **Faible**           | ⬜ Mineur    | ⬜ Mineur    | 🟡 Majeur   | 🟡 Majeur   |
| **Normal**           | ⬜ Mineur    | 🟡 Majeur   | 🟡 Majeur   | 🔴 Critique |
| **Urgent**           | 🟡 Majeur   | 🟡 Majeur   | 🔴 Critique | 🔴 Critique |
| **Critique**         | 🟡 Majeur   | 🔴 Critique | 🔴 Critique | 🔴 Critique |

### Signification des priorités
| Priorité        | Action                                   |
| --------------- | ---------------------------------------- |
| ⬜ **Mineur**    | Résolution à long terme, faible urgence  |
| 🟡 **Majeur**   | Intervention rapide nécessaire           |
| 🔴 **Critique** | Bloquant → procédure de gestion de crise |

### Exemples à mémoriser
| Situation | Résultat |
|---|---|
| Gravité Urgente + Impact Utilisateur | 🟡 **Majeur** |
| Gravité Critique + Impact Service | 🔴 **Critique** |
| Gravité Normale + Impact Entreprise | 🔴 **Critique** |
| Gravité Faible + Impact Site | 🟡 **Majeur** |

---

## 🔍 4. La CMDB dans la gestion d'incident

**Quand ?** À l'étape **5 — Diagnostic et Investigation**

**Rôle :** Fournir au technicien toutes les infos sur l'équipement concerné :
- Configuration matérielle et logicielle
- Historique des incidents précédents
- Dépendances avec d'autres équipements
- Utilisateur assigné

---

## ⬆️ 5. L'Escalade

### Quand déclencher une escalade ?
- Le technicien **ne peut pas résoudre** dans le délai SLA
- L'incident **dépasse ses compétences** ou droits d'accès

### Deux types d'escalade
| Type | Vers qui | Exemple |
|---|---|---|
| **Fonctionnelle** | Niveau supérieur (N1→N2→N3) | Panne serveur non résolue par N1 |
| **Hiérarchique** | Management | Incident impactant tout le site |

---

## 📝 6. Documentation et Clôture

### Pourquoi documenter ?
- Éviter la **récurrence** de l'incident
- Capitaliser pour les collègues
- Construire une base de connaissances

### Supports de documentation
1. **Wiki** (ex : Redmine, Confluence)
2. **FAQ** (Foire Aux Questions)
3. **Base de connaissances** intégrée au GLPI

### Conditions de clôture ✅
- Service **restauré**
- Résolution **confirmée**
- Utilisateur a **testé et validé**
- Utilisateur est **satisfait**

---

## ✅ À retenir absolument

- [ ] Les 8 étapes **dans l'ordre**
- [ ] 4 gravités + 4 impacts
- [ ] La matrice de priorisation (au moins les cas extrêmes)
- [ ] Rôle de la CMDB à l'étape diagnostic
- [ ] 2 types d'escalade
- [ ] 3 supports de documentation
- [ ] Conditions de clôture

---
*Fiche 2/5 — CP1 TSSR | Révision*
