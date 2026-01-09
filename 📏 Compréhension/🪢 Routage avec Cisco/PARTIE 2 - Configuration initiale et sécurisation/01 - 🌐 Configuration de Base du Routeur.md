# 

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

## 1. Nom d'hôte (Hostname)

### 🎯 Pourquoi configurer un hostname ?

Le nom d'hôte identifie de manière unique votre routeur sur le réseau. C'est essentiel pour :

- **Identifier rapidement** le périphérique lors de connexions multiples
- **Différencier** plusieurs routeurs dans une topologie
- **Faciliter la gestion** et le dépannage
- **Améliorer la sécurité** en évitant les noms par défaut (Router, Switch)

> [!info] Convention de nommage Utilisez des noms descriptifs qui indiquent l'emplacement, la fonction ou le site du routeur. Exemple : `R1-Paris-Gateway`, `RouterCore-Batiment-A`, `Edge-Router-NYC`

### ⚙️ Configuration

```bash
Router> enable
Router# configure terminal
Router(config)# hostname R1-Paris

# Le prompt change immédiatement
R1-Paris(config)#
```

> [!warning] Restrictions importantes
> 
> - **Pas d'espaces** : utilisez des tirets (-) ou underscores (_)
> - **Caractères autorisés** : lettres, chiffres, tirets et underscores
> - **Longueur maximale** : 63 caractères
> - **Sensible à la casse** : `Router1` ≠ `router1`

### ✅ Bonnes pratiques

|✓ À faire|✗ À éviter|
|---|---|
|`R1-HQ-Core`|`Router` (nom par défaut)|
|`SW-Access-B2`|`Mon Routeur` (espaces)|
|`FW-DMZ-01`|`Test123` (peu descriptif)|
|`RTR-Paris-Edge`|`ééé-routeur` (caractères spéciaux)|

> [!tip] Astuce professionnelle Adoptez une convention de nommage cohérente dans toute votre infrastructure :
> 
> - **Préfixe** : Type d'équipement (R=Router, SW=Switch, FW=Firewall)
> - **Site** : Localisation géographique
> - **Fonction** : Rôle dans le réseau

---

## 2. Bannières (Banners)

### 🎯 Pourquoi configurer des bannières ?

Les bannières affichent des messages aux utilisateurs qui se connectent au routeur. Elles servent à :

- **Avertir légalement** contre les accès non autorisés
- **Informer** sur les conditions d'utilisation
- **Dissuader** les intrusions (aspect juridique important)
- **Communiquer** des messages de maintenance ou d'urgence

### 📋 Types de bannières

|Type|Commande|Quand s'affiche-t-elle ?|
|---|---|---|
|**MOTD** (Message of the Day)|`banner motd`|Avant le login, pour tous les utilisateurs|
|**Login**|`banner login`|Juste avant la demande de mot de passe|
|**Exec**|`banner exec`|Après connexion réussie (mode privilégié)|

### ⚙️ Configuration MOTD (la plus utilisée)

```bash
R1-Paris# configure terminal
R1-Paris(config)# banner motd #
Enter TEXT message. End with the character '#'.
***********************************************
* ACCES RESERVE - AUTHORIZED ACCESS ONLY     *
* Toute tentative non autorisee sera         *
* enregistree et poursuivie judiciairement   *
* Unauthorized access is strictly prohibited *
***********************************************
#

R1-Paris(config)#
```

> [!info] Caractère délimiteur Le symbole `#` est le délimiteur qui marque le début ET la fin du message. Vous pouvez utiliser n'importe quel caractère qui n'apparaît PAS dans votre message : `$`, `%`, `@`, `&`, etc.

### ⚙️ Configuration Login Banner

```bash
R1-Paris(config)# banner login $
Enter TEXT message. End with the character '$'.
====================================
  Authentification requise
  Login required for access
====================================
$
```

### ⚙️ Configuration Exec Banner

