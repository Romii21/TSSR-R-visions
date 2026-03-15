# 📘 Fiche 2 — Utilisateurs, Groupes & Permissions
### Formation TSSR | CP3 — Exploiter des serveurs Linux

---

## 1. Les fichiers système des utilisateurs

### /etc/passwd — Liste des comptes

```
alice : x : 1000 : 1000 : Alice Martin : /home/alice : /bin/bash
  1    2    3      4          5               6              7
```

| Champ | Contenu |
|-------|---------|
| 1 — Nom | Login de l'utilisateur |
| 2 — Mot de passe | Toujours `x` (stocké dans `/etc/shadow`) |
| 3 — UID | Identifiant unique utilisateur |
| 4 — GID | Identifiant du groupe principal |
| 5 — GECOS | Commentaire (nom complet, etc.) |
| 6 — Home | Répertoire personnel |
| 7 — Shell | Shell par défaut (`/bin/bash`, `/bin/sh`, `/usr/sbin/nologin`) |

### /etc/shadow — Mots de passe hashés (root seulement)

```
alice : $6$salt$hash... : 19000 : 0 : 99999 : 7 : : :
  1          2              3    4    5      6  7  8  9
```

Champs importants : nom (1), hash du mdp (2), date dernier changement (3), âge max (5).

### /etc/group — Groupes

```
devops:x:1001:alice,bob
  1    2   3     4
```
Nom (1), mdp (2, inutilisé), GID (3), membres (4).

### Convention des UID

| UID | Catégorie |
|-----|-----------|
| **0** | root — superadministrateur |
| **1 – 999** | Comptes système (`www-data`, `mysql`, `sshd`…) |
| **1000+** | Utilisateurs humains normaux |

---

## 2. Gestion des utilisateurs — Commandes

```bash
# Créer un utilisateur (avec home automatique)
adduser thomas               # Interactif, recommandé Debian/Ubuntu
useradd -m -s /bin/bash thomas   # Bas niveau, toutes distros

# Supprimer
deluser --remove-home thomas     # Supprime user + home
userdel -r thomas                # Équivalent bas niveau

# Modifier
usermod -aG sudo thomas          # Ajouter au groupe sudo SANS toucher aux autres
usermod -s /bin/bash thomas      # Changer le shell
usermod -d /home/nouveau thomas  # Changer le home

# Mot de passe
passwd thomas                    # Changer le mot de passe
passwd -l thomas                 # Verrouiller le compte
passwd -u thomas                 # Déverrouiller

# Informations
id thomas                        # UID, GID, groupes
whoami                           # Utilisateur courant
groups thomas                    # Groupes de thomas
```

> ⚠️ `usermod -G sudo thomas` SANS `-a` écrase tous les groupes existants !
> Toujours utiliser `-aG` pour ajouter sans écraser.

---

## 3. Élévation de privilèges

### su vs sudo

| Critère | `su -` | `sudo commande` |
|---------|--------|----------------|
| **Mécanisme** | Ouvre une session complète en tant que root | Exécute une commande en root |
| **Mot de passe** | Demande le mdp **root** | Demande son **propre** mdp |
| **Traçabilité** | Non tracé | ✅ Tracé dans `/var/log/auth.log` |
| **Usage prod** | ❌ Déconseillé | ✅ Recommandé |

```bash
sudo commande                # Exécuter en tant que root
sudo -u alice commande       # Exécuter en tant qu'alice
sudo -l                      # Lister ses propres droits sudo
sudo -l -U thomas            # Voir les droits sudo de thomas (root)
```

Configuration : **`/etc/sudoers`** (éditer avec `visudo` — vérifie la syntaxe)
```
thomas ALL=(ALL) /usr/bin/systemctl restart nginx   # Droit limité
alice  ALL=(ALL) NOPASSWD: ALL                       # Tout sans mdp (dangereux)
```

---

## 4. Droits et permissions — Modèle UGO

### Lecture d'un `ls -l`

```
-  rwx  r-x  ---   2   alice  devops  4096  Jan 10  script.sh
↑  ↑↑↑  ↑↑↑  ↑↑↑   ↑     ↑      ↑      ↑            ↑
│   │    │    │    │     │      │      │              └─ Nom du fichier
│   │    │    │    │     │      │      └──────────────── Taille
│   │    │    │    │     │      └─────────────────────── Groupe propriétaire
│   │    │    │    │     └────────────────────────────── Propriétaire
│   │    │    │    └──────────────────────────────────── Nb de liens
│   │    │    └───────────────────────────────────────── Droits AUTRES
│   │    └────────────────────────────────────────────── Droits GROUPE
│   └─────────────────────────────────────────────────── Droits PROPRIÉTAIRE
└─────────────────────────────────────────────────────── Type (- fichier, d dossier, l symlink)
```

