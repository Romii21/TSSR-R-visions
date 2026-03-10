# ✅ CORRECTION — CP3 : Exploiter des serveurs Linux
### Formation TSSR | Correction détaillée

> **Légende :** ✅ Bonne réponse | ⚠️ Partiellement juste | ❌ Incorrecte ou incomplète | ⬜ Sans réponse

---

## 🟢 Partie 1 – Fondamentaux du système Linux

---

**Q1.** Différence entre chemin absolu et chemin relatif ?

> **Ton niveau : ✅ Bonne réponse**

**Correction :**
Tu as la bonne idée. Pour compléter :
- **Chemin absolu** : part toujours de la racine `/`. Ex : `/home/alice/documents/rapport.txt`
- **Chemin relatif** : part de ton emplacement actuel. Ex : si tu es dans `/home/alice/`, tu écris `documents/rapport.txt` ou `./documents/rapport.txt`

---

**Q2.** Citez 6 répertoires de l'arborescence Linux et leur rôle

> **Ton niveau : ⚠️ Partiellement juste — les noms sont corrects mais les rôles manquent**

**Correction :**

| Répertoire | Rôle |
|-----------|------|
| `/etc` | Fichiers de configuration du système et des services |
| `/home` | Répertoires personnels des utilisateurs (ex: `/home/alice`) |
| `/opt` | Logiciels tiers installés manuellement (hors gestionnaire de paquets) |
| `/var` | Données variables : logs, bases de données, emails (`/var/log`) |
| `/bin` | Commandes essentielles pour tous les utilisateurs (`ls`, `cp`, `cat`…) |
| `/usr` | Applications et utilitaires pour les utilisateurs |
| `/tmp` | Fichiers temporaires (vidé au redémarrage) |
| `/dev` | Pseudo-fichiers représentant les périphériques (`/dev/sda`, `/dev/null`) |
| `/proc` | Pseudo-fichiers représentant les processus en cours et infos noyau |

---

**Q3.** Philosophie "tout est fichier" — deux exemples

> **Ton niveau : ❌ Hors sujet**

**Correction :**
"Tout est fichier" ne veut **pas** dire "tout est modifiable". Cela signifie que Linux **représente tout** (périphériques, processus, sockets…) sous forme de fichiers accessibles dans l'arborescence.

- **Périphérique** : Un disque dur est accessible via `/dev/sda`. Tu peux y lire/écrire comme un fichier.
- **Processus** : Le processus PID 1234 est visible dans `/proc/1234/`. Tu peux lire son état, ses fichiers ouverts, etc.

Avantage : on utilise les mêmes outils (`cat`, `echo`, `read`…) pour tout manipuler.

---

**Q4.** Commande pour afficher le répertoire courant ? Se déplacer vers le répertoire personnel ?

> **Ton niveau : ⚠️ Incomplet — tu donnes `~` mais pas `pwd`**

**Correction :**
- **Afficher le répertoire courant** : `pwd` (Print Working Directory)
- **Aller dans le répertoire personnel** :
  - `cd ~` — utilise la variable `~` (alias du home)
  - `cd` (sans argument) — va directement dans le home
  - `cd $HOME` — équivalent, via la variable d'environnement

---

**Q5.** Hard link vs lien symbolique ? Quand un symlink est-il "cassé" ?

> **Ton niveau : ⚠️ Partiellement juste — la logique est bonne mais imprécise**

**Correction :**
- **Hard link (lien physique)** : pointe directement vers le même **inode** (emplacement physique sur le disque) que le fichier d'origine. Si l'original est supprimé, le hard link continue de fonctionner car les deux partagent le même inode. Limité à la même partition.
- **Lien symbolique (symlink)** : fichier spécial qui contient le **chemin** vers le fichier cible (comme un raccourci Windows). Fonctionne entre partitions.
- **Symlink "cassé"** : quand le fichier cible est supprimé ou déplacé. Le symlink pointe vers un chemin qui n'existe plus.

```bash
ln fichier.txt hardlink.txt        # Hard link
ln -s fichier.txt symlink.txt      # Lien symbolique
```

---

**Q6.** Rôle de `/etc/fstab` ?

> **Ton niveau : ⚠️ Idée correcte mais imprécise**

**Correction :**
`/etc/fstab` est un **fichier** (pas un dossier) qui liste les systèmes de fichiers à monter automatiquement au démarrage.

Chaque ligne contient 6 champs :
```
[PÉRIPHÉRIQUE]  [POINT DE MONTAGE]  [TYPE FS]  [OPTIONS]  [DUMP]  [PASS]
/dev/sda1       /                   ext4       defaults   0       1
UUID=xxxx       /home               ext4       defaults   0       2
```

---

## 🟢 Partie 2 – Gestion des utilisateurs et des groupes

---

**Q7.** Fichier listant les comptes — structure d'une ligne (7 champs)

> **Ton niveau : ⚠️ Fichier correct, mais les 7 champs sont incomplets**

**Correction :**
Fichier : `/etc/passwd`

Structure (7 champs séparés par `:`) :
```
alice:x:1000:1000:Alice Martin:/home/alice:/bin/bash
  1   2   3    4       5            6          7
```

1. **Nom d'utilisateur**
2. **Mot de passe** (toujours `x` → stocké dans `/etc/shadow`)
3. **UID** (User ID)
4. **GID** (Group ID principal)
5. **GECOS** (commentaire, souvent le nom complet)
6. **Répertoire personnel** (`/home/alice`)
7. **Shell par défaut** (`/bin/bash`)

---

**Q8.** Pourquoi les mots de passe ne sont pas dans `/etc/passwd` ? Où sont-ils ?

> **Ton niveau : ⬜ Sans réponse**

