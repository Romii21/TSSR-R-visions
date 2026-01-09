

## 📑 Table des matières

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

## 🔍 Configuration DNS statique {#dns-statique}

### Pourquoi configurer le DNS sur un routeur ?

Le DNS (Domain Name System) permet de résoudre des noms de domaine en adresses IP. Configurer le DNS sur un routeur Cisco permet :

- Au routeur lui-même de résoudre des noms (pour ping, traceroute, etc.)
- De définir les serveurs DNS que le routeur distribuera via DHCP aux clients
- D'améliorer la lisibilité des logs et diagnostics

> [!info] DNS vs résolution locale Le DNS statique sur un routeur diffère de la table de résolution locale (commande `ip host`). Le DNS statique interroge des serveurs externes, tandis que `ip host` crée des entrées locales statiques.

### Configuration de base

#### Activer la résolution DNS

```bash
Router(config)# ip domain-lookup
```

> [!tip] Par défaut La résolution DNS est activée par défaut sur les routeurs Cisco. Utilisez `no ip domain-lookup` pour la désactiver.

#### Définir le nom de domaine par défaut

```bash
Router(config)# ip domain-name exemple.local
```

Cette commande ajoute automatiquement le suffixe `.exemple.local` aux noms non qualifiés.

**Exemple** : Si vous tapez `ping serveur1`, le routeur tentera de résoudre `serveur1.exemple.local`.

#### Configurer les serveurs DNS

```bash
Router(config)# ip name-server 8.8.8.8
Router(config)# ip name-server 8.8.4.4
Router(config)# ip name-server 1.1.1.1
```

> [!warning] Limite de serveurs Vous pouvez configurer jusqu'à 6 serveurs DNS. Le routeur les interroge dans l'ordre jusqu'à obtenir une réponse.

### Configuration avancée

#### Liste de domaines de recherche

```bash
Router(config)# ip domain-list entreprise.local
Router(config)# ip domain-list lab.local
Router(config)# ip domain-list test.local
```

Le routeur essaiera chaque suffixe dans l'ordre lors de la résolution de noms courts.

#### Résolution de noms locaux statiques

```bash
Router(config)# ip host serveur1 192.168.1.100
Router(config)# ip host serveur2 192.168.1.101
Router(config)# ip host switch1 10.0.0.1
```

Ces entrées ont la priorité sur la résolution DNS externe.

### Vérification et diagnostic

```bash
# Tester la résolution DNS
Router# ping google.com

# Afficher la configuration DNS
Router# show hosts

# Afficher les serveurs DNS configurés
Router# show running-config | include name-server

# Debug DNS (utiliser avec précaution)
Router# debug ip domain
```

> [!example] Sortie de `show hosts`
> 
> ```
> Default domain is exemple.local
> Name/address lookup uses domain service
> Name servers are 8.8.8.8, 8.8.4.4
> 
> Host               Port  Flags      Age Type   Address(es)
> serveur1           None  (perm, OK)  0   IP    192.168.1.100
> google.com         None  (temp, OK)  0   IP    142.250.185.78
> ```

### Pièges courants

|Problème|Cause|Solution|
|---|---|---|
|Le routeur ne résout pas les noms|DNS désactivé|`ip domain-lookup`|
|Résolution lente|Serveurs DNS injoignables|Vérifier la connectivité, retirer les serveurs morts|
|Noms courts non résolus|Pas de domaine par défaut|Configurer `ip domain-name`|
|Conflit entre entrées locales et DNS|Priorité des `ip host`|Supprimer les entrées locales inutiles avec `no ip host`|

> [!tip] Bonne pratique Configurez toujours au moins deux serveurs DNS pour la redondance (un primaire, un secondaire).

---

## 🖥️ Serveur HTTP/HTTPS du routeur {#serveur-http-https}

### Qu'est-ce que le serveur HTTP du routeur ?

Le routeur Cisco embarque un serveur web minimal permettant :

- L'accès à une interface de gestion graphique (limitée)
- La configuration basique via navigateur web
- L'accès à certaines informations de diagnostic

> [!warning] Sécurité Par défaut, le serveur HTTP est souvent **activé** sur les routeurs Cisco. Pour des raisons de sécurité, il est recommandé de le désactiver en production ou de n'utiliser que HTTPS.

