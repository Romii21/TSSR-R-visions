## 📋 Sommaire

1. [Introduction au parc informatique](https://claude.ai/chat/893c59df-3765-4839-85d9-0acc0c45d2ed#1-introduction-au-parc-informatique)
2. [Les trois piliers de la gestion](https://claude.ai/chat/893c59df-3765-4839-85d9-0acc0c45d2ed#2-les-trois-piliers-de-la-gestion)
3. [Méthodes et outils de gestion](https://claude.ai/chat/893c59df-3765-4839-85d9-0acc0c45d2ed#3-m%C3%A9thodes-et-outils-de-gestion)
4. [La gestion des appareils mobiles (MDM)](https://claude.ai/chat/893c59df-3765-4839-85d9-0acc0c45d2ed#4-la-gestion-des-appareils-mobiles-mdm)
5. [Focus sur GLPI](https://claude.ai/chat/893c59df-3765-4839-85d9-0acc0c45d2ed#5-focus-sur-glpi)
6. [Points clés à retenir](https://claude.ai/chat/893c59df-3765-4839-85d9-0acc0c45d2ed#6-points-cl%C3%A9s-%C3%A0-retenir)

---

## 1. Introduction au parc informatique

### Définition

Un **parc informatique** regroupe l'ensemble des ressources matérielles et logicielles d'un système informatique.

### Composition du parc

|Catégorie|Éléments|
|---|---|
|**Ordinateurs**|Unités centrales, portables, écrans, claviers, souris|
|**Équipements réseaux**|Switchs, bornes WiFi, firewall, câblages|
|**Logiciels**|Applications et licences associées|
|**Appareils mobiles**|Smartphones, tablettes, lecteurs de code-barres|
|**Périphériques**|Imprimantes, copieurs, tablettes graphiques, caméras, disques durs externes|

### Acteurs concernés

- Personnel du SI (Système d'Information)
- Prestataires externes
- Utilisateurs référents hors SI

---

## 2. Les trois piliers de la gestion

### 2.1 Entretenir le parc

**Objectif** : Maintenir le parc en état de fonctionnement

|Action|Détails|
|---|---|
|**Recensement et localisation**|• Par découverte logicielle : Fusion Inventory, NextThink, SCCM<br>• Méthode humaine (recensement manuel)<br>• Données d'achat (bons de commande, réceptions)|
|**Sauvegarde des inventaires**|Utilisation d'une CMDB (Configuration Management Database)<br>Outils : GLPI, Ivanti Landesk|
|**Procédures d'administration**|• Procédures internes au SI<br>• Services Help-Desk et/ou Infrastructures Réseaux<br>• Maintenance interne ou externe|
|**Maintenance préventive**|• Mensuelle ou annuelle<br>• Matérielle ou fonctionnelle|
|**Dépannage**|Classification par criticité : standard, bloquant, urgent<br>Via le service Help-Desk|

**CMDB** = Configuration Management Database = Base de données qui stocke toutes les informations sur les équipements

### 2.2 Développer le parc

**Objectif** : Maîtriser l'évolution et l'expansion du parc

|Action|Détails|
|---|---|
|**Renouvellement matériel**|• Cycle de vie prédéfini<br>• Turn-over : achat tous les 4-6 ans pour 1/3 ou 1/4 du parc<br>• Leasing sur 3 ans (durée standard)|
|**Charte informatique**|Document définissant les règles d'utilisation|
|**Prévision budgétaire**|• Budgets établis en septembre-novembre<br>• Prévision sur N+3 ans|
|**Gestion de l'expansion**|• Évolution interne<br>• Acquisitions : mise en conformité et uniformisation|

**Cycles de vie recommandés** :

- PC fixe : **5 ans**
- PC portable : **3 ans**
- Serveur : **5 ans**
- Périphériques (écran) : **3-5 ans**

> ⚠️ Au-delà de ces durées : risque de perte de performance, augmentation des pannes, fin de garantie constructeur

### 2.3 Optimiser le parc

**Objectif** : Rendre chaque élément plus efficace

|Action|Détails|
|---|---|
|**Protection**|Outils de sécurité et veille technologique|
|**Formation utilisateurs**|• Sensibilisation à la sécurité SI<br>• Règles RGPD<br>• Charte informatique de bonne utilisation|
|**Gestion des prestataires**|Définir le niveau de connaissance technique à partager|
|**Conformité**|Matérielle et logicielle (méthode 5M, etc.)|

---

## 3. Méthodes et outils de gestion

### 3.1 Principes de gestion

**Comment gérer efficacement ?**

- Utilisation de logiciels spécialisés
- Mise en place de procédures
- Process qualité
- Uniformisation matérielle et logicielle
- Profils de postes standardisés
- Gestion des cycles de vie matériel

### 3.2 L'uniformisation

**Avantages** :

- Meilleure répartition matérielle
- Optimisation des process
- Réduction des coûts de maintenance et réparation
- Gestion budgétaire simplifiée

### 3.3 Logiciels de gestion de parc

#### BDD matériel - Fonctionnalités requises

- Modification, mise à jour, sauvegarde, suppression en temps réel ou par cycle (1h, 6h, 12h, 24h)
- Consultation et export des données
- Nettoyage régulier pour maintenir l'actualité

#### Logiciels disponibles

|Type|Logiciels|
|---|---|
|**Sous licence**|• Microsoft SCCM (System Center Configuration Manager)<br>• Ivanti LanDesk|
|**Libre et Open Source**|• GLPI (Logiciel Libre de Gestion de Parc Informatique)|

**SCCM** = System Center Configuration Manager

### 3.4 Informations conservées

#### Pour les ordinateurs :

|Catégorie|Données|
|---|---|
|**Identification**|Nom, code-barre, numéro de série (S/N)|
|**Référence matérielle**|Marque, constructeur, référence, modèle|
|**Référence réseau**|Adresse IP, adresse MAC|
|**Composants**|CPU, carte mère, disque dur, RAM, etc.|
|**Logiciels**|OS (système d'exploitation), pilotes, applications|
|**Domaine**|Domaine lié, OU (Organizational Unit)|
|**Statut**|En production, en stock, en réparation|
|**Informations budgétaires**|Prix, date d'achat, livraison, installation|
|**Utilisateurs**|Nom de l'utilisateur assigné|

#### Pour les autres équipements :

- **Périphériques** : nom, marque, modèle
- **Logiciels** : nom, licence, utilisateurs cibles

### 3.5 Les procédures

**Cycle de vie des procédures** :

|Élément|Description|
|---|---|
|**Métadonnées**|Date de création, n° de révision|
|**Cible**|Personnel concerné|
|**Objet**|But de la procédure|
|**Cycle de validation**|Processus d'approbation|
|**Cycle de révision**|Fréquence de mise à jour|

**Objectif** : Permettre aux utilisateurs authentifiés de gérer les tâches et actions

### 3.6 Process qualité

**Principe** : Convertir des éléments d'entrée en éléments de sortie

|Entrées|Sorties|
|---|---|
|• Demandes de service<br>• Alertes système<br>• Données d'inventaire<br>• Rapports d'incidents<br>• Politiques et normes de conformité|• Réponses aux demandes<br>• Amélioration système<br>• Rapports mis à jour<br>• Résolutions d'incidents<br>• Conformité aux politiques|

### 3.7 Méthode 5M (Ishikawa)

**Aussi appelée** : Diagramme d'Ishikawa ou "diagramme en arête de poisson"

**Origine** : Méthode japonaise de l'industrie automobile

**Objectif** : Analyser les liens de cause à effet d'un problème

#### Les 5M :

|M|Signification|Description|
|---|---|---|
|**Main d'œuvre**|Personnel|Interne/externe, savoir-faire, connaissances|
|**Matériel**|Équipements|Outils de travail, installations|
|**Méthode**|Procédures|Référentiels, instructions, notices|
|**Matière**|Ressources|Matières premières, énergie, composants|
|**Milieu**|Environnement|Locaux, ambiance, hiérarchie|

**Utilisation** : Identifier toutes les causes possibles d'un problème (ex : évaluation du taux de satisfaction client)

---

## 4. La gestion des appareils mobiles (MDM)

### 4.1 Appareils concernés

**En entreprise, il est crucial d'intégrer les appareils mobiles dans le parc informatique.**

|Catégorie|Exemples|
|---|---|
|**Smartphones et tablettes**|Téléphones professionnels, tablettes|
|**Appareils connectés**|IoT (Internet of Things)|
|**Appareils industriels**|Matériel durci mobile, terminaux de vente, scanners code-barres|
|**Ordinateurs mobiles**|Portables, hybrides|

**IoT** = Internet of Things = Objets connectés

### 4.2 Qu'est-ce qu'un MDM ?

**MDM** = Mobile Device Management = Gestion des Appareils Mobiles

**Définition** : Solution technologique permettant de gérer, sécuriser et surveiller les appareils mobiles en environnement professionnel.

### 4.3 Caractéristiques du MDM

|Type|MDM|
|---|---|
|**Nature**|Dynamique (gestion en temps réel)|
|**Mode**|Gestion active avec intervention directe|
|**Intégration**|S'intègre dans la stratégie globale de gestion de parc|

### 4.4 Fonctionnalités principales

- **Gestion en temps réel** : Nature dynamique
- **Intervention directe** : Actions immédiates sur les appareils
- **Gestion centralisée** : Tous les appareils depuis une console unique
- **Gestion système** : Mises à jour, politiques de sécurité
- **Gestion utilisateurs** : Installation d'applications, suivi de localisation

### 4.5 Logiciels MDM

**Exemples d'outils** :

- IBM MaaS 360
- MobileIron

---

## 5. Focus sur GLPI

### 5.1 Présentation

**GLPI** = Logiciel Libre de Gestion de Parc Informatique

**Type** : Solution Open Source qui combine :

- CMDB (Configuration Management Database)
- Gestion de help-desk

### 5.2 Rôle principal

**Conçu pour** :

1. La gestion d'inventaire de parc informatique
2. La gestion des services d'assistance (helpdesk)

**Permet de cataloguer** : ordinateurs, logiciels, périphériques, imprimantes, câbles, etc.

### 5.3 Type de gestion : Passive

|Caractéristique|GLPI|
|---|---|
|**Nature**|Statique|
|**Orientation**|Suivi et documentation des ressources|
|**Fonctions**|• Enregistrer les matériels<br>• Suivre les incidents<br>• Produire des rapports|
|**Temps réel**|Non (pas d'intervention en temps réel)|
|**Usage**|Gestion de parc et support informatique|

### 5.4 Comparaison MDM vs GLPI

|Critère|MDM|GLPI|
|---|---|---|
|**Type de gestion**|Active|Passive|
|**Nature**|Dynamique|Statique|
|**Temps réel**|Oui|Non|
|**Intervention**|Directe sur les appareils|Documentation et suivi|
|**Cible principale**|Appareils mobiles|Ensemble du parc informatique|

---

## 6. Points clés à retenir

### 🎯 L'essentiel

#### Définitions

- **Parc informatique** = Ensemble des ressources matérielles et logicielles
- **CMDB** = Base de données centralisant les informations de configuration
- **MDM** = Outil de gestion active des appareils mobiles en temps réel
- **GLPI** = Outil de gestion passive du parc et du helpdesk

#### Les 3 piliers de la gestion

1. **Entretenir** : Maintenir en état de fonctionnement (maintenance, dépannage, inventaire)
2. **Développer** : Gérer l'évolution (renouvellement, budget, expansion)
3. **Optimiser** : Améliorer l'efficacité (sécurité, formation, conformité)

#### Cycles de vie à respecter

|Équipement|Durée|
|---|---|
|PC fixe|5 ans|
|PC portable|3 ans|
|Serveur|5 ans|
|Écrans|3-5 ans|

#### Méthodes et outils essentiels

- **Uniformisation** : Réduction des coûts et simplification de la gestion
- **Méthode 5M** : Analyse des problèmes (Main d'œuvre, Matériel, Méthode, Matière, Milieu)
- **Process qualité** : Transformation des demandes en solutions
- **Procédures documentées** : Standardisation des actions

#### Outils logiciels

- **Gestion de parc** : GLPI (libre), SCCM, Ivanti LanDesk (payants)
- **Gestion mobile** : IBM MaaS 360, MobileIron
- **Découverte automatique** : Fusion Inventory, NextThink, SCCM

#### Données à conserver

Pour chaque équipement : identification, références matérielles/réseau, composants, logiciels, statut, informations budgétaires, utilisateur assigné

#### Acteurs de la gestion

- Personnel SI (équipes Help-Desk, Infrastructures Réseaux)
- Prestataires externes
- Utilisateurs référents

---

## 📚 Sources

- Document de cours "Suivi de parc informatique.pdf"
- Notes personnelles "Gestion de parc informatique.md"