**Correction :**
Historiquement, les mots de passe étaient dans `/etc/passwd`. Ce fichier doit être lisible par tous (pour résoudre les noms d'utilisateurs). Stocker les mots de passe (même hashés) là rendait possible les attaques par dictionnaire.

Ils ont été déplacés dans **`/etc/shadow`**, accessible uniquement par root.

Structure de `/etc/shadow` (9 champs) :
```
alice:$6$salt$hash...:19000:0:99999:7:::
  1        2             3   4   5   6 7 8 9
```
1. Nom d'utilisateur
2. Mot de passe hashé (algorithme + sel + hash)
3. Date du dernier changement
4. Âge minimum
5. Âge maximum
6. Délai d'avertissement
7/8/9. Désactivation, expiration, réservé

---

**Q9.** Commande pour créer `thomas` avec son répertoire personnel automatiquement ?

> **Ton niveau : ✅ Bonne réponse**

**Correction :**
`adduser thomas` ✅ — crée l'utilisateur, le répertoire `/home/thomas`, et demande le mot de passe interactivement.

On peut aussi utiliser : `useradd -m thomas` (option `-m` = créer le home) mais `adduser` est plus convivial sur Debian/Ubuntu.

---

**Q10.** Ajouter `thomas` au groupe `sudo` sans modifier ses autres appartenances ?

> **Ton niveau : ✅ Bonne réponse**

**Correction :**
`usermod -aG sudo thomas` ✅

- `-a` = **append** (ajouter sans retirer les autres groupes)
- `-G` = groupes secondaires
- Sans `-a`, cette commande **remplacerait** tous les groupes secondaires existants — piège classique !

---

**Q11.** Différence entre `su -` et `sudo` ?

> **Ton niveau : ✅ Bonne réponse, bien argumentée**

**Correction :**
Tu as raison. Pour être plus précis :
- **`su -`** : ouvre une nouvelle session en tant que root (ou autre utilisateur), charge l'environnement complet du compte cible. Nécessite le mot de passe **root**.
- **`sudo commande`** : exécute une commande spécifique avec les privilèges root, en utilisant son **propre mot de passe**. Actions tracées dans les logs.

En production : `sudo` est préféré car il est **auditable** (qui a fait quoi, quand) et respecte le principe du moindre privilège.

---

**Q12.** Supprimer un utilisateur et son répertoire personnel en une commande ?

> **Ton niveau : ⚠️ Commande correcte mais incomplète**

**Correction :**
`deluser --remove-home thomas` (Debian/Ubuntu)

ou

`userdel -r thomas` (commande bas niveau, toutes distributions)

L'option `-r` (ou `--remove-home`) est indispensable pour supprimer `/home/thomas`. Sans elle, le répertoire reste.

---

**Q13.** Convention Linux pour les UID ?

> **Ton niveau : ⬜ Sans réponse**

**Correction :**

| UID | Catégorie |
|-----|-----------|
| **0** | root (superadministrateur) |
| **1 – 999** | Comptes système/services (`www-data`, `mysql`, `nobody`…) |
| **1000+** | Utilisateurs normaux (humains) |

La limite entre système et normal peut varier selon les distributions (certaines commencent les users à 500).

---

## 🟢 Partie 3 – Droits et permissions

---

**Q14.** Analyse de `-rwxr-x--- 2 alice devops 4096 Jan 10 09:00 script.sh`

> **Ton niveau : ⚠️ Partiellement juste — quelques erreurs**

**Correction :**
```
- rwx r-x --- 2 alice devops 4096 Jan 10 09:00 script.sh
│  │   │   │
│  │   │   └── Autres (other) : --- = aucun droit
│  │   └────── Groupe devops  : r-x = lecture + exécution (pas écriture)
│  └────────── Propriétaire   : rwx = lecture + écriture + exécution
└───────────── Type : - = fichier ordinaire (d = dossier)
```

- **Type** : fichier ordinaire (pas "script" comme type)
- **Propriétaire** : alice, avec tous les droits (rwx) ✅
- **Groupe devops** : lecture + exécution seulement (pas d'écriture) ✅
- **Autres** : **aucun droit** (tu as oublié de le mentionner)
- **`2`** = nombre de liens physiques

---

**Q15.** Donner les droits `rwxr-xr--` en notation numérique à `deploy.sh` ?

> **Ton niveau : ✅ Bonne réponse**

**Correction :**
`chmod 754 deploy.sh` ✅

Calcul :
- `rwx` = 4+2+1 = **7**
- `r-x` = 4+0+1 = **5**
- `r--` = 4+0+0 = **4**

---

**Q16.** Expliquer `chmod +x`, `chown bob:marketing`, `chgrp rh contrat.pdf`

> **Ton niveau : ⚠️ Erreur sur le premier, incomplet sur le troisième**

**Correction :**
- `chmod +x script.sh` : ajoute le droit d'**exécution** (pas d'écriture — tu as fait une confusion avec `+w`)
- `chown bob:marketing fichier.txt` : change le **propriétaire** en `bob` et le **groupe** en `marketing` ✅
- `chgrp rh contrat.pdf` : change uniquement le **groupe** du fichier en `rh`

---

**Q17.** Qu'est-ce qu'une ACL Linux ?

> **Ton niveau : ❌ Hors sujet total — tu parles d'ACL réseau (firewall)**

**Correction :**
Il existe deux types d'ACL très différents :

- **ACL réseau** (ce que tu décris) : règles sur les routeurs/firewalls pour filtrer le trafic → **hors sujet ici**
- **ACL Linux (Access Control List)** : mécanisme de permission **plus fin que UGO** (User-Group-Other) permettant de définir des droits pour des utilisateurs/groupes spécifiques sur un fichier.

**Pourquoi plus flexible que UGO ?**
UGO ne permet de définir des droits que pour 3 entités (propriétaire, 1 groupe, les autres). Avec les ACL, on peut donner des droits à n'importe quel utilisateur ou groupe spécifique.

**Cas d'usage** : Alice possède un fichier. Bob (hors du groupe) a besoin d'un accès en lecture seul. Sans ACL, impossible. Avec ACL :
```bash
setfacl -m u:bob:r-- rapport.txt
getfacl rapport.txt    # Vérifier
```

---

**Q18.** Donner les droits `rw` à `stagiaire` sur `rapport.txt` via ACL ?

> **Ton niveau : ⬜ Sans réponse**

**Correction :**
```bash
setfacl -m u:stagiaire:rw rapport.txt
```
- `setfacl` = set file ACL
- `-m` = modifier
- `u:stagiaire:rw` = user:nom:droits

---

**Q19.** Qu'est-ce que le bit SUID ?

> **Ton niveau : ⬜ Sans réponse**

**Correction :**
Le **bit SUID** (Set User ID) sur un exécutable fait qu'il s'exécute avec les droits de son **propriétaire** plutôt que ceux de l'utilisateur qui le lance.

**Exemple concret : `passwd`**
```bash
ls -l /usr/bin/passwd
-rwsr-xr-x 1 root root ... /usr/bin/passwd
       ↑
       s = bit SUID activé
```
Un utilisateur normal peut changer son mot de passe avec `passwd`. Or modifier `/etc/shadow` requiert les droits root. Le bit SUID permet à `passwd` de s'exécuter **en tant que root** le temps de l'opération.

---

## 🟡 Partie 4 – Gestion des processus

---

**Q20.** Différence entre programme et processus ?

> **Ton niveau : ⬜ Sans réponse**

**Correction :**
- **Programme** : fichier exécutable statique stocké sur le disque (`/usr/bin/nginx`, `script.sh`…). C'est du code inerte.
- **Processus** : instance **en cours d'exécution** d'un programme. Un même programme peut générer plusieurs processus simultanément. Le processus a un PID, une mémoire allouée, des fichiers ouverts, un état.

---

**Q21.** PID 1 sous Linux moderne — rôle au démarrage ?

> **Ton niveau : ⬜ Sans réponse**

**Correction :**
Le **PID 1** est le premier processus lancé par le noyau Linux après le démarrage. Sur les distributions modernes, c'est **systemd**.

Son rôle :
- Démarrer tous les autres services (dépendances gérées)
- Gérer le démarrage parallèle des services (plus rapide que l'ancien `init`)
- Rester actif en permanence comme processus parent de tous les autres
- Gérer les signaux d'extinction/redémarrage du système

---

**Q22.** Commande pour afficher les processus en temps réel ? Touche kill ?

> **Ton niveau : ⚠️ Partiellement juste**

**Correction :**
- `btop` est un outil tiers (valide mais pas natif)
- La commande native standard est **`top`**
- Une meilleure alternative native : **`htop`** (plus lisible)
- Depuis `top` ou `htop` : la touche **`k`** permet d'envoyer un signal kill à un processus (tu entres le PID puis le signal)

---

**Q23.** Différence SIGTERM / SIGKILL ? Pourquoi éviter `-9` ?

> **Ton niveau : ⬜ Sans réponse**

**Correction :**
- **SIGTERM (signal 15)** : demande **poliment** au processus de s'arrêter. Le processus peut l'intercepter, sauvegarder ses données, fermer ses connexions proprement, puis s'arrêter.
- **SIGKILL (signal 9)** : force l'arrêt **immédiat** par le noyau. Le processus ne peut pas l'intercepter ni l'ignorer.

**Pourquoi éviter `-9` en premier recours ?**
Car le processus n'a pas le temps de nettoyer : fichiers temporaires laissés, connexions non fermées, données non sauvegardées, bases de données corrompues potentiellement.

Bonne pratique : `kill PID` (SIGTERM) → attendre → si refus → `kill -9 PID`

---

**Q24.** Lancer une commande en arrière-plan ? La ramener au premier plan ?

> **Ton niveau : ❌ Incorrect**

**Correction :**
- **Lancer en arrière-plan** : ajouter `&` à la fin de la commande
  ```bash
  long_script.sh &
  ```
- **Lister les jobs** : `jobs`
- **Ramener au premier plan** : `fg` (foreground) ou `fg %1` pour le job n°1
- **Mettre en pause** (suspendre) un processus en cours : `Ctrl+Z`
- **Reprendre en arrière-plan** : `bg`

---

**Q25.** Commande pour qu'un processus continue après fermeture du terminal SSH ?

> **Ton niveau : ⬜ Sans réponse**

**Correction :**
- **`nohup commande &`** : ignore le signal HUP (hangup) envoyé à la fermeture du terminal
- **`screen`** ou **`tmux`** : multiplexeurs de terminal — permettent de détacher/rattacher une session
- **`disown`** : détache un job déjà lancé du terminal courant

Exemple :
```bash
nohup python3 mon_script.py > output.log 2>&1 &
```

---

**Q26.** Répertoire des infos sur les processus en cours ?

> **Ton niveau : ⬜ Sans réponse**

**Correction :**
Le répertoire **`/proc`** contient un sous-dossier pour chaque processus actif, nommé par son PID.

Exemple pour le PID 1234 :
- `/proc/1234/status` → état du processus (nom, PID, PPID, état)
- `/proc/1234/cmdline` → commande utilisée pour lancer le processus
- `/proc/1234/fd/` → liste des descripteurs de fichiers ouverts
- `/proc/1234/mem` → mémoire du processus

---

## 🟡 Partie 5 – Services et systemd

---

**Q27.** Différence service / démon ? Deux démons classiques ?

> **Ton niveau : ⬜ Sans réponse**

**Correction :**
- **Démon (daemon)** : processus qui tourne en **arrière-plan** en permanence, sans interaction directe avec l'utilisateur. Convention de nommage : finit souvent par `d` (`sshd`, `httpd`, `crond`).
- **Service** : terme plus générique qui désigne un démon géré par le système (via systemd).

Deux démons classiques d'un serveur :
- **`sshd`** : serveur SSH (accès distant sécurisé)
- **`nginx`** ou **`apache2`** : serveur web

---

**Q28.** Commandes systemctl

> **Ton niveau : ✅ Bonne réponse**

**Correction :**
```bash
systemctl start nginx       # Démarrer
systemctl stop nginx        # Arrêter
systemctl restart nginx     # Redémarrer
systemctl status nginx      # Vérifier l'état
systemctl enable nginx      # Activer au démarrage
systemctl disable nginx     # Désactiver du démarrage
systemctl reload nginx      # Recharger la config sans redémarrer (si supporté)
```

---

**Q29.** `systemctl enable` sans `systemctl start` ?

> **Ton niveau : ⚠️ L'idée est bonne mais mal formulée**

**Correction :**
`enable` et `start` sont deux actions **indépendantes** :
- `systemctl enable` : crée un lien symbolique pour **démarrer le service au prochain boot**. Le service **n'est pas démarré maintenant**.
- `systemctl start` : démarre le service **immédiatement**, sans rien changer au comportement au démarrage.

Donc `enable` seul = le service démarrera **au prochain reboot**, mais pas maintenant. C'est parfaitement valide. Pour les deux en une commande : `systemctl enable --now nginx`

---

**Q30.** Fichiers unit systemd — différence entre `/lib/systemd/system/` et `/etc/systemd/system/` ?

> **Ton niveau : ⬜ Sans réponse**

**Correction :**
- **`/lib/systemd/system/`** (ou `/usr/lib/systemd/system/`) : fichiers unit fournis par les **paquets installés**. Ne pas modifier directement (écrasés à la mise à jour).
- **`/etc/systemd/system/`** : fichiers unit **personnalisés par l'administrateur**. Prennent priorité sur `/lib/`. C'est ici qu'on surcharge ou crée des services custom.

---

**Q31.** Qu'est-ce qu'une target systemd ?

> **Ton niveau : ⬜ Sans réponse**

**Correction :**
Une **target** est l'équivalent des runlevels SysV. C'est un groupe de services à atteindre.

| Target | Équivalent | Description |
|--------|-----------|-------------|
| `multi-user.target` | runlevel 3 | Mode multi-utilisateurs, sans interface graphique (serveur) |
| `graphical.target` | runlevel 5 | Mode multi-utilisateurs + interface graphique |
| `rescue.target` | runlevel 1 | Mode maintenance (root seul) |
| `poweroff.target` | runlevel 0 | Extinction |
| `reboot.target` | runlevel 6 | Redémarrage |

---

## 🟡 Partie 6 – Gestion des paquets

---

**Q32.** `apt update` vs `apt upgrade` ?

> **Ton niveau : ✅ Bonne réponse**

**Correction :**
- `apt update` : met à jour la **liste des paquets disponibles** (les métadonnées des dépôts). N'installe rien.
- `apt upgrade` : **installe les nouvelles versions** des paquets déjà installés.

Il faut `update` avant `upgrade` pour connaître les dernières versions disponibles.

---

**Q33.** Installer nginx et vérifier ?

> **Ton niveau : ✅ Bonne réponse**

**Correction :**
```bash
apt update && apt install nginx    ✅
systemctl status nginx             ✅
```
On peut aussi vérifier avec : `nginx -v` (version) ou `curl localhost` (test HTTP).

---

**Q34.** Rechercher un paquet dont on ne connaît pas le nom exact ?

> **Ton niveau : ⬜ Sans réponse**

**Correction :**
```bash
apt search mot_clé          # Recherche dans les noms et descriptions
apt-cache search mot_clé    # Équivalent (plus bas niveau)
```
Exemple : `apt search "web server"` → liste nginx, apache2, lighttpd…

---

**Q35.** `apt remove` vs `apt purge` ?

> **Ton niveau : ✅ Bonne réponse**

**Correction :**
- `apt remove paquet` : désinstalle le paquet mais **conserve les fichiers de configuration** (utile si réinstallation prévue)
- `apt purge paquet` : désinstalle + **supprime tous les fichiers de configuration**

Quand utiliser `purge` ? Quand on veut une réinstallation propre, ou supprimer définitivement un service et sa configuration.

---

**Q36.** Où sont les dépôts APT ? Comment en ajouter un ?

> **Ton niveau : ⬜ Sans réponse**

**Correction :**
- Fichier principal : **`/etc/apt/sources.list`**
- Dépôts supplémentaires : **`/etc/apt/sources.list.d/`** (un fichier `.list` par dépôt)

Ajouter manuellement :
```bash
# Méthode moderne (recommandée)
add-apt-repository "deb http://archive.ubuntu.com/ubuntu focal main"
# Ou éditer directement
echo "deb http://dépôt.exemple.com/ubuntu focal main" >> /etc/apt/sources.list.d/custom.list
apt update   # Toujours mettre à jour après ajout
```

---

## 🟡 Partie 7 – Configuration réseau

---

**Q37.** Commande remplaçant `ifconfig` ?

> **Ton niveau : ✅ Bonne réponse**

**Correction :**
`ip a` (ou `ip addr show`) ✅ — fait partie du paquet `iproute2`, remplace `ifconfig` (paquet `net-tools` déprécié).

Autres commandes de la suite `ip` :
- `ip link` → interfaces réseau
- `ip route` → table de routage
- `ip neigh` → table ARP

---

**Q38.** Tester la joignabilité d'un hôte ? D'un port spécifique ?

> **Ton niveau : ⬜ Sans réponse**

**Correction :**
- **Joignabilité (couche réseau)** : `ping 192.168.1.1`
- **Port spécifique** :
  - `telnet 192.168.1.1 80` (basique)
  - `nc -zv 192.168.1.1 80` (netcat — plus propre)
  - `nmap -p 80 192.168.1.1` (scan de port)
  - `curl -v http://192.168.1.1` (test HTTP)

---

**Q39.** Fichiers de configuration réseau Netplan ? Format ?

> **Ton niveau : ⬜ Sans réponse**

**Correction :**
- **Emplacement** : `/etc/netplan/` (fichiers `.yaml`)
- **Format** : YAML

Exemple :
```yaml
network:
  version: 2
  ethernets:
    eth0:
      dhcp4: false
      addresses: [192.168.1.10/24]
      gateway4: 192.168.1.1
      nameservers:
        addresses: [8.8.8.8, 1.1.1.1]
```
Appliquer : `netplan apply`

---

**Q40.** Table de routage ? Que représente `default via` ?

> **Ton niveau : ✅ Bonne réponse**

**Correction :**
`ip route` ou `route -n` ✅

`default via 192.168.1.254` = la **passerelle par défaut** (gateway). Tout trafic destiné à une adresse inconnue dans la table de routage sera envoyé vers cette passerelle. C'est généralement l'adresse du routeur.

---

**Q41.** Lister les connexions actives et ports en écoute ?

> **Ton niveau : ⚠️ nmap n'est pas la bonne commande ici**

**Correction :**
`nmap 127.0.0.1` est un **scanner de ports externe** — il fonctionne mais ce n'est pas l'outil approprié pour lister les ports locaux.

Les bonnes commandes :
```bash
ss -tulnp           # Socket Statistics — remplace netstat (moderne)
netstat -tulnp      # Ancienne commande (paquet net-tools)
```
- `-t` = TCP, `-u` = UDP, `-l` = listening, `-n` = numérique, `-p` = processus

---

## 🟡 Partie 8 – SSH

---

**Q42.** Port SSH par défaut ? TCP ou UDP ?

> **Ton niveau : ✅ Bonne réponse**

**Correction :** Port **22**, protocole **TCP** ✅

---

**Q43.** Authentification par clé publique/privée SSH — fichier côté serveur ?

> **Ton niveau : ⬜ Sans réponse**

**Correction :**
Principe asymétrique :
1. L'utilisateur génère une **paire de clés** : clé privée (gardée secrète) + clé publique (partageable)
2. La clé publique est déposée sur le serveur dans **`~/.ssh/authorized_keys`**
3. Lors de la connexion, le serveur envoie un challenge chiffré avec la clé publique. Seul le détenteur de la clé privée peut le déchiffrer → authentification prouvée sans envoi de mot de passe

Fichier côté serveur : **`~/.ssh/authorized_keys`**

---

**Q44.** Générer une paire de clés ed25519 ? Copier la clé publique ?

> **Ton niveau : ⬜ Sans réponse**

**Correction :**
```bash
# Générer une paire de clés ed25519
ssh-keygen -t ed25519 -C "commentaire@exemple.com"
# → Génère ~/.ssh/id_ed25519 (privée) et ~/.ssh/id_ed25519.pub (publique)

# Copier la clé publique sur le serveur
ssh-copy-id -i ~/.ssh/id_ed25519.pub user@serveur
# → Ajoute la clé dans ~/.ssh/authorized_keys sur le serveur
```

---

**Q45.** Fichier de configuration du serveur SSH ? 4 directives de sécurité ?

> **Ton niveau : ⬜ Sans réponse**

**Correction :**
Fichier : **`/etc/ssh/sshd_config`**

4 directives essentielles en production :
```
PermitRootLogin no              # Interdire la connexion root directe
PasswordAuthentication no       # Forcer l'authentification par clé
Port 2222                       # Changer le port (sécurité par obscurité)
MaxAuthTries 3                  # Limiter les tentatives
AllowUsers alice bob            # Whitelist des utilisateurs autorisés
```
Redémarrer après modification : `systemctl restart sshd`

---

**Q46.** Copier un fichier via SSH ? Synchronisation incrémentale ?

> **Ton niveau : ✅ Bonne réponse sur scp**

**Correction :**
- `scp /chemin/fichier user@ip:/chemin/destination` ✅
- **Synchronisation incrémentale** : `rsync -avz /source/ user@ip:/destination/`
  - `-a` = archive (récursif + préserve droits)
  - `-v` = verbose
  - `-z` = compression

`rsync` est bien supérieur à `scp` pour les sauvegardes car il ne retransfère que les fichiers modifiés.

---

**Q47.** Qu'est-ce que Fail2Ban ?

> **Ton niveau : ⬜ Sans réponse**

**Correction :**
**Fail2Ban** est un outil de protection contre les attaques **brute force**. Il surveille les fichiers de logs et **bannit automatiquement** les IP qui ont trop de tentatives d'authentification échouées.

- **Problème résolu** : attaques par dictionnaire/force brute sur SSH (et autres services)
- **Fichier de log SSH** : `/var/log/auth.log` (Debian/Ubuntu) ou `/var/log/secure` (RHEL/CentOS)
- **Action** : crée des règles iptables/nftables pour bloquer l'IP pendant une durée configurable

Configuration : `/etc/fail2ban/jail.conf` (ou `jail.local`)

---

## 🟠 Partie 9 – Journalisation

---

**Q48.** 4 fichiers de logs importants dans `/var/log/`

> **Ton niveau : ⬜ Sans réponse**

**Correction :**

| Fichier | Contenu |
|---------|---------|
| `/var/log/auth.log` | Authentifications, tentatives de connexion, sudo, su |
| `/var/log/syslog` | Messages système généraux (tous services) |
| `/var/log/kern.log` | Messages du noyau Linux |
| `/var/log/nginx/access.log` | Requêtes HTTP reçues par nginx |
| `/var/log/nginx/error.log` | Erreurs nginx |
| `/var/log/dpkg.log` | Installations/suppressions de paquets |

---

**Q49.** Différence rsyslog / journal systemd ?

> **Ton niveau : ⬜ Sans réponse**

**Correction :**
- **rsyslog** : daemon héritant de syslog. Stocke les logs en **texte clair** dans `/var/log/`. Compatible avec les serveurs syslog distants. Très configurable.
- **Journal systemd (journald)** : intégré à systemd, stocke en format **binaire** dans `/run/systemd/journal/`. Interrogé via `journalctl`. Plus structuré, indexé, filtrable.

Sur les distributions modernes : les deux coexistent. **journald** capte en premier, peut relayer vers **rsyslog** pour la persistance et l'envoi distant.

---

**Q50.** Suivre les logs en temps réel avec journalctl ? Filtrer par service ?

> **Ton niveau : ⬜ Sans réponse**

**Correction :**
```bash
journalctl -f                    # Suivre en temps réel (équivalent tail -f)
journalctl -u nginx              # Logs du service nginx uniquement
journalctl -u nginx -f           # Logs nginx en temps réel
journalctl -b                    # Logs depuis le dernier démarrage
journalctl --since "1 hour ago"  # Logs depuis 1h
journalctl -p err                # Filtrer par niveau d'erreur
```

---

**Q51.** Qu'est-ce que logrotate ?

> **Ton niveau : ⬜ Sans réponse**

**Correction :**
**logrotate** est l'outil standard qui gère la **rotation des fichiers de logs**. Il résout le problème de la croissance infinie des logs qui finiraient par remplir le disque.

- **Ce qu'il fait** : archive et compresse les anciens logs (ex: `syslog` → `syslog.1.gz`), supprime les archives trop anciennes, peut redémarrer les services après rotation.
- **Déclenchement** : via **cron** (tâche planifiée quotidienne dans `/etc/cron.daily/logrotate`)
- **Configuration** : `/etc/logrotate.conf` et `/etc/logrotate.d/` (un fichier par service)

---

**Q52.** Pourquoi centraliser les logs sur un serveur distant ?

> **Ton niveau : ✅ Bonne réponse — tu as la raison principale**

**Correction :**
Tu as raison sur le point principal. Pour compléter, les raisons de la centralisation :
1. **Disponibilité** : si la machine est compromise/tombée, les logs sont préservés ✅
2. **Sécurité** : un attaquant qui prend le contrôle d'une machine peut effacer les logs locaux — pas les logs centralisés
3. **Corrélation** : analyser des événements sur plusieurs machines en même temps (SIEM)
4. **Obligations légales** : RGPD, LPM imposent la conservation des logs

---

**Q53.** 8 niveaux de sévérité syslog (du plus au moins critique)

> **Ton niveau : ⬜ Sans réponse**

**Correction :**

| Code | Niveau | Description |
|------|--------|-------------|
| 0 | **Emergency** (emerg) | Système inutilisable |
| 1 | **Alert** (alert) | Intervention immédiate requise |
| 2 | **Critical** (crit) | Erreur critique |
| 3 | **Error** (err) | Erreur de fonctionnement |
| 4 | **Warning** (warn) | Avertissement |
| 5 | **Notice** (notice) | Événement normal notable |
| 6 | **Informational** (info) | Information |
| 7 | **Debug** (debug) | Messages de débogage |

---

## 🟠 Partie 10 – Stockage et systèmes de fichiers

---

**Q54.** Espace disque disponible par partition ? Taille d'un répertoire ?

> **Ton niveau : ⚠️ lsblk liste les disques mais ne montre pas l'espace disponible**

**Correction :**
- **Espace disque par partition** : `df -h` (disk free, `-h` = human readable)
- **Taille d'un répertoire** : `du -sh /chemin/du/dossier/` (`-s` = résumé, `-h` = lisible)
- `lsblk` = liste les périphériques blocs (disques, partitions) mais ne montre pas l'espace utilisé/libre

---

**Q55.** MBR vs GPT ?

> **Ton niveau : ⬜ Sans réponse**

**Correction :**

| Critère | MBR | GPT |
|---------|-----|-----|
| Norme | Ancienne (1983) | Moderne (UEFI) |
| Partitions | 4 primaires max | 128 partitions |
| Taille disque max | 2 To | 9,4 Zo (pratiquement illimité) |
| Redondance | Non | Oui (table de partition dupliquée) |
| Compatibilité | BIOS | UEFI (BIOS possible via CSM) |

**Recommandation** : **GPT** sur tous les systèmes modernes (disques > 2 To obligatoire).

---

**Q56.** Principe du LVM — les 3 niveaux PV, VG, LV

> **Ton niveau : ⬜ Sans réponse**

**Correction :**
Le **LVM (Logical Volume Manager)** est une couche d'abstraction qui permet de gérer le stockage de manière flexible (redimensionnement à chaud, agrégation de disques…).

Les 3 niveaux :
1. **PV (Physical Volume)** : disque ou partition physique initialisé pour LVM (`/dev/sda1`, `/dev/sdb`)
2. **VG (Volume Group)** : groupe qui agrège un ou plusieurs PV en un "pool" de stockage
3. **LV (Logical Volume)** : volume logique créé dans un VG, sur lequel on crée un système de fichiers et qu'on monte

```
Disques physiques → PV → VG (pool) → LV (partitions virtuelles) → Montage
```

---

**Q57.** RAID 0, 1, 5 et 6

> **Ton niveau : ⬜ Sans réponse**

**Correction :**

| RAID | Disques min | Redondance | Performance | Description |
|------|------------|-----------|------------|-------------|
| **0** | 2 | ❌ Aucune | ✅✅ Lecture+Écriture | Agrégation (striping) — si 1 disque tombe, tout est perdu |
| **1** | 2 | ✅ Miroir | ✅ Lecture / = Écriture | Miroir exact — survie à 1 panne |
| **5** | 3 | ✅ 1 disque | ✅ Lecture | Striping + parité répartie — survie à 1 panne |
| **6** | 4 | ✅✅ 2 disques | = Lecture | Striping + double parité — survie à 2 pannes |

---

**Q58.** Monter manuellement une partition ? Montage permanent ?

> **Ton niveau : ⬜ Sans réponse**

**Correction :**
```bash
# Montage manuel
mount /dev/sdb1 /mnt/data

# Montage permanent → ajouter dans /etc/fstab
UUID=xxxx-xxxx   /mnt/data   ext4   defaults   0   2
# Retrouver l'UUID avec : blkid /dev/sdb1
```

---

**Q59.** UUID de partition — pourquoi préférer à `/dev/sda1` dans fstab ?

> **Ton niveau : ⬜ Sans réponse**

**Correction :**
Un **UUID** (Universally Unique Identifier) est un identifiant unique attribué à chaque partition lors de sa création.

**Pourquoi préférer l'UUID ?**
Le nom `/dev/sda1` peut **changer** selon l'ordre de détection des disques au démarrage (ajout/retrait d'un disque, changement de port SATA). L'UUID, lui, est **immuable** et identifie toujours la même partition.

```bash
blkid           # Afficher les UUID de toutes les partitions
```

---

## 🟠 Partie 11 – Planification de tâches

---

**Q60.** Différence cron / at ?

> **Ton niveau : ⬜ Sans réponse**

**Correction :**
- **cron** : exécution **récurrente** selon un planning (toutes les nuits, chaque lundi…). Géré par le daemon `crond`.
- **at** : exécution **unique, différée** à un moment précis. Exemple : `at 22:00` → exécuter une fois ce soir.

---

**Q61.** Interpréter : `30 2 * * 1 /usr/bin/rsync -avz /data/ /backup/`

> **Ton niveau : ⬜ Sans réponse**

**Correction :**
Format cron : `minute heure jour_mois mois jour_semaine commande`

```
30  2  *  *  1
↓   ↓  ↓  ↓  ↓
30  2  *  *  lundi
```

- **Quand** : tous les **lundis à 2h30** du matin
- **Ce qu'elle fait** : synchronise le répertoire `/data/` vers `/backup/` en utilisant rsync en mode archive+verbose+compressé (typiquement une sauvegarde hebdomadaire)

---

**Q62.** Éditer la crontab ? Lister les tâches ?

> **Ton niveau : ⬜ Sans réponse**

**Correction :**
```bash
crontab -e       # Éditer la crontab de l'utilisateur courant
crontab -l       # Lister les tâches cron de l'utilisateur courant
crontab -u alice -l  # Lister les tâches d'un autre utilisateur (root)
```

---

**Q63.** Crontabs système — répertoires ?

> **Ton niveau : ⬜ Sans réponse**

**Correction :**
- **`/etc/crontab`** : crontab système globale (avec champ utilisateur supplémentaire)
- **`/etc/cron.d/`** : fichiers cron par application/service
- **`/etc/cron.hourly/`** : scripts exécutés toutes les heures
- **`/etc/cron.daily/`** : scripts exécutés tous les jours
- **`/etc/cron.weekly/`** : hebdomadaire
- **`/etc/cron.monthly/`** : mensuel

---

## 🔴 Partie 12 – Scripting Bash

---

**Q64.** Qu'est-ce que le shebang `#!/bin/bash` ?

> **Ton niveau : ⚠️ Partiellement juste**

**Correction :**
Le **shebang** (ou hashbang) indique au système quel **interpréteur** utiliser pour exécuter le script. Sans lui, le shell tenterait d'exécuter le script avec le shell courant (qui peut ne pas être bash).

- `#!/bin/bash` → interprété par bash
- `#!/usr/bin/env python3` → interprété par Python 3
- `#!/bin/sh` → shell POSIX basique

Ce n'est pas une "initialisation" — le script resterait un script bash même sans, mais son exécution directe (`./script.sh`) serait non déterministe.

---

**Q65.** Redirections Bash

> **Ton niveau : ⚠️ 2/4 bonnes réponses**

**Correction :**
- `commande > fichier.txt` : redirige la sortie standard (stdout) vers le fichier — **écrase** le contenu ✅
- `commande >> fichier.txt` : redirige stdout vers le fichier en **ajoutant** à la fin ✅
- `commande 2>&1` : redirige la sortie d'**erreur** (stderr, fd 2) vers stdout (fd 1) — pour capturer les deux ensembles ❌ (tu n'as pas répondu)
- `commande 2>/dev/null` : redirige stderr vers `/dev/null` (poubelle) — **supprime les messages d'erreur** ❌ (tu n'as pas répondu)

