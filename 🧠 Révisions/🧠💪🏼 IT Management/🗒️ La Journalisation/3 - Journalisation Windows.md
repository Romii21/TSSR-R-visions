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
