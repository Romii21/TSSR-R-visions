

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

## ⏱️ Timers RIP

### Comprendre les timers

RIP utilise quatre timers principaux qui contrôlent la stabilité et la réactivité du protocole. Ces timers déterminent comment et quand les routes sont mises à jour, invalidées et supprimées.

> [!info] Les 4 Timers RIP
> - **Update Timer** : Intervalle entre les mises à jour périodiques
> - **Invalid Timer** : Délai avant de marquer une route comme invalide
> - **Holddown Timer** : Période de stabilisation après invalidation
> - **Flush Timer** : Délai avant suppression définitive de la route

#### 📊 Tableau récapitulatif des timers

| Timer | Valeur par défaut | Fonction | Déclencheur |
|-------|-------------------|----------|-------------|
| **Update** | 30 secondes | Envoi des mises à jour RIP | Périodique |
| **Invalid** | 180 secondes | Marque la route comme invalide | Pas de mise à jour reçue |
| **Holddown** | 180 secondes | Empêche les mises à jour erronées | Route invalidée |
| **Flush** | 240 secondes | Supprime la route de la table | Après invalidation |

> [!warning] Relation entre les timers
> **Flush Timer ≥ Invalid Timer** pour éviter qu'une route soit supprimée avant d'être invalidée. Par défaut : Flush (240s) > Invalid (180s).

### Configuration des timers

#### Syntaxe de configuration

```bash
Router(config)# router rip
Router(config-router)# timers basic <update> <invalid> <holddown> <flush>
```

**Paramètres :**
- `update` : 1-4294967295 secondes (délai entre updates)
- `invalid` : 1-4294967295 secondes (délai avant invalidation)
- `holddown` : 0-4294967295 secondes (période de stabilisation)
- `flush` : 1-4294967295 secondes (délai avant suppression)

#### 💻 Exemple de configuration personnalisée

```bash
Router(config)# router rip
Router(config-router)# version 2
Router(config-router)# network 192.168.1.0
Router(config-router)# timers basic 15 90 90 120
```

> [!example] Explication de l'exemple
> - Update : **15s** (mises à jour plus fréquentes)
> - Invalid : **90s** (détection rapide des pannes)
> - Holddown : **90s** (stabilisation rapide)
> - Flush : **120s** (suppression après 120s)
> 
> Cette configuration accélère la convergence mais augmente le trafic réseau.

#### Vérification des timers

```bash
Router# show ip protocols

*** IP Routing is NSF aware ***

Routing Protocol is "rip"
  Outgoing update filter list for all interfaces is not set
  Incoming update filter list for all interfaces is not set
  Sending updates every 15 seconds
  Invalid after 90 seconds, hold down 90, flushed after 120
  Redistributing: rip
  Default version control: send version 2, receive version 2
```

### Impact sur la convergence

> [!tip] Optimisation des timers
> **Pour un réseau stable** : Garder les valeurs par défaut (30/180/180/240)
> **Pour convergence rapide** : Réduire les timers (10/60/60/80)
> **Pour réduire la bande passante** : Augmenter update timer (60/300/300/360)

> [!warning] Précautions importantes
> - **Cohérence** : Tous les routeurs doivent utiliser les mêmes timers
> - **Charge CPU** : Des timers trop courts augmentent la charge processeur
> - **Bande passante** : Update timer court = plus de trafic réseau
> - **Stabilité** : Holddown trop court peut causer des boucles de routage

#### Formule de dimensionnement recommandée

```
Invalid Timer = 3 × Update Timer (minimum)
Flush Timer = Invalid Timer + 60 secondes (minimum)
Holddown Timer = Invalid Timer (généralement)
```

---

## 🌐 Route par défaut avec RIP

### Principe et utilité

Une route par défaut (0.0.0.0/0) est une route de dernier recours utilisée lorsqu'aucune route spécifique n'existe pour une destination. Avec RIP, vous pouvez propager cette route à tous les routeurs du domaine.

> [!info] Cas d'usage typiques
> - **Routeur de bordure** : Connexion à Internet ou autre AS
> - **Topologie hub-and-spoke** : Le hub propage la route par défaut
> - **Simplification** : Éviter de propager toutes les routes externes

#### Schéma conceptuel

```
[Internet]
    |
[R1] ← Routeur de bordure avec route par défaut
    |
[R2]──[R3]──[R4] ← Reçoivent 0.0.0.0/0 via RIP
```

### Configuration default-information originate

#### Syntaxe de base

```bash
Router(config)# router rip
Router(config-router)# default-information originate
```