---

**Q66.** Structure d'un `if` Bash vérifiant si `/etc/passwd` existe

> **Ton niveau : ⚠️ Logique correcte mais syntaxe incorrecte**

**Correction :**
Ta syntaxe est incorrecte. Voici la bonne :
```bash
if [ -f /etc/passwd ]; then
    echo "Le fichier existe"
else
    echo "Le fichier n'existe pas"
fi
```
- `[ -f fichier ]` = test si le fichier existe et est un fichier ordinaire
- `[ -d dossier ]` = test si c'est un répertoire
- `[ -e chemin ]` = test si l'élément existe (fichier ou dossier)
- Ne pas oublier le `fi` pour fermer le bloc

---

**Q67.** Différence entre `$1`, `$@`, `$#` et `$?`

> **Ton niveau : ⚠️ Partiellement juste**

**Correction :**
- **`$1`** : premier argument passé au script ✅ (ex: `./script.sh Alice` → `$1` = "Alice")
- **`$@`** : **tous les arguments** (pas "le nom de l'argument") ❌ — ex: `./script.sh Alice Bob` → `$@` = "Alice Bob"
- **`$#`** : le **nombre** d'arguments ✅
- **`$?`** : le **code de retour** de la dernière commande (0 = succès, autre = erreur) ✅ — légèrement imprécis dans ta formulation
- Bonus : **`$0`** = nom du script lui-même

