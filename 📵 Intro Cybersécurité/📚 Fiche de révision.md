# Introduction à la Cybersécurité

## 📋 Sommaire

1. [Concepts Fondamentaux](https://claude.ai/chat/108212fc-ccfe-4923-884d-1b52988a9ae0#concepts-fondamentaux)
2. [Les Piliers de la Sécurité : DICP](https://claude.ai/chat/108212fc-ccfe-4923-884d-1b52988a9ae0#les-piliers-de-la-s%C3%A9curit%C3%A9--dicp)
3. [Politique de Sécurité (PSSI)](https://claude.ai/chat/108212fc-ccfe-4923-884d-1b52988a9ae0#politique-de-s%C3%A9curit%C3%A9-pssi)
4. [Analyse des Menaces et Vulnérabilités](https://claude.ai/chat/108212fc-ccfe-4923-884d-1b52988a9ae0#analyse-des-menaces-et-vuln%C3%A9rabilit%C3%A9s)
5. [L'Ingénierie Sociale](https://claude.ai/chat/108212fc-ccfe-4923-884d-1b52988a9ae0#ling%C3%A9nierie-sociale)
6. [Les Logiciels Malveillants](https://claude.ai/chat/108212fc-ccfe-4923-884d-1b52988a9ae0#les-logiciels-malveillants)
7. [Les Mots de Passe](https://claude.ai/chat/108212fc-ccfe-4923-884d-1b52988a9ae0#les-mots-de-passe)
8. [Conclusion](https://claude.ai/chat/108212fc-ccfe-4923-884d-1b52988a9ae0#conclusion)

---

## Concepts Fondamentaux

### Qu'est-ce que la Cybersécurité ?

La **cybersécurité** consiste à protéger le **Système d'Information (SI)** contre les risques de sécurité.

**Le Système d'Information (SI)** : ensemble organisé de ressources (ordinateurs, réseaux, données) qui permet à une organisation de collecter, stocker, traiter et distribuer l'information.

### Sécurité vs Sûreté

Il faut distinguer deux concepts différents :

|Sécurité|Sûreté|
|---|---|
|Protection contre les **actions malveillantes**|Protection contre les **dysfonctionnements et accidents**|
|Attaques intentionnelles|Pannes, erreurs naturelles|

---

## Les Piliers de la Sécurité : DICP

Pour assurer une bonne cybersécurité, tout SI doit respecter quatre piliers, connus sous l'acronyme **DICP** (ou **CIA** en anglais, auquel s'ajoute la Preuve) :

### 1️⃣ Disponibilité

Le service doit être **accessible aux personnes autorisées quand elles en ont besoin**.

- _Exemple_ : un serveur web doit fonctionner 24h/24

### 2️⃣ Intégrité

Les informations doivent être **exactes et complètes**, sans modification non autorisée.

- _Exemple_ : une facture ne doit pas être modifiée par un utilisateur malveillant

### 3️⃣ Confidentialité

L'information ne doit être accessible **que par les personnes autorisées**.

- _Exemple_ : vos données bancaires ne doivent pas être visibles par tous

### 4️⃣ Preuve

Capacité à **retrouver les responsables** des actions effectuées sur le SI.

Cette preuve repose sur trois éléments :

- **Traçabilité** : historique des modifications
- **Authentification** : reconnaître les utilisateurs
- **Imputabilité** : savoir qui a fait quoi

```
┌─────────────────────────────────┐
│      DICP (4 piliers)           │
├─────────────────────────────────┤
│ ✓ Disponibilité (Accès)         │
│ ✓ Intégrité (Exactitude)        │
│ ✓ Confidentialité (Secret)      │
│ ✓ Preuve (Traçabilité)          │
└─────────────────────────────────┘
```

---

## Politique de Sécurité (PSSI)

### Qu'est-ce que la PSSI ?

La **Politique de Sécurité du Système d'Information (PSSI)** est l'ensemble des règles et des mesures qu'une organisation met en place pour protéger son SI.

Elle est généralement définie par le **RSSI** (Responsable de la Sécurité des Systèmes d'Information), aussi appelé **CISO** en anglais.

### Comment construire une PSSI ?

**Étape 1 : Cartographier le SI**

- Identifier tous les éléments du système (serveurs, postes, logiciels, données)
- Distinguer la cartographie **physique** (matériel réel) et **virtuelle** (logiciels, données)

**Étape 2 : Analyse de Risques**

- Identifier les **menaces** potentielles
- Évaluer les **vulnérabilités**
- Définir le **modèle de menace** spécifique à l'organisation

**Étape 3 : Déployer des Solutions**

- Mettre en place les protections appropriées
- Adapter les mesures au contexte et aux besoins du SI

```
Cartographie SI → Identifier risques → Déployer solutions → Valider PSSI
```

---

## Analyse des Menaces et Vulnérabilités

### Les Trois Concepts Clés

#### 🔓 Vulnérabilité

**Faiblesse** de conception, réalisation, installation, configuration ou utilisation d'un système.

- _Exemple_ : un mot de passe trop simple, un logiciel sans mise à jour

#### ⚠️ Menace

**Cause potentielle** d'un dommage sur les éléments du SI (source extérieure ou interne).

- _Exemple_ : un attaquant externe, un employé mécontent

#### 🔥 Attaque

**Action concrète et malveillante** qui exploite une vulnérabilité pour réaliser une menace.

- _Exemple_ : un pirate qui utilise un mot de passe faible pour accéder au système

```
Vulnérabilité + Menace = Risque d'Attaque
```

### Étude des Vulnérabilités

Les vulnérabilités peuvent être classées selon :

**Par source** : d'où vient la menace ?

- Interne (employé)
- Externe (attaquant)
- Niveau de compétence et moyens de l'attaquant

**Par cible** : qu'est-ce qui est visé ?

- Logiciel
- Matériel
- Personnes (ingénierie sociale)

**Par nature** : quel type d'atteinte ?

- À la disponibilité (le système s'arrête)
- À l'intégrité (les données sont modifiées)
- À la confidentialité (les données sont fuites)

### Cycle de Traitement des Vulnérabilités

**🛡️ Prévenir** → Éviter que les vulnérabilités n'existent

**🔍 Détecter** → Savoir si et quand une attaque a lieu

**⚡ Réagir** → Décider de la réponse appropriée

**🔧 Réparer** → Remettre le SI en état opérationnel

**📈 Faire évoluer la PSSI** → Améliorer continuellement la sécurité

---

## L'Ingénierie Sociale

### Définition

**L'ingénierie sociale** est une attaque qui vise à **influencer les utilisateurs légitimes** pour qu'ils agissent dans l'intérêt du cybercriminel.

Contrairement aux attaques techniques, elle cible les **personnes**, pas les systèmes.

Les vecteurs d'attaque incluent : téléphone, email, réseaux sociaux, messages directs...

### Le Hameçonnage (Phishing)

L'**hameçonnage** est une technique d'ingénierie sociale très courante.

**Principe** : Mimer un site web ou un email légitime pour soutirer des informations sensibles.

**Cibles classiques** :

- Identifiants et mots de passe
- Numéros de carte bancaire
- Données personnelles

**Variantes** :

- **Phishing de masse** : envoi en masse à des milliers de personnes
- **Harponnage (Spear Phishing)** : ciblage personnalisé d'une personne ou d'une organisation

```
Email frauduleux → "Cliquez ici pour vérifier votre compte" 
                 → Vol d'identifiants
```

### Comment se Protéger de l'Ingénierie Sociale ?

**✅ Formation et sensibilisation** (pour TOUS les utilisateurs)

- Vérifier les URL avant de cliquer
- Analyser les métadonnées (expéditeur réel, domaine)
- Adopter un **regard critique** et vérifier les sources
- Douter des demandes urgentes ou inhabituelles

**✅ Mesures techniques**

- Utiliser des canaux de communication sécurisés et authentifiés
- Mettre en place des filtres antivirus sur la messagerie
- Implémenter des contrôles d'authentification forts

**✅ Bonnes pratiques**

- Ne jamais partager ses mots de passe
- Ne pas accepter les demandes suspectes
- Signaler les tentatives de phishing à son équipe IT

---

## Les Logiciels Malveillants

### Définition

Un **malware** (logiciel malveillant) est un programme s'installant dans un SI pour porter atteinte à :

- La **disponibilité** (le système s'arrête)
- L'**intégrité** (les données sont altérées)
- La **confidentialité** (les données sont exposées)

### Types de Malwares

#### Par Nature

**Virus**

- Contamine d'autres programmes
- S'exécute quand le programme hôte s'exécute
- Types : virus exécutables, macros, boot

**Vers**

- S'auto-réplique via des vulnérabilités réseau
- Ne nécessite pas de programme hôte
- Se propage rapidement d'un ordinateur à l'autre

#### Par Charge Utile

**Bombes logiques / Wipers**

- Effacent les données après une certaine date ou condition

**Rançongiciels (Ransomware)**

- Chiffrent les fichiers pour les rendre inaccessibles
- Exigent une rançon pour déchiffrer

**Chevaux de Troie (Trojans)**

- Se présentent comme un logiciel légitime
- Ouvrent des accès non autorisés au système
- Variante : **Backdoor** = porte dérobée pour accès futur

**Enregistreur de frappes (Keylogger)**

- Enregistre tous les caractères tapés au clavier
- Capture mots de passe et données sensibles

**Mouchards (Spyware)**

- Espionnent l'utilisateur et volent ses données

**Robots / Botnets**

- Ordinateurs contrôlés à distance
- Utilisés pour des attaques massives (DDoS)

```
┌──────────────────────────────────────┐
│     Classement des Malwares          │
├──────────────────────────────────────┤
│ Par nature : Virus, Vers             │
│ Par charge : Ransomware, Trojan...   │
│ Conséquence : Perte de contrôle      │
└──────────────────────────────────────┘
```

### Comment se Protéger des Malwares ?

**🔒 Fermer les portes**

- **Déployer des antivirus à jour** : détection et suppression des menaces connues
- **Limiter les droits** : appliquer le principe de moindre privilège (les utilisateurs n'ont que les permissions nécessaires)
- **Installer uniquement des logiciels de confiance** : sources officielles uniquement
- **Vérifier les téléchargements** : utiliser des empreintes ou signatures numériques
- **Limiter les périphériques d'entrée** : contrôler l'accès aux USB, CD...
- **Maintenir les systèmes à jour** : appliquer les patchs de sécurité

---

## Les Mots de Passe

### Définition

Un **mot de passe** est un moyen d'authentification basé sur la connaissance d'une **information secrète**.

On distingue :

- **Mot de passe** : court
- **Phrase de passe** : long (plus sécurisé)

### Caractéristiques

|Critère|Choisi|Généré|
|---|---|---|
|Création|L'utilisateur choisit|Générateur aléatoire|
|Mémorisation|Facile|Impossible (à noter)|
|Sécurité|Faible|Forte|

**Stockage** :

- En mémoire : mémorisé par l'utilisateur
- Stocké : dans un gestionnaire de mots de passe (chiffré)

### Les Attaques Courantes sur les Mots de Passe

#### 🔄 Force Brute

**Principe** : Tenter toutes les combinaisons possibles

- 4 chiffres = 10 000 combinaisons (0,01 secondes)
- 8 caractères alphanumériques = 2 x 10¹⁴ combinaisons (≈ 7 ans)
- 16 caractères ASCII = 10¹⁸ combinaisons (≈ 10 milliards d'années)

**Contre-mesure** :

- Augmenter la complexité (taille, variété de caractères)
- Limiter le nombre de tentatives
- Ralentir les tentatives (délai entre essais)

#### 📖 Attaque par Dictionnaire

**Principe** : Tester les mots de passe les plus probables

- Exemples courants : "123456", "qwerty", "password"

**Contre-mesure** :

- Interdire les mots de passe courants
- Limiter et ralentir les tentatives

#### 👁️ Capture en Clair

**Principe** : Lire simplement le mot de passe

- En transit (écoute réseau)
- Au stockage (base de données, système de fichiers, post-it)
- Lors de la saisie (enregistreur de frappe, caméra)

**Contre-mesure** :

- Chiffrer les communications réseau (TLS/SSL)
- Ne jamais stocker les mots de passe en clair
- Sécuriser son poste de travail
- Éviter de taper ses mots de passe en public

#### 🎮 Ingénierie Sociale

**Principe** : Convaincre l'utilisateur de révéler son mot de passe

- Prétexte d'une urgence
- Fausse autorité

**Contre-mesure** :

- Formation des utilisateurs
- Ne jamais demander un mot de passe par email/téléphone

### Le Problème des Mots de Passe

**Réalité désagréable** :

- Multiplication des services = réutilisation massive des mots de passe
- Les mots de passe sont un **compromis complexité / mémorisation**
- Les bases de données se font régulièrement **pirater** → fuites de données
- La plupart des utilisateurs ne respectent pas les bonnes pratiques

```
Plus de services = Plus de mots de passe = Réutilisation = Danger !
```

### 🔐 Bonne Solution : Hachage de Mots de Passe

**Problème** : Ne pas stocker les mots de passe en clair

**Solution** : Stocker l'**empreinte** (hash) du mot de passe, pas le mot de passe lui-même

**Fonctions de hachage** : Comparer l'empreinte ≈ Comparer le message

**Mais attention** : Même mot de passe = même empreinte → **Tables arc-en-ciel** (tables pré-calculées)

#### Salage

**Principe** : Ajouter un **sel** (valeur aléatoire mais non secrète) au moment du calcul de l'empreinte

**Avantage** :

- Mots de passe identiques → Empreintes différentes
- Élimine les tables arc-en-ciel

**Stockage** : empreinte + sel

**Coût de calcul** : Rendre le calcul volontairement long pour ralentir les attaques

**Algorithmes recommandés** :

- **Argon2** (moderne et recommandé)
- **bcrypt**
- **scrypt**
- **yescrypt**

```
┌────────────────────────────────────┐
│   Processus Sécurisé               │
├────────────────────────────────────┤
│ 1. Mot de passe + Sel             │
│ 2. Fonction de hachage             │
│ 3. Empreinte stockée               │
│ 4. À chaque connexion : comparer   │
│    l'empreinte, pas le mot         │
└────────────────────────────────────┘
```

### ✅ Bonnes Pratiques pour l'Utilisateur

**Gestionnaire de Mots de Passe**

- Stocke les mots de passe chiffrés
- Clé dérivée d'une phrase de passe principale
- Auto-remplissage sécurisé
- _Exemples_ : Bitwarden, 1Password, KeePass

**Règles d'Or**

- ✅ Utiliser un gestionnaire de mots de passe
- ✅ Générer des mots de passe longs et aléatoires
- ✅ Un mot de passe unique par service
- ✅ Changement régulier (si compromission suspectée)
- ❌ Ne jamais partager un mot de passe
- ❌ Ne jamais réutiliser le même mot de passe

---

## Conclusion

### Les Points Clés à Retenir

✅ **DICP** : Les 4 piliers de la sécurité (Disponibilité, Intégrité, Confidentialité, Preuve)

✅ **PSSI** : Politique documentée et mise en place par le RSSI

✅ **Analyse de risques** : Identifier vulnérabilités, menaces et attaques

✅ **Ingénierie sociale** : L'attaquant cible l'humain, pas juste la machine

✅ **Malwares** : Nombreuses formes, détection et prévention essentielles

✅ **Mots de passe** : Élément clé, à gérer correctement (hachage + sel)

### Hiérarchie des Menaces

```
1. Facteur humain (ingénierie sociale)
   ↓
2. Mots de passe faibles
   ↓
3. Logiciels malveillants
   ↓
4. Vulnérabilités techniques
```

### Pour Aller Plus Loin

- Familiariser tous les utilisateurs avec les bases de la cybersécurité
- Appliquer régulièrement les mises à jour
- Conduire des audits de sécurité réguliers
- Utiliser des outils spécialisés (gestionnaire de mots de passe, 2FA)
- Mettre en place une culture de sécurité dans l'organisation

---

## 📚 Sources

- Cours : _Introduction à la cybersécurité_ (documents fournis)
- Méthodologie EBIOS RM pour l'analyse de risques
- Recommandations OWASP et NIST en matière de cybersécurité