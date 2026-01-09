

## 📋 Table des matières

```table-of-contents
title: 
style: nestedList # TOC style (nestedList|nestedOrderedList|inlineFirstLevel)
minLevel: 2 # Include headings from the specified level
maxLevel: 2 # Include headings up to the specified level
include: 
exclude: 
includeLinks: true # Make headings clickable
hideWhenEmpty: false # Hide TOC if no headings are found
debugInConsole: false # Print debug info in Obsidian console
```

---

## 🎯 Introduction à la sécurisation {#introduction}

La sécurisation des accès à un routeur Cisco est **critique** pour la protection de votre infrastructure réseau. Sans sécurisation appropriée, n'importe qui ayant un accès physique ou réseau pourrait modifier la configuration, consulter des informations sensibles ou compromettre l'ensemble du réseau.

> [!info] Pourquoi sécuriser les accès ?
> 
> - Empêcher les modifications non autorisées de la configuration
> - Protéger contre les accès malveillants (externes et internes)
> - Répondre aux exigences de conformité et d'audit
> - Tracer les actions effectuées sur l'équipement

### Les différents types d'accès

|Type d'accès|Description|Ligne à sécuriser|
|---|---|---|
|**Console**|Accès physique via câble console|`line console 0`|
|**Enable**|Mode privilégié (configuration)|`enable secret`|
|**VTY**|Accès distant (Telnet/SSH)|`line vty 0 4` ou plus|

---

## 🖥️ Mots de passe console {#console}

La **ligne console** est le port physique sur le routeur permettant une connexion directe via un câble console. C'est souvent le premier point d'accès lors de la configuration initiale.

### Pourquoi protéger la console ?

Même si l'accès console nécessite une présence physique, il reste vulnérable :

- Accès en salle serveur par du personnel non autorisé
- Intervention de maintenance par des tiers
- Vol ou accès physique non contrôlé

### Configuration de base

```bash
Router> enable
Router# configure terminal
Router(config)# line console 0
Router(config-line)# password votreMotDePasse
Router(config-line)# login
Router(config-line)# exit
```

> [!warning] Attention Sans la commande `login`, le mot de passe configuré ne sera **pas demandé** ! C'est une erreur très fréquente.

### Configuration avancée recommandée

```bash
Router(config)# line console 0
Router(config-line)# password Cons0le@2024!
Router(config-line)# login
Router(config-line)# exec-timeout 5 0          # Déconnexion après 5 min d'inactivité
Router(config-line)# logging synchronous       # Empêche les messages système d'interrompre la saisie
Router(config-line)# exit
```

> [!tip] Astuce - Timeout personnalisé La commande `exec-timeout` utilise le format `minutes secondes`. Par exemple :
> 
> - `exec-timeout 5 0` = 5 minutes exactement
> - `exec-timeout 0 30` = 30 secondes
> - `exec-timeout 0 0` = Pas de timeout (déconseillé !)

### Vérification

```bash
Router# show running-config | section line con
line con 0
 password Cons0le@2024!
 exec-timeout 5 0
 logging synchronous
 login
```

> [!example] Cas d'usage typique Vous configurez un nouveau routeur en salle serveur. Vous connectez votre laptop via câble console, mais vous devez temporairement vous absenter. Avec `exec-timeout`, la session se fermera automatiquement, empêchant un accès non autorisé.

---

## 🔑 Mot de passe enable et enable secret {#enable}

Le mode **enable** (ou mode privilégié) permet d'accéder à toutes les commandes de configuration du routeur. Sa protection est **absolument essentielle**.

### Différence entre enable password et enable secret

|Commande|Chiffrement|Sécurité|Recommandation|
|---|---|---|---|
|`enable password`|Texte clair ou type 7 (faible)|❌ Très faible|**Ne jamais utiliser**|
|`enable secret`|MD5 (type 5)|✅ Acceptable|**Toujours utiliser**|

> [!warning] Important Si `enable password` et `enable secret` sont **tous les deux configurés**, c'est toujours `enable secret` qui sera utilisé. Mais pour éviter toute confusion, configurez uniquement `enable secret`.

### Configuration de enable secret

```bash
Router(config)# enable secret VotreMotDePasseComplexe123!
```

> [!info] Bonne pratique - Complexité du mot de passe Un bon mot de passe enable secret devrait :
> 
> - Contenir au minimum 10 caractères
> - Mélanger majuscules, minuscules, chiffres et caractères spéciaux
> - Ne pas contenir de mots du dictionnaire
> - Être unique à cet équipement

### Vérification dans la configuration

