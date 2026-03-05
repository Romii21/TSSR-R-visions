## 🟢 Partie 1 — Fondamentaux du support SI

---

### Q1. Citez quatre appellations courantes utilisées pour désigner un centre de services.

**Ta réponse :** « Je peux citer la SI »

**❌ Insuffisant** — "La SI" désigne le Système d'Information dans son ensemble, pas le centre de services.

**✅ Réponse attendue :**

- **Helpdesk**
- **Hot line**
- **Centre de services**
- **Support informatique** (ou DSI selon le contexte)

> 💡 Ces termes désignent tous l'équipe chargée d'assister les utilisateurs, mais avec des nuances : le Helpdesk est souvent réactif, le Centre de services est plus structuré (ITIL).

---

### Q2. Distinction incident / problème avec exemples concrets.

**Ta réponse :** Bonne compréhension de base, mais l'exemple est confus (oubli de sauvegarde → c'est une erreur humaine, pas vraiment un incident ITIL standard).

**✅ Réponse attendue :**

|Incident|Problème|
|---|---|---|
|**Définition**|Événement imprévu perturbant le SI|La cause inconnue d'un ou plusieurs incidents|
|**C'est...**|L'**effet**|La **cause**|
|**Exemple**|Un utilisateur ne peut pas se connecter au réseau|La mauvaise configuration du service DHCP qui a empêché plusieurs connexions|

> 💡 Un même **problème** peut générer plusieurs **incidents**. ITIL distingue les deux pour traiter la cause racine séparément des symptômes.

---

### Q3. Les cinq grandes missions d'une équipe de support SI.

**Ta réponse :** Tu en as cité deux, il en manque trois.

**✅ Réponse attendue :**

1. **Faciliter l'exploitation du parc informatique**
2. **Assister les utilisateurs** (technique et formation)
3. **Gérer le MCO** (Maintien en Condition Opérationnelle)
4. **Maintenir un catalogue de services**
5. **Mettre en place des processus qualité** (ITIL, ISO)

---

### Q4. Les quatre niveaux de support N0 à N3.

**Ta réponse :** Non renseignée.

**✅ Réponse attendue :**

|Niveau|Rôle|
|---|---|
|**N0**|Enregistrement et catégorisation des incidents (premier contact)|
|**N1**|N0 + priorisation + résolution par procédures standards|
|**N2**|Analyse approfondie et résolution sur des cas plus complexes, suivi|
|**N3**|Expertise avancée (éditeurs, constructeurs, spécialistes externes)|

---

### Q5. Qu'est-ce qu'un SLA ? Ses trois éléments principaux.

**Ta réponse :** Non renseignée.

**✅ Réponse attendue :** Le **SLA (Service Level Agreement)** est un contrat définissant les niveaux de service attendus entre le support SI et ses utilisateurs/clients.

Ses trois éléments principaux :

1. **Temps de réponse** : délai de prise en charge selon la criticité
2. **Disponibilité** : taux de disponibilité des services (ex : 99,9% pour un serveur critique)
3. **Qualité** : taux de résolution, satisfaction utilisateur

---

### Q6. Canaux de notification d'un incident.

**Ta réponse :** GLPI, mail, téléphone ✅ — il en manquait.

**✅ Réponse complète :**

1. Téléphone
2. Email
3. SMS
4. Logiciel de ticketing (ex : GLPI)
5. **Face à face** (en direct)

> 💡 GLPI n'est pas un canal de notification à proprement parler, c'est l'outil de gestion. Le canal est plutôt « formulaire web » ou « portail utilisateur ».

---

## 🟡 Partie 2 — Gestion des incidents

---

### Q7. Les 8 étapes de la gestion d'un incident dans l'ordre.

**Ta réponse :** Non renseignée.

**✅ Réponse attendue :**

1. **Identification / Détection** — repérer l'anomalie
2. **Notification** — alerter le support
3. **Enregistrement** — créer le ticket
4. **Catégorisation et Priorisation** — classer et hiérarchiser
5. **Diagnostic et Investigation** — analyser la cause
6. **Suivi (ou Escalade)** — suivre l'avancement
7. **Résolution et Documentation** — corriger et documenter
8. **Clôture** — fermer officiellement le ticket

---

### Q8. Formes de création d'un ticket à l'enregistrement.