---

**Q68.** Boucle `for` parcourant les `.log` de `/var/log/`

> **Ton niveau : ⬜ Sans réponse**

**Correction :**
```bash
for fichier in /var/log/*.log; do
    echo "$fichier"
done
```

Ou avec `find` pour les sous-dossiers :
```bash
find /var/log/ -name "*.log" | while read fichier; do
    echo "$fichier"
done
```

---

**Q69.** Script de sauvegarde — 3 lignes

> **Ton niveau : ⬜ Sans réponse**

**Correction :**
```bash
# 1. Compresser /var/www/html/ avec la date du jour
tar -czf /backup/html_$(date +%Y%m%d).tar.gz /var/www/html/

# 2. Le déplacer en /backup/ est déjà fait ci-dessus (tar crée directement là)
# Si besoin de déplacer :
# mv /tmp/html_$(date +%Y%m%d).tar.gz /backup/

# 3. Supprimer les archives de plus de 30 jours
find /backup/ -name "html_*.tar.gz" -mtime +30 -delete
```

---

## 🔴 Partie 13 – Sécurité et supervision

---

**Q70.** Afficher les tentatives de connexion échouées ? Quel log ?

> **Ton niveau : ⬜ Sans réponse**

**Correction :**
```bash
# Consulter le fichier de log directement
grep "Failed password" /var/log/auth.log
grep "authentication failure" /var/log/auth.log

# Commande dédiée aux échecs de connexion
lastb               # Historique des connexions échouées

# Avec journalctl
journalctl -u sshd | grep "Failed"
```

