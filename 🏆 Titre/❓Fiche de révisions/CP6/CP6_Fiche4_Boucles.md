# CP6 — Fiche de Révision 4 : Boucles et Itération

> **Bash & PowerShell | TSSR**

---

## BASH — Les Boucles

### Boucle FOR sur liste

Parcourir une liste de valeurs connues à l'avance :

```bash
for variable in liste
do
    instructions
done
```

**Exemples :**

```bash
# Liste de valeurs
for fruit in "Pomme" "Banane" "Orange"
do
    echo "Je mange une $fruit"
done

# Avec seq (générer une séquence de nombres)
for numero in $(seq 1 10)
do
    echo "Nombre : $numero"
done

# Parcourir les fichiers d'un dossier
for fichier in /etc/*.conf
do
    echo "Fichier : $fichier"
done
```

### Boucle FOR arithmétique (style C)

Pour les compteurs avec incrémentation :

```bash
for (( i=1 ; i <= 10 ; i++ ))
do
    echo "Itération $i"
done
```

**Table de multiplication par 3 :**

```bash
for (( i=1 ; i <= 10 ; i++ ))
do
    echo "3 x $i = $(( 3 * i ))"
done
```

### Boucle WHILE

Tant qu'une condition est vraie, exécuter les instructions :

```bash
while [ condition ]
do
    instructions
done
```

**Exemple — Compte à rebours :**

```bash
#!/bin/bash
nombre=10

while [ $nombre -ge 0 ]
do
    echo $nombre
    nombre=$(( nombre - 1 ))
done

echo "Décollage !"
```

> **Quand utiliser `for` vs `while` ?**
> - `for` → quand on connaît la liste ou le nombre d'itérations à l'avance
> - `while` → quand on itère jusqu'à ce qu'une condition change

### Boucle UNTIL

L'inverse du `while` : exécute **jusqu'à ce que** la condition devienne vraie :

```bash
compteur=0
until [ $compteur -ge 5 ]
do
    echo "Compteur : $compteur"
    compteur=$(( compteur + 1 ))
done
```

### Commandes de contrôle

| Commande | Effet |
|---|---|
| `break` | Quitte la boucle immédiatement |
| `continue` | Passe à l'itération suivante |

---

## POWERSHELL — Les Boucles

### Boucle For

```powershell
For ($i = 1; $i -le 10; $i++) {
    Write-Host "Valeur : $i"
}
```

### Boucle Foreach (syntaxe classique)

```powershell
$Services = Get-Service
Foreach ($Service in $Services) {
    Write-Host "$($Service.Name) --> $($Service.Status)"
}
```

### Boucle Foreach en pipeline (ForEach-Object)

```powershell
# $_ représente l'objet courant dans le pipeline
Get-Service | ForEach-Object {
    Write-Host "$($_.Name) --> $($_.Status)"
}

# Alias % (plus court)
Get-Service | % { Write-Host "$($_.Name) --> $($_.Status)" }
```

**Comparaison des deux syntaxes Foreach :**

| Syntaxe                       | Quand l'utiliser                  |
| ----------------------------- | --------------------------------- |
| `Foreach ($x in $collection)` | Collection déjà dans une variable |
| `cmd \| ForEach-Object { }`   | Chaînage direct depuis une cmdlet |

### Boucle While

```powershell
$Count = 0
While ($Count -le 10) {
    Write-Host "Compteur : $Count"
    $Count++
}
```

### Boucle Do While / Do Until

Ces boucles garantissent **au moins une exécution** avant de tester la condition :

```powershell
# Do While : continue TANT QUE la condition est vraie
$i = 0
Do {
    Write-Host "i = $i"
    $i++
} While ($i -lt 5)

# Do Until : continue JUSQU'À ce que la condition soit vraie
$i = 0
Do {
    Write-Host "i = $i"
    $i++
} Until ($i -ge 5)
```

> **While vs Do While :**
> - `While` : condition testée **avant** la 1ère exécution (peut ne jamais s'exécuter)
> - `Do While` : condition testée **après** (s'exécute au moins une fois)

---

## Comparatif Bash / PowerShell

| Boucle | Bash | PowerShell |
|---|---|---|
| For compteur | `for (( i=0; i<10; i++ ))` | `For ($i=0; $i -lt 10; $i++)` |
| For liste | `for x in liste` | `Foreach ($x in $collection)` |
| Pipeline | `for x in $(cmd)` | `cmd \| ForEach-Object { }` |
| While | `while [ cond ]` | `While (cond)` |
| Until | `until [ cond ]` | `Do { } Until (cond)` |
| Quitter | `break` | `break` |
| Continuer | `continue` | `continue` |

---

## Points Clés à Retenir ✅

- `for` Bash : utiliser `$(seq 1 10)` pour une plage de nombres
- Le `for` arithmétique Bash `(( ))` s'écrit comme en langage C
- `while` = **tant que** ; `until` = **jusqu'à ce que** (logiques opposées)
- `Do While` / `Do Until` s'exécutent **au minimum une fois**
- En PowerShell, `$_` représente l'objet courant dans un `ForEach-Object`
- `%` est l'alias de `ForEach-Object` dans un pipeline PowerShell
