# 📙 Fiche 3 — ITIL, MCO & Démarche de Diagnostic
**Formation TSSR | CP1 — Support utilisateur en centre de services**

---

## 📚 1. ITIL — Le cadre méthodologique

### Définition
**ITIL** (Information Technology Infrastructure Library) est un **cadre de bonnes pratiques** pour la gestion des services informatiques (ITSM — IT Service Management).

### Objectifs
- Standardiser les processus de gestion des incidents
- Minimiser l'impact des perturbations sur le SI
- Améliorer la qualité de service et la satisfaction utilisateur

### Pourquoi ITIL et pas Agile ou Scrum ?

| | ITIL | Agile / Scrum |
|---|---|---|
| **Conçu pour** | Gestion opérationnelle des services IT | Développement logiciel et gestion de projets |
| **Usage** | Helpdesk, incidents, problèmes, changements | Sprints, backlogs, livraisons logicielles |
| **Approche** | Processus récurrents et stables | Itérations rapides et adaptatives |

> 💡 On utilise **ITIL** pour faire tourner un centre de services, pas pour développer une application.

---

## 🔧 2. Le MCO — Maintien en Condition Opérationnelle

### Définition
Le **MCO** désigne l'ensemble des actions permettant de s'assurer que le SI reste **disponible, fonctionnel et sécurisé** dans le temps.

### Le support SI est acteur direct du MCO via :
- La **gestion des incidents** (restaurer le service)
- La **maintenance préventive** (éviter les pannes)
- Les **mises à jour** (sécurité et performance)
- La **surveillance du parc** (monitoring/supervision)

> 🔑 **Pas de MCO = système qui se dégrade progressivement.**

---

## 🔎 3. Les 7 Étapes du Diagnostic Structuré

```
1. RECUEIL DES INFORMATIONS
        ↓
2. ANALYSE DES INFORMATIONS
        ↓
3. TEST DES HYPOTHÈSES
        ↓
4. DÉTERMINATION DE LA CAUSE
        ↓
5. PLANIFICATION DE LA RÉSOLUTION
        ↓
6. RÉSOLUTION
        ↓
7. SUIVI
```

### Détail

| Étape | Ce qu'on fait | Outils / Sources |
|---|---|---|
| **1. Recueil** | Collecter les données | Entretiens, logs, supervision (SolarWinds, PRTG, Wireshark) |
| **2. Analyse** | Identifier les causes possibles | Analyse OSI, Ishikawa |
| **3. Test** | Vérifier chaque hypothèse | Tests de régression, de charge, de sécurité |
| **4. Détermination** | Identifier la cause exacte | ⚠️ Étape critique — une erreur ici = actions inutiles |
| **5. Planification** | Organiser les actions correctives | Priorité, disponibilité ressources, impact utilisateur |
| **6. Résolution** | Mettre en œuvre les corrections | Actions techniques |
| **7. Suivi** | Vérifier post-résolution | Contrôle, retour utilisateur |

---

## 🌿 4. L'Analyse de la Cause Première (Root Cause Analysis)

### Principe
Remonter des **symptômes** jusqu'à la **cause profonde** plutôt que traiter les effets.

### Outil : Le Diagramme d'Ishikawa (5M)

```
                    ┌─────────────────────┐
  Main d'œuvre ────►│                     │
  Matériel     ────►│   EFFET / PROBLÈME  │
  Méthode      ────►│                     │
  Matière      ────►│                     │
  Milieu       ────►│                     │
                    └─────────────────────┘
```

**Les 5M :**
| M | Ce qu'il représente |
|---|---|
| **Main d'œuvre** | Erreur humaine, compétences |
| **Matériel** | Panne matérielle, vétusté |
| **Méthode** | Mauvaise procédure, configuration |
| **Matière** | Données corrompues, logiciel défectueux |
| **Milieu** | Environnement (réseau, température, alimentation) |

**Quand l'utiliser ?** Pour traiter un **problème** (incidents récurrents), pas un incident isolé.

---

## ⚠️ 5. Exemples d'Erreurs de Diagnostic

> Ces 3 cas illustrent pourquoi bien diagnostiquer **avant** d'agir est crucial.

| Symptôme | Diagnostic erroné | Action inutile | Vraie cause |
|---|---|---|---|
| Connexion réseau impossible | Problème de carte réseau | Remplacement carte | Mauvaise config routeur / câble |
| Serveur défaillant | Problème disque dur | Remplacement HDD | Mauvaise config carte mère |
| BDD lente | Manque de RAM | Ajout RAM | Mauvaise config allocation mémoire BDD |

**Conséquences d'un mauvais diagnostic :**
- Coûts supplémentaires inutiles
- Incident récurrent non résolu
- Perte de temps et dégradation du service

---

## 🧪 6. Les Types de Tests

| Test | Objectif |
|---|---|
| **Test de régression** | Vérifier qu'une modification n'a pas cassé autre chose |
| **Test de charge** | Vérifier les performances sous forte utilisation |
| **Test de sécurité** | Détecter les failles ou vulnérabilités |
| **Test fonctionnel** | Vérifier que la fonction concernée fonctionne correctement |

> 💡 L'**approche par couches OSI** est une méthode d'analyse réseau complémentaire (pas un type de test, mais une bonne pratique de diagnostic réseau).

---

## ✅ À retenir absolument

- [ ] Définition ITIL + différence avec Agile/Scrum
- [ ] Définition MCO + lien avec le support SI
- [ ] Les 7 étapes du diagnostic **dans l'ordre**
- [ ] Les 5M du diagramme d'Ishikawa
- [ ] Quand utiliser l'analyse de cause première
- [ ] 3 exemples d'erreurs de diagnostic
- [ ] 4 types de tests

---
*Fiche 3/5 — CP1 TSSR | Révision*
