# 📋 Synthèse — Journalisation

> Cours Wild Code School

---

## 📑 Sommaire

1. [Introduction](https://claude.ai/chat/c33a996b-b8c6-442f-bb14-938325a5489b#1---introduction)
    - [1.1 Qu'est-ce qu'un journal (log) ?](https://claude.ai/chat/c33a996b-b8c6-442f-bb14-938325a5489b#11--quest-ce-quun-journal-log-)
    - [1.2 À quoi servent les logs ?](https://claude.ai/chat/c33a996b-b8c6-442f-bb14-938325a5489b#12--%C3%A0-quoi-servent-les-logs-)
    - [1.3 Historique et conservation](https://claude.ai/chat/c33a996b-b8c6-442f-bb14-938325a5489b#13--historique-et-conservation)
    - [1.4 Pertinence des informations journalisées](https://claude.ai/chat/c33a996b-b8c6-442f-bb14-938325a5489b#14--pertinence-des-informations-journalis%C3%A9es)
    - [1.5 Obligations légales](https://claude.ai/chat/c33a996b-b8c6-442f-bb14-938325a5489b#15--obligations-l%C3%A9gales)
    - [1.6 Stockage et archivage](https://claude.ai/chat/c33a996b-b8c6-442f-bb14-938325a5489b#16--stockage-et-archivage)
    - [1.7 Standardisation et centralisation](https://claude.ai/chat/c33a996b-b8c6-442f-bb14-938325a5489b#17--standardisation-et-centralisation)
2. [Journalisation GNU/Linux](https://claude.ai/chat/c33a996b-b8c6-442f-bb14-938325a5489b#2---journalisation-gnulinux)
    - [2.1 Syslog](https://claude.ai/chat/c33a996b-b8c6-442f-bb14-938325a5489b#21--syslog)
    - [2.2 Structure d'un message syslog](https://claude.ai/chat/c33a996b-b8c6-442f-bb14-938325a5489b#22--structure-dun-message-syslog)
    - [2.3 Les catégories (Facility)](https://claude.ai/chat/c33a996b-b8c6-442f-bb14-938325a5489b#23--les-cat%C3%A9gories-facility)
    - [2.4 Les niveaux de gravité (Severity)](https://claude.ai/chat/c33a996b-b8c6-442f-bb14-938325a5489b#24--les-niveaux-de-gravit%C3%A9-severity)
    - [2.5 Envoyer un message syslog](https://claude.ai/chat/c33a996b-b8c6-442f-bb14-938325a5489b#25--envoyer-un-message-syslog)
    - [2.6 Stocker les logs avec rsyslog](https://claude.ai/chat/c33a996b-b8c6-442f-bb14-938325a5489b#26--stocker-les-logs-avec-rsyslog)
    - [2.7 Rotation des logs avec logrotate](https://claude.ai/chat/c33a996b-b8c6-442f-bb14-938325a5489b#27--rotation-des-logs-avec-logrotate)
    - [2.8 Consulter les journaux](https://claude.ai/chat/c33a996b-b8c6-442f-bb14-938325a5489b#28--consulter-les-journaux)
    - [2.9 Journalisation avec systemd](https://claude.ai/chat/c33a996b-b8c6-442f-bb14-938325a5489b#29--journalisation-avec-systemd)
    - [2.10 Outils d'analyse](https://claude.ai/chat/c33a996b-b8c6-442f-bb14-938325a5489b#210--outils-danalyse)
3. [Journalisation Windows](https://claude.ai/chat/c33a996b-b8c6-442f-bb14-938325a5489b#3---journalisation-windows)
    - [3.1 L'Observateur d'événements (Event Viewer)](https://claude.ai/chat/c33a996b-b8c6-442f-bb14-938325a5489b#31--lobservateur-d%C3%A9v%C3%A9nements-event-viewer)
    - [3.2 Accéder à l'Event Viewer](https://claude.ai/chat/c33a996b-b8c6-442f-bb14-938325a5489b#32--acc%C3%A9der-%C3%A0-levent-viewer)
    - [3.3 Contenu et organisation des journaux](https://claude.ai/chat/c33a996b-b8c6-442f-bb14-938325a5489b#33--contenu-et-organisation-des-journaux)
    - [3.4 Centralisation avec WEF](https://claude.ai/chat/c33a996b-b8c6-442f-bb14-938325a5489b#34--centralisation-avec-wef)
    - [3.5 Niveaux de criticité et Event ID](https://claude.ai/chat/c33a996b-b8c6-442f-bb14-938325a5489b#35--niveaux-de-criticit%C3%A9-et-event-id)
    - [3.6 Event IDs importants à surveiller](https://claude.ai/chat/c33a996b-b8c6-442f-bb14-938325a5489b#36--event-ids-importants-%C3%A0-surveiller)
    - [3.7 Analyse et parsing de logs](https://claude.ai/chat/c33a996b-b8c6-442f-bb14-938325a5489b#37--analyse-et-parsing-de-logs)
4. [🎯 Points clés à retenir](https://claude.ai/chat/c33a996b-b8c6-442f-bb14-938325a5489b#-points-cl%C3%A9s-%C3%A0-retenir)

---

## 1 - Introduction


## 2 - Journalisation GNU/Linux


## 3 - Journalisation Windows

### 3.1 L'Observateur d'événements (Event Viewer)

L'**Observateur d'événements** (Event Viewer en anglais) est l'outil natif Windows qui enregistre toute l'activité du système.

|Caractéristique|Détail|
|---|---|
|**Disponibilité**|Nativement sur tous les OS Windows (client et serveur)|
|**Emplacement des fichiers**|`C:\Windows\System32\winevt\Logs`|
|**Format des fichiers**|`.evtx` (binaire, non lisible directement)|
|**Accès**|Local ou **distant** via la console MMC|

---

### 3.2 Accéder à l'Event Viewer

```
# Méthode 1 : Boîte d'exécution
Windows + R → taper : eventvwr

# Méthode 2 : PowerShell / CMD
eventvwr.msc

# Méthode 3 : Recherche Windows
Rechercher "Observateur d'événements"

# Méthode 4 : Server Manager (sur un serveur)
Tools > Event Viewer
```

---

### 3.3 Contenu et organisation des journaux

Les journaux Windows sont organisés en **4 catégories** :

|Catégorie|Description|
|---|---|
|**Affichages personnalisés**|Vues filtrées créées par l'administrateur|
|**Journaux Windows**|Logs de l'OS (Application, Sécurité, Système...)|
|**Journaux des applications et services**|Logs des applications et rôles installés|
|**Abonnements**|Centralisation des logs de machines distantes|

**Types de messages dans chaque journal :**

- 🔴 **Erreurs** — dysfonctionnements
- 🟡 **Avertissements** — situation à surveiller
- ⚪ **Informations** — activité normale

---

### 3.4 Centralisation avec WEF

**WEF** (Windows Event Forwarding) permet de **collecter les logs de plusieurs machines Windows** vers un serveur central.

|Élément|Détail|
|---|---|
|**Protocole**|WinRM (Windows Remote Management)|
|**Port HTTP**|`5985`|
|**Port HTTPS**|`5986` (recommandé)|
|**Configuration**|Via GPO (Group Policy Object) possible|

> - **WEF** = Windows Event Forwarding (transfert d'événements Windows)
> - **WinRM** = Windows Remote Management (gestion à distance Windows)
> - **GPO** = Group Policy Object (stratégie de groupe pour configurer les machines du domaine)

**Événements minimaux à centraliser :**

|Event ID|Description|
|---|---|
|`4624`|Authentification réussie|
|`4625`|Échec d'authentification|
|`4740`|Verrouillage de compte|
|`4728` / `4732` / `4756`|Modifications de groupes de sécurité|
|`1102`|Suppression des journaux (**très suspect !**)|
|`4663`|Accès à des objets sensibles|

---

### 3.5 Niveaux de criticité et Event ID

Windows utilise **3 niveaux de criticité** (plus simples que syslog) :

|Niveau|Description|
|---|---|
|🔴 **High**|Critique, action immédiate requise|
|🟡 **Medium**|Important, à surveiller|
|🟢 **Low**|Informatif|

> **Event ID** = code numérique identifiant le **type d'événement**. Chaque action Windows a un Event ID unique.

---

### 3.6 Event IDs importants à surveiller

|Event ID|Description|Intérêt sécurité|
|---|---|---|
|**4624**|Connexion réussie (logon success)|🔵 Surveillance normale|
|**4625**|Échec de connexion (logon failure)|🟠 Tentative d'intrusion ?|
|**4740**|Compte verrouillé|🟠 Attaque par force brute ?|
|**4728**|Ajout dans un groupe de sécurité global|🟠 Élévation de privilèges ?|
|**4732**|Ajout dans un groupe de sécurité local|🟠 Élévation de privilèges ?|
|**4756**|Ajout dans un groupe de sécurité universel|🟠 Élévation de privilèges ?|
|**4663**|Tentative d'accès à des objets sensibles|🔴 Exfiltration de données ?|
|**1102**|Suppression des journaux|🔴 Effacement des traces !|

**Consulter les logs en PowerShell :**

```powershell
# Lister les 10 derniers événements de Sécurité
Get-EventLog -LogName Security -Newest 10

# Filtrer par Event ID (ex: échecs de connexion)
Get-EventLog -LogName Security -InstanceId 4625

# Avec Get-WinEvent (plus puissant)
Get-WinEvent -FilterHashtable @{LogName='Security'; Id=4625}

# Exporter les logs
wevtutil epl Security C:\backup\security.evtx
```

---

### 3.7 Analyse et parsing de logs

**Parsing** = analyse du contenu textuel des logs pour en extraire des informations structurées.

Les **regex** (expressions régulières / Regular Expressions) sont des motifs de recherche puissants utilisés pour automatiser le parsing.

```powershell
# Exemple : rechercher les échecs de connexion avec PowerShell + regex
Get-Content "C:\log.txt" | Select-String -Pattern "\b4625\b"
```

**Outils d'analyse pour Windows :**

- **Event Viewer** (natif) — analyse manuelle
- **Graylog**, **Splunk**, **Elastic Stack (ELK)** — plateformes centralisées
- **SIEM** (ex: Microsoft Sentinel) — analyse avancée et corrélation d'événements

> - **regex** = Regular Expression (expression régulière) — motif de recherche dans du texte
> - **parsing** = analyse et extraction d'informations structurées depuis un texte brut
> - **ELK** = Elasticsearch, Logstash, Kibana (stack open-source d'analyse de logs)

---

## 🎯 Points clés à retenir

### Ce qu'il faut absolument savoir

|Thème|À retenir|
|---|---|
|**Définition**|Un log = trace de l'activité d'un système, application ou réseau|
|**Utilité**|Debug, administration, cybersécurité|
|**Syslog Linux**|Protocole standard, RFC 5424, port 514/UDP ou 6514/TCP+TLS|
|**Gravité syslog**|8 niveaux : 0 (Emergency) → 7 (Debug)|
|**Catégories syslog**|24 catégories (kern, auth, daemon, cron...)|
|**Stockage Linux**|`/var/log/` via rsyslog, format texte|
|**Rotation Linux**|`logrotate` + `cron` pour archiver et supprimer les vieux logs|
|**Systemd**|`journalctl` pour consulter les logs en format binaire|
|**Lire les logs Linux**|`tail -f`, `grep`, `journalctl -f`|
|**Windows Event Viewer**|Logs dans `C:\Windows\System32\winevt\Logs`|
|**Accès Windows**|`eventvwr` depuis Exécuter (Win+R)|
|**Centralisation Windows**|WEF via WinRM (port 5985/5986)|
|**Event IDs critiques**|4625 (échec connexion), 1102 (suppression logs), 4740 (compte bloqué)|
|**Obligations légales**|RGPD art. 32, LPM — respecter la durée de conservation|
|**Analyse avancée**|SIEM, HIDS, SOC pour détecter les menaces automatiquement|

### 🔑 Règles d'or

1. **Centraliser** les logs pour ne pas les perdre en cas d'incident
2. **Protéger** les logs (une suppression de logs = Event ID 1102 = alerte !)
3. **Chiffrer** les logs en transit (TLS/HTTPS)
4. **Archiver** régulièrement pour économiser l'espace disque
5. **Conserver** selon les obligations légales en vigueur

---

## 📚 Sources

- Cours Wild Code School — _Journalisation_ (support PDF)
- [RFC 5424 — The Syslog Protocol](https://datatracker.ietf.org/doc/html/rfc5424)
- [Syslog — Wikipédia](https://fr.wikipedia.org/wiki/Syslog)
- [Microsoft Docs — Windows Event IDs](https://docs.microsoft.com/en-us/windows/security/threat-protection/auditing/security-auditing-overview)
- [rsyslog Documentation](https://www.rsyslog.com/doc/)
- [SIEM — Wikipedia](https://en.wikipedia.org/wiki/Security_information_and_event_management)