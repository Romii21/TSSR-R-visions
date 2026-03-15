### Formation TSSR | Bash & PowerShell

> **Consignes** : Répondez sans support dans un premier temps. Les questions montent progressivement en difficulté. Certaines demandent d'écrire du code ou d'analyser un extrait de script.

---

## PARTIE 1 — Fondamentaux du scripting _(Débutant)_

**1.** Quelle est la différence fondamentale entre un **programme** et un **script** ?

**R1.**

**2.** À quoi sert la ligne `#!/bin/bash` placée en première ligne d'un script ? Comment s'appelle-t-elle ?

**

**3.** Quelle est la différence entre `#!/bin/bash` et `#!/usr/bin/env bash` ? Laquelle est recommandée et pourquoi ?

**4.** Que signifie un code de sortie à **0** ? Et à **1** ? Quelle variable permet de consulter le code de sortie de la dernière commande exécutée ?

**5.** Quelle commande utilise-t-on pour rendre un script Bash exécutable ? Donnez la commande exacte pour le fichier `backup.sh`.

**6.** Expliquez la différence de comportement entre ces deux lignes :

bash

```bash
echo 'Bonjour $USER'
echo "Bonjour $USER"
```

**7.** En PowerShell, quelle est l'extension d'un fichier script ? Avec quel exécutable est-il lancé ?

**8.** Quelle est la différence majeure entre PowerShell et un shell Bash concernant la nature des données manipulées ?

**9.** En PowerShell, les cmdlets respectent une convention de nommage stricte. Laquelle ? Donnez deux exemples de cmdlets.

**10.** PowerShell est-il sensible à la casse ? Justifiez avec un exemple.

---

## PARTIE 2 — Variables et données _(Intermédiaire)_

**11.** En Bash, déclarez une variable `prenom` avec la valeur `"Alice"`, puis affichez-la. Quelle est la règle concernant les espaces autour du `=` ?

**12.** Citez et expliquez **4 variables prédéfinies** parmi : `$0`, `$1`, `$#`, `$@`, `$?`, `$$`.

**13.** Quelle est la syntaxe de la **substitution de commande** en Bash ? Donnez un exemple qui stocke la date courante dans une variable.

**14.** Comment réalise-t-on une **opération arithmétique** en Bash ? Donnez la syntaxe et un exemple calculant `(10 * 3) - 5`.

**15.** Qu'est-ce que le mot-clé `export` permet de faire en Bash ? Dans quel contexte est-il nécessaire ?

**16.** En PowerShell, comment déclare-t-on une variable contenant le texte `"Serveur01"` ? Quelle différence existe-t-il entre `'$var'` et `"$var"` en PowerShell ?

**17.** En PowerShell, qu'est-ce qu'un **hashtable** ? Donnez un exemple avec au moins deux paires clé/valeur, puis montrez comment accéder à une valeur précise.

**18.** En PowerShell, comment déclarer un tableau contenant `"Lundi"`, `"Mardi"`, `"Mercredi"` ? Comment accéder au dernier élément ?

---

## PARTIE 3 — Structures de contrôle _(Intermédiaire)_

**19.** En Bash, quelle est la valeur logique de **0** et d'un **nombre non nul** dans un test conditionnel ? Pourquoi est-ce l'inverse du fonctionnement habituel en programmation ?

**20.** Écrivez en Bash une structure `if/elif/else` qui vérifie si `$age` passé en argument est inférieur à 18 ("Mineur"), supérieur ou égal à 65 ("Senior"), ou autre ("Adulte").

**21.** Quelle est la différence entre ces deux tests en Bash ?

bash

```bash
[ $a -eq $b ]
[ "$a" = "$b" ]
```

**22.** Expliquez le rôle des opérateurs `&&`, `||` et `;` en Bash. Donnez un exemple pour chacun.

**23.** Écrivez une structure `case` en Bash qui accepte un numéro de jour (`1` à `5`) et affiche le nom correspondant. Pour toute autre valeur : `"Jour invalide"`.

**24.** En PowerShell, écrivez une condition `if/elseif/else` qui vérifie si `$statut` est `"Running"`, `"Stopped"`, ou dans un autre état, avec un message adapté pour chaque cas.

**25.** En PowerShell, quels sont les opérateurs de comparaison équivalents à `==`, `!=`, `<`, `>` ? Pourquoi ne peut-on pas utiliser directement `<` et `>` ?

---

## PARTIE 4 — Boucles et itération _(Intermédiaire-Avancé)_

**26.** Écrivez en Bash une boucle `for` qui affiche les nombres de 1 à 10 avec la commande `seq`.

**27.** Écrivez en Bash une boucle `for` de style arithmétique (syntaxe C) qui affiche la table de multiplication par 3 (de 3×1 à 3×10).

**28.** Écrivez un script Bash avec une boucle `while` qui effectue un compte à rebours de 10 à 0, puis affiche `"Décollage !"`.

**29.** Quelle est la différence entre une boucle `while` et une boucle `for` ? Dans quel cas utilise-t-on l'une plutôt que l'autre ?

**30.** En PowerShell, écrivez une boucle `foreach` qui parcourt tous les services Windows et affiche le **nom** et l'**état** de chacun.

**31.** En PowerShell, quelle est la différence entre `Do While` et `Do Until` ? Donnez un exemple concret pour chacune.