**Ta réponse :** Non renseignée (numérotée Q7 par erreur dans ton fichier).

**✅ Réponse attendue :**

- **Formulaire logiciel** (ex : interface GLPI)
- **Formulaire web** (portail utilisateur)
- **Enregistrement direct en BDD** (Base De Données)

---

### Q9. Les quatre niveaux de gravité et d'impact.

**Ta réponse :** ✅ Bonne réponse ! Tu as correctement identifié les niveaux.

**✅ Complément pour être précis :**

**Gravité :**

|Gravité|Description|
|---|---|
|**Faible**|L'utilisateur peut continuer à travailler, problème peu gênant|
|**Normal**|L'utilisateur peut continuer mais c'est gênant|
|**Urgent**|L'utilisateur est bloqué|
|**Critique**|L'utilisateur ne peut plus du tout travailler|

**Impact :**

|Impact|Périmètre|
|---|---|
|Utilisateur|1 personne|
|Service|1 service/département|
|Site|1 site géographique|
|Entreprise|Toute l'entreprise|

---

### Q10. Matrice de priorisation — 4 situations.

**Ta réponse :** Bonne logique générale, mais classement incorrect et vocabulaire non conforme à la matrice.

**✅ Réponse attendue (avec la matrice) :**

|Impact → / Gravité ↓|Utilisateur|Service|Site|Entreprise|
|---|---|---|---|---|
|**Faible**|Mineur|Mineur|Majeur|Majeur|
|**Normal**|Mineur|Majeur|Majeur|**Critique**|
|**Urgent**|Majeur|Majeur|Critique|Critique|
|**Critique**|Majeur|**Critique**|Critique|Critique|

- **a) Gravité Urgente / Impact Utilisateur → MAJEUR**
- **b) Gravité Critique / Impact Service → CRITIQUE**
- **c) Gravité Normale / Impact Entreprise → CRITIQUE**
- **d) Gravité Faible / Impact Site → MAJEUR**

> ⚠️ Ton classement était : c > b > d > a, ce qui est **partiellement correct** (c et b à égalité Critique, d et a à égalité Majeur). Mais b et c sont tous les deux **Critique** donc ex-aequo. Bonne intuition globale !

---

### Q11. La CMDB dans la gestion d'incident — à quelle étape et quel rôle ?

**Ta réponse :** Non renseignée.

**✅ Réponse attendue :** La CMDB intervient à l'étape **5 — Diagnostic et Investigation**.

Son rôle : fournir au technicien toutes les informations sur la configuration de l'équipement concerné (matériel, logiciels installés, historique, dépendances) pour aider à identifier la cause de l'incident rapidement.

---

### Q12. Déclenchement de l'escalade.

**Ta réponse :** ✅ Bonne compréhension. À compléter sur les types d'escalade.

**✅ Réponse complète :** L'escalade se déclenche quand :

- Le technicien du niveau actuel ne peut pas résoudre l'incident dans le délai SLA
- L'incident dépasse ses compétences ou ses droits d'accès

**Deux types d'escalade :**

- **Escalade fonctionnelle** : vers un niveau supérieur (N1 → N2 → N3)
- **Escalade hiérarchique** : vers le management (si impact majeur ou client important)

**Exemples :**

1. Un N1 ne parvient pas à résoudre une panne serveur après 30 min → escalade vers N2
2. Incident impactant tout le site → escalade hiérarchique pour décision de crise

---

### Q13. Pourquoi documenter après résolution ?

**Ta réponse :** ✅ Très bonne réponse ! Tu as bien compris l'enjeu.

**✅ Compléments — 3 supports de documentation :**

1. **Wiki** (ex : Redmine, Confluence)
2. **FAQ** (Foire Aux Questions)
3. **Base de connaissances** intégrée à GLPI

---

### Q14. Conditions pour clôturer un ticket.

**Ta réponse :** ✅ Bonne réponse globale.

**✅ Réponse complète :**

- Le service a été **restauré**
- La résolution a été **confirmée**
- L'utilisateur a **validé** que tout fonctionne (test de sa part)
- L'utilisateur est **satisfait**

---

## 🟠 Partie 3 — Méthodes ITIL et qualité de service

---

### Q15. Qu'est-ce qu'ITIL ? Pourquoi pas Agile/Scrum ?

**Ta réponse :** Non renseignée.