Fichier de log SSH : **`/var/log/auth.log`** (Debian/Ubuntu)

---

**Q71.** `sudo` et sa configuration — vérifier les droits sans ouvrir le fichier ?

> **Ton niveau : ⚠️ Partiellement juste mais erreurs**

**Correction :**
- **sudo** = outil d'élévation de privilèges ✅
- **Configuration** : `/etc/sudoers` (et `/etc/sudoers.d/`) — **pas dans `/etc/`** directement, c'est le fichier `/etc/sudoers`
- **Vérifier les droits sudo d'un utilisateur** : `sudo -l -U thomas` (liste les droits de thomas)
  - Ta réponse (`passwd`) ne montre pas les droits sudo

---

**Q72.** Bonne pratique pour la connexion SSH root ?

> **Ton niveau : ✅ Bonne réponse**

**Correction :**
Interdire la connexion SSH directe en root ✅. La directive dans `/etc/ssh/sshd_config` :
```
PermitRootLogin no
```
En production, on se connecte avec un compte normal puis on utilise `sudo` pour les opérations privilégiées.

---

**Q73.** 3 indicateurs système à surveiller et commandes associées

> **Ton niveau : ⬜ Sans réponse**

**Correction :**

| Indicateur | Commandes |
|-----------|----------|
| **Charge CPU** | `top`, `htop`, `uptime`, `vmstat` |
| **Utilisation mémoire/RAM** | `free -h`, `htop`, `vmstat` |
| **Espace disque** | `df -h`, `du -sh /` |
| **Réseau (connexions/trafic)** | `ss -tulnp`, `iftop`, `nethogs` |
| **Processus gourmands** | `top` (tri par CPU/mémoire), `ps aux --sort=-%cpu` |