```bash
Router# show running-config | include enable
enable secret 5 $1$mERr$hx5rVt7rPNoS4wqbXKX7m0
```

Le `5` indique que c'est un hash MD5. Le mot de passe original n'est **jamais visible**.

> [!tip] Astuce - Test du mot de passe Pour vérifier que votre mot de passe fonctionne sans quitter votre session :
> 
> ```bash
> Router# exit                    # Retour en mode utilisateur
> Router> enable                  # Demande le mot de passe
> Password: 
> Router#                         # Vous êtes de nouveau en mode privilégié
> ```

### Piège courant à éviter

```bash
# ❌ MAUVAIS - Utilise enable password
Router(config)# enable password cisco123

# ✅ BON - Utilise enable secret
Router(config)# enable secret C1sc0@Secure!2024
```

> [!warning] Erreur fréquente Ne tapez **jamais** le mot de passe directement visible dans la salle ou devant une caméra. Les mots de passe enable sont la clé de votre infrastructure !

---

## 🔐 Chiffrement des mots de passe {#chiffrement}

Par défaut, certains mots de passe (comme ceux des lignes console et VTY) sont stockés **en clair** dans la configuration. C'est un énorme problème de sécurité !

### Le problème du texte clair

```bash
# Sans chiffrement - DANGEREUX
Router# show running-config | section line con
line con 0
 password monsupermotdepasse    # ❌ Visible en clair !
 login
```

Toute personne ayant accès en lecture à la configuration voit tous les mots de passe.

### Activation du chiffrement

```bash
Router(config)# service password-encryption
```

Cette simple commande active le chiffrement de **tous les mots de passe** stockés en clair dans la configuration.

### Résultat après activation

```bash
Router# show running-config | section line con
line con 0
 password 7 0822455D0A16    # ✅ Mot de passe chiffré
 login
```

Le `7` indique un chiffrement de type 7 (algorithme Cisco propriétaire).

> [!warning] Limitation importante Le chiffrement type 7 est **faible** et peut être cassé facilement avec des outils en ligne. Il protège contre la lecture passive, mais pas contre une attaque déterminée.

### Tableau récapitulatif des types de chiffrement

|Type|Algorithme|Niveau de sécurité|Utilisation|
|---|---|---|---|
|**0**|Texte clair|❌ Aucune|Jamais|
|**5**|MD5|✅ Correct|`enable secret`|
|**7**|Cisco propriétaire|⚠️ Faible|Mots de passe lignes (avec `service password-encryption`)|
|**8**|PBKDF2-SHA256|✅✅ Fort|IOS récents uniquement|
|**9**|Scrypt|✅✅✅ Très fort|IOS récents uniquement|

> [!tip] Astuce - Vérifier le chiffrement Pour voir quels mots de passe sont chiffrés :
> 
> ```bash
> Router# show running-config | include password
> enable secret 5 $1$mERr$hx5rVt7rPNoS4wqbXKX7m0
> password 7 0822455D0A16
> ```

### Désactivation (déconseillé)

```bash
Router(config)# no service password-encryption
```

> [!warning] Attention critique La commande `no service password-encryption` **ne déchiffre pas** les mots de passe existants. Elle empêche simplement le chiffrement des nouveaux mots de passe. Les mots de passe déjà chiffrés restent chiffrés.

### Bonne pratique

```bash
# Configuration complète recommandée
Router(config)# enable secret VotreMotDePasseFort2024!
Router(config)# service password-encryption
Router(config)# line console 0
Router(config-line)# password ConsoleSecure2024!
Router(config-line)# login
```

> [!example] Scénario réel Un technicien fait un `show running-config` et envoie la sortie par email pour demander de l'aide. Avec `service password-encryption`, les mots de passe ne sont pas exposés en clair dans cet email.

---

## 🌐 Configuration des lignes VTY {#vty}

Les lignes **VTY** (Virtual TeletYpe) permettent l'accès distant au routeur via Telnet ou SSH. C'est le vecteur d'accès le plus utilisé en production.

### Comprendre les lignes VTY

Un routeur Cisco possède typiquement **16 lignes VTY** (0 à 15), ce qui signifie que 16 connexions distantes simultanées sont possibles.

```bash
Router(config)# line vty 0 15    # Configure toutes les lignes VTY
```

> [!info] Pourquoi 16 lignes ?
> 
> - Ligne 0 à 4 : Connexions standard (5 sessions)
> - Ligne 5 à 15 : Connexions supplémentaires (11 sessions)
> 
> Il est important de configurer **toutes** les lignes pour éviter les failles de sécurité.