**✅ Réponse attendue :** **ITIL** (Information Technology Infrastructure Library) est un cadre de **bonnes pratiques** pour la gestion des services informatiques (ITSM).

**Pourquoi ITIL et pas Agile/Scrum ?**

- **Agile/Scrum** : conçus pour le développement logiciel et la gestion de projets itératifs
- **ITIL** : spécifiquement conçu pour la gestion opérationnelle des services IT (incidents, problèmes, changements…)

> 💡 On utilise ITIL pour structurer les processus du helpdesk, pas pour développer une application.

---

### Q16. Qu'est-ce que le MCO ?

**Ta réponse :** Non renseignée.

**✅ Réponse attendue :** Le **MCO (Maintien en Condition Opérationnelle)** désigne l'ensemble des actions permettant de s'assurer que le SI reste disponible, fonctionnel et sécurisé dans le temps.

Le support SI en est acteur direct car il gère : les incidents, les mises à jour, la maintenance préventive, la surveillance du parc — toutes des actions qui maintiennent le SI en état de marche.

---

### Q17. Les 7 étapes d'une démarche de diagnostic structurée.

**Ta réponse :** Non renseignée.

**✅ Réponse attendue :**

1. **Recueil des informations** (utilisateur, logs, supervision)
2. **Analyse des informations** (identifier les causes possibles)
3. **Test des hypothèses** (vérifier chaque piste)
4. **Détermination de la cause** (identifier la cause exacte)
5. **Planification de la résolution** (organiser les actions)
6. **Résolution** (mettre en œuvre les corrections)
7. **Suivi** (vérification post-résolution)

---

### Q18. L'analyse de la cause première.

**Ta réponse :** Non renseignée.

**✅ Réponse attendue :** L'**analyse de la cause première** (ou Root Cause Analysis) consiste à remonter des symptômes jusqu'à la cause profonde d'un incident, plutôt que de traiter uniquement les effets.

**Outil graphique :** Le **Diagramme d'Ishikawa** (aussi appelé diagramme en arêtes de poisson ou diagramme des 5M : Main d'œuvre, Matériel, Méthode, Matière, Milieu).

**Quand l'utiliser :** Pour traiter un **problème** (cause récurrente d'incidents), pas un incident isolé.

---

### Q19. Identifier les erreurs de diagnostic.

**Ta réponse :** Bonne piste pour a), b) et c) non complétées.

**✅ Réponse attendue :**

**a) Connexion réseau impossible → remplacement carte réseau → problème persiste**

- ✅ Ta réponse était bonne : câble débranché, ou mauvaise configuration IP/routeur
- La vraie cause probable était une **mauvaise configuration du routeur** ou **un câble défectueux**
- Conséquence : achat matériel inutile + perte de temps

**b) Serveur défaillant → remplacement disque dur → problème persiste**

- La vraie cause probable : **mauvaise configuration de la carte mère** ou problème d'alimentation
- Conséquence : remplacement coûteux inutile, serveur toujours indisponible

**c) Base de données lente → ajout RAM → problème persiste**

- La vraie cause probable : **mauvaise configuration de l'allocation mémoire de la BDD** (ex : paramètre `innodb_buffer_pool_size` trop faible sous MySQL)
- Conséquence : coût matériel sans amélioration des performances

> 💡 Ces trois exemples illustrent l'importance de diagnostiquer **avant** d'agir.

---

### Q20. Types de tests pour valider une hypothèse.

**Ta réponse :** Tu as cité le modèle OSI (approche valide mais partielle).

**✅ Réponse complète :**

|Test|Objectif|
|---|---|
|**Test de régression**|Vérifier qu'une modification n'a pas cassé autre chose|
|**Test de charge**|Vérifier les performances sous forte utilisation|
|**Test de sécurité**|Détecter les failles ou vulnérabilités|
|**Test fonctionnel**|Vérifier que la fonction concernée marche correctement|

> 💡 L'approche par couches OSI est une **méthode d'analyse**, pas un type de test en soi — mais elle est pertinente pour le diagnostic réseau.

---

## 🔴 Partie 4 — Outils et gestion de parc

---

### Q21. GLPI — type de gestion et signification.

**Ta réponse :** Non renseignée.

**✅ Réponse attendue :** **GLPI** (Logiciel Libre de Gestion de Parc Informatique) est une solution **open source** combinant CMDB et helpdesk.