### Configuration du serveur HTTP

#### Activer/Désactiver HTTP

```bash
# Activer le serveur HTTP (port 80)
Router(config)# ip http server

# Désactiver le serveur HTTP (recommandé)
Router(config)# no ip http server
```

#### Configurer le port HTTP personnalisé

```bash
Router(config)# ip http port 8080
```

#### Définir l'authentification

```bash
# Utiliser l'authentification AAA (si configurée)
Router(config)# ip http authentication aaa

# Utiliser l'authentification locale
Router(config)# ip http authentication local

# Utiliser enable password
Router(config)# ip http authentication enable
```

### Configuration du serveur HTTPS (sécurisé)

#### Activer HTTPS

```bash
# Activer le serveur HTTPS (port 443)
Router(config)# ip http secure-server
```

> [!info] Prérequis pour HTTPS Avant d'activer HTTPS, vous devez avoir :
> 
> - Un nom de domaine configuré (`ip domain-name`)
> - Des clés RSA générées (`crypto key generate rsa`)

**Exemple complet** :

```bash
Router(config)# ip domain-name entreprise.local
Router(config)# crypto key generate rsa general-keys modulus 2048
Router(config)# ip http secure-server
```

#### Configurer le port HTTPS personnalisé

```bash
Router(config)# ip http secure-port 8443
```

#### Désactiver HTTP et n'utiliser que HTTPS

```bash
Router(config)# no ip http server
Router(config)# ip http secure-server
```

> [!tip] Meilleure pratique de sécurité En production, **désactivez toujours HTTP** et n'utilisez que HTTPS avec des certificats appropriés.

### Configuration avancée

#### Limiter l'accès par ACL

```bash
# Créer une ACL pour autoriser uniquement certaines IP
Router(config)# access-list 10 permit 192.168.1.0 0.0.0.255
Router(config)# access-list 10 deny any log

# Appliquer l'ACL au serveur HTTP/HTTPS
Router(config)# ip http access-class 10
```

#### Configurer le nombre maximum de connexions

```bash
Router(config)# ip http max-connections 5
```

#### Définir le timeout de session

```bash
Router(config)# ip http timeout-policy idle 60 life 86400 requests 100
```

- **idle** : timeout d'inactivité (en secondes)
- **life** : durée de vie maximale d'une session (en secondes)
- **requests** : nombre maximal de requêtes par session

#### Chemins personnalisés

```bash
# Définir le chemin racine du serveur web
Router(config)# ip http path flash:/www
```

### Vérification

```bash
# Afficher l'état des serveurs HTTP/HTTPS
Router# show ip http server status

# Afficher les connexions actives
Router# show ip http server connection

# Vérifier la configuration
Router# show running-config | include ip http
```

> [!example] Sortie de `show ip http server status`
> 
> ```
> HTTP server status: Enabled
> HTTP server port: 80
> HTTP server authentication method: enable
> HTTP secure server status: Enabled
> HTTP secure server port: 443
> HTTP secure server ciphersuite: 3des-ede-cbc-sha des-cbc-sha rc4-128-md5 rc4-128-sha
> Maximum number of concurrent server connections allowed: 16
> ```

### Pièges courants

|Problème|Cause|Solution|
|---|---|---|
|Impossible de se connecter en HTTPS|Pas de clés RSA|Générer les clés avec `crypto key generate rsa`|
|Avertissement de certificat dans le navigateur|Certificat auto-signé|Normal pour les certificats auto-signés, installer un certificat CA si nécessaire|
|Accès refusé|ACL trop restrictive|Vérifier `ip http access-class`|
|Serveur HTTP exposé|HTTP activé par défaut|Désactiver avec `no ip http server`|

> [!warning] Important Le serveur HTTP/HTTPS du routeur n'est PAS destiné à héberger des sites web. C'est uniquement pour la gestion du routeur lui-même.

---

## 📊 Syslog {#syslog}

### Qu'est-ce que Syslog ?

Syslog est un protocole standard de journalisation qui permet :

- L'enregistrement des événements système (erreurs, changements d'état, etc.)
- L'envoi centralisé des logs vers un serveur externe
- La surveillance et l'audit du réseau
- Le diagnostic de problèmes

### Niveaux de sévérité Syslog

