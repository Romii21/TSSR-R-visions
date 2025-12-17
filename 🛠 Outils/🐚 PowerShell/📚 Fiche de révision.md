# Synthèse PowerShell - Guide Complet

## Sommaire

1. [Introduction à PowerShell](https://claude.ai/chat/3376a5f6-1fc8-4873-a003-b85a225b6bcf#1-introduction-%C3%A0-powershell)
2. [Les Bases du Langage](https://claude.ai/chat/3376a5f6-1fc8-4873-a003-b85a225b6bcf#2-les-bases-du-langage)
    - [Syntaxe et Conventions](https://claude.ai/chat/3376a5f6-1fc8-4873-a003-b85a225b6bcf#syntaxe-et-conventions)
    - [Les Cmdlets](https://claude.ai/chat/3376a5f6-1fc8-4873-a003-b85a225b6bcf#les-cmdlets)
    - [Quotes et Caractères d'Échappement](https://claude.ai/chat/3376a5f6-1fc8-4873-a003-b85a225b6bcf#quotes-et-caract%C3%A8res-d%C3%A9chappement)
3. [Les Variables](https://claude.ai/chat/3376a5f6-1fc8-4873-a003-b85a225b6bcf#3-les-variables)
    - [Déclaration et Utilisation](https://claude.ai/chat/3376a5f6-1fc8-4873-a003-b85a225b6bcf#d%C3%A9claration-et-utilisation)
    - [Types de Données](https://claude.ai/chat/3376a5f6-1fc8-4873-a003-b85a225b6bcf#types-de-donn%C3%A9es)
    - [Portée des Variables](https://claude.ai/chat/3376a5f6-1fc8-4873-a003-b85a225b6bcf#port%C3%A9e-des-variables)
    - [Variables Spéciales](https://claude.ai/chat/3376a5f6-1fc8-4873-a003-b85a225b6bcf#variables-sp%C3%A9ciales)
4. [Wildcards et Expressions Régulières](https://claude.ai/chat/3376a5f6-1fc8-4873-a003-b85a225b6bcf#4-wildcards-et-expressions-r%C3%A9guli%C3%A8res)
5. [Structures Conditionnelles](https://claude.ai/chat/3376a5f6-1fc8-4873-a003-b85a225b6bcf#5-structures-conditionnelles)
    - [Opérateurs de Comparaison](https://claude.ai/chat/3376a5f6-1fc8-4873-a003-b85a225b6bcf#op%C3%A9rateurs-de-comparaison)
    - [If/Else et Switch](https://claude.ai/chat/3376a5f6-1fc8-4873-a003-b85a225b6bcf#ifelse-et-switch)
6. [Structures Itératives (Boucles)](https://claude.ai/chat/3376a5f6-1fc8-4873-a003-b85a225b6bcf#6-structures-it%C3%A9ratives-boucles)
7. [Les Tableaux](https://claude.ai/chat/3376a5f6-1fc8-4873-a003-b85a225b6bcf#7-les-tableaux)
8. [Les Fonctions](https://claude.ai/chat/3376a5f6-1fc8-4873-a003-b85a225b6bcf#8-les-fonctions)
9. [PowerShell à Distance](https://claude.ai/chat/3376a5f6-1fc8-4873-a003-b85a225b6bcf#9-powershell-%C3%A0-distance)
10. [Exécution Parallèle](https://claude.ai/chat/3376a5f6-1fc8-4873-a003-b85a225b6bcf#10-ex%C3%A9cution-parall%C3%A8le)

---

## 1. Introduction à PowerShell

### Qu'est-ce que PowerShell ?

PowerShell est un **interpréteur de commandes** (shell) et un **langage de script** développé par Microsoft pour :

- **Automatiser des tâches répétitives**
- **Gérer la configuration** de systèmes Microsoft
- **Gagner du temps** en évitant de répéter les mêmes commandes

### Particularités de PowerShell

**Scripts PowerShell** :

- Extension : `.PS1`
- Exécutés avec `PowerShell.exe`
- Fichiers texte contenant des commandes

**Différence majeure** : PowerShell manipule des **objets .NET** et non du simple texte comme d'autres shells.

**Framework .NET** :

- Environnement pour construire et exécuter des applications
- Les données sont des **objets** (instances de classes)
- Un objet combine **propriétés** (données) et **méthodes** (actions)

### Premier Script

```powershell
Write-Host "Hello World !"
```

**Bonnes pratiques** :

- Nommer les scripts avec l'extension `.PS1`
- Utiliser `#` pour les commentaires
- Choisir des noms de variables clairs

---

## 2. Les Bases du Langage

### Syntaxe et Conventions

**PowerShell n'est PAS sensible** :

- À la **casse** (majuscules/minuscules)
- Aux **espaces** et **tabulations**

```powershell
wRite-ouTput "Ceci fonctionne"
```

### Les Cmdlets

**Format standard** : `<Verbe>-<Nom>`

Exemples :

- `Get-ChildItem` : Liste les fichiers/dossiers
- `Set-Item` : Modifie un élément
- `Where-Object` : Filtre des objets

**Alias prédéfinis** :

- PowerShell intègre des alias de commandes Linux (`ls`, `cp`) et Batch (`cd`, `dir`)
- Ces alias pointent vers des cmdlets PowerShell natifs

### Quotes et Caractères d'Échappement

**Single quotes** (`'`) :

- Texte littéral, aucune interprétation
- `'$i'` affiche littéralement `$i`

**Double quotes** (`"`) :

- Interprète les variables et expressions
- `"$i"` affiche la valeur de `$i`

**Caractère d'échappement** : `` ` `` (backtick)

- `` `n `` : Nouvelle ligne
- `` `t `` : Tabulation
- `` `$ `` : Affiche le symbole `$` littéralement

```powershell
Write-Output "`nSaut de ligne`nTabulation `tici"
```

---

## 3. Les Variables

### Déclaration et Utilisation

**Syntaxe** : `$NomVariable = valeur`

```powershell
$Var = "Bonjour"
Write-Host $Var  # Affiche : Bonjour

$Var1 = $Var
$Var = "Hello"
Write-Host "$Var1 et $Var"  # Affiche : Bonjour et Hello
```

**Supprimer une variable** :

```powershell
Remove-Variable Var
```

**Règles de nommage** :

- Commence par `$`
- Lettres, chiffres, caractères spéciaux autorisés
- Non sensible à la casse
- Convention : **PascalCase**

### Types de Données

**Types principaux** :

- `[int]` : Entier
- `[string]` : Chaîne de caractères
- `[bool]` : Booléen

**Connaître le type** :

```powershell
"Texte".GetType()  # Retourne : String
```

**Transtypage (casting)** :

```powershell
[int]$Nombre = "10"
$Nombre + 5  # Résultat : 15
```

### Portée des Variables

|Portée|Description|
|---|---|
|**Global**|Disponible partout (console, scripts)|
|**Script**|Limité au script en cours|
|**Local**|Limité au bloc actuel|
|**Private**|Non visible en dehors de sa portée|

**Modificateurs** :

```powershell
$global:Variable = "Valeur globale"
$script:Variable = "Valeur script"
```

### Variables Spéciales

|Variable|Description|
|---|---|
|`$?`|État de la dernière commande (True/False)|
|`$_`|Objet actuel dans un pipeline|
|`$args`|Paramètres passés à une fonction/script|
|`$null`|Valeur vide|
|`$true` / `$false`|Booléens|

**Variables d'environnement** : `$env:NomVariable`

```powershell
$env:PROCESSOR_ARCHITECTURE  # Affiche l'architecture CPU
```

---

## 4. Wildcards et Expressions Régulières

### Wildcards (Caractères Génériques)

- `*` : Remplace zéro ou plusieurs caractères
- `?` : Remplace exactement un caractère

```powershell
Get-ChildItem C:\*.txt      # Tous les fichiers .txt
Get-ChildItem C:\file?.txt  # file1.txt, file2.txt, etc.
```

### Expressions Régulières (Regex)

Utilisées avec les opérateurs `-match` et `-replace`

```powershell
"123" -match "\d"              # True (\d = chiffre)
"Hello" -match "^H"            # True (^ = début)
"Hello boy" -replace "boy", "everybody"  # Hello everybody
```

---

## 5. Structures Conditionnelles

### Opérateurs de Comparaison

**Comparaison** :

- `-eq` : Égal à
- `-ne` : Différent de
- `-gt` : Plus grand que
- `-lt` : Plus petit que
- `-ge` : Plus grand ou égal
- `-le` : Plus petit ou égal

**Correspondance** :

- `-like` / `-notLike` : Avec wildcards
- `-match` / `-notMatch` : Avec regex

**Logiques** :

- `-and` : ET logique
- `-or` : OU logique
- `-xor` : OU exclusif
- `-not` ou `!` : NON logique

```powershell
$trois = 3
$trois -eq 3              # True
2 -lt $trois -and $trois -lt 4  # True
```

### If/Else et Switch

**Structure If** :

```powershell
If (condition) {
    # Instructions
} Else {
    # Instructions alternatives
}
```

**Structure Switch** :

```powershell
$Valeur = 5
Switch ($Valeur) {
    1 { Write-Host "Un" }
    5 { Write-Host "Cinq" }
    default { Write-Host "Autre" }
}
```

**⚠️ Important** : Les accolades `{ }` remplacent `then`, `elif`, etc. du Bash

---

## 6. Structures Itératives (Boucles)

### Opérateurs Unaires

```powershell
$i++  # Incrémente après utilisation
++$i  # Incrémente avant utilisation
$i--  # Décrémente
```

### Boucle For

```powershell
For ($i = 0; $i -le 10; $i++) {
    Write-Host "Valeur : $i"
}
```

### Boucle Foreach

**Syntaxe classique** :

```powershell
$Services = Get-Service
Foreach ($Service in $Services) {
    Write-Host "$($Service.Name) --> $($Service.Status)"
}
```

**Pipeline** :

```powershell
Get-Service | ForEach { Write-Host "$($_.Name) --> $($_.Status)" }
# Alias : % au lieu de ForEach
```

### Boucle While

```powershell
$Count = 0
While ($Count -le 10) {
    Write-Host "Compteur : $Count"
    $Count++
}
```

### Boucle Do While / Do Until

**Do While** : Condition à la fin (au moins 1 passage)

```powershell
Do {
    # Instructions
} While (condition)
```

**Do Until** : Jusqu'à ce que la condition soit vraie

```powershell
Do {
    # Instructions
} Until (condition)
```

---

## 7. Les Tableaux

### Initialisation

```powershell
$Tab = @()                    # Tableau vide
$Tab = @("Lundi", "Mardi")   # Avec valeurs
$Tab = 1..5                  # Tableau 1 à 5
```

### Accès aux Éléments

```powershell
$Tab[0]        # Premier élément
$Tab[-1]       # Dernier élément
$Tab[1,3,5]    # Éléments multiples
$Tab.Count     # Nombre d'éléments
```

### Parcourir un Tableau

```powershell
Foreach ($Element in $Tab) {
    Write-Host $Element
}
```

### Hashtables (Tableaux Associatifs)

**Clé/Valeur** :

```powershell
$Hash = @{
    1 = "Lundi"
    2 = "Mardi"
    3 = "Mercredi"
}

$Hash[2]           # "Mardi"
$Hash.Keys         # Liste des clés
$Hash.Values       # Liste des valeurs
$Hash.Add(4, "Jeudi")
```

---

## 8. Les Fonctions

### Déclaration Basique

```powershell
function Hello {
    Write-Host "Bonjour !"
}

Hello  # Appel de la fonction
```

**⚠️ Important** : Déclarer la fonction **avant** de l'appeler

### Fonctions avec Paramètres

```powershell
function Greet {
    param ([string]$Name)
    Write-Host "Bonjour $Name !"
}

Greet -Name "Alice"
```

### Fonctions Avancées

**Validation des paramètres** :

```powershell
function Conversion {
    param (
        [Parameter(Mandatory=$True)]
        [ValidateRange(0, 255)]
        [int]$Number,
        
        [ValidateSet('Binaire', 'Octal')]
        [string]$Type
    )
    
    Switch ($Type) {
        'Binaire' { [convert]::ToString($Number, 2) }
        'Octal' { [convert]::ToString($Number, 8) }
    }
}
```

---

## 9. PowerShell à Distance

### Cmdlets avec ComputerName

Certaines cmdlets acceptent le paramètre `-ComputerName` :

```powershell
Test-Connection -ComputerName client1
Restart-Computer -ComputerName serveur2
```

### Session Interactive

```powershell
Enter-PSSession -ComputerName client1
# Vous êtes maintenant sur client1
Exit-PSSession
```

**Prérequis** : Service WinRM démarré sur la machine distante

### Exécution de Commandes

```powershell
Invoke-Command -ComputerName client1 -ScriptBlock {
    Get-ChildItem C:\
}
```

---

## 10. Exécution Parallèle

### Les Jobs

**Commandes principales** :

- `Start-Job` : Démarre un job
- `Get-Job` : Liste les jobs
- `Wait-Job` : Attend la fin d'un job
- `Receive-Job` : Récupère le résultat
- `Remove-Job` : Supprime un job

**Exemple** :

```powershell
$Job1 = Start-Job -ScriptBlock {
    Start-Sleep -Seconds 10
    Write-Host "Job terminé"
}

Wait-Job $Job1
Receive-Job $Job1
Remove-Job $Job1
```

### Avantages de l'Exécution Parallèle

- **Jobs** : Tâches asynchrones longues
- **Runspaces** : Contrôle fin des threads
- **Workflows** : Processus complexes
- **Parallel.ForEach** : Traitement de collections en parallèle

---

## Schéma Récapitulatif : Flux de Travail PowerShell

```
┌─────────────────────────────────────────────────────┐
│              SCRIPT POWERSHELL (.PS1)                │
└─────────────────────────────────────────────────────┘
                         │
                         ▼
        ┌────────────────────────────────┐
        │   PowerShell.exe (Console)     │
        └────────────────────────────────┘
                         │
        ┌────────────────┴────────────────┐
        │                                  │
        ▼                                  ▼
┌──────────────┐                  ┌──────────────┐
│  Variables   │                  │   Cmdlets    │
│  Fonctions   │                  │   Alias      │
└──────────────┘                  └──────────────┘
        │                                  │
        └────────────────┬────────────────┘
                         ▼
            ┌────────────────────────┐
            │    Objets .NET         │
            │  (Propriétés/Méthodes) │
            └────────────────────────┘
                         │
                         ▼
            ┌────────────────────────┐
            │   Résultat / Action    │
            └────────────────────────┘
```

---

## Points Clés à Retenir

✅ **PowerShell manipule des objets .NET**, pas du texte simple  
✅ **Les variables commencent par `$`** et ont une portée définie  
✅ **Les cmdlets suivent la convention `Verbe-Nom`**  
✅ **Les accolades `{ }` délimitent les blocs** (if, for, etc.)  
✅ **PowerShell intègre des alias** pour faciliter la transition depuis d'autres shells  
✅ **L'exécution à distance nécessite WinRM**  
✅ **Les jobs permettent l'exécution parallèle** de tâches

---

## Sources

- Documents de cours fournis : _Scripting PowerShell - Parties 1, 2 et 3_
- Notes personnelles de l'étudiant
- Documentation officielle Microsoft PowerShell : https://docs.microsoft.com/powershell/

---

_Synthèse réalisée pour faciliter l'apprentissage et la révision de PowerShell_