**Type de gestion : Passive (Statique)** Cela signifie que GLPI ne peut **pas intervenir directement** sur les machines en temps réel. Il enregistre, suit et documente les ressources, mais n'exécute pas d'actions à distance sur les postes.

---

### Q22. Comparaison GLPI vs MDM.

**Ta réponse :** Non renseignée.

**✅ Réponse attendue :**

|Critère|MDM|GLPI|
|---|---|---|
|**Type de gestion**|Active|Passive|
|**Nature**|Dynamique|Statique|
|**Temps réel**|Oui|Non|
|**Type d'intervention**|Directe sur les appareils|Documentation et suivi|
|**Cible principale**|Appareils mobiles|Ensemble du parc informatique|

---

### Q23. Qu'est-ce qu'une CMDB ? 6 catégories d'informations.

**Ta réponse :** Non renseignée.

**✅ Réponse attendue :** Une **CMDB** (Configuration Management Database) est une base de données centralisant toutes les informations de configuration du parc informatique.

**6 catégories minimum pour un ordinateur :**

1. **Identification** : nom, code-barre, numéro de série
2. **Référence matérielle** : marque, modèle, constructeur
3. **Référence réseau** : adresse IP, adresse MAC
4. **Composants** : CPU, RAM, disque dur, carte mère
5. **Logiciels** : OS, pilotes, applications installées
6. **Statut** : en production / en stock / en réparation
7. _(Bonus)_ **Informations budgétaires** : prix, date d'achat
8. _(Bonus)_ **Utilisateur assigné**

---

### Q24. Logiciels de découverte automatique pour alimenter une CMDB.

**Ta réponse :** Non renseignée.

**✅ Réponse attendue :**

1. **Fusion Inventory** (plugin GLPI, open source)
2. **NextThink**
3. **SCCM** (Microsoft System Center Configuration Manager)

---

### Q25. Les trois piliers de la gestion de parc + actions concrètes.

**Ta réponse :** Non renseignée.

**✅ Réponse attendue :**

|Pilier|Actions concrètes|
|---|---|
|**1. Entretenir**|Maintenance préventive, dépannage par criticité|
|**2. Développer**|Renouvellement matériel, gestion budgétaire|
|**3. Optimiser**|Formation des utilisateurs, mise en conformité sécurité|

---

### Q26. Cycles de vie recommandés et risques.

**Ta réponse :** Non renseignée.

**✅ Réponse attendue :**

|Équipement|Cycle de vie|
|---|---|
|PC fixe|**5 ans**|
|PC portable|**3 ans**|
|Serveur|**5 ans**|
|Écran|**3 à 5 ans**|

**Risques si non respectés :**

- Perte de performance générale
- Augmentation des pannes et des coûts de réparation
- Fin de garantie constructeur → coûts de support élevés
- Incompatibilité avec les logiciels récents
- Risques de sécurité (OS non supporté)

---

## 🔴🔴 Partie 5 — Mise en situation et analyse critique

---

### Q27. « Mon ordinateur ne s'allume plus, rapport dans 2h »

**Ta réponse :** Partiellement correcte.

**✅ Réponse complète :**

**a) Informations à collecter :**

- Nom de l'utilisateur et du poste (pour chercher dans CMDB)
- Depuis quand le problème est apparu
- Des modifications récentes (mise à jour, chute, coupure de courant ?)
- Si le PC est sur batterie ou sur secteur
- Messages d'erreur affichés
- Si des données importantes sont sauvegardées (et où)

> ✅ Ta réponse sur la batterie était bonne ! Mais l'identité du poste et l'historique sont prioritaires.

**b) Priorité (matrice) :**

- Gravité **Urgente** (bloqué) / Impact **Utilisateur** (lui seul) → **MAJEUR**
- ✅ Tu avais la bonne combinaison, mais en ajoutant le contexte du rapport urgent, on peut argumenter une montée en urgence.

**c) Actions dans l'ordre :**

1. Créer le ticket et l'enregistrer
2. Vérifier que le câble d'alimentation est branché et que la prise fonctionne
3. Tester la batterie (essai sur secteur)
4. Vérifier les voyants et bips POST au démarrage
5. Si toujours HS : proposer un poste de remplacement pour ne pas bloquer la remise du rapport
6. Escalader si problème matériel non résolvable N1

