### 1.1 Qu'est-ce qu'un journal (log) ?

Les **journaux** (ou **logs**) sont des enregistrements de l'activité d'un système, d'un serveur ou d'une application. Ils permettent de **tracer les événements** qui se produisent sur une machine ou un réseau.

|Caractéristique|Description|
|---|---|
|**Forme**|Fichiers texte, bases de données, format binaire...|
|**Format**|Défini par les développeurs de l'application|
|**Contenu**|Événements, erreurs, connexions, actions utilisateurs...|

> 💡 Chaque application et chaque système d'exploitation choisit son propre format de journal.

---

### 1.2 À quoi servent les logs ?

|Qui ?|Usage|
|---|---|
|**Développeurs**|Découvrir les causes d'un bug|
|**Administrateurs systèmes**|Comprendre les dysfonctionnements, analyser l'utilisation|
|**Cybersécurité**|Détecter des tentatives d'intrusion|

---

### 1.3 Historique et conservation

Idéalement, les journaux devraient être analysés **en temps réel**. Dans la pratique, il est indispensable de conserver un **historique** pour :

- Analyser des événements passés
- Relier des événements non consécutifs pour reconstituer un incident

---

### 1.4 Pertinence des informations journalisées

On ne journalise pas tout de la même façon : les informations à enregistrer **dépendent de l'usage**.

|Contexte|Exemples d'événements à journaliser|
|---|---|
|**Cybersécurité**|Échecs de connexion, requêtes invalides|
|**Administration**|Anomalies de fonctionnement, pannes|
|**Développement**|Erreurs applicatives, traces de débogage|

> ⚠️ Il faut **catégoriser** les informations et ajuster le niveau de journalisation selon le besoin.

---

### 1.5 Obligations légales

La journalisation n'est pas seulement une bonne pratique, c'est parfois une **obligation légale**.

|Réglementation|Description|
|---|---|
|**RGPD** (Règlement Général sur la Protection des Données) — Art. 32|Impose des mesures de sécurité incluant la journalisation|
|**LPM** (Loi de Programmation Militaire)|Obligations pour les opérateurs d'importance vitale|
|**HDS** (Hébergeur de Données de Santé)|Cadre spécifique aux données de santé|
|**Audit d'authentification**|Journalisation des succès/échecs de connexion (ok/nok)|

> ⚠️ **Points de vigilance :**
> 
> - **Durée de conservation** : elle est encadrée par la loi et varie selon le type de données
> - **Protection** : les logs eux-mêmes doivent être sécurisés pour éviter toute falsification

---

### 1.6 Stockage et archivage

Enregistrer toute l'activité peut rapidement consommer beaucoup d'espace disque, surtout en cas de dysfonctionnement (boucles d'erreurs, etc.).

La conservation de l'historique est donc un **compromis** entre :

- Le besoin d'information
- Les capacités de stockage disponibles

**Solutions :** utilisation de techniques d'**archivage** et de **compression** des anciens journaux.

---

### 1.7 Standardisation et centralisation

Analyser des journaux peut être complexe pour deux raisons principales :

|Problème|Solution|
|---|---|
|Les logs sont dispersés sur plusieurs machines|**Centralisation** (rassembler les logs en un seul endroit)|
|Les formats de logs sont hétérogènes|**Standardisation** (utiliser un format commun)|

---
