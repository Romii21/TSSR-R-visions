# Commandes Bash essentielles – Cheat Sheet

## 1. Navigation et fichiers de base

| Commande        | Description             | Exemple                         |
| --------------- | ----------------------- | ------------------------------- |
| `pwd`           | Répertoire courant      | `pwd`                           |
| `ls`            | Lister                  | `ls -lha`                       |
| `cd`            | Changer de dossier      | `cd /var/log` │ `cd ..`         |
| `mkdir`         | Créer un dossier        | `mkdir -p /tmp/nouveau/dossier` |
| `touch`         | Créer fichier vide      | `touch fichier.txt`             |
| `cp`            | Copier                  | `cp -r source/ dest/`           |
| `mv`            | Déplacer / renommer     | `mv old.txt new.txt`            |
| `rm`            | Supprimer               | `rm -rf dossier/`               |
| `cat`           | Afficher contenu        | `cat /etc/hosts`                |
| `less` / `more` | Parcourir fichier       | `less gros.log`                 |
| `head` / `tail` | Début ou fin de fichier | `tail -f /var/log/auth.log`     |

## 2. Recherche et texte

| Commande          | Description                                      | Exemple                                  |
|-------------------|--------------------------------------------------|------------------------------------------|
| `find`            | Chercher fichiers                                | `find / -name "*.conf" 2>/dev/null`      |
| `grep`            | Chercher dans fichiers                           | `grep -r "error" /var/log/`              |
| `wc`              | Compter lignes/mots                              | `wc -l access.log`                       |
| `sort`            | Trier                                            | `sort fichier.txt`                       |
| `uniq`            | Supprimer doublons                               | `sort fichier.txt \| uniq -c`             |
| `cut`             | Extraire colonnes                                | `cut -d: -f1 /etc/passwd`                |
| `awk`             | Traitement texte avancé                          | `awk '{print $1}' fichier.txt`           |
| `sed`             | Remplacer texte                                  | `sed 's/ancien/nouveau/g' fichier.txt`   |

## 3. Permissions

| Commande          | Description                                      | Exemple                              |
|-------------------|--------------------------------------------------|--------------------------------------|
| `chmod`           | Changer permissions                              | `chmod 755 script.sh`                |
| `chmod +x`        | Rendre exécutable                                | `chmod +x myscript.sh`               |
| `chown`           | Changer propriétaire                             | `chown user:group fichier`           |
| `ls -l`           | Voir permissions                                 | `ls -l`                              |

## 4. Utilisateurs et groupes

| Commande          | Description                                      | Exemple                                  |
|-------------------|--------------------------------------------------|------------------------------------------|
| `whoami`          | Utilisateur actuel                               | `whoami`                                 |
| `id`              | Infos utilisateur                                | `id john`                                |
| `sudo`            | Exécuter en root                                 | `sudo apt update`                        |
| `su -`            | Changer d'utilisateur                            | `su - marie`                             |
| `adduser`         | Créer utilisateur                                | `sudo adduser paul`                      |
| `usermod`         | Modifier utilisateur                             | `usermod -aG docker paul`                |
| `userdel -r`      | Supprimer utilisateur + home                     | `sudo userdel -r paul`                   |
| `passwd`          | Changer mot de passe                             | `passwd paul`                            |

## 5. Processus

| Commande          | Description                                      | Exemple                              |
|-------------------|--------------------------------------------------|--------------------------------------|
| `ps aux`          | Voir tous les processus                          | `ps aux \| grep python`              |
| `top` / `htop`    | Moniteur interactif                              | `htop`                               |
| `kill`            | Tuer par PID                                     | `kill -9 1234`                       |
| `pkill`           | Tuer par nom                                     | `pkill firefox`                     |
| `pgrep`           | Trouver PID                                      | `pgrep nginx`                        |
| `jobs` / `fg` / `bg` | Gérer jobs en arrière-plan                    | Ctrl+Z → `bg`                        |

## 6. Réseau

| Commande          | Description                                      | Exemple                                  |
|-------------------|--------------------------------------------------|------------------------------------------|
| `ip a`            | Interfaces réseau                                | `ip a`                                   |
| `ping`            | Tester connectivité                              | `ping 8.8.8.8`                           |
| `curl`            | Requêtes HTTP                                    | `curl -I https://x.com`                  |
| `wget`            | Télécharger                                      | `wget https://site.com/fichier`          |
| `ss`              | Connexions actives                               | `ss -tuln`                               |
| `dig`             | Interroger DNS                                   | `dig google.com`                         |

## 7. SSH (les plus utiles)

| Commande                          | Description                                        | Exemple                                      |
|-----------------------------------|----------------------------------------------------|----------------------------------------------|
| `ssh user@ip`                     | Connexion                                          | `ssh root@51.15.123.45`                      |
| `ssh -p 2222`                     | Port personnalisé                                  | `ssh -p 2222 user@serveur`                   |
| `ssh-keygen -t ed25519`           | Générer clé moderne                                | `ssh-keygen -t ed25519 -C "mon pc"`          |
| `ssh-copy-id user@ip`             | Envoyer clé publique                               | `ssh-copy-id user@serveur`                   |
| `scp`                             | Copie sécurisée                                    | `scp fichier user@ip:/tmp/`                  |
| `rsync -avz`                      | Synchro performante                                | `rsync -avz /local/ user@ip:/dest/`          |

## 8. Gestion paquets (Debian/Ubuntu)

| Commande                  | Description                                      |
|---------------------------|--------------------------------------------------|
| `apt update`              | Rafraîchir liste paquets                         |
| `apt upgrade`             | Mettre à jour tout                               |
| `apt install paquet`      | Installer                                        |
| `apt remove paquet`       | Désinstaller                                     |
| `apt autoremove`          | Nettoyer paquets inutiles                        |
| `apt search mot`          | Rechercher                                       |

## 9. Système & divers

| Commande              | Description                                      | Exemple                          |
|-----------------------|--------------------------------------------------|----------------------------------|
| `df -h`               | Espace disque                                    | `df -h`                          |
| `free -h`             | Mémoire                                          | `free -h`                        |
| `uptime`              | Temps activité + charge                          | `uptime`                         |
| `reboot`              | Redémarrer                                       | `sudo reboot`                    |
| `shutdown -h now`     | Éteindre immédiatement                           | `sudo shutdown -h now`           |
| `history`             | Historique commandes                             | `history \| grep git`             |
| `alias`               | Créer raccourci                                  | `alias ll='ls -la'`              |

