# 📚 La Cryptographie - Synthèse Complète

## 📑 Sommaire

1. [Introduction](https://claude.ai/chat/101ab17d-e665-42e1-aa0e-fa2749deb05d#introduction)
2. [Concepts Fondamentaux](https://claude.ai/chat/101ab17d-e665-42e1-aa0e-fa2749deb05d#concepts-fondamentaux)
3. [Cryptographie Symétrique](https://claude.ai/chat/101ab17d-e665-42e1-aa0e-fa2749deb05d#cryptographie-sym%C3%A9trique)
4. [Cryptographie Asymétrique](https://claude.ai/chat/101ab17d-e665-42e1-aa0e-fa2749deb05d#cryptographie-asym%C3%A9trique)
5. [Fonctions de Hachage](https://claude.ai/chat/101ab17d-e665-42e1-aa0e-fa2749deb05d#fonctions-de-hachage)
6. [Authentification Cryptographique](https://claude.ai/chat/101ab17d-e665-42e1-aa0e-fa2749deb05d#authentification-cryptographique)
7. [Les Certificats](https://claude.ai/chat/101ab17d-e665-42e1-aa0e-fa2749deb05d#les-certificats)
8. [Applications Pratiques](https://claude.ai/chat/101ab17d-e665-42e1-aa0e-fa2749deb05d#applications-pratiques)

---

## Introduction

### Qu'est-ce que la Cryptographie ?

La **cryptographie** est la science qui rend un message incompréhensible sauf pour son destinataire légitime. Son objectif principal est de **protéger les informations** contre les accès non autorisés.

**Vocabulaire clé :**

- **Clé** : un secret utilisé pour chiffrer/déchiffrer
- **Chiffrer** : transformer un message clair en message illisible
- **Déchiffrer** : récupérer le message clair avec la bonne clé
- **Décrypter** : récupérer le message clair **sans** la clé (très difficile, voire impossible)

### Les Objectifs Cryptographiques Majeurs

|Objectif|Définition|
|---|---|
|**Confidentialité**|Seul le destinataire peut lire le message|
|**Authenticité**|Vérifier l'identité de l'expéditeur|
|**Intégrité**|S'assurer que le message n'a pas été modifié|
|**Non-répudiation**|L'expéditeur ne peut pas nier avoir envoyé le message|

---

## Concepts Fondamentaux

### Historique Rapide

- **Antiquité** : Débuts de la cryptographie (César, chiffre de substitution)
- **WWII** : Automatisation avec Enigma
- **Années 1950** : Cryptographie mathématique (Shannon)
- **Années 1970** : Passage au numérique et cryptographie à clé publique
- **Aujourd'hui** : Cryptographie quantique en horizon

### Les Trois Piliers de la Cryptographie Moderne

```
┌─────────────────────────────────────────────┐
│     CRYPTOGRAPHIE MODERNE                   │
├─────────────────────────────────────────────┤
│                                             │
│  1️⃣ SYMÉTRIQUE        2️⃣ ASYMÉTRIQUE      │
│     (Une seule clé)    (Paire de clés)    │
│     Rapide ⚡          Flexible 🔐         │
│                                             │
│     3️⃣ HACHAGE                             │
│        (Empreinte fixe)                    │
│        Unidirectionnel 👇                  │
│                                             │
└─────────────────────────────────────────────┘
```

---

## Cryptographie Symétrique

### Concept Fondamental

La cryptographie symétrique utilise **une seule clé** pour chiffrer ET déchiffrer le message. C'est comme une serrure classique : même clé pour fermer et ouvrir.

```
MESSAGE CLAIR
     ↓
┌────────────────────┐
│   Chiffrement      │
│   + Clé Secrète    │
└────────────────────┘
     ↓
MESSAGE CHIFFRÉ (incompréhensible)
     ↓
┌────────────────────┐
│   Déchiffrement    │
│   + Même Clé       │
└────────────────────┘
     ↓
MESSAGE CLAIR (récupéré)
```

### Caractéristiques des Clés Symétriques

Une bonne clé symétrique doit être :

✅ **Aléatoire** : Imprévisible et impossible à deviner  
✅ **Longue** : Rendre une attaque par force brute trop coûteuse  
✅ **Secrète** : Partagée entre les correspondants uniquement

**Les 3 Défis :**

1. Générer les clés de manière aléatoire (générateurs pseudo-aléatoires)
2. Stocker les clés en sécurité (HSM - Hardware Security Module)
3. Partager les clés sans les exposer (problème clé de Diffie-Hellman)

### Chiffrement par Blocs vs. Flux

#### Chiffrement par Blocs

Divise le message en blocs de taille fixe et les chiffre l'un après l'autre.

**Modes opératoires (lien entre les blocs) :**

|Mode|Description|Sécurité|
|---|---|---|
|**ECB**|Chaque bloc chiffré indépendamment|❌ Non recommandé|
|**CBC**|Chaque bloc dépend du bloc précédent|✅ Recommandé|
|**CTR, CFB, OFB**|Modes de flux à partir de blocs|✅ Sûrs|

**Concept important : Vecteur d'initialisation (IV)**

- Aléatoire mais **pas secret** (peut être transmis avec le message)
- **Jamais réutilisé** avec la même clé !

#### Chiffrement de Flux

Chiffre le message **bit par bit** avec un générateur pseudo-aléatoire, généralement par XOR.

```
Message clair    : 1 0 1 0 1 1 0 1
Générateur PA    : 1 1 0 1 0 0 1 0
     XOR          ─────────────────
Message chiffré  : 0 1 1 1 1 1 1 1
```

### Algorithmes Symétriques Recommandés

|Algorithme|Type|Taille Clé|Statut|
|---|---|---|---|
|**AES** (Rijndael)|Bloc|128/192/256 bits|✅ Standard moderne|
|**ChaCha20**|Flux|256 bits|✅ Recommandé|
|**Blowfish**|Bloc|32-448 bits|⚠️ Obsolète|
|**DES / 3DES**|Bloc|56/168 bits|❌ Obsolète|

### Dérivation de Clés à partir de Mots de Passe

**Fonction PBKDF2** (Password-Based Key Derivation Function) : Transforme un mot de passe en clé cryptographique robuste.

⚠️ **Important** : Les mots de passe doivent être **forts et longs** !

### Cryptographie Hybride en Pratique

**Le problème** : Comment partager une clé symétrique de manière sûre ?

**La solution** : Combiner symétrique + asymétrique !

```
1️⃣  Alice génère une clé symétrique (session)
2️⃣  Alice la chiffre avec la clé publique de Bob
3️⃣  Bob déchiffre avec sa clé privée
4️⃣  Échange symétrique rapide et sûr ⚡🔐
```

---

## Cryptographie Asymétrique

### Concept Fondamental

La cryptographie asymétrique utilise **deux clés liées mathématiquement** :

```
┌──────────────────────────────────────┐
│   PAIRE DE CLÉS ASYMÉTRIQUES        │
├──────────────────────────────────────┤
│                                      │
│  🔓 CLÉ PUBLIQUE                    │
│     Distribuée à tout le monde      │
│     Utilisée pour CHIFFRER          │
│                                      │
│  🔐 CLÉ PRIVÉE                      │
│     Gardée secrète absolument       │
│     Utilisée pour DÉCHIFFRER        │
│                                      │
└──────────────────────────────────────┘
```

### Deux Applications Principales

#### 1. Confidentialité

```
Alice veut envoyer un secret à Bob

Alice              Bob
  │                 │
  ├─ Récupère clé publique de Bob
  ├─ Chiffre le message
  └─ Envoie message chiffré ──────→ Bob
                                    ├─ Utilise sa clé privée
                                    └─ Déchiffre ✅
```

#### 2. Authentification (Signature Numérique)

```
Bob veut prouver qu'il a envoyé un message

Bob                 Alice
  │                  │
  ├─ Chiffre empreinte avec clé privée (signature)
  └─ Envoie message + signature ──────→ Alice
                                       ├─ Déchiffre signature avec clé publique de Bob
                                       ├─ Compare avec empreinte du message
                                       └─ Si identique → authentifié ✅
```

### Algorithmes Asymétriques Recommandés

|Algorithme|Base Mathématique|Taille Clé|Statut|
|---|---|---|---|
|**RSA**|Factorisation premiers|3072-4096 bits|✅ Standard classique|
|**ECC** (Courbes elliptiques)|Logarithme discret|256-512 bits|✅ Recommandé moderne|
|**Ed25519** (EdDSA)|Edwards-curve|256 bits|✅ Idéal signatures|
|**DSA**|Logarithme discret|1024-3072 bits|❌ Obsolète|

### Points Clés

⚠️ **Algorithmes lents** : Chiffrement/déchiffrement beaucoup plus coûteux qu'en symétrique  
⚠️ **Clés plus longues nécessaires** : 3072 bits RSA vs 256 bits AES pour sécurité équivalente  
⚠️ **Menace quantique** : Théoriquement cassable par ordinateur quantique (Shor's algorithm)

---

## Fonctions de Hachage

### Concept Fondamental

Une fonction de hachage transforme **n'importe quelle donnée** en une **empreinte unique de taille fixe et petite**. C'est une **fonction à sens unique** : impossible de retrouver l'entrée à partir de la sortie.

```
ENTRÉE (n'importe quelle taille)
     ↓
┌──────────────────────┐
│   FONCTION HACHAGE   │
│   (SHA-256, MD5...)  │
└──────────────────────┘
     ↓
EMPREINTE (32 caractères hex pour SHA-256)
Ex: a665a45920422f9d417e4867efdc4fb8a04a1f3fff1fa07e998e86f7f7a27ae3

❌ IMPOSSIBLE d'inverser
❌ IMPOSSIBLE de retrouver l'entrée
```

### Propriétés Essentielles

|Propriété|Signification|Importance|
|---|---|---|
|**Déterministe**|Même entrée = même sortie|Permettre la vérification|
|**Effet avalanche**|Petite modification entrée → complètement différente|Détecter les modifications|
|**À sens unique**|Impossible d'inverser|Sécurité garantie|
|**Collision rare**|Impossible de trouver 2 entrées → même hash|Authenticité|

### Le Concept de Collision

**Collision** : Deux messages différents produisent la même empreinte.

```
Message A ──┐
            ├─→ HACHAGE ──→ MÊME EMPREINTE ❌
Message B ──┘
```

C'est le point faible : si on trouve une collision, on peut forger des messages !

### Fonctions de Hachage Recommandées

|Fonction|Taille Empreinte|Statut|Notes|
|---|---|---|---|
|**SHA-2**|224/256/384/512 bits|✅ Standard|SHA-256 le plus courant|
|**SHA-3** (Keccak)|Flexible|✅ Remplaçant moderne|Pas de collision connue|
|**Argon2**|Variable|✅ Mots de passe|Résistant aux attaques|
|**MD5**|128 bits|❌ Cassée|Collisions trouvées|
|**SHA-1**|160 bits|❌ Faible|À abandonner|

### Applications Pratiques

#### 1. Contrôle d'Intégrité de Messages

```
Envoyeur :
  ├─ Calcule HASH(message)
  └─ Envoie : MESSAGE + HASH

Récepteur :
  ├─ Reçoit : MESSAGE + HASH
  ├─ Calcule HASH(message reçu)
  └─ Compare :
     • Identique → Message intact ✅
     • Différent → Message modifié ⚠️
```

#### 2. Stockage Sécurisé de Mots de Passe

```
❌ MAUVAIS        ✅ BON
┌─────────────┐   ┌─────────────────────┐
│ Mot de passe│   │ HASH(mot de passe)  │
│ stocké      │   │ + SALT (aléatoire)  │
│ en clair ❌ │   │ stocké sécurisé ✅  │
└─────────────┘   └─────────────────────┘
```

**Concept de SALT** : Ajoute un nombre aléatoire avant de hacher. Deux mots de passe identiques donnent des hashes différents !

```
Mot de passe : "azerty"
Salt : "9f3$k2m"

HASH("azerty" + "9f3$k2m") = a1b2c3d4...
HASH("azerty" + "7x9Q1p8") = z9y8x7w6...  ← Différent !
```

**Fonctions spécialisées pour mots de passe :**

- **Bcrypt** : Basée sur Blowfish, très sûre
- **Argon2** : Gagnante de Password Hashing Competition
- **Yescrypt** : Par défaut sur Debian

#### 3. Vérification de Fichiers

```
Fichier original → HACHAGE → Empreinte A

Fichier téléchargé → HACHAGE → Empreinte B

Empreinte A = Empreinte B → Fichier correct ✅
Empreinte A ≠ Empreinte B → Fichier corrompu ⚠️
```

---

## Authentification Cryptographique

### Le Challenge

**Comment s'assurer que le message provient réellement de la personne affichée ?**

Deux approches : symétrique ou asymétrique.

---

### HMAC (Keyed-Hash Message Authentication Code)

**Concept** : Hachage + secret partagé

```
┌─────────────────────────────────────┐
│  HMAC = Authentification Symétrique │
└─────────────────────────────────────┘

ENVOI :
  Alice & Bob partagent clé secrète 🔑
  ├─ MESSAGE = "Salut Bob"
  ├─ HMAC = HASH(MESSAGE + CLÉ_SECRÈTE)
  └─ Envoie : MESSAGE + HMAC

RÉCEPTION :
  Bob reçoit MESSAGE + HMAC
  ├─ Calcule : HASH(MESSAGE_REÇU + CLÉ_SECRÈTE)
  ├─ Compare avec HMAC reçu
  └─ Match → Message authentique ✅
```

**Avantages** :

- Rapide ⚡
- Vérifie authentification + intégrité

**Limitations** :

- Nécessite clé secrète partagée
- Pas de preuve de non-répudiation (Bob pourrait avoir envoyé !)

---

### Signature Numérique (Asymétrique)

**Concept** : Authentification avec clé privée

```
┌──────────────────────────────────────┐
│   SIGNATURE NUMÉRIQUE (RSA, EdDSA)   │
└──────────────────────────────────────┘

ENVOI (Bob signe) :
  ├─ MESSAGE = "Accord deal"
  ├─ EMPREINTE = HASH(MESSAGE)
  ├─ SIGNATURE = CHIFFRE(EMPREINTE, CLÉ_PRIVÉE_BOB)
  └─ Envoie : MESSAGE + SIGNATURE

RÉCEPTION (Alice vérifie) :
  ├─ Reçoit : MESSAGE + SIGNATURE
  ├─ CALCULE_EMPREINTE = HASH(MESSAGE)
  ├─ DÉCHIFFRE_SIGNATURE = DÉCHIFFRE(SIGNATURE, CLÉ_PUBLIQUE_BOB)
  └─ Vérifie :
     • CALCULE_EMPREINTE = DÉCHIFFRE_SIGNATURE → Signé par Bob ✅
     • Sinon → Signature invalide ❌
```

**Avantages** :

- Authentification + Non-répudiation
- Pas besoin de clé partagée
- Preuve légale possible

---

### Chiffrement Authentifié

**Objectif** : Confidentialité + Authentification + Intégrité en même temps

**Formules :**

- **Encrypt-then-MAC (EtM)** ⭐ Recommandé
    
    ```
    1. Chiffrer le message
    2. Calculer MAC du chiffré
    3. Envoyer : chiffré + MAC
    ```
    
- **MAC-then-Encrypt (MtE)**
    
    ```
    1. Calculer MAC du message
    2. Chiffrer message + MAC
    ```
    
- **Modes authentifiés** (GCM, CCM)
    
    ```
    Fusion de chiffrement et authentification
    ```
    

⚠️ **Important** : Utiliser des clés différentes pour authentification et confidentialité !

---

## Les Certificats

### Pourquoi les Certificats ?

**Le problème fondamental** :

```
Alice veut vérifier que la clé publique qu'elle reçoit
appartient RÉELLEMENT à Bob et pas à un attaquant.
```

**Solution** : Un tiers de confiance **signe** la clé publique de Bob.

---

### Concept du Certificat

Un certificat numérique contient :

```
┌────────────────────────────────────────┐
│         CERTIFICAT NUMÉRIQUE           │
├────────────────────────────────────────┤
│                                        │
│  📋 CLÉ PUBLIQUE                      │
│  👤 IDENTITÉ (Nom, email, etc.)       │
│  📅 DATE VALIDITÉ (Du / Au)           │
│  🏢 INFORMATIONS D'ORGANISATION       │
│  ✍️  SIGNATURE du Tiers de Confiance  │
│                                        │
└────────────────────────────────────────┘
```

### Deux Standards Majeurs

#### 1. X.509 (Standard Hiérarchique)

- Format normalisé ITU
- Utilise une **Autorité de Certification (CA)**
- Modèle hiérarchique : certificats racines connus
- Exemple : Let's Encrypt

```
Racine (Auto-signé)
  ↓
Autorité intermédiaire
  ↓
Certificat serveur HTTPS
```

#### 2. OpenPGP (Approche Décentralisée)

- Protocole IETF (RFC 4880)
- Version ouverte de PGP (Philip Zimmermann)
- **Toile de confiance (Web of Trust)** : Chacun certifie les autres
- Moins d'autorités centrales
- Exemple : GNU Privacy Guard (GPG)

```
Utilisateur A certifie Utilisateur B
Utilisateur B certifie Utilisateur C
→ Réseau décentralisé de confiance
```

---

### Cycle de Vie d'un Certificat

#### Étape 1 : Création (CSR - Certificate Signing Request)

```
1. Générer paire de clés publique/privée
   ↓
2. Créer CSR (informations + clés publiques)
   ↓
3. Envoyer au Tiers de Confiance
```

#### Étape 2 : Vérification et Signature

```
Tiers de Confiance reçoit CSR
   ↓
Vérifie l'identité du demandeur
   ↓
Signe avec sa clé privée
   ↓
Retourne certificat signé
```

#### Étape 3 : Utilisation et Vérification

```
À réception d'un nouveau certificat :

1. Vérifier identité du propriétaire
   ↓
2. Vérifier signature du certificat
   └─ Besoin de clé publique du Tiers
      └─ Charger certificat du Tiers
         └─ Vérifier son certificat...
            └─ CHAÎNE DE CERTIFICATION
               
3. Si signature valide → Certificat approuvé ✅
```

**Problème circulaire** : Comment vérifier le certificat du Tiers ?

→ **Solution** : Certificats racines pré-installés dans le système !

#### Étape 4 : Révocation (Quand tout va mal)

**Cas de révocation** :

- Vol/perte de clé privée
- Compromission identité
- Fin de validité anticipée

**Méthodes de diffusion** :

- **CRL** (Certificate Revocation List) : Liste noire périodique
- **OCSP** (Online Certificate Status Protocol) : Vérification en direct
- **OCSP Stapling** : Agrafage OCSP pour performance

---

### Public Key Infrastructure (PKI)

**PKI** = Ensemble complet pour émettre des certificats :

```
┌──────────────────────────┐
│   PUBLIC KEY INFRASTRUCTURE (PKI)
├──────────────────────────┤
│                          │
│  🖥️  Matériel           │
│  └─ Serveurs            │
│  └─ HSM (sécurité)      │
│                          │
│  💾 Logiciel             │
│  └─ Autorité de CA       │
│  └─ Registre             │
│                          │
│  📋 Procédures           │
│  └─ Vérification ID      │
│  └─ Renouvellement       │
│  └─ Révocation           │
│                          │
└──────────────────────────┘
```

---

### ACME (Automated Certificate Management Environment)

**Problème** : Certificats nécessitent renouvellement régulier

**Solution** : Automatiser complètement le processus

```
┌─────────────────────────────────┐
│  ACME PROTOCOL (Let's Encrypt)  │
├─────────────────────────────────┤
│                                 │
│  ✅ Automatisation complète     │
│  ✅ Certificats courte durée    │
│  ✅ Renouvellement sans effort  │
│  ✅ Gratuit & open source       │
│                                 │
│  Implémentation : certbot       │
│                                 │
└─────────────────────────────────┘
```

---

## Applications Pratiques

### 1. HTTPS (Web Sécurisé)

```
Protocole : TLS (Transport Layer Security)

CLIENT (Navigateur)              SERVEUR
    │                               │
    ├─ Demande certificat ─────────→
    │                               │
    ├─← Reçoit certificat (X.509)  │
    │                               │
    ├─ Vérifie certificat           │
    │   • Signature valide ?        │
    │   • Domaine correspond ?      │
    │   • Dates valides ?           │
    │                               │
    ├─ Échange clé de session ─────→
    │  (chiffrement asymétrique)    │
    │                               │
    ├─ Communication symétrique ───→
    │  (AES + HMAC)                 │
    │                               │
```

**Indicateurs visuels** :

- 🔒 Cadenas vert → Certificat valide
- ⚠️ Cadenas gris → Certificat auto-signé
- ❌ Cadenas rouge → Certificat invalide

---

### 2. Email Sécurisé

#### S/MIME (Secure/Multipurpose Internet Mail Extensions)

- Standard X.509
- Chiffrement + Signature
- Support natif dans clients principaux

#### PGP/MIME (OpenPGP)

- Approche alternative
- Plus décentralisée
- Pour utilisateurs techniques

```
EMAIL SIGNÉ :
Expéditeur
  ├─ Message
  ├─ Signature (clé privée)
  └─ Clé publique (certificat)

Destinataire
  ├─ Vérifie signature
  ├─ Récupère clé publique
  └─ Confirme authenticité ✅
```

---

### 3. VPN (Réseau Privé Virtuel)

**OpenVPN** :

- Certificats X.509
- Chiffrement bout en bout
- Authentification mutuelle client-serveur

**IPsec** :

- Standard réseau
- Support certificats
- Niveau transport

---

### 4. Autres Applications

- **SSH** : Connexion serveur (clés asymétriques)
- **Code signing** : Certifier l'authenticité de logiciels
- **Blockchain** : Signatures transactions
- **IoT** : Authentification devices

---

## 📋 Tableau Récapitulatif

|Type|Clés|Vitesse|Usage|Exemple|
|---|---|---|---|---|
|**Symétrique**|1 (secrète)|⚡ Rapide|Chiffrement données|AES-256|
|**Asymétrique**|2 (pub/priv)|🐢 Lent|Signature & échange clés|RSA, Ed25519|
|**Hachage**|0 (sens unique)|⚡ Très rapide|Intégrité & mots de passe|SHA-256|
|**HMAC**|1 (secrète)|⚡ Rapide|Authentification|HMAC-SHA256|

---

## 🎯 Points Clés à Retenir

✅ **Cryptographie symétrique** = Rapide, partagée, c'est le cœur des communications  
✅ **Cryptographie asymétrique** = Flexible, signatures, mais lent  
✅ **Hachage** = Sens unique, vérification intégrité et mots de passe  
✅ **Certificats** = Association clé publique ↔ identité via Tiers de confiance  
✅ **Hybride** = Combiner symétrique (rapide) + asymétrique (flexible)  
⚠️ **Menace quantique** = Développement cryptographie post-quantique en cours  
⚠️ **Clés longues** = Plus grande taille pour sécurité équivalente asymétrique

---

## 📚 Sources et Références

- **Documents fournis** : Note & Cryptographie.pdf
- **Standards** :
    - RFC 8017 (PKCS #1: RSA)
    - RFC 8018 (PBKDF2)
    - RFC 4880 (OpenPGP)
    - NIST SP 800-38 (Modes de chiffrement par blocs)
- **Organisations** :
    - NIST (National Institute of Standards and Technology)
    - Let's Encrypt (ACME & PKI)
    - IETF (Internet Engineering Task Force)

---

**Dernière mise à jour** : 2025  
**Niveau** : Débutant à intermédiaire  
**Durée de lecture** : ~20-30 minutes