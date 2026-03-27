# 📒 Fiche 5 — Mises en Situation & Qualité de Service (KPI)
**Formation TSSR | CP1 — Support utilisateur en centre de services**

---

## 🎭 1. Méthode de Traitement d'une Mise en Situation

Quand tu reçois un appel ou un ticket, applique ce réflexe en 3 temps :

### ① Collecter les informations
- Nom de l'utilisateur et du poste (→ CMDB)
- Depuis quand ? Modifications récentes ?
- Symptômes précis / messages d'erreur
- Données sauvegardées ?
- Combien de personnes sont impactées ?

### ② Prioriser avec la matrice
- Évaluer la **gravité** (l'utilisateur peut-il travailler ?)
- Évaluer l'**impact** (combien de personnes touchées ?)
- Croiser → obtenir la priorité (Mineur / Majeur / Critique)

### ③ Agir dans l'ordre
1. Créer le ticket (enregistrement)
2. Appliquer le diagnostic structuré
3. Escalader si nécessaire
4. Résoudre + documenter
5. Valider avec l'utilisateur
6. Clôturer

---

## 📋 2. Exemples de Priorisation — Cas Pratiques

### Cas 1 — « Mon PC ne s'allume plus, rapport dans 2h »
| Critère | Valeur |
|---|---|
| Gravité | **Urgente** (bloqué) |
| Impact | **Utilisateur** (lui seul) |
| Priorité | 🟡 **Majeur** |

**Actions dans l'ordre :**
1. Vérifier le câble d'alimentation + prise de courant
2. Tester sur secteur (problème batterie ?)
3. Écouter les bips POST au démarrage
4. Si toujours HS → **proposer un poste de remplacement** (urgence rapport !)
5. Escalader si problème matériel hors portée N1

---

### Cas 2 — Classement de 3 tickets simultanés

| Ticket | Situation | Gravité | Impact | Priorité | Ordre |
|---|---|---|---|---|---|
| **B** | Serveur inaccessible, toute l'entreprise bloquée | Critique | Entreprise | 🔴 Critique | **1er** |
| **A** | Imprimante HS, 20 personnes | Normal | Service | 🟡 Majeur | **2ème** |
| **C** | Logiciel inaccessible, 1 personne, solution alternative | Faible | Utilisateur | ⬜ Mineur | **3ème** |

> 💡 Si une **solution alternative** existe → la priorité baisse.

---

## 🔁 3. Incidents Récurrents — La Notion de Problème

### Situation typique
> Le même incident se produit plusieurs fois en quelques jours avec des utilisateurs différents.

### Comment réagir selon ITIL ?
| Étape | Action |
|---|---|
| **1. Identifier** | Relier tous les incidents → ouvrir un **ticket Problème** |
| **2. Analyser** | Lancer une analyse de cause première (Ishikawa) |
| **3. Contourner** | Mettre en place un workaround temporaire si possible |
| **4. Résoudre** | Traiter la cause racine définitivement |
| **5. Documenter** | Mettre à jour la base de connaissances |

### Risques si non traité
- ♾️ Récurrence infinie des incidents
- 📉 Dégradation de la QoS et satisfaction utilisateur
- 🔥 Surcharge du support (tickets inutiles répétés)

---

## ⬆️ 4. Gérer une Escalade — Le Bon Réflexe

### Quand escalader ?
- Temps SLA dépassé sans résolution
- Compétences ou droits insuffisants
- Impact trop important pour le niveau actuel

### Ce qu'on fait sur le ticket
1. Mettre à jour toutes les actions réalisées
2. Changer le statut → **"En cours d'escalade"**
3. Transférer au N2 avec le contexte complet

### Ce qu'on transmet au niveau supérieur
- Description précise du **symptôme**
- Toutes les **actions déjà effectuées** (et leurs résultats)
- Configuration du poste (via **CMDB**)
- **Contexte utilisateur** (urgence, contraintes métier)

### Comment gérer la communication utilisateur
- ✅ Informer calmement qu'on passe la main à un expert
- ✅ Donner une estimation du délai si possible
- ✅ Rester professionnel et empathique
- ❌ Ne pas promettre ce qu'on ne peut pas tenir
- ❌ Ne pas s'énerver si l'utilisateur s'impatiente

---

## 📊 5. Les KPI — Mesurer la Qualité du Support

> **KPI** = Key Performance Indicator = Indicateur clé de performance

| KPI | Ce qu'il mesure | Pourquoi c'est utile |
|---|---|---|
| **FCR** — First Call Resolution | % d'incidents résolus au premier contact | Mesure l'efficacité du N1 |
| **MTTR** — Mean Time To Resolve | Temps moyen de résolution | Évalue la réactivité du support |
| **Respect du SLA** | % d'incidents traités dans les délais contractuels | Mesure la conformité |
| **CSAT** — Customer Satisfaction | Satisfaction des utilisateurs | Qualité perçue |
| **Taux d'incidents récurrents** | Nb d'incidents identiques répétés | Indicateur de problèmes non résolus |
| **Taux d'escalade** | % d'incidents escaladés | Trop élevé = N1 sous-formé ou SLA mal calibré |

---

## 🧠 6. Mémo — Les Réflexes Pro à Avoir

| Situation | Bon réflexe |
|---|---|
| Plusieurs utilisateurs avec le même problème | Ouvrir un **ticket Problème**, pas juste des incidents |
| Bloqué depuis 30-45 min | **Escalader** sans attendre |
| Utilisateur urgent (réunion dans 1h, etc.) | Proposer un **contournement temporaire** en attendant |
| Incident résolu | Toujours **documenter** avant de clôturer |
| Clôturer un ticket | Toujours faire **valider l'utilisateur** d'abord |

---

## ✅ À retenir absolument

- [ ] Réflexe en 3 temps : Collecter → Prioriser → Agir
- [ ] Savoir prioriser un incident avec la matrice (gravité × impact)
- [ ] Notion de Problème selon ITIL (incidents récurrents)
- [ ] 5 informations à transmettre lors d'une escalade
- [ ] 6 KPI du support SI et leur signification
- [ ] Réflexes pro : workaround, escalade à temps, documentation systématique

---
*Fiche 5/5 — CP1 TSSR | Révision*
