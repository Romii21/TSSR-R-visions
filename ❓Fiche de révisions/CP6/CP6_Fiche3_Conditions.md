# CP6 — Fiche de Révision 3 : Structures Conditionnelles

> **Bash & PowerShell | TSSR**

---

## BASH — Conditions

### Logique inversée des codes de sortie

> ⚠️ **Particularité Bash** : Dans un test conditionnel, **0 = VRAI** et **non-zéro = FAUX**.
> C'est l'inverse des conventions habituelles en programmation (où 0 = faux).
>
> **Raison** : Le code `0` signifie "succès" en Bash. Un `if` vérifie si la commande a réussi.

```bash
if commande_qui_reussit; then
    echo "Succès = 0 = VRAI"
fi
```

### Structure if / elif / else

```bash
if [ condition ]
then
    # instructions
elif [ autre_condition ]
then
    # instructions
else
    # instructions par défaut
fi
```

**Exemple complet :**

```bash
#!/bin/bash
age=$1

if [ "$age" -lt 18 ]
then
    echo "Mineur"
elif [ "$age" -ge 65 ]
then
    echo "Senior"
else
    echo "Adulte"
fi
```

### Opérateurs de comparaison numérique

| Opérateur | Signification |
|---|---|
| `-eq` | Égal à (equal) |
| `-ne` | Différent de (not equal) |
| `-lt` | Inférieur à (less than) |
| `-le` | Inférieur ou égal (less or equal) |
| `-gt` | Supérieur à (greater than) |
| `-ge` | Supérieur ou égal (greater or equal) |

```bash
[ $a -eq $b ]    # Comparaison numérique
[ "$a" = "$b" ]  # Comparaison de chaînes
[ "$a" != "$b" ] # Différent (chaînes)
```

> **Différence importante** :
> - `-eq` → compare des **nombres** : `[ 10 -eq 10 ]`
> - `=` → compare des **chaînes de caractères** : `[ "alice" = "alice" ]`

### Opérateurs sur les fichiers

| Opérateur | Signification |
|---|---|
| `-f fichier` | C'est un fichier régulier |
| `-d dossier` | C'est un dossier |
| `-e chemin` | Le chemin existe |
| `-r fichier` | Le fichier est lisible |
| `-w fichier` | Le fichier est modifiable |
| `-x fichier` | Le fichier est exécutable |

```bash
if [ -f "/etc/passwd" ]
then
    echo "Le fichier existe"
fi

if [ -d "/home/alice" ]
then
    echo "Le répertoire existe"
fi
```

### Opérateurs logiques

| Opérateur | Signification | Exemple |
|---|---|---|
| `&&` | ET — exécute si la précédente réussit | `cmd1 && cmd2` |
| `\|\|` | OU — exécute si la précédente échoue | `cmd1 \|\| cmd2` |
| `;` | Séquence — exécute toujours | `cmd1 ; cmd2` |

```bash
# && : n'exécute cmd2 que si cmd1 réussit (code 0)
cd /backups && tar -czf backup.tar.gz /home

# || : exécute cmd2 uniquement si cmd1 échoue
cd /dossier 2>/dev/null || echo "Erreur : dossier introuvable"

# Combiné : pattern gestion d'erreur classique
cd /dossier 2>/dev/null && echo "Succès" || echo "Erreur"

# ; : exécute cmd2 dans tous les cas
echo "Début" ; echo "Fin"
```

### Structure case

Alternative au `if/elif` pour tester une valeur contre plusieurs cas :

```bash
case $variable in
    valeur1)
        instructions
        ;;
    valeur2)
        instructions
        ;;
    *)
        # Cas par défaut
        ;;
esac
```

**Exemple — Jours de la semaine :**

```bash
#!/bin/bash
jour=$1

case $jour in
    1) echo "Lundi" ;;
    2) echo "Mardi" ;;
    3) echo "Mercredi" ;;
    4) echo "Jeudi" ;;
    5) echo "Vendredi" ;;
    *) echo "Jour invalide" ;;
esac
```

---

## POWERSHELL — Conditions

### Structure If / ElseIf / Else

```powershell
If (condition) {
    # Instructions
} ElseIf (autre_condition) {
    # Instructions
} Else {
    # Instructions par défaut
}
```

**Exemple :**

```powershell
$Statut = "Running"

If ($Statut -eq "Running") {
    Write-Host "Service en cours d'exécution"
} ElseIf ($Statut -eq "Stopped") {
    Write-Host "Service arrêté"
} Else {
    Write-Host "État inconnu : $Statut"
}
```

### Opérateurs de comparaison PowerShell

| Opérateur PS | Équivalent | Signification |
|---|---|---|
| `-eq` | `==` | Égal à |
| `-ne` | `!=` | Différent de |
| `-lt` | `<` | Inférieur à |
| `-le` | `<=` | Inférieur ou égal |
| `-gt` | `>` | Supérieur à |
| `-ge` | `>=` | Supérieur ou égal |
| `-like` | — | Avec wildcards (`*`, `?`) |
| `-match` | — | Avec expressions régulières |

> ⚠️ **Pourquoi pas `<` et `>` directement ?**
> En PowerShell, `<` et `>` sont réservés pour la **redirection** de flux (comme en Bash).
> Il faut donc utiliser `-lt` et `-gt` pour les comparaisons.

### Opérateurs logiques PowerShell

| Opérateur | Signification |
|---|---|
| `-and` | ET logique |
| `-or` | OU logique |
| `-not` ou `!` | NON logique |
| `-xor` | OU exclusif |

```powershell
$Age = 25
If ($Age -ge 18 -and $Age -lt 65) {
    Write-Host "Adulte actif"
}
```

### Structure Switch (PowerShell)

```powershell
$Valeur = 3

Switch ($Valeur) {
    1 { Write-Host "Un" }
    2 { Write-Host "Deux" }
    3 { Write-Host "Trois" }
    default { Write-Host "Autre valeur" }
}
```

---

## Comparatif Bash / PowerShell

| Notion | Bash | PowerShell |
|---|---|---|
| Structure if | `if [ cond ]; then ... fi` | `If (cond) { ... }` |
| Égalité nombre | `-eq` | `-eq` |
| Égalité chaîne | `=` dans `[ ]` | `-eq` |
| Inférieur | `-lt` | `-lt` |
| ET logique | `&&` | `-and` |
| OU logique | `\|\|` | `-or` |
| Switch/Case | `case ... esac` | `Switch { ... }` |

---

## Points Clés à Retenir ✅

- En Bash, **0 = VRAI** dans un test (succès de commande)
- `-eq` compare des **nombres**, `=` compare des **chaînes** en Bash
- Toujours utiliser `"$variable"` dans les tests Bash pour éviter les erreurs sur les espaces
- PowerShell utilise `-lt`, `-gt`... car `<` et `>` sont des opérateurs de redirection
- `&&` en Bash = `-and` en PowerShell
- `||` en Bash = `-or` en PowerShell
- Le `case` Bash se termine par `esac` ; chaque cas par `;;`