### Configuration de base sécurisée

```bash
Router(config)# line vty 0 15
Router(config-line)# password TelnetSecure2024!
Router(config-line)# login
Router(config-line)# exec-timeout 10 0
Router(config-line)# transport input ssh    # Force SSH uniquement (recommandé)
Router(config-line)# exit
```

> [!warning] Telnet vs SSH
> 
> - **Telnet** : Tout transite en clair (mots de passe inclus) ❌
> - **SSH** : Communication chiffrée ✅
> 
> **En production, utilisez TOUJOURS SSH**, jamais Telnet !

### Configuration SSH recommandée

```bash
# Étape 1 : Configurer le hostname et le domaine
Router(config)# hostname R1
R1(config)# ip domain-name monentreprise.local

# Étape 2 : Générer les clés RSA
R1(config)# crypto key generate rsa
How many bits in the modulus [512]: 2048    # Minimum 2048 bits

# Étape 3 : Configurer SSH version 2
R1(config)# ip ssh version 2
R1(config)# ip ssh time-out 60              # Timeout de 60 secondes
R1(config)# ip ssh authentication-retries 3  # 3 tentatives maximum

# Étape 4 : Sécuriser les lignes VTY
R1(config)# line vty 0 15
R1(config-line)# transport input ssh         # SSH uniquement
R1(config-line)# login local                 # Authentification locale (utilisateurs configurés)
R1(config-line)# exec-timeout 10 0
R1(config-line)# exit
```

> [!tip] Astuce - Taille des clés RSA
> 
> - **1024 bits** : Minimum absolu (déprécié)
> - **2048 bits** : Standard actuel recommandé
> - **4096 bits** : Sécurité maximale (plus lent à générer)

### Limitation par Access-List (optionnel mais recommandé)

Vous pouvez restreindre les adresses IP autorisées à se connecter :

```bash
# Créer une ACL autorisant uniquement certaines IPs
R1(config)# access-list 10 permit 192.168.1.0 0.0.0.255
R1(config)# access-list 10 deny any log

# Appliquer l'ACL aux lignes VTY
R1(config)# line vty 0 15
R1(config-line)# access-class 10 in
R1(config-line)# exit
```

> [!example] Cas d'usage Vous souhaitez que seuls les administrateurs du réseau 192.168.1.0/24 puissent accéder au routeur à distance. L'ACL bloquera toute tentative depuis d'autres réseaux.

### Vérification de la configuration VTY

```bash
# Voir la configuration des lignes VTY
R1# show running-config | section line vty
line vty 0 4
 access-class 10 in
 exec-timeout 10 0
 password 7 0822455D0A16
 transport input ssh
line vty 5 15
 access-class 10 in
 exec-timeout 10 0
 password 7 0822455D0A16
 transport input ssh

# Vérifier les sessions actives
R1# show users
    Line       User       Host(s)              Idle       Location
*  0 con 0                idle                 00:00:00   
   2 vty 0     admin      idle                 00:01:23 192.168.1.50

# Vérifier la configuration SSH
R1# show ip ssh
SSH Enabled - version 2.0
Authentication timeout: 60 secs; Authentication retries: 3
```

> [!warning] Piège fréquent Si vous configurez `transport input ssh` mais que SSH n'est pas activé (pas de clés RSA générées), **personne ne pourra se connecter** aux lignes VTY, même en console vous ne pourrez pas y accéder ! Toujours générer les clés RSA avant.

### Tableau comparatif des options de transport

|Commande|Effet|Sécurité|
|---|---|---|
|`transport input telnet`|Telnet uniquement|❌ Très faible|
|`transport input ssh`|SSH uniquement|✅ Recommandé|
|`transport input all`|Telnet + SSH|⚠️ Déconseillé|
|`transport input none`|Aucun accès distant|✅ Pour désactiver temporairement|

---

## 🚫 Limitation des tentatives {#limitation}

La limitation des tentatives d'authentification protège contre les attaques par force brute.

### Limitation pour SSH

Déjà couverte dans la section VTY ci-dessus :

```bash
Router(config)# ip ssh authentication-retries 3
```

Après 3 échecs, la connexion SSH est fermée et l'attaquant doit recommencer.

> [!info] Comportement
> 
> - Tentative 1 : Échec → "Password:" réaffiché
> - Tentative 2 : Échec → "Password:" réaffiché
> - Tentative 3 : Échec → Déconnexion automatique

### Blocage temporaire (IOS récents)

Sur les IOS récents, vous pouvez bloquer temporairement une IP après plusieurs échecs :