|Niveau|Nom|Description|Utilisation typique|
|---|---|---|---|
|0|emergencies|Système inutilisable|Panne critique|
|1|alerts|Action immédiate requise|Défaillance matérielle|
|2|critical|Condition critique|Température critique|
|3|errors|Messages d'erreur|Interface down|
|4|warnings|Messages d'avertissement|Duplex mismatch|
|5|notifications|Condition normale mais significative|Interface up|
|6|informational|Messages informatifs|Accès config|
|7|debugging|Messages de debug|Protocoles détaillés|

> [!info] Hiérarchie des niveaux Quand vous configurez un niveau, tous les niveaux **supérieurs** (plus critiques) sont également inclus. Par exemple, le niveau 4 (warnings) inclut automatiquement 0-3.

### Configuration de base

#### Activer la journalisation

```bash
Router(config)# logging on
```

> [!tip] Déjà activé La journalisation est généralement activée par défaut sur les routeurs Cisco.

#### Journalisation vers la console

```bash
# Activer les logs sur la console
Router(config)# logging console

# Définir le niveau de sévérité
Router(config)# logging console warnings
```

> [!warning] Console flooding Attention : trop de logs sur la console peuvent rendre le routeur difficile à utiliser. En production, utilisez un niveau de sévérité élevé (errors ou critical) ou désactivez les logs console.

#### Journalisation vers le terminal (lignes VTY)

```bash
# Activer les logs pour les sessions Telnet/SSH
Router(config)# logging monitor

# Définir le niveau de sévérité
Router(config)# logging monitor informational

# Activer l'affichage pour la session en cours
Router# terminal monitor
```

#### Journalisation dans le buffer interne

```bash
# Activer la journalisation dans le buffer
Router(config)# logging buffered

# Définir la taille du buffer (en octets)
Router(config)# logging buffered 16384

# Définir le niveau de sévérité
Router(config)# logging buffered informational
```

### Configuration d'un serveur Syslog externe

#### Configurer le serveur Syslog

```bash
# Définir l'adresse du serveur Syslog
Router(config)# logging host 192.168.1.50

# Ou avec un nom de domaine
Router(config)# logging host syslog.entreprise.local

# Définir le niveau de sévérité pour ce serveur
Router(config)# logging trap informational
```

> [!tip] Multiples serveurs Vous pouvez configurer plusieurs serveurs Syslog pour la redondance :
> 
> ```bash
> Router(config)# logging host 192.168.1.50
> Router(config)# logging host 192.168.1.51
> ```

#### Configurer l'interface source

```bash
# Utiliser une interface spécifique comme source des messages Syslog
Router(config)# logging source-interface Loopback0
```

Cela garantit que les messages proviennent toujours de la même adresse IP, facilitant le filtrage côté serveur.

#### Configurer le port et le protocole

```bash
# Par défaut, Syslog utilise UDP port 514
# Pour changer le port :
Router(config)# logging host 192.168.1.50 transport udp port 1514
```

### Configuration avancée

#### Horodatage des logs

```bash
# Ajouter des timestamps aux logs
Router(config)# service timestamps log datetime msec localtime show-timezone

# Format : Jan 15 2025 14:32:45.123 CET
```

**Options de timestamps** :

- `datetime` : date et heure complètes
- `msec` : millisecondes
- `localtime` : heure locale (pas UTC)
- `show-timezone` : affiche le fuseau horaire
- `year` : inclut l'année

#### Numéros de séquence

```bash
# Ajouter des numéros de séquence aux logs
Router(config)# service sequence-numbers
```

Cela facilite le suivi et l'identification des messages manquants.

#### Filtrage des messages

```bash
# Désactiver certains types de messages spécifiques
Router(config)# no logging event link-status

# Activer seulement certains événements
Router(config)# logging event link-status default
```

#### Messages de connexion

```bash
# Enregistrer les tentatives de connexion
Router(config)# login on-success log
Router(config)# login on-failure log
```

### Vérification et diagnostic

```bash
# Afficher les logs du buffer
Router# show logging

# Afficher uniquement les statistiques
Router# show logging summary

# Afficher l'historique des logs (derniers messages)
Router# show logging history

# Effacer le buffer de logs
Router# clear logging
```

