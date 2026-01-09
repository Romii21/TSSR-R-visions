# Liste Complète des Commandes PowerShell

## Sommaire

1. [Aide et Documentation](https://claude.ai/chat/3376a5f6-1fc8-4873-a003-b85a225b6bcf#1-aide-et-documentation)
2. [Gestion des Fichiers et Dossiers](https://claude.ai/chat/3376a5f6-1fc8-4873-a003-b85a225b6bcf#2-gestion-des-fichiers-et-dossiers)
3. [Navigation](https://claude.ai/chat/3376a5f6-1fc8-4873-a003-b85a225b6bcf#3-navigation)
4. [Affichage et Sortie](https://claude.ai/chat/3376a5f6-1fc8-4873-a003-b85a225b6bcf#4-affichage-et-sortie)
5. [Variables et Environnement](https://claude.ai/chat/3376a5f6-1fc8-4873-a003-b85a225b6bcf#5-variables-et-environnement)
6. [Processus et Services](https://claude.ai/chat/3376a5f6-1fc8-4873-a003-b85a225b6bcf#6-processus-et-services)
7. [Réseau](https://claude.ai/chat/3376a5f6-1fc8-4873-a003-b85a225b6bcf#7-r%C3%A9seau)
8. [Filtrage et Tri](https://claude.ai/chat/3376a5f6-1fc8-4873-a003-b85a225b6bcf#8-filtrage-et-tri)
9. [PowerShell à Distance](https://claude.ai/chat/3376a5f6-1fc8-4873-a003-b85a225b6bcf#9-powershell-%C3%A0-distance)
10. [Jobs (Exécution Parallèle)](https://claude.ai/chat/3376a5f6-1fc8-4873-a003-b85a225b6bcf#10-jobs-ex%C3%A9cution-parall%C3%A8le)
11. [Modules et Fonctions](https://claude.ai/chat/3376a5f6-1fc8-4873-a003-b85a225b6bcf#11-modules-et-fonctions)
12. [Système et Administration](https://claude.ai/chat/3376a5f6-1fc8-4873-a003-b85a225b6bcf#12-syst%C3%A8me-et-administration)
13. [Texte et Manipulation de Chaînes](https://claude.ai/chat/3376a5f6-1fc8-4873-a003-b85a225b6bcf#13-texte-et-manipulation-de-cha%C3%AEnes)
14. [Conversion et Formatage](https://claude.ai/chat/3376a5f6-1fc8-4873-a003-b85a225b6bcf#14-conversion-et-formatage)
15. [Gestion des Erreurs](https://claude.ai/chat/3376a5f6-1fc8-4873-a003-b85a225b6bcf#15-gestion-des-erreurs)
16. [Sécurité et Permissions](https://claude.ai/chat/3376a5f6-1fc8-4873-a003-b85a225b6bcf#16-s%C3%A9curit%C3%A9-et-permissions)
17. [Active Directory](https://claude.ai/chat/3376a5f6-1fc8-4873-a003-b85a225b6bcf#17-active-directory)
18. [Registre Windows](https://claude.ai/chat/3376a5f6-1fc8-4873-a003-b85a225b6bcf#18-registre-windows)
19. [Comparaison et Test](https://claude.ai/chat/3376a5f6-1fc8-4873-a003-b85a225b6bcf#19-comparaison-et-test)
20. [Import/Export de Données](https://claude.ai/chat/3376a5f6-1fc8-4873-a003-b85a225b6bcf#20-importexport-de-donn%C3%A9es)

---

## 1. Aide et Documentation

|Commande|Description|Exemple|
|---|---|---|
|`Get-Help`|Affiche l'aide d'une cmdlet|`Get-Help Get-Process`|
|`Get-Command`|Liste toutes les commandes disponibles|`Get-Command *Service*`|
|`Get-Member`|Affiche les propriétés et méthodes d'un objet|`Get-Process \| Get-Member`|
|`Get-Alias`|Liste les alias de commandes|`Get-Alias ls`|
|`Set-Alias`|Crée un alias personnalisé|`Set-Alias ll Get-ChildItem`|
|`Update-Help`|Met à jour l'aide PowerShell|`Update-Help`|

**Astuces** :

```powershell
Get-Help Get-Process -Full      # Aide complète
Get-Help Get-Process -Examples   # Affiche les exemples
Get-Help Get-Process -Online     # Ouvre l'aide en ligne
Get-Help about_*                 # Aide conceptuelle
```

---

## 2. Gestion des Fichiers et Dossiers

### Création et Suppression

|Commande|Alias|Description|Exemple|
|---|---|---|---|
|`New-Item`|`ni`|Crée un fichier ou dossier|`New-Item -ItemType File -Name "test.txt"`|
|`Remove-Item`|`rm, del`|Supprime un élément|`Remove-Item "test.txt"`|
|`New-Item -ItemType Directory`|`mkdir`|Crée un dossier|`New-Item -ItemType Directory -Name "MonDossier"`|

### Copie et Déplacement

|Commande|Alias|Description|Exemple|
|---|---|---|---|
|`Copy-Item`|`cp, copy`|Copie un fichier/dossier|`Copy-Item "source.txt" "destination.txt"`|
|`Move-Item`|`mv, move`|Déplace un fichier/dossier|`Move-Item "fichier.txt" "C:\Temp\"`|
|`Rename-Item`|`ren`|Renomme un élément|`Rename-Item "ancien.txt" "nouveau.txt"`|

### Lecture et Modification

| Commande        | Alias           | Description                      | Exemple                                  |
| --------------- | --------------- | -------------------------------- | ---------------------------------------- |
| `Get-Content`   | `gc, cat, type` | Lit le contenu d'un fichier      | `Get-Content "fichier.txt"`              |
| `Set-Content`   | `sc`            | Écrit dans un fichier (écrase)   | `Set-Content "fichier.txt" "Contenu"`    |
| `Add-Content`   | `ac`            | Ajoute du contenu à un fichier   | `Add-Content "log.txt" "Nouvelle ligne"` |
| `Clear-Content` | `clc`           | Efface le contenu d'un fichier   | `Clear-Content "fichier.txt"`            |
| `Out-File`      | -               | Envoie la sortie vers un fichier | `Get-Process \| Out-File "process.txt"`  |

### Exploration

|Commande|Alias|Description|Exemple|
|---|---|---|---|
|`Get-ChildItem`|`gci, ls, dir`|Liste les fichiers/dossiers|`Get-ChildItem C:\`|
|`Get-Item`|`gi`|Obtient un élément spécifique|`Get-Item "C:\Windows"`|
|`Test-Path`|-|Vérifie si un chemin existe|`Test-Path "C:\Temp"`|
|`Resolve-Path`|-|Résout un chemin relatif|`Resolve-Path "..\fichier.txt"`|

**Exemples avancés** :

```powershell
# Lister uniquement les fichiers
Get-ChildItem -File

# Lister uniquement les dossiers
Get-ChildItem -Directory

# Recherche récursive
Get-ChildItem -Recurse -Filter "*.txt"

# Taille des fichiers
Get-ChildItem | Select-Object Name, Length

# Fichiers modifiés récemment
Get-ChildItem | Where-Object {$_.LastWriteTime -gt (Get-Date).AddDays(-7)}
```

---

## 3. Navigation

|Commande|Alias|Description|Exemple|
|---|---|---|---|
|`Set-Location`|`cd, chdir, sl`|Change de répertoire|`Set-Location C:\Windows`|
|`Get-Location`|`pwd, gl`|Affiche le répertoire actuel|`Get-Location`|
|`Push-Location`|`pushd`|Sauvegarde et change de répertoire|`Push-Location C:\Temp`|
|`Pop-Location`|`popd`|Retourne au répertoire précédent|`Pop-Location`|

---

## 4. Affichage et Sortie

|Commande|Description|Exemple|
|---|---|---|
|`Write-Host`|Affiche du texte (coloré possible)|`Write-Host "Message" -ForegroundColor Green`|
|`Write-Output`|Envoie des objets dans le pipeline|`Write-Output "Résultat"`|
|`Write-Warning`|Affiche un avertissement|`Write-Warning "Attention !"`|
|`Write-Error`|Affiche une erreur|`Write-Error "Erreur critique"`|
|`Write-Verbose`|Affiche des infos détaillées|`Write-Verbose "Détails" -Verbose`|
|`Write-Debug`|Affiche des infos de débogage|`Write-Debug "Debug info" -Debug`|
|`Clear-Host`|`cls, clear`|Efface la console|

**Couleurs disponibles** :

```powershell
# Black, DarkBlue, DarkGreen, DarkCyan, DarkRed, DarkMagenta
# DarkYellow, Gray, DarkGray, Blue, Green, Cyan, Red, Magenta
# Yellow, White

Write-Host "Succès" -ForegroundColor Green -BackgroundColor Black
```

---

## 5. Variables et Environnement

### Variables

|Commande|Description|Exemple|
|---|---|---|
|`Get-Variable`|Liste les variables|`Get-Variable`|
|`Set-Variable`|Définit une variable|`Set-Variable -Name "Test" -Value "Valeur"`|
|`Remove-Variable`|Supprime une variable|`Remove-Variable -Name "Test"`|
|`Clear-Variable`|Vide une variable|`Clear-Variable -Name "Test"`|
|`New-Variable`|Crée une nouvelle variable|`New-Variable -Name "Var" -Value 10`|

### Variables d'Environnement

| Commande                                  | Description                      | Exemple                                                           |
| ----------------------------------------- | -------------------------------- | ----------------------------------------------------------------- |
| `Get-ChildItem Env:`                      | Liste toutes les variables d'env | `Get-ChildItem Env:`                                              |
| `$env:Variable`                           | Accède à une variable d'env      | `$env:USERNAME`                                                   |
| `[Environment]::GetEnvironmentVariable()` | Obtient une variable d'env       | `[Environment]::GetEnvironmentVariable("PATH")`                   |
| `[Environment]::SetEnvironmentVariable()` | Définit une variable d'env       | `[Environment]::SetEnvironmentVariable("TEST", "Valeur", "User")` |

**Variables d'environnement courantes** :

```powershell
$env:USERNAME              # Nom utilisateur
$env:COMPUTERNAME          # Nom de l'ordinateur
$env:USERPROFILE           # Profil utilisateur
$env:TEMP                  # Dossier temporaire
$env:PATH                  # Chemins système
$env:PROCESSOR_ARCHITECTURE # Architecture CPU
$env:OS                    # Système d'exploitation
```

---

## 6. Processus et Services

### Processus

|Commande|Alias|Description|Exemple|
|---|---|---|---|
|`Get-Process`|`ps, gps`|Liste les processus|`Get-Process`|
|`Start-Process`|`start`|Démarre un processus|`Start-Process notepad`|
|`Stop-Process`|`kill, spps`|Arrête un processus|`Stop-Process -Name "notepad"`|
|`Wait-Process`|-|Attend la fin d'un processus|`Wait-Process -Name "setup"`|

**Exemples** :

```powershell
# Processus par nom
Get-Process -Name "chrome"

# Tuer un processus par ID
Stop-Process -Id 1234

# Processus consommant le plus de CPU
Get-Process | Sort-Object CPU -Descending | Select-Object -First 5

# Démarrer un programme en tant qu'admin
Start-Process powershell -Verb RunAs
```

### Services

|Commande|Alias|Description|Exemple|
|---|---|---|---|
|`Get-Service`|`gsv`|Liste les services|`Get-Service`|
|`Start-Service`|`sasv`|Démarre un service|`Start-Service -Name "Spooler"`|
|`Stop-Service`|`spsv`|Arrête un service|`Stop-Service -Name "Spooler"`|
|`Restart-Service`|-|Redémarre un service|`Restart-Service -Name "Spooler"`|
|`Set-Service`|-|Modifie un service|`Set-Service -Name "Spooler" -StartupType Automatic`|
|`Suspend-Service`|-|Suspend un service|`Suspend-Service -Name "ServiceName"`|
|`Resume-Service`|-|Reprend un service|`Resume-Service -Name "ServiceName"`|

**Exemples** :

```powershell
# Services en cours d'exécution
Get-Service | Where-Object {$_.Status -eq "Running"}

# Services arrêtés
Get-Service | Where-Object {$_.Status -eq "Stopped"}

# Démarrer plusieurs services
Start-Service -Name "Service1", "Service2"
```

---

## 7. Réseau

### Connectivité

|Commande|Description|Exemple|
|---|---|---|
|`Test-Connection`|Ping une machine|`Test-Connection google.com`|
|`Test-NetConnection`|Test de connexion avancé|`Test-NetConnection -ComputerName google.com -Port 80`|
|`Resolve-DnsName`|Résolution DNS|`Resolve-DnsName google.com`|
|`Get-NetIPAddress`|Affiche les adresses IP|`Get-NetIPAddress`|
|`Get-NetAdapter`|Liste les cartes réseau|`Get-NetAdapter`|
|`Get-NetRoute`|Affiche la table de routage|`Get-NetRoute`|

### Configuration

|Commande|Description|Exemple|
|---|---|---|
|`Set-NetIPAddress`|Configure une IP|`Set-NetIPAddress -InterfaceAlias "Ethernet" -IPAddress "192.168.1.100"`|
|`New-NetIPAddress`|Crée une nouvelle IP|`New-NetIPAddress -InterfaceAlias "Ethernet" -IPAddress "192.168.1.10" -PrefixLength 24`|
|`Set-DnsClientServerAddress`|Configure DNS|`Set-DnsClientServerAddress -InterfaceAlias "Ethernet" -ServerAddresses "8.8.8.8"`|
|`Enable-NetAdapter`|Active une carte réseau|`Enable-NetAdapter -Name "Ethernet"`|
|`Disable-NetAdapter`|Désactive une carte réseau|`Disable-NetAdapter -Name "Ethernet"`|

**Exemples** :

```powershell
# Tester un port spécifique
Test-NetConnection -ComputerName serveur.local -Port 3389

# IP et masque
Get-NetIPAddress | Select-Object InterfaceAlias, IPAddress, PrefixLength

# Vider le cache DNS
Clear-DnsClientCache
```

---

## 8. Filtrage et Tri

|Commande|Alias|Description|Exemple|
|---|---|---|---|
|`Where-Object`|`where, ?`|Filtre des objets|`Get-Process \| Where-Object {$_.CPU -gt 100}`|
|`Select-Object`|`select`|Sélectionne des propriétés|`Get-Process \| Select-Object Name, CPU`|
|`Sort-Object`|`sort`|Trie des objets|`Get-Process \| Sort-Object CPU -Descending`|
|`Group-Object`|`group`|Groupe des objets|`Get-Service \| Group-Object Status`|
|`Measure-Object`|`measure`|Calcule des statistiques|`Get-ChildItem \| Measure-Object -Property Length -Sum`|
|`Compare-Object`|`compare, diff`|Compare deux ensembles|`Compare-Object -ReferenceObject $a -DifferenceObject $b`|
|`ForEach-Object`|`foreach, %`|Exécute une action sur chaque objet|`Get-Process \| ForEach-Object {$_.Name}`|

**Exemples avancés** :

```powershell
# Filtrer et trier
Get-Process | Where-Object {$_.WorkingSet -gt 100MB} | Sort-Object CPU -Descending

# Sélectionner les 5 premiers
Get-Process | Sort-Object CPU -Descending | Select-Object -First 5

# Grouper par statut
Get-Service | Group-Object Status | Select-Object Name, Count

# Statistiques
Get-ChildItem -File | Measure-Object -Property Length -Sum -Average -Maximum -Minimum

# Objets uniques
Get-Process | Select-Object -Property ProcessName -Unique
```

---

## 9. PowerShell à Distance

### Sessions

|Commande|Description|Exemple|
|---|---|---|
|`Enter-PSSession`|Démarre une session interactive|`Enter-PSSession -ComputerName Server01`|
|`Exit-PSSession`|Ferme la session interactive|`Exit-PSSession`|
|`New-PSSession`|Crée une session persistante|`$s = New-PSSession -ComputerName Server01`|
|`Get-PSSession`|Liste les sessions actives|`Get-PSSession`|
|`Remove-PSSession`|Ferme une session|`Remove-PSSession -Session $s`|
|`Disconnect-PSSession`|Déconnecte sans fermer|`Disconnect-PSSession -Session $s`|
|`Connect-PSSession`|Reconnecte à une session|`Connect-PSSession -Session $s`|

### Exécution à Distance

|Commande|Description|Exemple|
|---|---|---|
|`Invoke-Command`|Exécute une commande à distance|`Invoke-Command -ComputerName Server01 -ScriptBlock {Get-Process}`|
|`Enable-PSRemoting`|Active PowerShell à distance|`Enable-PSRemoting -Force`|
|`Disable-PSRemoting`|Désactive PowerShell à distance|`Disable-PSRemoting -Force`|
|`Test-WSMan`|Teste la connectivité WinRM|`Test-WSMan -ComputerName Server01`|

**Exemples** :

```powershell
# Exécuter sur plusieurs machines
Invoke-Command -ComputerName Server01, Server02 -ScriptBlock {Get-Service}

# Utiliser une session
$s = New-PSSession -ComputerName Server01
Invoke-Command -Session $s -ScriptBlock {Get-Process}

# Copier des fichiers
Copy-Item "local.txt" -Destination "C:\Remote\" -ToSession $s
```

---

## 10. Jobs (Exécution Parallèle)

|Commande|Description|Exemple|
|---|---|---|
|`Start-Job`|Démarre un job en arrière-plan|`Start-Job -ScriptBlock {Get-Process}`|
|`Get-Job`|Liste les jobs|`Get-Job`|
|`Receive-Job`|Récupère le résultat d'un job|`Receive-Job -Id 1`|
|`Wait-Job`|Attend la fin d'un job|`Wait-Job -Id 1`|
|`Stop-Job`|Arrête un job|`Stop-Job -Id 1`|
|`Remove-Job`|Supprime un job|`Remove-Job -Id 1`|
|`Suspend-Job`|Suspend un job|`Suspend-Job -Id 1`|
|`Resume-Job`|Reprend un job|`Resume-Job -Id 1`|

**Exemple complet** :

```powershell
# Démarrer plusieurs jobs
$job1 = Start-Job -ScriptBlock {Start-Sleep 5; "Job1"}
$job2 = Start-Job -ScriptBlock {Start-Sleep 3; "Job2"}

# Attendre tous les jobs
Get-Job | Wait-Job

# Récupérer les résultats
Receive-Job $job1
Receive-Job $job2

# Nettoyer
Get-Job | Remove-Job
```

---

## 11. Modules et Fonctions

### Modules

|Commande|Description|Exemple|
|---|---|---|
|`Get-Module`|Liste les modules chargés|`Get-Module`|
|`Get-Module -ListAvailable`|Liste tous les modules|`Get-Module -ListAvailable`|
|`Import-Module`|Importe un module|`Import-Module ActiveDirectory`|
|`Remove-Module`|Décharge un module|`Remove-Module ActiveDirectory`|
|`Find-Module`|Recherche des modules en ligne|`Find-Module -Name "Az"`|
|`Install-Module`|Installe un module|`Install-Module -Name "Az"`|
|`Update-Module`|Met à jour un module|`Update-Module -Name "Az"`|
|`Uninstall-Module`|Désinstalle un module|`Uninstall-Module -Name "Az"`|

### Fonctions

|Commande|Description|Exemple|
|---|---|---|
|`Get-Command -CommandType Function`|Liste les fonctions|`Get-Command -CommandType Function`|
|`New-Item Function:`|Crée une fonction|`New-Item Function:Test -Value {"Hello"}`|
|`Remove-Item Function:`|Supprime une fonction|`Remove-Item Function:Test`|

---

## 12. Système et Administration

### Informations Système

|Commande|Description|Exemple|
|---|---|---|
|`Get-ComputerInfo`|Infos complètes du système|`Get-ComputerInfo`|
|`Get-WmiObject Win32_OperatingSystem`|Infos OS|`Get-WmiObject Win32_OperatingSystem`|
|`Get-WmiObject Win32_ComputerSystem`|Infos matériel|`Get-WmiObject Win32_ComputerSystem`|
|`Get-WmiObject Win32_BIOS`|Infos BIOS|`Get-WmiObject Win32_BIOS`|
|`Get-CimInstance`|Version moderne de WMI|`Get-CimInstance Win32_OperatingSystem`|
|`Get-HotFix`|Liste les mises à jour|`Get-HotFix`|
|`Get-EventLog`|Lit les journaux d'événements|`Get-EventLog -LogName System -Newest 10`|
|`Get-WinEvent`|Version moderne des logs|`Get-WinEvent -LogName System -MaxEvents 10`|

### Gestion de l'Ordinateur

|Commande|Description|Exemple|
|---|---|---|
|`Restart-Computer`|Redémarre l'ordinateur|`Restart-Computer -Force`|
|`Stop-Computer`|Éteint l'ordinateur|`Stop-Computer -Force`|
|`Rename-Computer`|Renomme l'ordinateur|`Rename-Computer -NewName "NewPC" -Restart`|
|`Add-Computer`|Joint un domaine|`Add-Computer -DomainName "contoso.com"`|
|`Remove-Computer`|Quitte un domaine|`Remove-Computer -WorkgroupName "WORKGROUP"`|

### Utilisateurs et Groupes (Local)

|Commande|Description|Exemple|
|---|---|---|
|`Get-LocalUser`|Liste les utilisateurs locaux|`Get-LocalUser`|
|`New-LocalUser`|Crée un utilisateur local|`New-LocalUser -Name "User1"`|
|`Remove-LocalUser`|Supprime un utilisateur local|`Remove-LocalUser -Name "User1"`|
|`Set-LocalUser`|Modifie un utilisateur local|`Set-LocalUser -Name "User1" -Description "Test"`|
|`Get-LocalGroup`|Liste les groupes locaux|`Get-LocalGroup`|
|`New-LocalGroup`|Crée un groupe local|`New-LocalGroup -Name "Group1"`|
|`Add-LocalGroupMember`|Ajoute à un groupe|`Add-LocalGroupMember -Group "Administrators" -Member "User1"`|
|`Remove-LocalGroupMember`|Retire d'un groupe|`Remove-LocalGroupMember -Group "Administrators" -Member "User1"`|

---

## 13. Texte et Manipulation de Chaînes

### Méthodes de String

```powershell
# Majuscules/Minuscules
"hello".ToUpper()           # HELLO
"HELLO".ToLower()           # hello

# Supprimer les espaces
"  hello  ".Trim()          # hello
"  hello  ".TrimStart()     # "hello  "
"  hello  ".TrimEnd()       # "  hello"

# Remplacer
"hello world".Replace("world", "PowerShell")  # hello PowerShell

# Diviser
"un,deux,trois".Split(",")  # Tableau: un, deux, trois

# Joindre
-join @("un", "deux")       # undeux
"un", "deux" -join ", "     # un, deux

# Longueur
"hello".Length              # 5

# Contient
"hello".Contains("ell")     # True
"hello".StartsWith("he")    # True
"hello".EndsWith("lo")      # True

# Sous-chaîne
"hello".Substring(1, 3)     # ell

# Index
"hello".IndexOf("l")        # 2
```

### Cmdlets de Texte

|Commande|Description|Exemple|
|---|---|---|
|`Select-String`|Recherche dans du texte (grep)|`Get-Content file.txt \| Select-String "motif"`|
|`Out-String`|Convertit en chaîne|`Get-Process \| Out-String`|
|`ConvertTo-String`|Convertit en chaîne|`$obj \| ConvertTo-String`|

---

## 14. Conversion et Formatage

### Conversion

|Commande|Description|Exemple|
|---|---|---|
|`ConvertTo-Json`|Convertit en JSON|`Get-Process \| Select Name \| ConvertTo-Json`|
|`ConvertFrom-Json`|Convertit depuis JSON|`$json \| ConvertFrom-Json`|
|`ConvertTo-Xml`|Convertit en XML|`Get-Process \| ConvertTo-Xml`|
|`ConvertTo-Html`|Convertit en HTML|`Get-Service \| ConvertTo-Html`|
|`ConvertTo-Csv`|Convertit en CSV|`Get-Process \| ConvertTo-Csv`|
|`ConvertFrom-Csv`|Convertit depuis CSV|`Import-Csv file.csv`|

### Formatage

|Commande|Alias|Description|Exemple|
|---|---|---|---|
|`Format-Table`|`ft`|Formate en tableau|`Get-Process \| Format-Table Name, CPU`|
|`Format-List`|`fl`|Formate en liste|`Get-Process \| Format-List *`|
|`Format-Wide`|`fw`|Formate en colonnes larges|`Get-Process \| Format-Wide Name`|
|`Format-Custom`|`fc`|Formatage personnalisé|`Get-Process \| Format-Custom`|

**Exemples** :

```powershell
# Tableau avec colonnes spécifiques
Get-Process | Format-Table Name, CPU, WorkingSet -AutoSize

# Liste complète
Get-Service | Format-List *

# JSON avec indentation
Get-Process | Select Name, CPU | ConvertTo-Json -Depth 2
```

---

## 15. Gestion des Erreurs

### Commandes

|Commande|Description|Exemple|
|---|---|---|
|`Try/Catch/Finally`|Gestion structurée des erreurs|Voir exemple ci-dessous|
|`Throw`|Lance une erreur|`Throw "Erreur personnalisée"`|
|`$Error`|Contient les erreurs récentes|`$Error[0]`|
|`$?`|Statut de la dernière commande|`if ($?) { "Succès" }`|
|`Get-Error`|Détails de la dernière erreur|`Get-Error`|

**Exemple Try/Catch** :

```powershell
Try {
    Get-Item "C:\FichierInexistant.txt" -ErrorAction Stop
}
Catch {
    Write-Host "Erreur : $_" -ForegroundColor Red
}
Finally {
    Write-Host "Nettoyage..."
}
```

**ErrorAction** :

```powershell
# Stop : Arrête l'exécution
# Continue : Continue malgré l'erreur (défaut)
# SilentlyContinue : Ignore l'erreur silencieusement
# Inquire : Demande à l'utilisateur

Get-Item "inexistant.txt" -ErrorAction SilentlyContinue
```

---

## 16. Sécurité et Permissions

### Politique d'Exécution

|Commande|Description|Exemple|
|---|---|---|
|`Get-ExecutionPolicy`|Affiche la politique actuelle|`Get-ExecutionPolicy`|
|`Set-ExecutionPolicy`|Modifie la politique|`Set-ExecutionPolicy RemoteSigned`|

**Politiques** :

- `Restricted` : Aucun script
- `AllSigned` : Scripts signés uniquement
- `RemoteSigned` : Scripts locaux + signés pour distants
- `Unrestricted` : Tous les scripts
- `Bypass` : Aucune restriction

### Permissions (ACL)

|Commande|Description|Exemple|
|---|---|---|
|`Get-Acl`|Obtient les permissions|`Get-Acl C:\Dossier`|
|`Set-Acl`|Définit les permissions|`Set-Acl C:\Dossier -AclObject $acl`|
|`Get-AuthenticodeSignature`|Vérifie la signature|`Get-AuthenticodeSignature script.ps1`|
|`Set-AuthenticodeSignature`|Signe un script|`Set-AuthenticodeSignature script.ps1 -Certificate $cert`|

---

## 17. Active Directory

**Note** : Nécessite le module ActiveDirectory

|Commande|Description|Exemple|
|---|---|---|
|`Get-ADUser`|Obtient un utilisateur AD|`Get-ADUser -Identity "jdoe"`|
|`New-ADUser`|Crée un utilisateur AD|`New-ADUser -Name "John Doe"`|
|`Set-ADUser`|Modifie un utilisateur AD|`Set-ADUser -Identity "jdoe" -Title "Manager"`|
|`Remove-ADUser`|Supprime un utilisateur AD|`Remove-ADUser -Identity "jdoe"`|
|`Get-ADGroup`|Obtient un groupe AD|`Get-ADGroup -Identity "Admins"`|
|`New-ADGroup`|Crée un groupe AD|`New-ADGroup -Name "NewGroup" -GroupScope Global`|
|`Add-ADGroupMember`|Ajoute à un groupe AD|`Add-ADGroupMember -Identity "Admins" -Members "jdoe"`|
|`Get-ADComputer`|Obtient un ordinateur AD|`Get-ADComputer -Identity "PC01"`|
|`Get-ADDomain`|Infos sur le domaine|`Get-ADDomain`|
|`Get-ADForest`|Infos sur la forêt|`Get-ADForest`|

---

## 18. Registre Windows

|Commande|Description|Exemple|
|---|---|---|
|`Get-ItemProperty`|Lit une valeur de registre|`Get-ItemProperty -Path "HKLM:\Software\Microsoft"`|
|`Set-ItemProperty|||