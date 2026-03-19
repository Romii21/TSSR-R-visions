# CP6 — Fiche de Révision 2 : Variables et Données

> **Bash & PowerShell | TSSR**

---

## BASH — Variables

### Déclaration et utilisation

```bash
prenom="Alice"          # ✅ Pas d'espace autour du =
echo $prenom            # Affiche : Alice
echo "$prenom"          # ✅ Recommandé (guillemets doubles)
unset prenom            # Supprimer la variable
```

> ⚠️ **Règle critique** : Aucun espace autour du `=` en Bash.
> `prenom = "Alice"` → **Erreur** (bash interprète `prenom` comme une commande)

### Variables prédéfinies

| Variable | Signification |
|---|---|
| `$0` | Nom du script |
| `$1`, `$2`... | Arguments passés au script |
| `$#` | Nombre d'arguments |
| `$@` | Tous les arguments (liste séparée) |
| `$*` | Tous les arguments (en un seul mot) |
| `$?` | Code de sortie de la dernière commande |
| `$$` | PID du processus courant |

```bash
# Lancement : ./script.sh Alice Bob Charlie
echo $0     # ./script.sh
echo $1     # Alice
echo $2     # Bob
echo $#     # 3
echo $@     # Alice Bob Charlie
```

### Substitution de commande

Exécuter une commande et stocker son résultat dans une variable :

```bash
date_now=$(date)                # ✅ Syntaxe recommandée
date_old=`date`                 # Ancienne syntaxe (déconseillée)

echo "Nous sommes le : $date_now"

uid=$(id -u)                    # UID de l'utilisateur courant
nom_machine=$(hostname)         # Nom de la machine
```

### Opérations arithmétiques

Bash ne gère pas nativement les nombres — il faut utiliser la syntaxe `$(( ))` :

```bash
result=$(( 5 + 3 ))             # 8
total=$(( 10 * 3 - 5 ))         # 25
i=$(( $i + 1 ))                 # Incrémenter

# Exemples complets
a=10
b=4
echo $(( a + b ))   # 14
echo $(( a * b ))   # 40
echo $(( a / b ))   # 2 (division entière)
echo $(( a % b ))   # 2 (modulo)
```

### Le mot-clé `export`

Par défaut, une variable n'est visible que dans le shell courant. Pour la rendre accessible aux scripts/processus fils, on utilise `export` :

```bash
# Sans export
variable="test"
bash -c 'echo $variable'        # Affiche rien

# Avec export
export variable="test"
bash -c 'echo $variable'        # Affiche : test
```

```
Shell parent
├── export variable="test"
│
└── Shell fils (script lancé)
    └── $variable accessible ✅
```

---

## POWERSHELL — Variables

### Déclaration et utilisation

```powershell
$Prenom = "Alice"               # Déclaration
Write-Host $Prenom              # Affiche : Alice
Write-Host "Bonjour $Prenom"    # Interprétation dans les doubles guillemets

Remove-Variable Prenom          # Supprimer la variable
```

**Différence guillemets PowerShell :**

```powershell
$Var = "Serveur01"
Write-Host "$Var"     # Affiche : Serveur01  (interprétation)
Write-Host '$Var'     # Affiche : $Var       (texte brut)
```

### Portée des variables

| Portée | Description |
|---|---|
| `$global:` | Disponible partout (console + scripts) |
| `$script:` | Limité au script en cours |
| `$local:` | Limité au bloc actuel (défaut) |
| `$private:` | Non visible en dehors de sa portée |

```powershell
$global:Variable = "Valeur globale"    # Accessible partout
$script:Variable = "Valeur script"     # Dans le script uniquement
```

### Variables spéciales PowerShell

| Variable | Description |
|---|---|
| `$?` | État de la dernière commande (`$true` / `$false`) |
| `$_` | Objet courant dans un pipeline |
| `$args` | Arguments passés à un script/fonction |
| `$null` | Valeur vide/nulle |
| `$true` / `$false` | Booléens |
| `$env:NomVar` | Variables d'environnement |

```powershell
$env:USERNAME                   # Nom de l'utilisateur Windows
$env:COMPUTERNAME               # Nom de la machine
$env:PATH                       # Variable PATH
```

### Tableaux (Arrays)

```powershell
$Tab = @()                          # Tableau vide
$Tab = @("Lundi", "Mardi", "Mercredi")  # Avec valeurs
$Tab = 1..5                         # Tableau de 1 à 5

$Tab[0]         # Premier élément → "Lundi"
$Tab[-1]        # Dernier élément  → "Mercredi"
$Tab.Count      # Nombre d'éléments → 3

# Parcourir
Foreach ($Jour in $Tab) {
    Write-Host $Jour
}
```

### Hashtables (Tableaux Associatifs)

Structure clé/valeur, équivalent d'un dictionnaire :

```powershell
$Hash = @{
    Nom    = "Alice"
    Age    = 25
    Ville  = "Paris"
}

$Hash["Nom"]        # Alice
$Hash.Age           # 25
$Hash.Keys          # Nom, Age, Ville
$Hash.Values        # Alice, 25, Paris

$Hash.Add("Email", "alice@exemple.com")    # Ajouter une entrée
```

---

## Comparatif Bash / PowerShell

| Notion | Bash | PowerShell |
|---|---|---|
| Déclaration | `var="valeur"` | `$Var = "valeur"` |
| Lecture | `$var` ou `"$var"` | `$Var` |
| Arithmétique | `$(( a + b ))` | `$a + $b` (natif) |
| Variables spéciales | `$0`, `$1`, `$?`... | `$_`, `$?`, `$args`... |
| Tableau | `tab=("a" "b")` | `$Tab = @("a","b")` |
| Tableau assoc. | `declare -A hash` | `$Hash = @{cle="val"}` |
| Environnement | `export VAR` | `$env:VAR` |

---

## Points Clés à Retenir ✅

- En Bash, **pas d'espace autour du = lors d'une déclaration**
- `$?` donne le code de sortie en Bash ; `True`/`False` en PowerShell
- `export` rend une variable Bash visible par les processus fils
- `$(commande)` capture la sortie d'une commande dans une variable
- `$(( ))` est obligatoire pour les calculs en Bash
- En PowerShell, `$_` représente l'objet courant dans un pipeline
- Les **hashtables** PowerShell (`@{}`) sont des structures clé/valeur