### Les droits en détail

| Lettre | Valeur | Sur un fichier | Sur un dossier |
|--------|--------|----------------|----------------|
| `r` | 4 | Lire le contenu | Lister les fichiers |
| `w` | 2 | Modifier / supprimer | Créer / supprimer des fichiers |
| `x` | 1 | Exécuter | Traverser (accéder) |
| `-` | 0 | Droit absent | Droit absent |

### Notation numérique (octal)

```
rwxr-xr--
 7   5   4

rwx = 4+2+1 = 7
r-x = 4+0+1 = 5
r-- = 4+0+0 = 4
```

### Commandes chmod, chown, chgrp

```bash
# chmod — changer les droits
chmod 755 script.sh          # rwxr-xr-x
chmod 640 fichier.conf       # rw-r-----
chmod +x script.sh           # Ajouter exécution à tous
chmod -w fichier.txt         # Retirer écriture à tous
chmod u+x,g-w script.sh      # Ajouter x au proprio, retirer w au groupe

# chown — changer le propriétaire
chown alice fichier.txt           # Changer le propriétaire
chown alice:marketing fichier.txt # Changer propriétaire ET groupe
chown -R alice:web /var/www/      # Récursif

# chgrp — changer le groupe uniquement
chgrp rh contrat.pdf
```

---

## 5. Bits spéciaux

### SUID (Set User ID) — bit s sur le propriétaire

```bash
ls -l /usr/bin/passwd
-rwsr-xr-x 1 root root ... /usr/bin/passwd
    ↑
    s = SUID actif
```

Effet : le programme s'exécute avec les droits de son **propriétaire** (root) peu importe qui le lance. Utilisé pour `passwd` (modifier `/etc/shadow` requiert root).

```bash
chmod u+s fichier      # Activer SUID
chmod 4755 fichier     # SUID en notation octal (4xxx)
```

### SGID (Set Group ID) — bit s sur le groupe
Le fichier s'exécute avec les droits du **groupe propriétaire**. Sur un dossier : les nouveaux fichiers héritent du groupe du dossier.

### Sticky bit — bit t sur les autres
Sur un dossier (`/tmp`) : seul le propriétaire peut supprimer ses propres fichiers.
```bash
chmod +t /tmp           # Sticky bit
```

---

## 6. ACL Linux (Access Control List)

> ⚠️ ACL Linux ≠ ACL réseau (firewall). Ce sont deux concepts totalement différents.

Les ACL permettent des droits **plus granulaires qu'UGO** : définir des droits pour n'importe quel utilisateur ou groupe sur un fichier, sans le changer de groupe.

### Quand utiliser les ACL ?

Alice possède un fichier. Bob (pas dans son groupe) a besoin d'un accès lecture seule. UGO ne peut pas gérer ça proprement → ACL.

```bash
# Donner des droits à un utilisateur spécifique
setfacl -m u:bob:r-- rapport.txt            # Bob peut lire
setfacl -m u:stagiaire:rw rapport.txt       # Stagiaire peut lire+écrire

# Donner des droits à un groupe
setfacl -m g:comptabilite:r-x /opt/appli/

# Supprimer une entrée ACL
setfacl -x u:bob rapport.txt

# Voir les ACL
getfacl rapport.txt

# Supprimer toutes les ACL
setfacl -b rapport.txt
```

Un fichier avec ACL a un `+` à la fin dans `ls -l` :
```
-rw-rw-r--+ 1 alice alice 512 ... rapport.txt
                                             ↑
                                         ACL active
```

---

## 7. Points clés à retenir

```
✅ /etc/passwd   →  7 champs, mdp = x
✅ /etc/shadow   →  hash des mdp, root uniquement
✅ UID 0         →  root
✅ UID 1-999     →  comptes système
✅ UID 1000+     →  utilisateurs humains
✅ usermod -aG   →  TOUJOURS -a pour ne pas écraser
✅ sudo          →  tracé, recommandé en prod
✅ chmod 754     →  rwxr-xr--
✅ r=4, w=2, x=1 →  additionner pour la valeur
✅ SUID          →  exécuter avec droits du propriétaire (passwd)
✅ ACL Linux     →  setfacl/getfacl (≠ ACL firewall !)
```