> [!example] Sortie de `show logging`
> 
> ```
> Syslog logging: enabled (0 messages dropped, 3 messages rate-limited)
>     Console logging: level warnings, 45 messages logged
>     Monitor logging: level informational, 0 messages logged
>     Buffer logging:  level informational, 128 messages logged
>     Logging to: 192.168.1.50
>     Trap logging: level informational, 156 message lines logged
> 
> Log Buffer (16384 bytes):
> 
> *Jan 15 14:32:45.123: %SYS-5-CONFIG_I: Configured from console by admin
> *Jan 15 14:33:12.456: %LINEPROTO-5-UPDOWN: Line protocol on Interface Gi0/0, changed state to up
> ```

### Format des messages Syslog

Structure typique d'un message :

```
*Jan 15 14:32:45.123: %FACILITY-SEVERITY-MNEMONIC: Description
```

**Exemple** :

```
*Jan 15 14:32:45.123: %LINEPROTO-5-UPDOWN: Line protocol on Interface Gi0/0, changed state to up
```

- `Jan 15 14:32:45.123` : timestamp
- `LINEPROTO` : facility (module)
- `5` : niveau de sévérité (notification)
- `UPDOWN` : mnémonique (type de message)
- `Line protocol...` : description détaillée

### Pièges courants

|Problème|Cause|Solution|
|---|---|---|
|Logs non reçus sur le serveur|Firewall bloque UDP 514|Vérifier la connectivité et les ACL|
|Timestamps incorrects|Horloge non configurée|Configurer NTP ou `clock set`|
|Buffer plein, logs perdus|Buffer trop petit|Augmenter la taille avec `logging buffered`|
|Trop de messages de debug|Debug activé sans limite|Désactiver le debug avec `no debug all`|
|Console inutilisable|Trop de logs console|Réduire le niveau ou `no logging console`|

> [!warning] Debug vs Logging Les commandes `debug` génèrent énormément de logs (niveau 7). **Ne jamais activer debug en production** sans limiter le scope et surveiller la charge CPU.

### Bonnes pratiques

1. **Centralisation** : Envoyez toujours les logs vers un serveur Syslog externe
2. **Niveau approprié** : Utilisez `informational` (6) pour la plupart des environnements
3. **Timestamps** : Configurez toujours `service timestamps` avec NTP
4. **Source interface** : Utilisez `logging source-interface` avec une Loopback
5. **Désactiver console** : En production, limitez `logging console` à `critical` minimum
6. **Buffer raisonnable** : 16384 ou 32768 octets suffisent généralement

---

## 🔐 SNMP de base {#snmp}

### Qu'est-ce que SNMP ?

SNMP (Simple Network Management Protocol) permet :

- La supervision et le monitoring des équipements réseau
- La collecte d'informations (CPU, mémoire, interfaces, etc.)
- La gestion à distance des équipements
- La réception d'alertes (traps SNMP)

### Versions de SNMP

|Version|Sécurité|Description|Utilisation|
|---|---|---|---|
|SNMPv1|⚠️ Très faible|Community strings en clair|**Obsolète, éviter**|
|SNMPv2c|⚠️ Faible|Community strings en clair, améliorations protocole|Usage limité|
|SNMPv3|✅ Fort|Authentification et chiffrement|**Recommandé en production**|

> [!warning] SNMPv1 et v2c Ces versions transmettent les community strings (mots de passe) en clair sur le réseau. Utilisez SNMPv3 pour la sécurité.

### Configuration SNMPv2c de base

#### Configuration des community strings

```bash
# Community en lecture seule (RO - Read Only)
Router(config)# snmp-server community PUBLIC ro

# Community en lecture/écriture (RW - Read Write)
Router(config)# snmp-server community PRIVATE rw

# Community avec ACL pour limiter l'accès
Router(config)# access-list 20 permit 192.168.1.0 0.0.0.255
Router(config)# snmp-server community MONITORING ro 20
```

> [!tip] Sécurité
> 
> - **Jamais** de community "public" ou "private" par défaut
> - Toujours utiliser des ACL pour limiter l'accès
> - Préférer RO (read-only) sauf besoin spécifique

#### Configuration des informations système

```bash
# Définir le contact de l'équipement
Router(config)# snmp-server contact admin@entreprise.local

# Définir la localisation
Router(config)# snmp-server location Salle_serveur_A_Batiment_3

# Définir un nom de châssis
Router(config)# snmp-server chassis-id RTR-CORE-01
```