**32.** En PowerShell, quelle est la différence entre `ForEach` en syntaxe classique et en pipeline avec `ForEach-Object` ? Donnez un exemple des deux syntaxes.

---

## PARTIE 5 — Fonctions _(Avancé)_

**33.** En Bash, quelles sont les deux syntaxes valides pour déclarer une fonction ? Y a-t-il une contrainte sur l'ordre déclaration / appel ?

**34.** En Bash, comment une fonction récupère-t-elle ses paramètres ? Quelle variable indique le nombre de paramètres reçus ?

**35.** Expliquez la différence entre `return` et `echo` pour retourner une valeur depuis une fonction Bash. Quand utilise-t-on l'un ou l'autre ?

**36.** Analysez ce script et expliquez ce qu'il affiche et pourquoi :

bash

```bash
#!/bin/bash
ma_fonction() {
    var="Modifiée dans la fonction"
}
var="Valeur initiale"
ma_fonction
echo "$var"
```

**37.** Modifiez l'exemple précédent pour que `var` reste locale à la fonction. Quel mot-clé utilise-t-on ?

**38.** En PowerShell, écrivez une fonction `Get-InfoDisque` qui prend en paramètre un nom de disque (ex: `"C:"`) et affiche sa taille totale et l'espace libre disponible.

**39.** En PowerShell, comment déclare-t-on un paramètre **obligatoire** dans une fonction ? Donnez la syntaxe complète avec le bloc `param`.

---

## PARTIE 6 — Bonnes pratiques _(Avancé)_

**40.** Citez **5 bonnes pratiques** à respecter lors de l'écriture d'un script Bash professionnel.

**41.** Pourquoi est-il déconseillé d'utiliser `sudo` à l'intérieur d'un script ? Quelle est la bonne pratique alternative ?

**42.** Écrivez le début d'un script Bash professionnel comprenant : shebang, en-tête de commentaire (nom, utilité, usage, auteur), vérification des droits root, et initialisation d'un fichier de log.

**43.** Quel outil permet d'analyser la syntaxe d'un script Bash et de détecter des erreurs courantes ? Comment s'utilise-t-il ?

**44.** Expliquez ce que fait cette ligne et pourquoi elle est une bonne pratique :

bash

```bash
cd /dossier 2>/dev/null && echo "Succès" || echo "Erreur : dossier introuvable"
```

**45.** Pourquoi faut-il toujours entourer les variables de guillemets doubles (`"$variable"`) ? Donnez un exemple de problème que cela évite.

---

## PARTIE 7 — PowerShell avancé et administration _(Avancé)_

**46.** Qu'est-ce que la **politique d'exécution** (Execution Policy) en PowerShell ? Quelles sont les valeurs possibles et laquelle est recommandée en production ?

**47.** Expliquez le concept de **pipeline** en PowerShell. En quoi est-il différent du pipe `|` sous Linux/Bash ?

**48.** En PowerShell, qu'est-ce qu'un **Job** ? À quoi sert-il ? Citez les 5 cmdlets principales pour les gérer.

**49.** Quelle est la différence entre `Enter-PSSession` et `Invoke-Command` ? Dans quel cas utilise-t-on chacun ?

**50.** Quel prérequis est indispensable pour exécuter des commandes PowerShell à distance sur une machine cible ?

---

## PARTIE 8 — Mise en situation _(Expert TSSR)_

**51.** Rédigez un script Bash de sauvegarde automatique qui :

- Prend un répertoire source en argument `$1`
- Vérifie que ce répertoire existe (sinon quitte avec un message d'erreur)
- Crée une archive `.tar.gz` horodatée dans `/backups/`
- Logge l'opération dans `/var/log/backup.log`

Respectez les bonnes pratiques.

---

**52.** Rédigez un script PowerShell qui :

- Récupère la liste des services Windows **arrêtés**
- Pour chacun, tente de le redémarrer
- Affiche un message de succès ou d'échec pour chaque service

---

**53.** Identifiez **toutes les erreurs** (syntaxe, logique, mauvaises pratiques) dans ce script et proposez une version corrigée :

bash

```bash
#!/bin/bash
function verifier_espace
{
    seuil = 80
    utilisation=$(df / | grep / | awk '{print $5}' | sed 's/%//')
    
    if [ $utilisation > $seuil ]
    then
        echo "Espace disque critique : $utilisation%"
    fi
}
verifier_espace
```

---

**54.** Expliquez la différence entre `>` et `>>` en Bash. Dans le contexte d'un fichier de log, laquelle utilise-t-on et pourquoi ?

---

**55.** En PowerShell, vous devez créer un script qui génère automatiquement 10 utilisateurs Active Directory nommés `stagiaire01` à `stagiaire10`, tous membres du groupe `"Stagiaires"`. Décrivez la logique du script (structure, cmdlets utilisées, boucle) sans nécessairement écrire le code complet.

---

_Fin du questionnaire CP6 — Bonne révision !_

---

Le questionnaire couvre **55 questions** réparties en 8 parties qui montent progressivement : des fondamentaux (shebang, variables, codes de sortie) jusqu'aux mises en situation concrètes niveau TSSR (scripts complets, débogage, administration AD). Il couvre Bash et PowerShell en parallèle tout au long.

Pour la prochaine fois, dis-moi simplement quel **CP** tu veux travailler et je génèrerai le questionnaire correspondant en m'appuyant sur tes cours du projet.