```bash
R1-Paris(config)# banner exec |
Enter TEXT message. End with the character '|'.
Bienvenue sur R1-Paris
Contactez admin@entreprise.com pour assistance
|
```

> [!warning] Pièges courants
> 
> - **Ne jamais utiliser "Welcome"** dans une bannière MOTD : cela peut affaiblir votre position légale en cas d'intrusion
> - **Ne pas inclure le délimiteur** dans le texte de la bannière
> - **Éviter les informations sensibles** (version IOS, modèle de routeur) qui aident les attaquants

### ✅ Exemple de bannière juridiquement solide

```bash
R1-Paris(config)# banner motd @
***********************************************************
*                    AVERTISSEMENT                        *
*                                                         *
* Ce systeme est reserve aux utilisateurs autorises.     *
* L'acces non autorise est interdit par la loi.          *
* Toutes les activites sont surveillees et enregistrees. *
* Les contrevenants seront poursuivis.                   *
*                                                         *
* This system is for authorized users only.              *
* Unauthorized access is prohibited by law.              *
* All activities are monitored and logged.               *
***********************************************************
@
```

> [!tip] Astuce pour bannières multi-lignes Pour une meilleure lisibilité, utilisez des caractères ASCII pour créer des cadres :
> 
> - `*`, `#`, `=` pour les bordures
> - Alignez le texte pour un rendu professionnel
> - Incluez une version anglaise pour les contextes internationaux

### 🗑️ Supprimer une bannière

```bash
R1-Paris(config)# no banner motd
R1-Paris(config)# no banner login
R1-Paris(config)# no banner exec
```

---

## 3. Configuration de l'horloge

### 🎯 Pourquoi configurer l'horloge ?

Une horloge précise est **essentielle** pour :

- **Logs horodatés** précis pour le débogage et l'audit
- **Certificats SSL/TLS** (validation de dates)
- **Protocoles de sécurité** (Kerberos, certificats)
- **Corrélation d'événements** entre plusieurs équipements
- **Conformité réglementaire** (traçabilité temporelle)

> [!warning] Impact critique Sans horloge correcte, vos logs afficheront des dates incorrectes, rendant le dépannage et les investigations de sécurité quasi impossibles.

### ⚙️ Configuration manuelle de l'horloge

```bash
# Vérifier l'heure actuelle
R1-Paris# show clock
*00:05:23.456 UTC Mon Mar 1 1993

# Configurer l'heure (mode EXEC privilégié, PAS config)
R1-Paris# clock set 14:30:00 21 December 2025

# Vérification
R1-Paris# show clock
14:30:02.123 UTC Sun Dec 21 2025
```

> [!info] Syntaxe de la commande
> 
> ```bash
> clock set HH:MM:SS DD Month YYYY
> # HH:MM:SS = heure au format 24h
> # DD = jour (1-31)
> # Month = mois en anglais (January, February, etc.)
> # YYYY = année complète (2025)
> ```

### 📅 Format des mois (en anglais)

|Abréviation|Mois complet|Numéro|
|---|---|---|
|Jan|January|01|
|Feb|February|02|
|Mar|March|03|
|Apr|April|04|
|May|May|05|
|Jun|June|06|
|Jul|July|07|
|Aug|August|08|
|Sep|September|09|
|Oct|October|10|
|Nov|November|11|
|Dec|December|12|

> [!example] Exemples de configuration
> 
> ```bash
> # 15h45 le 25 décembre 2025
> R1-Paris# clock set 15:45:00 25 Dec 2025
> 
> # 08h30 le 1er janvier 2026
> R1-Paris# clock set 08:30:00 1 January 2026
> 
> # Minuit le 31 mars 2025
> R1-Paris# clock set 00:00:00 31 Mar 2025
> ```

> [!warning] Piège fréquent La commande `clock set` s'exécute en mode **EXEC privilégié** (#), PAS en mode configuration globale (config)#. C'est une exception !