> [!info] Fonctionnement
> Cette commande propage la route par défaut **SI et SEULEMENT SI** une route 0.0.0.0/0 existe dans la table de routage du routeur.

#### 💻 Exemple complet - Routeur de bordure

```bash
# Configuration du routeur de bordure R1
R1(config)# interface GigabitEthernet0/0
R1(config-if)# ip address 192.168.1.1 255.255.255.0
R1(config-if)# no shutdown
R1(config-if)# exit

R1(config)# interface GigabitEthernet0/1
R1(config-if)# ip address 203.0.113.2 255.255.255.0
R1(config-if)# description Connexion Internet
R1(config-if)# no shutdown
R1(config-if)# exit

# Création de la route par défaut statique vers l'ISP
R1(config)# ip route 0.0.0.0 0.0.0.0 203.0.113.1

# Configuration RIP avec propagation de la route par défaut
R1(config)# router rip
R1(config-router)# version 2
R1(config-router)# network 192.168.1.0
R1(config-router)# no auto-summary
R1(config-router)# default-information originate
```

#### 💻 Exemple - Routeur interne recevant la route

```bash
# Configuration d'un routeur interne R2
R2(config)# router rip
R2(config-router)# version 2
R2(config-router)# network 192.168.1.0
R2(config-router)# network 192.168.2.0
R2(config-router)# no auto-summary
```

### Propagation de la route par défaut

#### Vérification sur le routeur émetteur

```bash
R1# show ip route
Codes: L - local, C - connected, S - static, R - RIP, ...

Gateway of last resort is 203.0.113.1 to network 0.0.0.0

S*    0.0.0.0/0 [1/0] via 203.0.113.1
      192.168.1.0/24 is variably subnetted, 2 subnets, 2 masks
C        192.168.1.0/24 is directly connected, GigabitEthernet0/0
L        192.168.1.1/32 is directly connected, GigabitEthernet0/0
R     192.168.2.0/24 [120/1] via 192.168.1.2, 00:00:15, GigabitEthernet0/0
```

#### Vérification sur le routeur receveur

```bash
R2# show ip route
Codes: L - local, C - connected, S - static, R - RIP, ...

Gateway of last resort is 192.168.1.1 to network 0.0.0.0

R*    0.0.0.0/0 [120/1] via 192.168.1.1, 00:00:08, GigabitEthernet0/0
      192.168.1.0/24 is variably subnetted, 2 subnets, 2 masks
C        192.168.1.0/24 is directly connected, GigabitEthernet0/0
L        192.168.1.2/32 is directly connected, GigabitEthernet0/0
      192.168.2.0/24 is variably subnetted, 2 subnets, 2 masks
C        192.168.2.0/24 is directly connected, GigabitEthernet0/1
L        192.168.2.1/32 is directly connected, GigabitEthernet0/1
```

> [!example] Interprétation
> - **R\*** : Route par défaut apprise via RIP
> - **[120/1]** : Distance administrative 120, métrique RIP 1 saut
> - Le routeur R2 enverra tout trafic inconnu vers R1 (192.168.1.1)

#### Vérification de la propagation RIP

```bash
R2# show ip rip database
0.0.0.0/0    auto-summary
0.0.0.0/0
    [1] via 192.168.1.1, 00:00:12, GigabitEthernet0/0
192.168.1.0/24    auto-summary
192.168.1.0/24    directly connected, GigabitEthernet0/0
192.168.2.0/24    auto-summary
192.168.2.0/24    directly connected, GigabitEthernet0/1
```

> [!tip] Astuce de vérification
> Utilisez `show ip protocols` pour confirmer que default-information originate est actif :
> ```bash
> R1# show ip protocols
> Routing Protocol is "rip"
>   ...
>   Redistributing: rip
>   Default version control: send version 2, receive version 2
>   Automatic network summarization is not in effect
>   Default metric redistribution is not set
>   Redistributing: static
>   Default redistribution metric is 1
> ```

> [!warning] Pièges courants
> 1. **Route par défaut absente** : default-information originate ne fonctionne que si 0.0.0.0/0 existe dans la table de routage
> 2. **Boucles de routage** : Éviter plusieurs routeurs qui propagent 0.0.0.0/0 sans coordination
> 3. **Métrique** : La route par défaut RIP aura toujours une métrique de 1 depuis le routeur source

---

## 🔐 Authentication RIP

### Pourquoi sécuriser RIP

RIP, par défaut, accepte les mises à jour de n'importe quel routeur sans vérification. L'authentification protège contre :

> [!warning] Menaces sans authentification
> - **Injection de routes malveillantes** : Attaquant envoie de fausses routes
> - **Man-in-the-middle** : Interception et modification des updates
> - **Déni de service** : Surcharge avec des updates invalides
> - **Routeur non autorisé** : Appareil tiers rejoint le domaine RIP

