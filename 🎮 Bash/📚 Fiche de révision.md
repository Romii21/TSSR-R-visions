# Synthèse Complète des Scripts Bash

## 📋 Sommaire

1. [Fondations](https://claude.ai/chat/b92a8d2d-3af7-4e19-8252-378785e9b3a3#fondations)
2. [Variables](https://claude.ai/chat/b92a8d2d-3af7-4e19-8252-378785e9b3a3#variables)
3. [Structures Conditionnelles](https://claude.ai/chat/b92a8d2d-3af7-4e19-8252-378785e9b3a3#structures-conditionnelles)
4. [Structures Itératives](https://claude.ai/chat/b92a8d2d-3af7-4e19-8252-378785e9b3a3#structures-it%C3%A9ratives)
5. [Fonctions](https://claude.ai/chat/b92a8d2d-3af7-4e19-8252-378785e9b3a3#fonctions)
6. [Bonnes Pratiques](https://claude.ai/chat/b92a8d2d-3af7-4e19-8252-378785e9b3a3#bonnes-pratiques)
7. [Récapitulatif Visuel](https://claude.ai/chat/b92a8d2d-3af7-4e19-8252-378785e9b3a3#r%C3%A9capitulatif-visuel)

---

## Fondations

### Qu'est-ce qu'un script Bash ?

Un script Bash est un **fichier texte** contenant une suite de commandes interprétées ligne par ligne par le shell.

**Différence programme vs script :**

- **Programme** : indique au processeur ce qu'il doit faire
- **Script** : indique au système d'exploitation ou à une application ce qu'ils doivent faire

### Anatomie d'un script minimal

bash

```bash
#!/bin/bash
# Ceci est mon premier script
echo "Hello World !"
exit 0
```

#### Les éléments essentiels :

|Élément|Rôle|
|---|---|
|`#!/bin/bash`|**Shebang** - indique l'interpréteur à utiliser|
|`# commentaire`|Permet d'expliquer le code|
|`echo "texte"`|Affiche un texte|
|`exit 0`|Code de sortie (0 = succès)|

### Le Shebang 🎯

bash

```bash
#!/bin/bash              # Standard Debian/Ubuntu
#!/usr/bin/env bash      # Plus portable (recommandé)
```

Le shebang sur la première ligne indique quel shell va exécuter le script.

### Les codes de sortie

Chaque commande retourne un **code de sortie** (entre 0 et 255) :

- **0** = Succès ✅
- **> 0** = Erreur ❌

bash

```bash
mkdir newDir        # Crée un dossier
echo $?             # Affiche le code de sortie (0 si réussi, 1 si échoué)
```

### Les métacaractères

Ce sont des **séparateurs spéciaux** pour Bash :

bash

```bash
;     # Exécute la commande suivante (séquence)
&&    # Exécute si la précédente réussit
||    # Exécute si la précédente échoue
|     # Pipe (envoie la sortie à la commande suivante)
\     # Caractère d'échappement
" "   # Double guillemets (variable interprétée)
' '   # Guillemets simples (variable non interprétée)
```

**Exemples :**

bash

```bash
echo "Bonjour" ; echo "Monde"              # Les 2 s'exécutent
cd dossier && echo "Succès" || echo "Erreur"
echo 'Bonjour $name'  # Affiche littéralement : Bonjour $name
echo "Bonjour $name"  # Affiche : Bonjour [valeur de name]
```

---

## Variables

### Qu'est-ce qu'une variable ?

Une variable est un **contenant nommé** pour stocker une valeur qu'on peut réutiliser.

bash

```bash
prenom="Alice"          # Créer une variable
echo $prenom            # Accéder à la valeur
unset prenom            # Supprimer la variable
```

### Règles de nommage

✅ **Noms valides :**

- Commencent par une lettre ou `_`
- Contiennent lettres, chiffres, `_`
- Sensibles à la casse

❌ **Noms invalides :**

- `1variable` (commence par un chiffre)
- `some-variable` (tiret interdit)
- `my@mail` (caractères spéciaux)

### Convention : snake_case

bash

```bash
user_name="Alice"           # ✅ Recommandé
creation_user_account()     # ✅ Pour les fonctions
```

### Variables prédéfinies importantes

|Variable|Signification|
|---|---|
|`$0`|Nom du script|
|`$1, $2, $3...`|Arguments du script|
|`$#`|Nombre d'arguments|
|`$*`|Tous les arguments en un mot|
|`$@`|Tous les arguments séparés|
|`$?`|Code de sortie de la dernière commande|
|`$$`|ID du process courant|

**Exemple :**

bash

```bash
#!/bin/bash
./script.sh Alice Bob Charlie

echo $0     # Affiche : ./script.sh
echo $1     # Affiche : Alice
echo $#     # Affiche : 3
```

### Substitution de commandes

Exécuter une commande et stocker son résultat :

bash

```bash
date_now=$(date)                # Stocker le résultat
echo "Nous sommes le : $date_now"

uid=$(id -u)                    # Récupérer mon UID
list=$(ls -la)                  # Stocker une liste
```

### Substitution arithmétique

Faire des calculs :

bash

```bash
result=$(( 5 + 3 ))             # Résultat : 8
total=$(( 10 * 2 - 3 ))         # Résultat : 17
i=$(( $i + 1 ))                 # Incrémenter une variable
```

### Contexte des variables : Shell parent vs shell fils

```
┌─────────────────────────────────┐
│   Shell parent (courant)        │
│   variable = "valeur"           │
├─────────────────────────────────┤
│  ┌──────────────────────────┐   │
│  │  Shell fils (script)     │   │
│  │  variable non accessible │   │
│  └──────────────────────────┘   │
└─────────────────────────────────┘
```

**Solution : `export` la variable**

bash

```bash
export ma_variable="valeur"     # Visible dans tous les shell fils
```

---

## Structures Conditionnelles

### Concept : Test Vrai ou Faux

Pour Bash :

- **0** = Vrai ✅
- **Autre nombre** = Faux ❌

bash

```bash
[ condition ]                   # Test avec crochets
test condition                  # Ou avec test
```

### Comparaison de chaînes

bash

```bash
s1 = s2          # Égal
s1 != s2         # Différent
-z s1            # s1 est vide
-n s1            # s1 n'est pas vide

# Exemples
[ "chat" = "chat" ]       # Vrai (0)
[ "chat" != "souris" ]    # Vrai (0)
[ -z "" ]                 # Vrai, la chaîne est vide
```

### Comparaison de nombres

bash

```bash
n1 -eq n2       # Égal
n1 -ne n2       # Différent
n1 -lt n2       # Inférieur à
n1 -le n2       # Inférieur ou égal
n1 -gt n2       # Supérieur à
n1 -ge n2       # Supérieur ou égal

# Exemples
[ 5 -gt 3 ]     # Vrai
[ 10 -eq 10 ]   # Vrai
```

### Opérateurs logiques

bash

```bash
! condition          # NON logique (inverse)
c1 -a c2             # ET logique (AND)
c1 -o c2             # OU logique (OR)

# Exemples
[ ! 5 -eq 3 ]                      # Vrai (5 n'égale pas 3)
[ 2 -lt 5 -a 5 -lt 10 ]            # Vrai (ET des deux)
[ $age -lt 18 -o $age -gt 65 ]     # Vrai si âge < 18 OU > 65
```

### Opérateurs sur fichiers/dossiers

bash

```bash
-e p        # p existe
-f p        # p est un fichier
-d p        # p est un dossier
-s p        # p existe et taille > 0
-r p        # Je peux lire p
-w p        # Je peux écrire p
-x p        # Je peux exécuter p

# Exemples
[ -f /etc/passwd ]      # Vérifie que c'est un fichier
[ -d ~/Documents ]      # Vérifie que c'est un dossier
```

### Structure IF

bash

```bash
if [ condition ]
then
    # Instructions si vrai
elif [ condition2 ]
    # Instructions si condition2 vraie
else
    # Instructions par défaut
fi
```

**Exemple pratique :**

bash

```bash
#!/bin/bash
age=$1

if [ $age -lt 18 ]
then
    echo "Vous êtes mineur"
elif [ $age -ge 65 ]
then
    echo "Vous êtes senior"
else
    echo "Vous êtes adulte"
fi
```

### Structure CASE

Pour énumérer plusieurs cas :

bash

```bash
case $variable in
    valeur1)
        echo "C'est la valeur 1"
        ;;
    valeur2 | valeur3)
        echo "C'est la valeur 2 ou 3"
        ;;
    *)
        echo "Valeur par défaut"
        ;;
esac
```

**Exemple :**

bash

```bash
#!/bin/bash
read -p "Choisissez (1, 2 ou 3) : " choix

case $choix in
    1)
        echo "Vous avez choisi 1"
        ;;
    2)
        echo "Vous avez choisi 2"
        ;;
    3)
        echo "Vous avez choisi 3"
        ;;
    *)
        echo "Choix invalide"
        ;;
esac
```

---

## Structures Itératives

### Concept : Les boucles

Les boucles permettent de **répéter des instructions** un nombre de fois ou tant qu'une condition est remplie.

```
┌─────────────────────┐
│  Condition vraie?  │
├─────────────────────┤
│  Oui  ↓             │
│  ┌──────────────┐   │
│  │ Instructions │   │
│  └──────────────┘   │
│       ↓ Retour      │
│  Non ↓              │
│  Suite du script    │
└─────────────────────┘
```

### Boucle FOR (sur liste)

Parcourir une liste de valeurs :

bash

```bash
for variable in liste
do
    # Instructions
done
```

**Exemples :**

bash

```bash
# Liste simple
for fruit in "Pomme" "Banane" "Orange"
do
    echo "Je mange une $fruit"
done

# Avec substitution de commande
for numero in $(seq 1 5)
do
    echo "Nombre : $numero"
done

# Parcourir les fichiers du dossier
for fichier in *
do
    echo "Fichier : $fichier"
done
```

### Boucle FOR (alternative avec arithmétique)

Comme un `for` classique en C :

bash

```bash
for (( i=1 ; i <= 5 ; i++ ))
do
    echo "Itération $i"
done
```

**Expliqué :**

- `i=1` : initialisation
- `i <= 5` : condition (tant que vraie)
- `i++` : incrément après chaque tour

### Boucle WHILE

Tant qu'une condition est vraie :

bash

```bash
while [ condition ]
do
    # Instructions
done
```

**Exemple : Compte à rebours**

bash

```bash
#!/bin/bash
nombre=5

while [ $nombre -ge 0 ]
do
    echo $nombre
    nombre=$(( $nombre - 1 ))
done

echo "Décollage!"
```

---

## Fonctions

### Qu'est-ce qu'une fonction ?

Une fonction est un **bloc de code nommé** qu'on peut réutiliser plusieurs fois.

**Avantages :**

- Code plus lisible et organisé
- Réutilisabilité
- Maintenance facile
- Évite les répétitions

### Déclarer une fonction

bash

```bash
# Syntaxe 1
function nom_fonction()
{
    instructions
}

# Syntaxe 2 (équivalent)
nom_fonction()
{
    instructions
}
```

⚠️ **Important** : Les fonctions doivent être déclarées **avant** d'être utilisées.

**Exemple simple :**

bash

```bash
#!/bin/bash

function saluer()
{
    echo "Bonjour, bienvenue!"
}

saluer              # Appel de la fonction
echo "Au revoir"
saluer              # Deuxième appel
```

### Fonctions avec paramètres

Les paramètres fonctionnent comme dans un script :

bash

```bash
function greet()
{
    if [ $# -gt 0 ]
    then
        echo "Bonjour $1"
    else
        echo "Bonjour inconnu"
    fi
}

greet "Alice"       # Affiche : Bonjour Alice
greet               # Affiche : Bonjour inconnu
```

### Retour de valeur

**Option 1 : Code numérique avec `return`**

bash

```bash
fonction_test()
{
    if [ $1 -gt 0 ]
    then
        return 0        # Succès
    else
        return 1        # Erreur
    fi
}

fonction_test 5
if [ $? -eq 0 ]
then
    echo "Succès"
fi
```

**Option 2 : Texte avec `echo`** (plus courant)

bash

```bash
fonction_addition()
{
    somme=$(( $1 + $2 ))
    echo $somme
}

resultat=$(fonction_addition 5 3)
echo "Résultat : $resultat"     # Affiche : 8
```

### Périmètre des variables : LOCAL vs GLOBAL

**Par défaut : variables GLOBALES**

bash

```bash
#!/bin/bash

ma_fonction()
{
    var="Bonjour"
}

var="Au revoir"
ma_fonction
echo $var           # Affiche : Bonjour (modifiée par la fonction!)
```

**Utiliser LOCAL pour isoler**

bash

```bash
#!/bin/bash

ma_fonction()
{
    local var="Bonjour"    # Reste dans la fonction
}

var="Au revoir"
ma_fonction
echo $var           # Affiche : Au revoir (pas modifiée)
```

**Schéma :**

```
┌──────────────────────────────────┐
│  Script (Global scope)           │
│  var = "valeur globale"          │
│ ┌──────────────────────────────┐ │
│ │  Fonction                    │ │
│ │  local var = "locale"        │ │
│ │  (reste dans la fonction)    │ │
│ └──────────────────────────────┘ │
└──────────────────────────────────┘
```

---

## Bonnes Pratiques

### 1. Toujours ajouter le shebang

bash

```bash
#!/usr/bin/env bash     # Portable et recommandé
```

### 2. Utiliser les commentaires

bash

```bash
#!/bin/bash
# Script de sauvegarde
# Utilité: Sauvegarde automatique du dossier /home
# Usage: backup.sh
# Auteur: Jean DOE
# Mise à jour: 15/01/2025

# Créer le dossier de sauvegarde
mkdir -p /backups
```

### 3. Quoting des variables

Toujours entourer les variables de guillemets pour éviter les problèmes d'espaces :

bash

```bash
# ❌ Risqué
echo $variable

# ✅ Recommandé
echo "$variable"
```

### 4. Gestion d'erreurs avec `||`

bash

```bash
cd /dossier 2>/dev/null && echo "Succès" || echo "Erreur"
```

### 5. NE PAS mettre SUDO dans les scripts

❌ **Mauvais :**

bash

```bash
#!/bin/bash
sudo apt update
sudo apt upgrade -y
```

✅ **Bon :**

bash

```bash
#!/bin/bash
apt update
apt upgrade -y

# Exécuter avec : sudo ./script.sh
```

**Ou vérifier en début de script :**

bash

```bash
#!/bin/bash

if [[ $EUID -ne 0 ]]
then
    echo "Ce script doit être exécuté en sudo" >&2
    exit 1
fi

apt update
exit 0
```

### 6. Utiliser des variables pour la flexibilité

bash

```bash
#!/bin/bash

nom="$1"
age="$2"
dossier_travail="/tmp"

echo "Utilisateur : $nom, Âge : $age"
mkdir -p "$dossier_travail"
```

### 7. Utiliser des fonctions pour la réutilisabilité

bash

```bash
#!/bin/bash

# Fonction réutilisable
calculer()
{
    echo $(( $1 + $2 ))
}

resultat1=$(calculer 5 10)
resultat2=$(calculer 20 30)
resultat3=$(calculer 100 200)

echo "Résultat 1 : $resultat1"
echo "Résultat 2 : $resultat2"
echo "Résultat 3 : $resultat3"
```

### 8. Logs et traçabilité

bash

```bash
#!/bin/bash

LOG_FILE="/var/log/mon_script.log"

log_message()
{
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $1" >> $LOG_FILE
}

log_message "Script démarré"
mkdir /test && log_message "Dossier créé" || log_message "Erreur création"
log_message "Script terminé"
```

### 9. Outils de vérification

Utiliser **ShellCheck** pour vérifier la syntaxe :

bash

```bash
shellcheck mon_script.sh
```

---

## Récapitulatif Visuel

### Cycle de vie d'un script

```
1. Création du fichier
   ↓
2. Rendre exécutable (chmod u+x script.sh)
   ↓
3. Ajouter shebang (#!/bin/bash)
   ↓
4. Coder les instructions
   ↓
5. Ajouter exit 0 en fin
   ↓
6. Tester (./script.sh)
   ↓
7. Déboguer si nécessaire ($? pour voir le code de sortie)
```

### Hiérarchie des structures de contrôle

```
STRUCTURES DE CONTRÔLE
│
├─ CONDITIONNELLES
│  ├─ IF ... THEN ... ELSE ... FI
│  ├─ CASE ... ESAC
│  └─ Opérateurs: [ ], test, -a, -o, !
│
├─ ITÉRATIVES
│  ├─ FOR (liste)
│  ├─ FOR (arithmétique)
│  └─ WHILE
│
└─ FONCTIONS
   ├─ Déclaration
   ├─ Paramètres
   ├─ Retour (return, echo)
   └─ Périmètre (local, global)
```

### Tableau récapitulatif : Opérateurs et tests

|Type|Opérateur|Signification|
|---|---|---|
|**Chaînes**|`=`|Égal|
||`!=`|Différent|
||`-z`|Vide|
||`-n`|Non vide|
|**Nombres**|`-eq`|Égal|
||`-ne`|Différent|
||`-lt`|Inférieur|
||`-le`|Inférieur ou égal|
||`-gt`|Supérieur|
||`-ge`|Supérieur ou égal|
|**Fichiers**|`-e`|Existe|
||`-f`|Fichier|
||`-d`|Dossier|
||`-r`|Lisible|
||`-w`|Modifiable|
||`-x`|Exécutable|
|**Logiques**|`!`|NON|
||`-a`|ET|
||`-o`|OU|

### Exemple complet : Script professionnel

bash

```bash
#!/usr/bin/env bash

##########################################################
# Script de maintenance système
# Utilité: Nettoie les fichiers temporaires
# Usage: ./maintenance.sh <chemin>
# Auteur: Admin <admin@example.com>
# Mise à jour: 15/01/2025
##########################################################

LOG_FILE="/var/log/maintenance.log"

# Fonction de log
logger() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $1" >> "$LOG_FILE"
}

# Vérifier les droits
if [[ $EUID -ne 0 ]]; then
    echo "Ce script doit être exécuté en sudo" >&2
    exit 1
fi

chemin="${1:-.}"

# Vérifier que le chemin existe
if [ ! -d "$chemin" ]; then
    logger "Erreur: chemin inexistant"
    exit 1
fi

logger "Début du nettoyage: $chemin"

# Boucle sur les fichiers temporaires
for fichier in "$chemin"/*.tmp; do
    [ -e "$fichier" ] || continue
    rm -f "$fichier"
    logger "Fichier supprimé: $fichier"
done

logger "Nettoyage terminé"
exit 0
```

---

## 📚 Références

- Documentation officielle Bash : [GNU Bash Manual](https://www.gnu.org/software/bash/manual/)
- Wikibooks : [Programmation Bash](https://fr.wikibooks.org/wiki/Programmation_Bash)
- Wiki Bash Hackers : [Bash Hackers Wiki](https://wiki.bash-hackers.org/)
- Bash Guide de Greg : [Greg's Bash Guide](https://mywiki.wooledge.org/BashGuide)
- ExplainShell : [explainshell.com](https://explainshell.com/)
- ShellCheck : [www.shellcheck.net](https://www.shellcheck.net/)