---

**Q74.** Principe du moindre privilège — deux exemples

> **Ton niveau : ⚠️ Définition trop vague**

**Correction :**
Le **principe du moindre privilège** : chaque utilisateur/service/processus ne doit disposer que des **droits strictement nécessaires** à l'accomplissement de sa tâche. Ni plus.

**Exemples concrets :**
1. Le service web `nginx` tourne sous l'utilisateur `www-data` (pas root) → s'il est compromis, l'attaquant n'a que les droits de `www-data`
2. Un utilisateur de support peut lire les logs mais ne peut pas modifier les fichiers système (`chmod 440 /var/log/auth.log`)
3. `sudo` avec droits limités : `alice ALL=(ALL) /usr/bin/systemctl restart nginx` → alice peut uniquement redémarrer nginx, rien d'autre

---

**Q75.** Diagnostic SSH — utilisateur ne peut plus se connecter

> **Ton niveau : ⬜ Sans réponse**

**Correction :**
Démarche de diagnostic du plus simple au plus technique :

1. **Vérifier la connectivité réseau** : `ping IP_du_serveur` → si pas de réponse, problème réseau en amont
2. **Vérifier que le port 22 est ouvert** : `nc -zv IP_serveur 22` ou `nmap -p 22 IP_serveur`
3. **Vérifier l'état du service SSH** : `systemctl status sshd` sur le serveur
4. **Vérifier les droits sur les fichiers de clés** : `~/.ssh/` doit être en `700`, `authorized_keys` en `600`
5. **Consulter les logs SSH** : `journalctl -u sshd` ou `tail -f /var/log/auth.log` → chercher le message d'erreur exact
6. **Vérifier si le compte est verrouillé** : `passwd -S username` ou `getent shadow username`
7. **Vérifier `/etc/sshd_config`** : directives `AllowUsers`, `DenyUsers`, `PermitRootLogin`
8. **Vérifier Fail2Ban** : `fail2ban-client status sshd` → l'IP est peut-être bannie
9. **Vérifier le pare-feu** : `iptables -L` ou `ufw status` → règle bloquant le port 22