### 🔍 Commandes de vérification

```bash
# Afficher l'heure actuelle
R1-Paris# show clock

# Afficher l'heure avec plus de détails
R1-Paris# show clock detail
14:30:45.678 UTC Sun Dec 21 2025
Time source is hardware calendar

# Vérifier le fuseau horaire configuré
R1-Paris# show running-config | include clock
```

> [!tip] Bonne pratique Configurez toujours l'horloge **immédiatement** après la configuration initiale du routeur, même si vous prévoyez d'utiliser NTP plus tard. Cela garantit des logs cohérents dès le départ.

---

## 4. Fuseau horaire et synchronisation NTP

### 🎯 Pourquoi configurer le fuseau horaire et NTP ?

#### Fuseau horaire

- Affiche l'heure **locale** plutôt que UTC dans les logs
- Facilite la **lecture** et la **corrélation** des événements
- Gère automatiquement le **passage heure d'été/hiver**

#### NTP (Network Time Protocol)

- **Synchronisation automatique** avec une source de temps fiable
- **Précision** à la milliseconde (indispensable pour la sécurité)
- **Cohérence** temporelle entre tous les équipements du réseau
- **Maintenance simplifiée** (pas de configuration manuelle répétée)

> [!info] Hiérarchie NTP - Stratum NTP utilise une hiérarchie de niveaux (stratum) :
> 
> - **Stratum 0** : Horloge atomique, GPS (source de référence)
> - **Stratum 1** : Serveurs connectés directement au Stratum 0
> - **Stratum 2** : Serveurs synchronisés avec Stratum 1
> - **Stratum 3-15** : Niveaux suivants de synchronisation
> 
> Votre routeur sera typiquement Stratum 3 ou 4.

### ⚙️ Configuration du fuseau horaire

```bash
R1-Paris(config)# clock timezone CET 1
# CET = Central European Time
# 1 = décalage en heures par rapport à UTC (+1h)

# Pour la France métropolitaine
R1-Paris(config)# clock timezone CET 1

# Activer l'heure d'été (daylight saving time)
R1-Paris(config)# clock summer-time CEST recurring
# CEST = Central European Summer Time
# recurring = ajustement automatique chaque année
```

> [!info] Syntaxe complète
> 
> ```bash
> clock timezone ZONE hours [minutes]
> clock summer-time ZONE recurring [week day month hh:mm week day month hh:mm [offset]]
> 
> # ZONE = nom du fuseau (3-4 caractères recommandés)
> # hours = décalage par rapport à UTC (-23 à +23)
> # minutes = décalage en minutes (optionnel, 0-59)
> ```

### 🌍 Exemples de fuseaux horaires courants

```bash
# France / Europe centrale
R1-Paris(config)# clock timezone CET 1
R1-Paris(config)# clock summer-time CEST recurring

# Royaume-Uni
R1-London(config)# clock timezone GMT 0
R1-London(config)# clock summer-time BST recurring

# États-Unis - New York (EST)
R1-NYC(config)# clock timezone EST -5
R1-NYC(config)# clock summer-time EDT recurring

# États-Unis - Los Angeles (PST)
R1-LA(config)# clock timezone PST -8
R1-LA(config)# clock summer-time PDT recurring

# Tokyo (pas d'heure d'été)
R1-Tokyo(config)# clock timezone JST 9
```

|Fuseau|Abréviation|Décalage UTC|Heure d'été|
|---|---|---|---|
|France|CET / CEST|+1 / +2|Oui|
|UK|GMT / BST|0 / +1|Oui|
|New York|EST / EDT|-5 / -4|Oui|
|Los Angeles|PST / PDT|-8 / -7|Oui|
|Tokyo|JST|+9|Non|

### ⚙️ Configuration NTP

#### Étape 1 : Définir un serveur NTP