### Configuration des traps SNMP

Les traps sont des alertes envoyées automatiquement par le routeur vers un serveur de monitoring.

#### Configurer un serveur de traps

```bash
# Définir le serveur qui recevra les traps
Router(config)# snmp-server host 192.168.1.100 version 2c MONITORING

# Avec spécification de la community
Router(config)# snmp-server host 192.168.1.100 version 2c TRAPCOMM
```

#### Activer les types de traps

```bash
# Activer tous les traps
Router(config)# snmp-server enable traps

# Activer des traps spécifiques
Router(config)# snmp-server enable traps snmp linkdown linkup
Router(config)# snmp-server enable traps config
Router(config)# snmp-server enable traps syslog
Router(config)# snmp-server enable traps cpu threshold
```

**Types de traps courants** :

- `snmp` : événements SNMP généraux
- `linkdown/linkup` : changement d'état des interfaces
- `config` : changements de configuration
- `syslog` : messages syslog
- `cpu threshold` : seuils CPU dépassés
- `memory` : seuils mémoire dépassés

> [!example] Configuration complète de traps
> 
> ```bash
> Router(config)# snmp-server host 192.168.1.100 version 2c TRAPCOMM
> Router(config)# snmp-server enable traps snmp linkdown linkup
> Router(config)# snmp-server enable traps config
> Router(config)# snmp-server enable traps cpu threshold
> Router(config)# snmp-server enable traps memory bufferpeak
> ```

### Configuration SNMPv3 (sécurisé)

SNMPv3 introduit l'authentification et le chiffrement.

#### Créer un groupe SNMPv3

```bash
# Groupe avec authentification et chiffrement (priv)
Router(config)# snmp-server group ADMIN_GROUP v3 priv

# Groupe avec authentification seulement (auth)
Router(config)# snmp-server group MONITORING_GROUP v3 auth

# Groupe avec ACL
Router(config)# snmp-server group SECURE_GROUP v3 priv access 20
```

**Niveaux de sécurité** :

- `noauth` : pas d'authentification, pas de chiffrement
- `auth` : authentification, pas de chiffrement
- `priv` : authentification + chiffrement (recommandé)

#### Créer un utilisateur SNMPv3

```bash
# Utilisateur avec authentification SHA et chiffrement AES
Router(config)# snmp-server user admin ADMIN_GROUP v3 auth sha MotDePasseAuth priv aes 128 MotDePassePriv

# Utilisateur avec authentification MD5 (moins sécurisé)
Router(config)# snmp-server user monitoring MONITORING_GROUP v3 auth md5 MotDePasseMD5
```

**Algorithmes disponibles** :

- Authentification : `md5`, `sha`
- Chiffrement : `des`, `3des`, `aes 128`, `aes 192`, `aes 256`

> [!tip] Sécurité maximale Utilisez toujours `auth sha` et `priv aes 256` pour la meilleure sécurité.

#### Configuration complète SNMPv3

```bash
# ACL pour limiter l'accès
Router(config)# access-list 30 permit host 192.168.1.100
Router(config)# access-list 30 deny any log

# Groupe avec sécurité maximale
Router(config)# snmp-server group SECURE_ADMINS v3 priv access 30

# Utilisateur avec SHA-256 et AES-256
Router(config)# snmp-server user secadmin SECURE_ADMINS v3 auth sha PassAuth2024! priv aes 256 PassPriv2024!

# Informations système
Router(config)# snmp-server contact security@entreprise.local
Router(config)# snmp-server location DC1-RACK12

# Configuration des traps SNMPv3
Router(config)# snmp-server host 192.168.1.100 version 3 priv secadmin
Router(config)# snmp-server enable traps snmp linkdown linkup
Router(config)# snmp-server enable traps config
```

### Configuration avancée

#### Définir l'interface source SNMP

```bash
Router(config)# snmp-server source-interface traps Loopback0
```

#### Limiter le nombre de requêtes

```bash
# Limiter à 50 paquets par seconde
Router(config)# snmp-server packetsize 1500
Router(config)# snmp-server queue-length 50
```

#### Désactiver certains OID (Object Identifiers)