---

## 📊 Bilan Global

| Partie | Thème | Résultat |
|--------|-------|----------|
| 1 | Fondamentaux | ⚠️ Bases présentes, précisions manquantes |
| 2 | Utilisateurs | ⚠️ Commandes OK, structure fichiers incomplète |
| 3 | Droits | ⚠️ Calcul chmod OK, ACL/SUID inconnus |
| 4 | Processus | ❌ Majorité sans réponse |
| 5 | systemd | ⚠️ Commandes OK, concepts incomplets |
| 6 | Paquets | ✅ Bonne maîtrise |
| 7 | Réseau | ⚠️ Bases OK, outils manquants |
| 8 | SSH | ⚠️ Port/clés partiels, config manquante |
| 9 | Logs | ❌ Majorité sans réponse |
| 10 | Stockage | ❌ Majorité sans réponse |
| 11 | Cron | ❌ Sans réponse |
| 12 | Bash | ⚠️ Logique OK, syntaxe incorrecte |
| 13 | Sécurité | ⚠️ Concepts OK, commandes manquantes |

### 🎯 Points forts
- Commandes systemctl ✅
- Gestion des paquets apt ✅
- `chmod` en notation numérique ✅
- Port SSH ✅

### 📌 Axes à retravailler en priorité
1. **Journalisation** — niveaux syslog, journalctl, logrotate (partie 9)
2. **Processus** — signaux, /proc, foreground/background (partie 4)
3. **Stockage** — LVM, RAID, UUID, fstab (partie 10)
4. **Scripting Bash** — syntaxe if/for, redirections (partie 12)
5. **ACL Linux** (≠ ACL réseau !) — setfacl, getfacl
