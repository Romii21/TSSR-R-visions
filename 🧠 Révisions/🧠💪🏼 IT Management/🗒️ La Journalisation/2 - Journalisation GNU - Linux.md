### 2.1 Syslog

**Syslog** est le protocole standard de journalisation sous Unix/Linux.

- Créé dans les années **1980** par Eric Allman pour le logiciel Sendmail
- Standardisé par l'**IETF** (Internet Engineering Task Force) : **RFC 5424**
- Architecture **client-serveur** qui sépare clairement trois rôles :

|Rôle|Description|
|---|---|
|**Générateur**|L'application ou le service qui produit les messages|
|**Stockage**|Le daemon (rsyslog) qui enregistre les messages|
|**Analyseur**|L'outil qui lit et interprète les messages|

**Ports réseau utilisés par syslog :**

|Port|Protocole|Sécurité|
|---|---|---|
|`514/UDP`|UDP|❌ Non sécurisé|
|`514/TCP`|TCP|✅ Meilleure fiabilité|
|`6514/TCP`|TCP + TLS|✅✅ Chiffré (recommandé)|

> - **UDP** = User Datagram Protocol (rapide, sans garantie de livraison)
> - **TCP** = Transmission Control Protocol (avec garantie de livraison)
> - **TLS** = Transport Layer Security (chiffrement du trafic)
> - **IETF** = Internet Engineering Task Force (organisme de standardisation Internet)
> - **RFC** = Request For Comments (documents de standardisation)

---

### 2.2 Structure d'un message syslog

Un message syslog contient toujours les informations suivantes :

|Champ|Description|
|---|---|
|**Date**|Horodatage de l'événement|
|**Hôte**|Machine qui émet le message|
|**Service/Processus**|Application ou service à l'origine du message|
|**PID**|Identifiant du processus (Process ID)|
|**Type**|Identifiant du type de message|
|**Priorité**|Combinaison de la catégorie (Facility) et du niveau de sévérité (Severity)|
|**Message**|Texte décrivant l'événement|

---

### 2.3 Les catégories (Facility)

Syslog définit **24 catégories** de messages numérotées de 0 à 23. Les 8 dernières (16 à 23) sont **réservées pour un usage local** (local0 à local7).

|Code|Mot-clé|Description|
|---|---|---|
|0|`kern`|Messages du noyau|
|1|`user`|Messages de l'espace utilisateur|
|2|`mail`|Système de messagerie|
|3|`daemon`|Services (daemons)|
|4|`auth`|Messages d'authentification|
|5|`syslog`|Messages internes de syslogd|
|6|`lpr`|Messages d'impression|
|7|`news`|Messages d'actualités|
|8|`uucp`|Messages UUCP|
|9|`cron`|Tâches planifiées (at/cron)|
|10|`authpriv`|Sécurité / élévation de privilèges|
|11|`ftp`|Logiciel FTP|
|12|`ntp`|Synchronisation du temps (NTP)|
|13|`security`|Log audit|
|14|`console`|Log alert console|
|15|`solaris-cron`|Tâches planifiées (variante)|
|16–23|`local0`–`local7`|Usage local, non défini dans la norme|

