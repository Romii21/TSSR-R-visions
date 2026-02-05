## 1. Sysmon (System Monitor)

**Rôle :** "Caméra de surveillance" du système.

Il remplace/complète l'observateur d'événements standard en enregistrant des détails précis (hash de fichiers, lignes de commande exécutées, connexions réseaux, etc.).

### 🛠️ Commandes Principales

**Installation (CMD Admin)**

Installe le service, active le monitoring réseau et accepte la licence automatiquement.

DOS

```
sysmon64.exe -i -n -accepteula
```

**Vérification du service (PowerShell)**

Permet de confirmer que Sysmon tourne bien en arrière-plan.

PowerShell

```
Get-Service -Name Sysmon64
```

_Résultat attendu : Status `Running`._

**Mise à jour de la configuration**

Applique un nouveau filtre (fichier XML) pour affiner ce qu'on enregistre (filtrer le bruit).

DOS

```
sysmon64.exe -c "C:\chemin\vers\config.xml"
```

### 📍 Localisation des logs

Les logs ne sont pas dans "Application" ou "Système", mais ici :

`Observateur d'événements` > `Journaux des applications et des services` > `Microsoft` > `Windows` > `Sysmon` > `Operational`

---

## 2. Log Parser 2.2

**Rôle :** "Moteur de recherche SQL" pour logs.

Il permet de transformer des fichiers de logs illisibles (EVTX, CSV, TXT) en tableau clair, en utilisant le langage SQL.

### 🛠️ Structure d'une commande

La commande s'exécute depuis le dossier d'installation (`C:\Program Files (x86)\Log Parser 2.2`).

DOS

```
LogParser.exe -stats:OFF -i:EVT "SELECT [Champs] FROM '[Fichier]' WHERE [Condition]"
```

### 🔍 Détail des paramètres

|**Paramètre / Mot-clé**|**Utilité**|
|---|---|
|**`-i:EVT`**|**Input format.** Indique à l'outil qu'on lit un fichier d'événement Windows (.evtx).|
|**`-stats:OFF`**|Supprime les statistiques de fin (nombre de fichiers traités, temps d'exécution) pour un affichage propre.|
|**`SELECT`**|Choisit les colonnes à afficher (ex: `EventID`, `TimeGenerated`, `Message`).|
|**`FROM`**|Indique le chemin du fichier source (ex: `'C:\Temp\Application.evtx'`).|
|**`WHERE`**|Filtre les résultats (ex: `EventID = 4625` pour les échecs de connexion).|
|**`SUBSTR(x, 0, 50)`**|Fonction pour couper un texte trop long (prend les 50 premiers caractères).|

### 💡 Exemple concret (Forensics)

_Pour retrouver toutes les erreurs générées par la machine 'WINSERV1' dans un fichier exporté :_

SQL

```
LogParser.exe -stats:OFF -i:EVT "SELECT TimeGenerated, EventID, Message FROM 'C:\temp\Application.evtx' WHERE ComputerName = 'WINSERV1' AND EventID = 1"
```

---

## 🏆 Synthèse

1. **Sysmon** génère la donnée (il voit ce qu'il se passe).
    
2. **L'Event Viewer** stocke la donnée (sous format .evtx).
    
3. **Log Parser** analyse la donnée (il extrait l'information pertinente).