```bash
Router(config)# login block-for 300 attempts 3 within 60
```

**Explication** :

- `block-for 300` : Bloquer pendant 300 secondes (5 minutes)
- `attempts 3` : Après 3 tentatives échouées
- `within 60` : Dans une fenêtre de 60 secondes

> [!example] Scénario Un attaquant tente 3 connexions SSH échouées en 30 secondes. Son IP est bloquée pendant 5 minutes. Il ne peut plus tenter de connexion pendant ce délai.

### Autoriser des IPs de confiance (quiet mode)

Pour éviter de vous bloquer vous-même :

```bash
Router(config)# login quiet-mode access-class 20
Router(config)# access-list 20 permit 192.168.1.100    # IP de l'administrateur
```

L'IP 192.168.1.100 ne sera jamais bloquée, même en cas d'échecs répétés.

### Logging des tentatives échouées

```bash
Router(config)# login on-failure log
Router(config)# login on-success log
```

Toutes les tentatives (réussies et échouées) seront enregistrées dans les logs.

> [!tip] Astuce - Surveillance Combiné avec un serveur syslog, vous pouvez surveiller en temps réel toutes les tentatives de connexion et détecter les attaques.

### Vérification

```bash
# Voir les statistiques de blocage
Router# show login

# Voir les IPs actuellement bloquées
Router# show login failures
```

> [!warning] Attention en production Testez toujours la limitation depuis une IP de test avant de l'activer en production. Vous pourriez accidentellement vous bloquer !

---

## 📝 Récapitulatif des bonnes pratiques {#recap}

### Configuration complète recommandée

```bash
# === HOSTNAME ET DOMAINE ===
Router(config)# hostname R1
R1(config)# ip domain-name monentreprise.local

# === ENABLE SECRET ===
R1(config)# enable secret VotreMotDePasseComplexe123!

# === CHIFFREMENT ===
R1(config)# service password-encryption

# === CONSOLE ===
R1(config)# line console 0
R1(config-line)# password ConsoleSecure2024!
R1(config-line)# login
R1(config-line)# exec-timeout 5 0
R1(config-line)# logging synchronous
R1(config-line)# exit

# === SSH ===
R1(config)# crypto key generate rsa
# Choisir 2048 bits minimum
R1(config)# ip ssh version 2
R1(config)# ip ssh time-out 60
R1(config)# ip ssh authentication-retries 3

# === VTY ===
R1(config)# access-list 10 permit 192.168.1.0 0.0.0.255
R1(config)# access-list 10 deny any log
R1(config)# line vty 0 15
R1(config-line)# transport input ssh
R1(config-line)# login local
R1(config-line)# exec-timeout 10 0
R1(config-line)# access-class 10 in
R1(config-line)# exit

# === LIMITATION TENTATIVES (optionnel) ===
R1(config)# login block-for 300 attempts 3 within 60
R1(config)# login on-failure log
R1(config)# login on-success log

# === SAUVEGARDER ===
R1(config)# exit
R1# write memory
```

### Checklist de sécurisation

- [ ] Mot de passe console configuré avec `login`
- [ ] `enable secret` configuré (pas `enable password`)
- [ ] `service password-encryption` activé
- [ ] SSH v2 configuré et clés RSA générées (2048 bits minimum)
- [ ] `transport input ssh` sur toutes les lignes VTY (0 à 15)
- [ ] Timeouts configurés (console et VTY)
- [ ] Access-list sur les lignes VTY pour limiter les IPs autorisées
- [ ] Limitation des tentatives SSH activée
- [ ] Logging des connexions activé
- [ ] Configuration sauvegardée (`write memory`)

> [!tip] Astuce finale Créez un script ou un template de configuration sécurisée que vous pourrez réutiliser sur tous vos routeurs. Cela garantit une sécurité homogène sur toute l'infrastructure.

---

### Commandes de vérification essentielles

```bash
# Vérifier toute la configuration de sécurité
R1# show running-config

# Vérifier spécifiquement les lignes
R1# show running-config | section line

# Vérifier SSH
R1# show ip ssh
R1# show ssh

# Vérifier les utilisateurs connectés
R1# show users

# Vérifier les tentatives de connexion
R1# show login
R1# show login failures

# Vérifier les ACL
R1# show access-lists
```

> [!info] Documentation complète Ce cours couvre la sécurisation des accès de base. Pour aller plus loin, vous pourriez explorer l'authentification AAA (RADIUS/TACACS+) qui sera abordée dans une partie ultérieure dédiée.

---

**✅ Vous maîtrisez maintenant la sécurisation des accès sur un routeur Cisco !**