# Questionnaire – CP6 : Automatiser des tâches à l'aide de scripts

**Formation TSSR | Scripting Bash** **15 questions – Réponses ouvertes**

---

## Question 1

Qu'est-ce que le **shebang** dans un script Bash ? Donnez sa syntaxe exacte et expliquez pourquoi il doit obligatoirement figurer en **première ligne** du fichier.

---

## Question 2

Quelle est la différence entre ces deux façons d'exécuter un script Bash ?

```bash
bash mon_script.sh
./mon_script.sh
```

Quelle commande est nécessaire avant de pouvoir utiliser la seconde méthode ?

---

## Question 3

Que représente la variable spéciale `$?` en Bash ? Quelles sont les deux valeurs principales qu'elle peut renvoyer et que signifient-elles ?

---

## Question 4

Expliquez la différence de comportement entre les guillemets simples `' '` et les guillemets doubles `" "` en Bash concernant les variables. Illustrez votre réponse avec un exemple concret pour chacun des deux cas.

---

## Question 5

Citez et expliquez les trois variables spéciales permettant de gérer les **arguments passés à un script** :

- `$1`, `$2`, `$3...`
- `$#`
- `$@`

---

## Question 6

Analysez le script suivant et décrivez précisément ce qu'il affiche :

```bash
#!/bin/bash
nom="Dupont"
age=30

if [ $age -ge 18 ] && [ -n "$nom" ]
then
    echo "Bienvenue $nom, vous êtes majeur."
else
    echo "Accès refusé."
fi
```

Quel opérateur de comparaison numérique aurait-il fallu utiliser si l'on voulait vérifier que l'âge est **strictement inférieur** à 18 ?

---

## Question 7

Quelle est la différence entre les opérateurs `-a` / `-o` et les opérateurs `&&` / `||` dans un script Bash ? Dans quel contexte utilise-t-on chacun ?

---

## Question 8

Complétez et expliquez le rôle de chaque test sur les fichiers/dossiers ci-dessous :

|Opérateur|Signification|
|---|---|
|`-e`|?|
|`-f`|?|
|`-d`|?|
|`-x`|?|
|`-w`|?|

---

## Question 9

Quel est le résultat produit par le script suivant ? Expliquez le fonctionnement de la boucle utilisée.

```bash
#!/bin/bash
for i in $(seq 1 5)
do
    if [ $(( $i % 2 )) -eq 0 ]
    then
        echo "$i est pair"
    fi
done
```

---

## Question 10

Quelle est la différence fondamentale entre une boucle `for` et une boucle `while` en Bash ? Donnez un exemple d'utilisation typique pour chacune.

---

## Question 11

Observez la fonction suivante :

```bash
#!/bin/bash

calculer()
{
    local resultat=$(( $1 * $2 ))
    echo $resultat
}

val=$(calculer 6 7)
echo "Résultat : $val"
```

- Quel est le rôle du mot-clé `local` ici ?
- Comment la valeur est-elle **retournée** depuis la fonction ?
- Qu'affiche ce script ?

---

## Question 12

Expliquez pourquoi il est considéré comme une **mauvaise pratique** d'inclure `sudo` à l'intérieur d'un script Bash. Quelle alternative propre peut-on mettre en place pour s'assurer que le script est bien exécuté avec les droits appropriés ?

---

## Question 13

Dans le cadre de l'automatisation de tâches système, expliquez l'intérêt d'intégrer un **mécanisme de logs** dans un script. Écrivez une fonction Bash simple appelée `log_message` qui :

- Prend un message en paramètre
- L'écrit dans un fichier `/var/log/mon_script.log`
- Préfixe chaque ligne de la date et de l'heure au format `[YYYY-MM-DD HH:MM:SS]`

---

## Question 14

Quel est le rôle de la commande `export` en Bash ? Expliquez la notion de **shell parent / shell fils** et dans quel cas l'utilisation d'`export` est indispensable.

---

## Question 15

Analysez le script suivant et identifiez **au moins trois erreurs ou mauvaises pratiques** qu'il contient. Pour chaque problème identifié, proposez la correction appropriée.

```bash
#!/bin/bash
sudo mkdir /backup
cp -r /home/user/documents /backup
echo Script terminé
```

---

_Bonne chance ! Transmettez vos réponses pour correction._