### Types d'authentification

RIP supporte deux modes d'authentification :

| Type | Sécurité | Configuration | Utilisation |
|------|----------|---------------|-------------|
| **Clear Text** | ⚠️ Faible | Simple | Tests uniquement |
| **MD5** | ✅ Forte | Recommandée | Production |

> [!info] Différences clés
> - **Clear Text** : Mot de passe visible dans les paquets (Wireshark)
> - **MD5** : Hash cryptographique, mot de passe jamais transmis

### Configuration Clear Text

> [!warning] Ne pas utiliser en production
> L'authentification en texte clair est visible en sniffant le réseau. À utiliser uniquement pour des tests ou des environnements isolés.

#### Étapes de configuration

**Étape 1 : Créer la clé d'authentification**

```bash
Router(config)# interface GigabitEthernet0/0
Router(config-if)# ip rip authentication key-chain NOM_KEYCHAIN
Router(config-if)# ip rip authentication mode text
```

**Étape 2 : Définir le key-chain**

```bash
Router(config)# key chain NOM_KEYCHAIN
Router(config-keychain)# key 1
Router(config-keychain-key)# key-string MOT_DE_PASSE
```

#### 💻 Exemple complet - Clear Text

```bash
# Configuration du routeur R1
R1(config)# key chain RIP_AUTH
R1(config-keychain)# key 1
R1(config-keychain-key)# key-string Cisco123
R1(config-keychain-key)# exit
R1(config-keychain)# exit

R1(config)# interface GigabitEthernet0/0
R1(config-if)# ip rip authentication key-chain RIP_AUTH
R1(config-if)# ip rip authentication mode text
R1(config-if)# exit

# Configuration identique sur R2
R2(config)# key chain RIP_AUTH
R2(config-keychain)# key 1
R2(config-keychain-key)# key-string Cisco123
R2(config-keychain-key)# exit
R2(config-keychain)# exit

R2(config)# interface GigabitEthernet0/0
R2(config-if)# ip rip authentication key-chain RIP_AUTH
R2(config-if)# ip rip authentication mode text
```

### Configuration MD5

L'authentification MD5 est la méthode recommandée pour la production. Elle utilise un hash MD5 du message + clé secrète.

#### Syntaxe MD5

```bash
Router(config)# interface GigabitEthernet0/0
Router(config-if)# ip rip authentication key-chain NOM_KEYCHAIN
Router(config-if)# ip rip authentication mode md5
```

#### 💻 Exemple complet - MD5

```bash
# Configuration du routeur R1
R1(config)# key chain RIP_MD5
R1(config-keychain)# key 1
R1(config-keychain-key)# key-string SecurePass2024!
R1(config-keychain-key)# exit
R1(config-keychain)# exit

R1(config)# interface GigabitEthernet0/0
R1(config-if)# ip address 10.1.1.1 255.255.255.0
R1(config-if)# ip rip authentication key-chain RIP_MD5
R1(config-if)# ip rip authentication mode md5
R1(config-if)# exit

R1(config)# router rip
R1(config-router)# version 2
R1(config-router)# network 10.0.0.0

# Configuration du routeur R2 (doit être identique)
R2(config)# key chain RIP_MD5
R2(config-keychain)# key 1
R2(config-keychain-key)# key-string SecurePass2024!
R2(config-keychain-key)# exit
R2(config-keychain)# exit

R2(config)# interface GigabitEthernet0/0
R2(config-if)# ip address 10.1.1.2 255.255.255.0
R2(config-if)# ip rip authentication key-chain RIP_MD5
R2(config-if)# ip rip authentication mode md5
R2(config-if)# exit

R2(config)# router rip
R2(config-router)# version 2
R2(config-router)# network 10.0.0.0
```

#### Configuration avancée avec clés multiples

> [!tip] Rotation des clés
> Utilisez plusieurs clés avec des périodes de validité pour une rotation sans interruption.

```bash
R1(config)# key chain RIP_MULTIKEY
R1(config-keychain)# key 1
R1(config-keychain-key)# key-string OldPassword
R1(config-keychain-key)# accept-lifetime 00:00:00 Jan 1 2024 23:59:59 Dec 31 2024
R1(config-keychain-key)# send-lifetime 00:00:00 Jan 1 2024 23:59:59 Jun 30 2024
R1(config-keychain-key)# exit

R1(config-keychain)# key 2
R1(config-keychain-key)# key-string NewPassword
R1(config-keychain-key)# accept-lifetime 00:00:00 Jun 1 2024 infinite
R1(config-keychain-key)# send-lifetime 00:00:00 Jul 1 2024 infinite
```

