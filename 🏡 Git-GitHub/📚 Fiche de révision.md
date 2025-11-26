# Git & GitHub - Aide-Mémoire Complet des Commandes

## 📑 Table des Matières

1. [Configuration Initiale](https://claude.ai/chat/3f1c4fff-2fa3-43ae-8439-ed2fee4da3f3#configuration-initiale)
2. [Créer / Cloner un Dépôt](https://claude.ai/chat/3f1c4fff-2fa3-43ae-8439-ed2fee4da3f3#cr%C3%A9er--cloner-un-d%C3%A9p%C3%B4t)
3. [Gestion des Fichiers (Staging)](https://claude.ai/chat/3f1c4fff-2fa3-43ae-8439-ed2fee4da3f3#gestion-des-fichiers-staging)
4. [Commits](https://claude.ai/chat/3f1c4fff-2fa3-43ae-8439-ed2fee4da3f3#commits)
5. [Branches](https://claude.ai/chat/3f1c4fff-2fa3-43ae-8439-ed2fee4da3f3#branches)
6. [Fusion (Merge)](https://claude.ai/chat/3f1c4fff-2fa3-43ae-8439-ed2fee4da3f3#fusion-merge)
7. [Dépôt Distant (Remote)](https://claude.ai/chat/3f1c4fff-2fa3-43ae-8439-ed2fee4da3f3#d%C3%A9p%C3%B4t-distant-remote)
8. [Synchronisation avec le Distant](https://claude.ai/chat/3f1c4fff-2fa3-43ae-8439-ed2fee4da3f3#synchronisation-avec-le-distant)
9. [Stash](https://claude.ai/chat/3f1c4fff-2fa3-43ae-8439-ed2fee4da3f3#stash)
10. [Historique et Inspection](https://claude.ai/chat/3f1c4fff-2fa3-43ae-8439-ed2fee4da3f3#historique-et-inspection)
11. [Tags](https://claude.ai/chat/3f1c4fff-2fa3-43ae-8439-ed2fee4da3f3#tags)
12. [Annuler des Changements](https://claude.ai/chat/3f1c4fff-2fa3-43ae-8439-ed2fee4da3f3#annuler-des-changements)
13. [Rebase](https://claude.ai/chat/3f1c4fff-2fa3-43ae-8439-ed2fee4da3f3#rebase)
14. [Nettoyage](https://claude.ai/chat/3f1c4fff-2fa3-43ae-8439-ed2fee4da3f3#nettoyage)
15. [Alias Utiles](https://claude.ai/chat/3f1c4fff-2fa3-43ae-8439-ed2fee4da3f3#alias-utiles)

---

## Configuration Initiale

Configuration à faire une seule fois sur votre ordinateur.

```bash
# Configurer votre nom et email
git config --global user.name "Ton Nom"
git config --global user.email "ton.email@example.com"

# Choisir l'éditeur par défaut
git config --global core.editor "nano"    # ou vim, code, etc.

# Vérifier la configuration actuelle
git config --list

# Vérifier une paramètre spécifique
git config user.name
```

---

## Créer / Cloner un Dépôt

### Initialiser un nouveau dépôt local

```bash
# Créer un nouveau dépôt Git local
git init

# Initialiser dans un dossier spécifique
git init nom-dossier
```

### Cloner un dépôt existant

```bash
# Cloner un dépôt distant
git clone https://github.com/user/repo.git

# Cloner dans un dossier spécifique
git clone https://github.com/user/repo.git mon-dossier

# Cloner avec une branche spécifique
git clone -b develop https://github.com/user/repo.git

# Cloner sans historique complet (plus rapide)
git clone --depth 1 https://github.com/user/repo.git
```

---

## Gestion des Fichiers (Staging)

Préparer vos fichiers avant de les commiter.

```bash
# Voir l'état actuel du dépôt
git status

# Voir un résumé de l'état
git status -s

# Ajouter un fichier spécifique
git add fichier.txt

# Ajouter tous les fichiers modifiés
git add .

# Ajouter tous les fichiers d'une extension
git add *.js
git add src/

# Ajouter fichiers interactivement
git add -p

# Retirer un fichier du staging (mais le garder en local)
git reset fichier.txt

# Retirer tous les fichiers du staging
git reset
```

---

## Commits

Sauvegarder vos modifications avec un message explicite.

```bash
# Commit simple avec un message
git commit -m "Ajouter la fonctionnalité de login"

# Commit avec description longue (ouvre l'éditeur)
git commit

# Commit avec add inclus (seulement fichiers déjà tracked)
git commit -am "Message du commit"

# Voir ce qui sera commité avant de valider
git commit --dry-run

# Modifier le message du dernier commit
git commit --amend -m "Nouveau message"

# Ajouter des fichiers au dernier commit sans changer le message
git commit --amend --no-edit

# Commit vide (utile pour redémarrer les tests CI)
git commit --allow-empty -m "Redémarrer CI/CD"
```

---

## Branches

Gérer les lignes de développement parallèles.

### Afficher les branches

```bash
# Lister les branches locales
git branch

# Lister les branches locales avec dernier commit
git branch -v

# Lister les branches distantes
git branch -r

# Lister TOUTES les branches (locales et distantes)
git branch -a
```

### Créer et changer de branche

```bash
# Créer une branche
git branch nom-branche

# Créer et passer à une nouvelle branche
git checkout -b feature/authentification

# Ou avec la commande moderne (Git 2.23+)
git switch -c feature/authentification

# Passer à une branche existante
git checkout main
git switch main

# Créer une branche à partir d'une branche distante
git checkout -b local-branch origin/remote-branch
```

### Supprimer et renommer

```bash
# Supprimer une branche (fusionnée)
git branch -d nom-branche

# Forcer la suppression (pas fusionnée)
git branch -D nom-branche

# Supprimer une branche distante
git push origin --delete nom-branche

# Renommer une branche locale
git branch -m ancien-nom nouveau-nom

# Renommer la branche actuelle
git branch -m nouveau-nom
```

---

## Fusion (Merge)

Combiner deux branches ensemble.

```bash
# Fusionner une branche dans la branche actuelle
git merge nom-branche

# Fusionner avec un message personnalisé
git merge nom-branche -m "Message de merge"

# Fusionner sans créer de commit (fast-forward)
git merge --ff-only nom-branche

# Fusionner en forçant un commit de merge
git merge --no-ff nom-branche

# Annuler une fusion en cours
git merge --abort

# Lancer l'outil de résolution de conflits
git mergetool

# Voir les branches fusionnées
git branch --merged
git branch --no-merged
```

---

## Dépôt Distant (Remote)

Gérer les connexions avec les serveurs distants.

```bash
# Lister les dépôts distants
git remote

# Lister les dépôts avec URLs
git remote -v

# Ajouter un dépôt distant
git remote add origin https://github.com/user/repo.git

# Ajouter un second dépôt distant
git remote add upstream https://github.com/autre-user/repo.git

# Afficher les infos d'un dépôt distant
git remote show origin

# Changer l'URL d'un dépôt distant
git remote set-url origin https://nouveau-url.git

# Supprimer un dépôt distant
git remote remove origin

# Renommer un dépôt distant
git remote rename origin upstream
```

---

## Synchronisation avec le Distant

Récupérer et envoyer les modifications.

### Récupérer les changements

```bash
# Récupérer les changements SANS les fusionner
git fetch

# Récupérer depuis un dépôt spécifique
git fetch origin

# Récupérer une branche spécifique
git fetch origin develop

# Fusionner les changements en local
git merge origin/main

# Récupérer ET fusionner (raccourci)
git pull

# Pull avec rebase au lieu de merge
git pull --rebase
```

### Envoyer les changements

```bash
# Pousser la branche actuelle
git push

# Pousser vers une branche spécifique
git push origin main

# Pousser une branche nouvelle et lier (tracking)
git push -u origin feature/new-page

# Pousser toutes les branches
git push --all

# Pousser avec forces (attention, peut perdre des données !)
git push --force

# Pousser avec sécurité (refuse si rebase nécessaire)
git push --force-with-lease

# Pousser les tags
git push origin v1.0.0
git push origin --tags

# Supprimer une branche distante
git push --delete origin nom-branche
git push origin :nom-branche  # Ancienne syntaxe
```

---

## Stash

Sauvegarder temporairement les changements non commités.

```bash
# Sauvegarder les modifications en cours
git stash

# Sauvegarder avec un message
git stash save "Message descriptif"

# Sauvegarder en incluant les fichiers non trackés
git stash -u

# Voir la liste des stashs
git stash list

# Appliquer le dernier stash (sans le supprimer)
git stash apply

# Appliquer un stash spécifique
git stash apply stash@{2}

# Appliquer ET supprimer le stash
git stash pop

# Voir les changements d'un stash
git stash show stash@{0}

# Voir les changements en détail
git stash show -p stash@{0}

# Supprimer un stash
git stash drop stash@{0}

# Supprimer tous les stashs
git stash clear
```

---

## Historique et Inspection

Consulter l'historique des commits et les changements.

```bash
# Voir l'historique complet
git log

# Une ligne par commit
git log --oneline

# Voir les 5 derniers commits
git log -5

# Voir avec un graphe des branches
git log --graph --oneline --all

# Filtrer par auteur
git log --author="Nom"

# Filtrer par date
git log --since="2025-01-01"
git log --until="2025-12-31"

# Chercher dans les messages de commit
git log --grep="fix"
git log --grep="login" -i  # Insensible à la casse

# Voir les changements du dernier commit
git show HEAD

# Voir un commit spécifique
git show abc123

# Voir le commit d'il y a X commits
git show HEAD~2

# Voir les différences non stagées
git diff

# Voir les différences stagées
git diff --staged

# Voir les différences entre deux branches
git diff main..develop

# Voir les fichiers modifiés
git diff --name-only

# Voir les statistiques de changements
git diff --stat
```

---

## Tags

Marquer des versions importantes (releases).

```bash
# Lister tous les tags
git tag

# Lister les tags avec filtre
git tag -l "v1.*"

# Créer un tag léger
git tag v1.0.0

# Créer un tag annoté (recommandé)
git tag -a v1.0.0 -m "Version 1.0.0 - Production"

# Voir les infos d'un tag
git show v1.0.0

# Créer un tag sur un commit spécifique
git tag v1.0.0 abc123

# Pousser un tag spécifique
git push origin v1.0.0

# Pousser tous les tags
git push origin --tags

# Supprimer un tag localement
git tag -d v1.0.0

# Supprimer un tag distant
git push origin --delete v1.0.0
git push origin :refs/tags/v1.0.0  # Ancienne syntaxe
```

---

## Annuler des Changements

Revenir en arrière sur vos modifications.

```bash
# Annuler les modifications d'un fichier (avant staging)
git restore fichier.txt
git checkout HEAD -- fichier.txt  # Ancienne syntaxe

# Retirer un fichier du staging (mais le garder)
git restore --staged fichier.txt
git reset fichier.txt  # Ancienne syntaxe

# Supprimer le dernier commit (attention !)
# Option 1 : Garder les changements en staging
git reset --soft HEAD~1

# Option 2 : Garder les changements en local
git reset --mixed HEAD~1
git reset HEAD~1  # Pareil que --mixed

# Option 3 : SUPPRIMER les changements (dangereux !)
git reset --hard HEAD~1

# Annuler plusieurs commits
git reset --hard HEAD~3

# Créer un commit qui annule un autre commit
git revert HEAD

# Revenir à un commit spécifique
git reset --hard abc123

# Voir l'historique des reset
git reflog  # Permet de récupérer des commits "perdus"
```

---

## Rebase

Réappliquer les commits sur une autre branche (avancé).

```bash
# Rebase simple
git rebase main

# Rebase interactif (réorganiser, squash, modifier)
git rebase -i HEAD~3

# Continuer après résolution de conflits
git rebase --continue

# Annuler le rebase
git rebase --abort

# Rebase avec sélection de commits
git rebase -i abc123
```

---

## Nettoyage

Optimiser et nettoyer le dépôt.

```bash
# Voir ce qui sera supprimé (simulation)
git clean -n

# Supprimer les fichiers non trackés
git clean -f

# Supprimer fichiers + dossiers non trackés
git clean -fd

# Supprimer incluant les fichiers ignorés
git clean -fdx

# Optimiser la base de données Git
git gc

# Vérifier l'intégrité du dépôt
git fsck
```

---

## Alias Utiles

Créer des raccourcis pour les commandes fréquentes.

```bash
# Ajouter un alias global
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.ci commit
git config --global alias.st status

# Alias plus avancés
git config --global alias.unstage 'reset HEAD --'
git config --global alias.last 'log -1 HEAD'
git config --global alias.log-graph 'log --graph --oneline --all'
git config --global alias.see 'show --stat'

# Depuis maintenant, vous pouvez utiliser :
git co main           # au lieu de git checkout main
git br -a             # au lieu de git branch -a
git log-graph         # au lieu de git log --graph --oneline --all
```

Vérifier vos alias :

```bash
git config --global --list | grep alias
```

---

## 🎯 Commandes Essentielles à Mémoriser

```bash
# Les 5 PLUS importantes
git status              # Voir où vous en êtes
git add .               # Préparer vos changements
git commit -m "msg"     # Valider vos changements
git push                # Envoyer au serveur
git pull                # Récupérer du serveur

# Bonus : Autres très utiles
git log --oneline       # Voir l'historique
git branch -a           # Voir toutes les branches
git checkout -b feature # Créer une branche
git merge nom-branche   # Fusionner une branche
git stash               # Sauvegarder temporairement
```

---

## 💡 Astuces Pratiques

```bash
# Obtenir l'aide sur une commande
git [commande] --help

# Exempels : 
git merge --help
git reset --help

# Voir les changements en détail
git diff fichier.txt

# Comparer deux branches
git diff main..develop

# Chercher qui a modifié une ligne
git blame fichier.txt

# Copier un commit d'une autre branche
git cherry-pick abc123

# Chercher un commit par message
git log --grep="keyword"
```

---

## ⚠️ Danger : Les Commandes à Utiliser avec Prudence

```bash
# DANGER : Supprime les commits SANS possibilité de retour
git reset --hard HEAD~1

# DANGER : Force l'envoi et peut écraser le travail des autres
git push --force

# SAFER : Version sécurisée
git push --force-with-lease

# Pour récupérer un commit "perdu"
git reflog
git reset --hard abc123  # Le SHA du commit récupéré dans reflog
```

---

## 📝 Bonnes Pratiques pour les Commits

```bash
# ❌ Mauvais message
git commit -m "fix"
git commit -m "changements"

# ✅ Bon message
git commit -m "Fix: corriger le bug de pagination"
git commit -m "Feature: ajouter la connexion par Google"
git commit -m "Docs: mettre à jour le README"

# Format recommandé
[Type]: [Brève description]
Ex: "Feature: implémenter la barre de recherche"
Ex: "Fix: corriger l'affichage du footer sur mobile"
Ex: "Docs: documenter l'API REST"
```

---

## 🔗 Ressources Rapides

```bash
# Vous bloquez ?
git status              # D'abord voir l'état

# Vous devez fusionner
git fetch               # Récupérer les changements
git merge               # Fusionner
git push                # Envoyer

# Vous avez des conflits
git status              # Voir les fichiers en conflit
git merge --abort       # Annuler si c'est complexe
git mergetool           # Lancer l'outil de résolution
```

---

_Aide-mémoire compilée à partir des meilleures pratiques Git actuelles._