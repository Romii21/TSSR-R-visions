# Questionnaire – CP6 : Automatiser des tâches à l'aide de scripts

**Formation TSSR | Scripting PowerShell** **15 questions – Réponses ouvertes**

---

## Question 1

PowerShell est décrit comme manipulant des **objets .NET** plutôt que du simple texte. Expliquez concrètement ce que cela signifie. Quelle est la différence fondamentale avec un shell comme Bash, et quel avantage cela apporte-t-il dans un contexte d'administration système ?

---

## Question 2

Quelle est la convention de nommage des **cmdlets** PowerShell ? Donnez trois exemples de cmdlets en respectant cette convention, et précisez ce que fait chacune d'elles.

---

## Question 3

Expliquez la différence de comportement entre les **guillemets simples** `' '` et les **guillemets doubles** `" "` en PowerShell. Quel est le caractère d'échappement utilisé, et comment affiche-t-on un `$` littéralement dans une chaîne en double quotes ?

---

## Question 4

Analysez le code suivant et expliquez ce qui sera affiché :

```powershell
$Prenom = "Alice"
$Age = 30

Write-Host 'Utilisateur : $Prenom, Age : $Age'
Write-Host "Utilisateur : $Prenom, Age : $Age"
```

---

## Question 5

Citez et expliquez les quatre **niveaux de portée (scope)** des variables en PowerShell. Comment déclare-t-on explicitement une variable **globale** depuis l'intérieur d'une fonction ?

---

## Question 6

Donnez la liste des **opérateurs de comparaison** PowerShell équivalents aux symboles suivants, et précisez une particularité importante liée à la casse :

|Symbole mathématique|Opérateur PowerShell|
|---|---|
|`=`|?|
|`≠`|?|
|`>`|?|
|`<`|?|
|`≥`|?|
|`≤`|?|

---

## Question 7

Analysez le script suivant et décrivez précisément ce qu'il affiche :

```powershell
$Score = 75

If ($Score -ge 90) {
    Write-Host "Mention Très Bien"
} ElseIf ($Score -ge 70) {
    Write-Host "Mention Bien"
} ElseIf ($Score -ge 50) {
    Write-Host "Mention Passable"
} Else {
    Write-Host "Échec"
}
```

Quelle structure aurait-on pu utiliser à la place du `If/ElseIf` pour traiter plusieurs cas fixes ?

---

## Question 8

Quelle est la différence entre une boucle `For`, une boucle `Foreach` et une boucle `While` en PowerShell ? Pour chacune, décrivez un cas d'usage typique en administration système.

---

## Question 9

Observez le script suivant et expliquez son fonctionnement ligne par ligne, en précisant ce qu'affiche la boucle :

```powershell
$Services = Get-Service
Foreach ($Service in $Services) {
    If ($Service.Status -eq "Running") {
        Write-Host "$($Service.Name) est actif" -ForegroundColor Green
    }
}
```

Qu'est-ce que `$_` représente dans un pipeline PowerShell et comment aurait-on pu réécrire cette boucle avec un pipeline ?

---

## Question 10

Expliquez ce qu'est un **tableau (array)** en PowerShell. Comment initialise-t-on un tableau vide ? Comment accède-t-on au **premier** et au **dernier** élément ? Comment récupère-t-on le **nombre d'éléments** ?

---

## Question 11

Quelle est la différence entre un **tableau classique** (`@()`) et une **hashtable** (`@{}`) en PowerShell ? Donnez un exemple d'utilisation de chacun dans un contexte d'administration.

---

## Question 12

Analysez la fonction suivante :

```powershell
function Convertir {
    param (
        [Parameter(Mandatory=$True)]
        [ValidateRange(0, 255)]
        [int]$Nombre,

        [ValidateSet('Binaire', 'Hexadecimal')]
        [string]$Type
    )

    Switch ($Type) {
        'Binaire'     { [convert]::ToString($Nombre, 2) }
        'Hexadecimal' { [convert]::ToString($Nombre, 16) }
    }
}
```

- Que fait `[Parameter(Mandatory=$True)]` ?
- Que fait `[ValidateRange(0, 255)]` ?
- Que fait `[ValidateSet('Binaire', 'Hexadecimal')]` ?
- Comment appelle-t-on cette fonction pour convertir 42 en binaire ?

---

## Question 13

Expliquez le mécanisme **Try / Catch / Finally** en PowerShell. Dans quel cas est-il indispensable ? Écrivez un exemple simple qui tente de lire un fichier inexistant et affiche un message d'erreur personnalisé si le fichier n'est pas trouvé.

---

## Question 14

Qu'est-ce que **PowerShell Remoting** ? Citez les trois méthodes principales pour exécuter des commandes sur une machine distante, et indiquez le prérequis indispensable côté machine cible pour que cela fonctionne.

---

## Question 15

Analysez le script ci-dessous et identifiez **au moins trois problèmes ou mauvaises pratiques**. Pour chacun, proposez la correction appropriée.

```powershell
$users = get-aduser -filter *
foreach($u in $users){
write-host $u.name
$global:count = $global:count + 1
}
write-host "Total : " + $global:count
```

---

_Bonne chance ! Transmettez vos réponses pour correction._