```bash
# Créer une vue qui exclut certains OID sensibles
Router(config)# snmp-server view SAFE_VIEW internet included
Router(config)# snmp-server view SAFE_VIEW ciscoMgmt excluded

# Appliquer la vue à un groupe
Router(config)# snmp-server group LIMITED_GROUP v3 priv read SAFE_VIEW
```

### Vérification et diagnostic

```bash
# Afficher la configuration SNMP
Router# show snmp

# Afficher les community strings (soyez prudent !)
Router# show snmp community

# Afficher les utilisateurs SNMPv3
Router# show snmp user

# Afficher les groupes SNMPv3
Router# show snmp group

# Afficher les statistiques SNMP
Router# show snmp engineID

# Tester SNMP depuis un autre équipement (Linux)
$ snmpwalk -v2c -c MONITORING 192.168.1.1
$ snmpget -v3 -l authPriv -u secadmin -a SHA -A PassAuth2024! -x AES -X PassPriv2024! 192.168.1.1 sysDescr.0
```

> [!example] Sortie de `show snmp`
> 
> ```
> Chassis: RTR-CORE-01
> Contact: admin@entreprise.local
> Location: Salle_serveur_A_Batiment_3
> 
> 45 SNMP packets input
>     0 Bad SNMP version errors
>     0 Unknown community name
>     0 Illegal operation for community name supplied
> 
> 120 SNMP packets output
>     0 Too big errors
>     0 No such name errors
>     12 General errors
> 
> SNMP global trap: enabled
> ```

### Pièges courants

|Problème|Cause|Solution|
|---|---|---|
|Pas de réponse SNMP|Firewall ou ACL bloque|Vérifier UDP 161, autoriser IP source|
|Community invalide|Mauvais community string|Vérifier avec `show snmp community`|
|SNMPv3 auth failure|Mauvais mot de passe ou algorithme|Vérifier user/group et algorithmes|
|Pas de traps reçus|Configuration host manquante|Configurer `snmp-server host`|
|Traps non envoyés|Traps pas activés|Utiliser `snmp-server enable traps`|
|Accès depuis n'importe où|Pas d'ACL|Ajouter ACL à la community ou au groupe|

### OID utiles pour monitoring

|OID|Description|Utilité|
|---|---|---|
|1.3.6.1.2.1.1.1.0|sysDescr|Description du système|
|1.3.6.1.2.1.1.3.0|sysUpTime|Temps de fonctionnement|
|1.3.6.1.2.1.2.2.1.8|ifOperStatus|État opérationnel des interfaces|
|1.3.6.1.4.1.9.2.1.56.0|avgBusy5|CPU moyen sur 5 minutes|
|1.3.6.1.4.1.9.9.48.1.1.1.5|ciscoMemoryPoolUsed|Mémoire utilisée|

### Bonnes pratiques

1. **Toujours SNMPv3 en production** : SNMPv1/v2c = mots de passe en clair
2. **ACL strictes** : Limitez l'accès SNMP à vos serveurs de monitoring uniquement
3. **Read-Only par défaut** : N'utilisez RW que si absolument nécessaire
4. **Mots de passe forts** : SNMPv3 auth et priv avec 16+ caractères
5. **Interface source** : Utilisez une Loopback comme source pour la cohérence
6. **Monitoring des traps** : Configurez les traps pour être alerté des problèmes
7. **Documentation** : Gardez une trace des community strings et users SNMPv3

> [!warning] Sécurité critique SNMP mal configuré = porte dérobée vers votre réseau. Un attaquant avec RW peut reconfigurer vos équipements à distance !

---

## 🎯 Récapitulatif

Vous avez maintenant configuré les services réseau essentiels :

- **DNS statique** : Résolution de noms pour le routeur et les clients
- **Serveur HTTP/HTTPS** : Interface web de gestion (à sécuriser ou désactiver)
- **Syslog** : Journalisation centralisée et audit du réseau
- **SNMP** : Monitoring et supervision des équipements

> [!tip] Ordre de priorité en production
> 
> 1. **Syslog** → Indispensable pour l'audit et le diagnostic
> 2. **SNMP** → Essentiel pour le monitoring proactif
> 3. **DNS** → Facilite la gestion et l'utilisation
> 4. **HTTP/HTTPS** → Optionnel, souvent désactivé pour la sécurité

Ces services forment la base d'une infrastructure réseau bien gérée et monitorée.