> [!example] Explication de la rotation
> - **Key 1** : Valide jusqu'au 31/12/2024, envoyée jusqu'au 30/06/2024
> - **Key 2** : Acceptée dès 01/06/2024, envoyée dès 01/07/2024
> - **Période de chevauchement** : Juin-Juillet permet transition en douceur

#### Vérification de l'authentification

```bash
# Vérifier la configuration d'interface
R1# show ip interface GigabitEthernet0/0
GigabitEthernet0/0 is up, line protocol is up
  Internet address is 10.1.1.1/24
  ...
  RIP authentication mode is MD5, and keychain is RIP_MD5
  RIP authentication key timeout disabled

# Vérifier les key-chains
R1# show key chain
Key-chain RIP_MD5:
    key 1 -- text "SecurePass2024!"
        accept lifetime (always valid) - (always valid) [valid now]
        send lifetime (always valid) - (always valid) [valid now]

# Déboguer l'authentification RIP
R1# debug ip rip
RIP protocol debugging is on

*Mar 1 12:30:15.123: RIP: sending v2 update to 224.0.0.9 via GigabitEthernet0/0 (10.1.1.1)
*Mar 1 12:30:15.124: RIP: build update entries - authentication MD5
```

#### Test de la sécurité

> [!example] Test : Mot de passe incorrect
> Si les mots de passe ne correspondent pas, les updates sont rejetés :
> ```bash
> R2# debug ip rip
> *Mar 1 12:35:20.456: RIP: ignored v2 packet from 10.1.1.1 (invalid authentication)
> ```

> [!example] Test : Authentification désactivée sur un routeur
> ```bash
> R1# show ip route rip
> # Aucune route RIP reçue de R2
> 
> R1# show ip rip database
> # Base de données locale uniquement, pas de routes de R2
> ```

### Bonnes pratiques d'authentification

> [!tip] Recommandations de sécurité
> 1. **Toujours utiliser MD5** en production (jamais text)
> 2. **Mots de passe complexes** : 12+ caractères, majuscules, chiffres, symboles
> 3. **Cohérence** : Même configuration sur TOUS les routeurs du segment
> 4. **Rotation régulière** : Changer les clés tous les 3-6 mois
> 5. **Documentation** : Conserver un registre sécurisé des clés actives
> 6. **Test** : Valider l'authentification après chaque modification

> [!warning] Erreurs fréquentes
> - **Typo dans le mot de passe** : La différence d'un caractère bloque tout
> - **Nom de key-chain différent** : Peut fonctionner mais source de confusion
> - **Oublier une interface** : Crée un trou de sécurité
> - **Key-chain sans clé active** : L'authentification échoue
> - **Case-sensitive** : "Password" ≠ "password"

#### Dépannage de l'authentification

```bash
# Vérifier si l'authentification est la cause du problème
R1# show ip rip database
# Si vide ou incomplet, problème potentiel

# Vérifier les adjacences RIP
R1# show ip protocols | include Routing Information Sources
  Routing Information Sources:
    Gateway         Distance      Last Update
    10.1.1.2        120           00:00:25

# Désactiver temporairement pour tester (UNIQUEMENT EN TEST!)
R1(config)# interface GigabitEthernet0/0
R1(config-if)# no ip rip authentication mode
R1(config-if)# no ip rip authentication key-chain RIP_MD5

# Si cela résout le problème, reconfigurer correctement l'authentification
```

---

## 🎯 Résumé des commandes clés

### Timers RIP
```bash
router rip
 timers basic <update> <invalid> <holddown> <flush>

# Vérification
show ip protocols
```

### Route par défaut
```bash
# Créer la route statique
ip route 0.0.0.0 0.0.0.0 <next-hop>

# Activer la propagation
router rip
 default-information originate

# Vérification
show ip route
show ip rip database
```

### Authentification
```bash
# Configuration MD5 (recommandée)
key chain <nom>
 key <id>
  key-string <password>

interface <interface>
 ip rip authentication key-chain <nom>
 ip rip authentication mode md5

# Vérification
show ip interface <interface>
show key chain
debug ip rip
```

---

> [!tip] 💡 Points clés à retenir
> - Les **timers RIP** contrôlent la vitesse de convergence et la charge réseau
> - La **route par défaut** simplifie la configuration en évitant la propagation de toutes les routes externes
> - L'**authentification MD5** est indispensable pour sécuriser RIP en production
> - La **cohérence** de configuration entre tous les routeurs est critique pour le bon fonctionnement