---

### Q28. Classement des tickets A, B, C.

**Ta réponse :** ✅ Excellent ! Classement et justifications corrects.

**✅ Validation :**

- **Ticket B** (serveur inaccessible, toute l'entreprise) → Critique / Entreprise → **Priorité 1** ✅
- **Ticket A** (imprimante, 20 personnes) → Majeur / Service → **Priorité 2** ✅
- **Ticket C** (logiciel, 1 personne, solution alternative) → Mineur / Utilisateur → **Priorité 3** ✅

> 💡 Très bonne analyse. La précision sur le fait qu'il existe des solutions alternatives pour le ticket C est un excellent réflexe professionnel.

---

### Q29. Même incident signalé 5 fois en 3 jours.

**Ta réponse :** Non renseignée.

**✅ Réponse attendue :**

**a) Selon ITIL :** Cette situation caractérise un **Problème** (cause inconnue d'incidents répétés). Il faut ouvrir un **ticket Problème** distinct des tickets incidents.

**b) Démarche à engager :**

1. Relier tous les incidents dans un même problème dans l'outil de ticketing
2. Lancer une **analyse de cause première** (Ishikawa, logs…)
3. Mettre en place un **contournement temporaire** si possible
4. Documenter et résoudre la cause racine

**c) Risques si non traité :**

- Récurrence infinie des incidents
- Dégradation de la QoS et de la satisfaction utilisateur
- Surcharge du support (tickets inutiles répétés)
- Possible impact croissant si le problème s'aggrave

---

### Q30. Technicien N1 bloqué depuis 45 min, utilisateur impatient.

**Ta réponse :** ✅ Bonne réponse globale et professionnelle.

**✅ Compléments :**

**a) Action sur le ticket :**

- Mettre le ticket à jour avec toutes les actions réalisées
- Modifier le statut : "en cours d'escalade"
- **Escalader vers le N2** (escalade fonctionnelle)

**b) Communication avec l'utilisateur :**

- Informer calmement qu'on passe la main à un expert
- Donner une estimation du délai si possible
- Ne pas promettre ce qu'on ne peut pas tenir
- Rester professionnel, ne pas s'énerver

**c) Informations à transmettre au N2 :**

- Description précise du symptôme
- Toutes les actions déjà effectuées (et leurs résultats)
- Configuration du poste (via CMDB)
- Contexte utilisateur (urgence, contraintes)
- ✅ Tu avais bien cité ces éléments !

---

### Q31. KPI pour mesurer la qualité du support.

**Ta réponse :** Non renseignée.

**✅ Réponse attendue :**

|KPI|Justification|
|---|---|
|**Taux de résolution au premier contact (FCR)**|Mesure l'efficacité du N1 à résoudre sans escalade|
|**Temps moyen de résolution (MTTR)**|Évalue la réactivité du support|
|**Respect du SLA (% d'incidents traités dans les délais)**|Mesure la conformité contractuelle|
|**Taux de satisfaction utilisateur (CSAT)**|Évalue la qualité perçue par les utilisateurs|
|**Nombre d'incidents récurrents**|Indicateur de problèmes non résolus à la racine|
|**Taux d'escalade**|Trop élevé = N1 sous-formé ou SLA mal calibré|

---

## 📊 Bilan global

|Zone|Maîtrise|
|---|---|
|Incident vs Problème|✅ Bonne base, à affiner|
|Matrice de priorisation|✅ Bonne logique, vocabulaire à préciser|
|Étapes de gestion d'incident|⚠️ À retravailler (plusieurs réponses vides)|
|Niveaux de support|⚠️ À mémoriser précisément|
|GLPI / CMDB|❌ Non traité, à revoir|
|Diagnostic structuré|⚠️ Bonne intuition, méthode à formaliser|
|Mises en situation|✅ Bon sens professionnel, très bon ticket B/C/A|
|ITIL / MCO / SLA|❌ Non traité, à revoir en priorité|

---

### 🎯 Points prioritaires à retravailler :

1. Les 8 étapes de gestion d'incident **dans l'ordre**
2. La définition et le fonctionnement du **SLA**
3. **ITIL** et le **MCO**
4. **GLPI vs MDM** (tableau comparatif)
5. Les **KPI** du support SI

---

_Correction CP1 — TSSR | Mars 2026_