```bash
# Configurer un serveur NTP
R1-Paris(config)# ntp server 195.83.132.105
# Serveur NTP public français (ntp.obspm.fr)

# Configurer plusieurs serveurs pour la redondance
R1-Paris(config)# ntp server 195.83.132.105
R1-Paris(config)# ntp server 129.6.15.28
R1-Paris(config)# ntp server 216.239.35.0
```

> [!tip] Serveurs NTP publics recommandés **France :**
> 
> - `195.83.132.105` (ntp.obspm.fr - Observatoire de Paris)
> - `145.238.203.14` (ntp2.obspm.fr)
> 
> **Internationaux :**
> 
> - `pool.ntp.org` (pool mondial)
> - `time.google.com` (Google)
> - `time.cloudflare.com` (Cloudflare)
> - `time.windows.com` (Microsoft)

#### Étape 2 : Définir le serveur maître (optionnel)

```bash
# Désigner un serveur NTP comme source préférée
R1-Paris(config)# ntp server 195.83.132.105 prefer
```

#### Étape 3 : Configurer l'authentification NTP (sécurité renforcée)

```bash
# Activer l'authentification NTP
R1-Paris(config)# ntp authenticate

# Créer une clé d'authentification
R1-Paris(config)# ntp authentication-key 1 md5 SecureKey123

# Définir les clés de confiance
R1-Paris(config)# ntp trusted-key 1

# Associer la clé au serveur
R1-Paris(config)# ntp server 195.83.132.105 key 1
```

> [!warning] Sécurité NTP L'authentification NTP est **fortement recommandée** en environnement de production pour éviter :
> 
> - Les attaques de type "man-in-the-middle"
> - La synchronisation avec des serveurs malveillants
> - Les dérives temporelles intentionnelles

#### Étape 4 : Configurer le routeur comme serveur NTP (optionnel)

```bash
# Permettre au routeur de servir l'heure à d'autres équipements
R1-Paris(config)# ntp master 3
# 3 = niveau de stratum (1-15)
# Utilisez un niveau approprié (généralement 3-5 pour un routeur)
```

### 🔍 Commandes de vérification NTP

```bash
# Vérifier les associations NTP
R1-Paris# show ntp associations

     address         ref clock       st   when   poll reach  delay  offset   disp
*~195.83.132.105  .GPS.            1     64    128  377  15.23   -2.45   1.234
* = synchronized, ~ = configured

# Vérifier le statut NTP détaillé
R1-Paris# show ntp status
Clock is synchronized, stratum 2, reference is 195.83.132.105
nominal freq is 250.0000 Hz, actual freq is 249.9995 Hz, precision is 2**18
reference time is E8B4A3C1.4D5E6F7A (14:30:45.302 CET Sun Dec 21 2025)
clock offset is -2.4532 msec, root delay is 15.23 msec
root dispersion is 8.45 msec, peer dispersion is 1.23 msec

# Vérifier l'horloge avec fuseau horaire
R1-Paris# show clock detail
14:30:45.678 CET Sun Dec 21 2025
Time source is NTP
Summer time starts 02:00:00 CET Sun Mar 30 2025
Summer time ends 03:00:00 CEST Sun Oct 26 2025
```

> [!info] Interprétation du statut NTP
> 
> - **Clock is synchronized** : Synchronisation réussie
> - **stratum X** : Niveau dans la hiérarchie NTP
> - **clock offset** : Différence avec le serveur (en millisecondes)
> - **root delay** : Délai total vers la source stratum 0
> - **Time source is NTP** : L'heure provient bien de NTP

### 🔧 Dépannage NTP

```bash
# Vérifier les associations avec détails
R1-Paris# show ntp associations detail

# Vérifier la configuration NTP
R1-Paris# show running-config | include ntp

# Réinitialiser les associations NTP
R1-Paris# clear ntp associations

# Déboguer NTP (attention, verbeux !)
R1-Paris# debug ntp all
# Pour arrêter le debug
R1-Paris# undebug all
```

### ⚠️ Problèmes courants NTP

