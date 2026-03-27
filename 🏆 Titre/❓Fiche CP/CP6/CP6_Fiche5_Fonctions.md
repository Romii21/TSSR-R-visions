# CP6 — Fiche de Révision 5 : Fonctions

> **Bash & PowerShell | TSSR**

---

## BASH — Les Fonctions

### Déclaration

Deux syntaxes équivalentes :

```bash
# Syntaxe 1 (avec le mot-clé function)
function nom_fonction() {
    instructions
}

# Syntaxe 2 (sans le mot-clé function)
nom_fonction() {
    instructions
}
```

> ⚠️ **Contrainte** : Une fonction doit être **déclarée avant** d'être appelée dans le script.

```bash
#!/bin/bash

# ✅ Déclaration en premier
function saluer() {
    echo "Bonjour !"
}

saluer    # Appel ensuite → OK

# ❌ Appel avant déclaration → Erreur
bonjour
function bonjour() { echo "Hello"; }
```

### Paramètres d'une fonction

Les fonctions reçoivent leurs paramètres comme un script : `$1`, `$2`...

```bash
function greet() {
    if [ $# -gt 0 ]
    then
        echo "Bonjour $1 !"
    else
        echo "Bonjour inconnu !"
    fi
}

greet "Alice"   # Affiche : Bonjour Alice !
greet           # Affiche : Bonjour inconnu !
```

| Variable | Dans une fonction |
|---|---|
| `$1`, `$2`... | Arguments passés à la fonction |
| `$#` | Nombre d'arguments reçus |
| `$@` | Tous les arguments |

### Retour de valeur : `return` vs `echo`

**Option 1 : `return` — retourne un code numérique (0-255)**

```bash
verifier_age() {
    if [ $1 -ge 18 ]
    then
        return 0    # Succès / vrai
    else
        return 1    # Erreur / faux
    fi
}

verifier_age 25
if [ $? -eq 0 ]; then
    echo "Majeur"
fi
```

**Option 2 : `echo` — retourne du texte (plus courant)**

```bash
addition() {
    echo $(( $1 + $2 ))
}

resultat=$(addition 5 3)    # Capture la sortie
echo "Résultat : $resultat" # Affiche : 8
```

**Quand utiliser l'un ou l'autre ?**

| Méthode | Type retourné | Usage |
|---|---|---|
| `return` | Entier 0-255 | Succès/échec, vrai/faux |
| `echo` | Texte/nombre | Retourner une vraie valeur calculée |

### Portée des variables : LOCAL vs GLOBAL

**Par défaut, les variables sont GLOBALES** — une fonction peut modifier une variable du script :

```bash
#!/bin/bash

ma_fonction() {
    var="Modifiée dans la fonction"
}

var="Valeur initiale"
ma_fonction
echo "$var"     # Affiche : Modifiée dans la fonction ⚠️
```

**Utiliser `local` pour isoler la variable :**

```bash
#!/bin/bash

ma_fonction() {
    local var="Locale à la fonction"
    echo "Dans la fonction : $var"
}

var="Valeur globale"
ma_fonction
echo "Après appel : $var"   # Affiche : Valeur globale ✅
```

```
┌──────────────────────────────────┐
│  Script (scope global)           │
│  var = "valeur globale"          │
│ ┌──────────────────────────────┐ │
│ │  Fonction                    │ │
│ │  local var = "locale"        │ │
│ │  (reste dans la fonction)    │ │
│ └──────────────────────────────┘ │
└──────────────────────────────────┘
```

---

## POWERSHELL — Les Fonctions

### Déclaration basique

```powershell
function Saluer {
    Write-Host "Bonjour !"
}

Saluer    # Appel
```

> ⚠️ **Contrainte** : Déclarer la fonction **avant** de l'appeler.

### Fonctions avec paramètres (bloc param)

```powershell
function Greet {
    param ([string]$Name)
    Write-Host "Bonjour $Name !"
}

Greet -Name "Alice"    # Appel avec paramètre nommé
```

### Paramètre obligatoire

Pour forcer l'utilisateur à fournir un paramètre :

```powershell
function Get-InfoDisque {
    param (
        [Parameter(Mandatory=$True)]
        [string]$Disque
    )

    $Info = Get-PSDrive -Name $Disque.TrimEnd(':')
    Write-Host "Disque : $Disque"
    Write-Host "Espace libre  : $($Info.Free / 1GB) Go"
    Write-Host "Espace utilisé : $($Info.Used / 1GB) Go"
}

Get-InfoDisque -Disque "C:"
```

### Validation de paramètres

```powershell
function Conversion {
    param (
        [Parameter(Mandatory=$True)]
        [ValidateRange(0, 255)]       # Valeur entre 0 et 255
        [int]$Nombre,

        [ValidateSet('Binaire', 'Octal')]  # Valeur parmi une liste
        [string]$Type
    )

    Switch ($Type) {
        'Binaire' { [convert]::ToString($Nombre, 2) }
        'Octal'   { [convert]::ToString($Nombre, 8) }
    }
}
```

---

## Comparatif Bash / PowerShell

| Notion | Bash | PowerShell |
|---|---|---|
| Déclaration | `function nom() { }` | `function Nom { }` |
| Paramètres | `$1`, `$2`, `$#` | `param([type]$Nom)` |
| Param obligatoire | Vérification manuelle | `[Parameter(Mandatory=$True)]` |
| Retour valeur | `echo` ou `return` | `return` ou affichage |
| Variable locale | `local var="val"` | Portée `$local:` ou par défaut |
| Appel | `nom_fonction arg1` | `Nom-Fonction -Param val` |

---

## Points Clés à Retenir ✅

- En Bash, les fonctions doivent être déclarées **avant** d'être appelées
- `$1`, `$2`... dans une fonction Bash = arguments passés **à la fonction** (pas au script)
- `return` retourne un **code 0-255** ; `echo` retourne du **texte/valeur**
- Sans `local`, les variables Bash sont globales et peuvent modifier l'état du script
- En PowerShell, `param()` est le bloc de déclaration des paramètres
- `[Parameter(Mandatory=$True)]` rend un paramètre PowerShell obligatoire
- `[ValidateRange()]` et `[ValidateSet()]` permettent de valider les entrées