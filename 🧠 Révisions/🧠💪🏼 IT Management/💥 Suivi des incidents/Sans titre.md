# Synthèse - Suivi des incidents informatiques

## 📋 Sommaire

1. [Définitions essentielles](https://claude.ai/chat/cc03ef54-5931-4476-8cf4-2bbc46d8b1f1#1-d%C3%A9finitions-essentielles)
2. [Introduction à ITIL](https://claude.ai/chat/cc03ef54-5931-4476-8cf4-2bbc46d8b1f1#2-introduction-%C3%A0-itil)
3. [Organisation d'une équipe Support SI](https://claude.ai/chat/cc03ef54-5931-4476-8cf4-2bbc46d8b1f1#3-organisation-dune-%C3%A9quipe-support-si)
4. [Gestion des incidents - Procédure complète](https://claude.ai/chat/cc03ef54-5931-4476-8cf4-2bbc46d8b1f1#4-gestion-des-incidents---proc%C3%A9dure-compl%C3%A8te)
5. [Démarche de diagnostic](https://claude.ai/chat/cc03ef54-5931-4476-8cf4-2bbc46d8b1f1#5-d%C3%A9marche-de-diagnostic)
6. [🎯 Points clés à retenir](https://claude.ai/chat/cc03ef54-5931-4476-8cf4-2bbc46d8b1f1#-points-cl%C3%A9s-%C3%A0-retenir)

---

## 1. Définitions essentielles


## 2. Introduction à ITIL


## 3. Organisation d'une équipe Support SI

### Le rôle du Support SI

Le **Support SI** (aussi appelé Helpdesk, Hot line, Centre de services) fait partie du système opérationnel de l'entreprise.

**Missions principales :**

- Faciliter l'exploitation du parc informatique
- Assister les utilisateurs (technique, formation)
- Gérer le MCO (Maintien en Condition Opérationnelle)
- Maintenir un catalogue de services
- Mettre en place des processus qualité (ITIL, ISO)

### Compétences requises

- **Communication** et relation utilisateur
- **Diagnostic** et résolution rapide
- **Connaissances techniques** approfondies (logiciels, matériels, OS, réseaux)
- **Gestion de projet** et du temps
- **Capacité d'adaptation** et apprentissage continu

### Niveaux de support

|Niveau|Rôle|
|---|---|
|**Niveau 0**|Enregistrement, catégorisation|
|**Niveau 1**|Niveau 0 + priorisation, résolution (procédures standards)|
|**Niveau 2**|Analyse, résolution (analyse approfondie), suivi|
|**Niveau 3**|Analyse, résolution (expertise avancée), suivi|

### SLA (Service Level Agreement)

Contrat définissant les **niveaux de service attendus** :

- **Temps de réponse** : délai selon la criticité (critique, majeur, mineur)
- **Disponibilité** : taux de disponibilité des serveurs, réseau, etc.
- **Qualité** : taux de résolution, satisfaction utilisateurs

---

## 4. Gestion des incidents - Procédure complète

### Vue d'ensemble

|Ordre|Processus|
|---|---|
|1|Identification / Détection|
|2|Notification|
|3|Enregistrement|
|4|Catégorisation et priorisation|
|5|Diagnostic et investigation|
|6|Suivi (ou escalade)|
|7|Résolution (et documentation)|
|8|Clôture|

---

### 1️⃣ Identification / Détection

**Objectif :** Déterminer qu'un incident s'est produit (séparation du fonctionnement inhabituel de l'état normal)

**Moyens de détection :**

- Personnes (utilisateurs, techniciens)
- Logiciels de supervision (monitoring)
- Outils de supervision système et réseau

**⚠️ À distinguer des autres demandes :**

- Demandes de service (création compte, réinitialisation mot de passe)
- Demandes d'information (conseil, documentation)

---

### 2️⃣ Notification

**Objectif :** Signaler l'incident au support SI

**Canaux de notification :**

- Téléphone
- Email
- SMS
- Logiciel de gestion d'incidents (ticketing)
- Face à face

**Informations à fournir :**

- État actuel avec détails
- Type d'incident
- Utilisateur(s) impacté(s)

---

### 3️⃣ Enregistrement

**Objectif :** Créer le **ticket d'incident**

**Format :**

- Formulaire logiciel
- Formulaire web
- Enregistrement dans une BDD (Base De Données)

---

### 4️⃣ Catégorisation et Priorisation

#### Catégorisation

Classement selon :

- **Type d'incident** : téléphonie, messagerie, réseaux, etc.
- **Gravité** : faible, normal, urgent, critique
- **Impact** : nombre de personnes touchées
- **Urgence**

#### Niveaux de gravité

|Gravité|Description|
|---|---|
|**Faible**|L'utilisateur peut continuer à travailler, problème peu gênant|
|**Normal**|L'utilisateur peut continuer mais c'est gênant, à résoudre dans la journée|
|**Urgent**|L'utilisateur est bloqué|
|**Critique**|L'utilisateur ne peut plus travailler|

#### Niveaux d'impact

|Impact|Périmètre|
|---|---|
|Utilisateur|Une personne|
|Service|Un service/département|
|Site|Un site géographique|
|Entreprise|Toute l'entreprise|

#### Matrice de priorisation

|Niveau d'impact → <br> Gravité ↓|Utilisateur|Service|Site|Entreprise|
|---|---|---|---|---|
|**Faible**|Mineur|Mineur|Majeur|Majeur|
|**Normal**|Mineur|Majeur|Majeur|Critique|
|**Urgent**|Majeur|Majeur|Critique|Critique|
|**Critique**|Majeur|Critique|Critique|Critique|

**Priorités :**

- **Mineur** : faible impact, résolution à long terme
- **Majeur** : gênant, intervention rapide nécessaire
- **Critique** : bloquant, procédure de gestion de crise

---

### 5️⃣ Diagnostic et Investigation

**Objectif :** Analyser la situation pour arriver à une résolution

**Actions :**

- Comprendre pourquoi l'incident s'est produit
- Déterminer les actions à entreprendre
- Utiliser les informations de la notification
- Consulter la CMDB (Configuration Management Database)

---

### 6️⃣ Suivi (Escalade)

**Objectif :** Assurer le suivi des incidents avec des résolutions longues

**Actions :**

- Relancer un intervenant si nécessaire
- Relancer un prestataire
- Relancer l'utilisateur
- Escalader vers un niveau supérieur si besoin

---

### 7️⃣ Résolution et Documentation

#### Résolution

**Objectif :** Corriger l'incident pour revenir à un état d'utilisation normal

**Actions :**

- Mettre en œuvre les actions correctives
- Vérifier que l'incident est résolu
- Restaurer le service

#### Documentation

**Objectif :** Consigner les informations pour l'avenir

**Supports :**

- Wiki (ex : Redmine)
- FAQ (Foire Aux Questions)
- Base de connaissances
- Gestion documentaire

**Importance :** Permet d'éviter la récurrence et de capitaliser sur les résolutions

#### Communication

**Objectif :** Informer les personnes concernées

**Actions :**

- Communiquer sur la situation
- Expliquer la résolution
- Obtenir un retour utilisateur
- Valider la satisfaction

---

### 8️⃣ Clôture

**Objectif :** Fermer officiellement l'incident

**Conditions :**

- Service restauré
- Confirmation de la résolution
- Utilisateurs satisfaits

---

## 5. Démarche de diagnostic

### Importance du diagnostic

Un bon diagnostic permet :

- ✅ Une identification correcte de l'incident
- ✅ L'efficacité des actions de résolution
- ✅ L'amélioration de la QoS (Qualité de Service)
- ✅ Des économies (temps, coûts) - ROI positif
- ✅ La prévention des futurs incidents
- ✅ L'amélioration de la satisfaction utilisateur

---

### Étapes du diagnostic

|Étape|Description|
|---|---|
|**1. Recueil des informations**|Collecter les données nécessaires|
|**2. Analyse des informations**|Examiner et déterminer les causes possibles|
|**3. Test des hypothèses**|Vérifier les hypothèses par des tests|
|**4. Détermination de la cause**|Identifier la cause exacte|
|**5. Planification de la résolution**|Organiser les actions correctives|
|**6. Résolution**|Mettre en œuvre les corrections|
|**7. Suivi**|Assurer le suivi post-résolution|

---

### 1️⃣ Recueil d'informations

**Sources d'information :**

- Entretiens avec les utilisateurs
- Journaux d'événements (logs)
- Retours de supervision (monitoring)
- Outils de diagnostic (SolarWinds, PRTG, Wireshark)

---

### 2️⃣ Analyse des informations

**Méthodes d'analyse :**

|Méthode|Description|
|---|---|
|**Analyse de la chaîne de valeur**|Décomposer le système en parties (ex : modèle OSI)|
|**Analyse de la cause première**|Remonter aux causes profondes (ex : Diagramme d'Ishikawa)|

---

### 3️⃣ Test des hypothèses

**Types de tests :**

|Test|Objectif|
|---|---|
|**Test de régression**|Vérifier qu'une modification n'a pas causé de problème ailleurs|
|**Test de charge**|Vérifier la performance sous forte utilisation|
|**Test de sécurité**|Détecter les failles de sécurité|

---

### 4️⃣ Détermination de la cause

**⚠️ Importance de la précision !**

Une **mauvaise détermination** peut entraîner :

- Actions inappropriées
- Coûts supplémentaires inutiles
- Récurrence de l'incident

#### Exemples d'erreurs de diagnostic

**Exemple 1 :**

- **Symptôme :** Problème de connexion réseau
- **Diagnostic erroné :** Problème de carte réseau
- **Action :** Remplacement de la carte réseau
- **Résultat :** Problème persiste
- **Vraie cause :** Mauvaise configuration du routeur

**Exemple 2 :**

- **Symptôme :** Serveur défaillant
- **Diagnostic erroné :** Problème de disque dur
- **Action :** Remplacement du disque dur
- **Résultat :** Problème persiste
- **Vraie cause :** Mauvaise configuration de la carte mère

**Exemple 3 :**

- **Symptôme :** BDD lente
- **Diagnostic erroné :** Manque de RAM
- **Action :** Ajout de RAM
- **Résultat :** Problème persiste
- **Vraie cause :** Mauvaise configuration de l'allocation mémoire de la BDD

---

### 5️⃣ Planification de la résolution

**Éléments à prendre en compte :**

- **Priorité** de l'incident (critique, majeur, mineur)
- **Disponibilité** des ressources (personnel, matériel, temps)
- **Impact** sur les utilisateurs (minimiser les perturbations)

**Objectif :** Planifier de manière méthodique pour assurer l'efficacité et minimiser l'impact

---

## 🎯 Points clés à retenir

### Définitions fondamentales

- **Incident** = Effet (événement imprévu perturbant le SI)
- **Problème** = Cause (cause inconnue d'un ou plusieurs incidents)

### Méthodologie ITIL

- Cadre de bonnes pratiques pour l'ITSM
- Approche structurée pour minimiser les impacts

### Les 8 étapes de gestion d'incident

1. **Identification/Détection** → Repérer l'anomalie
2. **Notification** → Alerter le support
3. **Enregistrement** → Créer le ticket
4. **Catégorisation/Priorisation** → Classer et hiérarchiser
5. **Diagnostic** → Analyser la cause
6. **Suivi** → Suivre l'avancement
7. **Résolution/Documentation** → Corriger et documenter
8. **Clôture** → Fermer le ticket

### Priorisation : Gravité × Impact

- **Mineur** : faible impact, résolution différée
- **Majeur** : gênant, intervention rapide
- **Critique** : bloquant, gestion de crise

### Niveaux de support

- **N0** : Enregistrement
- **N1** : Résolution procédure
- **N2** : Analyse approfondie
- **N3** : Expertise avancée

### Diagnostic efficace

- Recueillir les informations complètes
- Analyser méthodiquement
- Tester les hypothèses
- **Précision cruciale** pour éviter coûts inutiles et récurrence

### Documentation

- Capitaliser sur les résolutions
- Éviter la récurrence
- Construire une base de connaissances

### Communication

- Informer les utilisateurs concernés
- Valider la satisfaction
- Maintenir la confiance

---

## 📚 Sources

- Documents fournis : "Suivi des incidents.pdf" et "Suivi des incidents.md"
- Méthodologie ITIL (Information Technology Infrastructure Library)

---

_Synthèse réalisée à partir du cours "Suivi des incidents" - Support de formation_