|Problème|Cause possible|Solution|
|---|---|---|
|Pas de synchronisation|Firewall bloque UDP 123|Autoriser le port NTP|
|"Clock not synchronized"|Serveur NTP inaccessible|Vérifier connectivité réseau|
|Décalage trop important|Différence initiale > 1000s|Régler manuellement l'horloge d'abord|
|Stratum 16|Aucune source valide|Vérifier la configuration du serveur|

> [!tip] Bonne pratique de déploiement **Ordre de configuration recommandé :**
> 
> 1. Configurer le fuseau horaire
> 2. Régler l'horloge manuellement (approximativement)
> 3. Configurer NTP
> 4. Vérifier la synchronisation
> 
> Cette approche évite les décalages temporels trop importants qui peuvent empêcher la synchronisation NTP initiale.

### 📋 Configuration complète recommandée

```bash
R1-Paris# configure terminal

# Fuseau horaire
R1-Paris(config)# clock timezone CET 1
R1-Paris(config)# clock summer-time CEST recurring

# NTP avec redondance et authentification
R1-Paris(config)# ntp authenticate
R1-Paris(config)# ntp authentication-key 1 md5 Cl3Secur1s3e!
R1-Paris(config)# ntp trusted-key 1

# Serveurs NTP multiples (redondance)
R1-Paris(config)# ntp server 195.83.132.105 key 1 prefer
R1-Paris(config)# ntp server 129.6.15.28 key 1
R1-Paris(config)# ntp server 216.239.35.0

# Sauvegarder
R1-Paris(config)# exit
R1-Paris# write memory
```

> [!tip] Astuce professionnelle Dans un réseau d'entreprise :
> 
> - Configurez **un routeur central** comme maître NTP (synchronisé avec Internet)
> - Faites pointer **tous les autres équipements** vers ce routeur
> - Cela réduit le trafic Internet et garantit la cohérence temporelle interne

---

## 🎯 Récapitulatif de la configuration initiale

### Séquence de configuration recommandée

```bash
# 1. Nom d'hôte
Router(config)# hostname R1-Paris

# 2. Bannière MOTD
R1-Paris(config)# banner motd #
ACCES RESERVE - Unauthorized access prohibited
#

# 3. Fuseau horaire
R1-Paris(config)# clock timezone CET 1
R1-Paris(config)# clock summer-time CEST recurring

# 4. Horloge manuelle (approximative)
R1-Paris(config)# exit
R1-Paris# clock set 14:30:00 21 Dec 2025

# 5. NTP
R1-Paris# configure terminal
R1-Paris(config)# ntp server 195.83.132.105 prefer
R1-Paris(config)# ntp server 129.6.15.28

# 6. Sauvegarder
R1-Paris(config)# exit
R1-Paris# write memory
```

> [!example] Vérification finale
> 
> ```bash
> R1-Paris# show running-config
> R1-Paris# show clock detail
> R1-Paris# show ntp status
> R1-Paris# show ntp associations
> ```

### ✅ Checklist de configuration de base

- [ ] Hostname configuré avec une convention de nommage cohérente
- [ ] Bannière MOTD configurée avec avertissement légal
- [ ] Fuseau horaire local défini
- [ ] Heure d'été/hiver configurée (si applicable)
- [ ] Horloge réglée manuellement
- [ ] Serveurs NTP configurés avec redondance
- [ ] Synchronisation NTP vérifiée et fonctionnelle
- [ ] Configuration sauvegardée (`write memory`)

---

> [!tip] 💡 Point clé à retenir La configuration de base (hostname, bannières, horloge, NTP) est **fondamentale** pour une gestion professionnelle. Ces éléments semblent basiques mais sont **essentiels** pour :
> 
> - L'identification et la traçabilité
> - La sécurité juridique
> - Le débogage et l'audit
> - La cohérence temporelle du réseau
> 
> Ne les négligez jamais, même sur des équipements de test !