> - **daemon** = service tournant en arrière-plan
> - **NTP** = Network Time Protocol (synchronisation de l'heure)
> - **FTP** = File Transfer Protocol (transfert de fichiers)
> - **UUCP** = Unix-to-Unix Copy Protocol (protocole historique)

---

### 2.4 Les niveaux de gravité (Severity)

Syslog définit **8 niveaux de gravité**, du plus critique au moins critique :

|Code|Gravité|Mot-clé|Description|
|---|---|---|---|
|**0**|Emergency|`emerg`|🔴 Système inutilisable|
|**1**|Alert|`alert`|🔴 Intervention immédiate nécessaire|
|**2**|Critical|`crit`|🟠 Erreur critique pour le système|
|**3**|Error|`err`|🟠 Erreur de fonctionnement|
|**4**|Warning|`warn`|🟡 Avertissement (erreur possible sans action)|
|**5**|Notice|`notice`|🔵 Événement normal méritant d'être signalé|
|**6**|Informational|`info`|⚪ Pour information|
|**7**|Debugging|`debug`|⚪ Message de débogage|

> 💡 En production, on configure généralement le niveau à **Warning** (4) ou **Notice** (5) pour ne pas générer trop de logs.

---

### 2.5 Envoyer un message syslog

Une application envoie des messages syslog via des **appels système**.

**Depuis un script Bash :**

```bash
# Envoyer un message syslog depuis le terminal ou un script
logger "Mon message de test"

# Avec une priorité spécifique (catégorie.sévérité)
logger -p auth.warning "Tentative de connexion suspecte"

# Avec un tag (nom du programme)
logger -t monscript -p local0.info "Démarrage du script"

# Voir le manuel
man logger
```

**Depuis Python :**

```python
import syslog

syslog.openlog("mon_application")
syslog.syslog(syslog.LOG_WARNING, "Message d'avertissement")
syslog.closelog()
```

---

### 2.6 Stocker les logs avec rsyslog

Le stockage est assuré par le **daemon rsyslog** (Reliable Syslog), successeur moderne de syslog.

**Fichiers de logs principaux sous Linux (`/var/log/`) :**

|Catégorie|Fichier|
|---|---|
|`auth`|`/var/log/auth.log`|
|`kern`|`/var/log/kern.log`|
|Tous les messages|`/var/log/syslog`|
|Messages système|`/var/log/messages`|

**Autres destinations possibles :**

- Base de données (PostgreSQL, MySQL)
- Transmission à un serveur syslog distant

**Configuration client syslog (`/etc/rsyslog.conf`) :**

```bash
# Modifier la configuration rsyslog
sudo nano /etc/rsyslog.conf

# Redémarrer rsyslog après modification
sudo systemctl restart rsyslog
```

**Bonnes pratiques de configuration :**

- ✅ Toujours préférer **TCP** à UDP (garantie de livraison des messages)
- ✅ Utiliser **TLS** pour chiffrer les logs en transit (port 6514)
- ✅ Filtrer les **IP sources autorisées** via le pare-feu

---

### 2.7 Rotation des logs avec logrotate

**logrotate** est l'outil standard pour gérer la taille des fichiers de logs. Il est installé par défaut et lancé automatiquement par **cron**.

**Fonctionnement :**

- Archive et compresse les anciens fichiers de logs (ex: `syslog.1.gz`)
- Supprime les archives trop anciennes
- Peut envoyer les logs par mail avant suppression

```bash
# Voir la configuration de logrotate
cat /etc/logrotate.conf

# Forcer une rotation manuellement
sudo logrotate -f /etc/logrotate.conf

# Documentation
man logrotate
man cron
```

> - **cron** = planificateur de tâches sous Linux
> - **logrotate** = outil de rotation/compression automatique des logs

---

### 2.8 Consulter les journaux

**Commandes CLI pour lire les logs :**

|Commande|Usage|Exemple|
|---|---|---|
|`cat`|Afficher un fichier entier|`cat /var/log/syslog`|
|`less` / `more`|Afficher page par page|`less /var/log/auth.log`|
|`zcat` / `zless`|Lire un fichier **compressé** (.gz)|`zcat /var/log/syslog.1.gz`|
|`head`|Afficher les premières lignes|`head -20 /var/log/syslog`|
|`tail`|Afficher les dernières lignes|`tail -f /var/log/syslog`|
|`grep`|Rechercher dans un fichier|`grep "error" /var/log/syslog`|
|`zgrep`|Rechercher dans un fichier **compressé**|`zgrep "error" /var/log/syslog.1.gz`|
|`watch`|Répéter une commande périodiquement|`watch -n 5 tail /var/log/syslog`|
|`dmesg`|Logs du **noyau** Linux|`dmesg|
|`last`|Historique des connexions utilisateurs|`last`|
|`lastb`|Historique des **échecs** de connexion|`lastb`|

> - **tail -f** = affiche les nouvelles lignes en temps réel (très utile pour surveiller un log)
> - **grep** = filtre les lignes contenant un motif recherché

---

### 2.9 Journalisation avec systemd

Les distributions Linux modernes utilisent **systemd** qui intègre son propre système de journalisation : le **journal systemd**.

|Caractéristique|Détail|
|---|---|
|**Format de stockage**|Binaire (non lisible directement) dans `/run/systemd/journal`|
|**Outil de consultation**|`journalctl`|
|**Rotation**|Mécanisme de rotation interne intégré|
|**Compatibilité**|Peut transmettre à syslog pour compatibilité|

**Commandes journalctl :**

```bash
# Voir tous les logs
journalctl

# Suivre les logs en temps réel
journalctl -f

# Logs d'un service spécifique
journalctl -u nginx

# Logs depuis le dernier démarrage
journalctl -b

# Filtrer par niveau de sévérité (0=emerg, 3=err, 4=warning...)
journalctl -p 3

# Logs entre deux dates
journalctl --since "2024-01-01" --until "2024-01-31"
```

> - **systemd** = système d'initialisation et de gestion des services sous Linux moderne
> - **journalctl** = outil de consultation du journal systemd

---

### 2.10 Outils d'analyse

Pour automatiser le traitement et l'analyse des logs :

|Outil|Type|Description|
|---|---|---|
|**logwatch**|Analyse locale|Génère des rapports résumés des logs système|
|**Graylog**|Centralisation|Plateforme de gestion centralisée des logs|
|**LogAnalyzer**|Interface web|Visualisation des logs syslog|
|**SIEM**|Sécurité avancée|Security Information and Event Management — corrèle des événements multi-sources|
|**HIDS**|Sécurité avancée|Host-based Intrusion Detection System — détecte les intrusions sur un hôte|

> - **SIEM** = Security Information and Event Management (centralise et corrèle les logs pour détecter les menaces)
> - **HIDS** = Host Intrusion Detection System (surveille les activités suspectes sur une machine)
> - **SOC** = Security Operations Center (équipe dédiée à la surveillance de la sécurité)

---
