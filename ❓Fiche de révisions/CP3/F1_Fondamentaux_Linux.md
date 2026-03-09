# 📘 Fiche 1 — Fondamentaux du système Linux
### Formation TSSR | CP3 — Exploiter des serveurs Linux

---

## 1. Philosophie Linux

### "Tout est fichier"
Linux représente **tout** sous forme de fichiers accessibles dans l'arborescence :

| Élément | Localisation | Exemple |
|---------|-------------|---------|
| Fichiers de données | Partout | `/home/alice/rapport.txt` |
| Périphériques | `/dev/` | `/dev/sda` (disque), `/dev/null` (poubelle) |
| Processus en cours | `/proc/` | `/proc/1234/status` |
| Infos matérielles | `/sys/` | `/sys/class/net/` |

> ⚠️ Ce n'est pas "tout est modifiable" — c'est "tout est accessible comme un fichier"

---

## 2. Arborescence Linux (à connaître par cœur)

```
/                       ← Racine unique du système
├── bin/                ← Commandes essentielles (ls, cp, cat, chmod...)
├── sbin/               ← Commandes d'administration système (root)
├── boot/               ← Noyau Linux + fichiers de démarrage (GRUB)
├── dev/                ← Pseudo-fichiers des périphériques
├── etc/                ← Fichiers de CONFIGURATION du système et services
├── home/               ← Répertoires personnels des utilisateurs
│   └── alice/          ← Home de l'utilisatrice alice
├── lib/                ← Bibliothèques partagées
├── opt/                ← Logiciels tiers (hors gestionnaire de paquets)
├── proc/               ← Pseudo-fichiers des processus (infos noyau)
├── root/               ← Home de l'utilisateur root
├── tmp/                ← Fichiers temporaires (vidé au reboot)
├── usr/                ← Applications et utilitaires utilisateurs
│   ├── bin/            ← Commandes non-essentielles
│   └── local/          ← Logiciels compilés manuellement
└── var/                ← Données variables
    ├── log/            ← Logs système
    ├── www/            ← Fichiers web
    └── lib/            ← Bases de données des applications
```

---

## 3. Navigation dans l'arborescence

### Chemin absolu vs chemin relatif

| Type | Définition | Exemple |
|------|-----------|---------|
| **Absolu** | Part toujours de `/` | `/home/alice/documents/rapport.txt` |
| **Relatif** | Part du répertoire courant | `documents/rapport.txt` (si on est dans `/home/alice/`) |

### Commandes essentielles

```bash
pwd                   # Afficher le répertoire courant (Print Working Directory)
cd /etc/nginx         # Aller dans un répertoire (chemin absolu)
cd documents          # Aller dans un sous-dossier (chemin relatif)
cd ~                  # Aller dans son répertoire personnel
cd                    # Idem (sans argument = home)
cd ..                 # Remonter d'un niveau
cd -                  # Retourner au répertoire précédent
ls -la                # Lister avec détails (droits, propriétaire, taille)
```

---

## 4. Liens physiques et liens symboliques

### Comparaison

| Critère | Hard link (lien physique) | Symlink (lien symbolique) |
|---------|--------------------------|--------------------------|
| **Mécanisme** | Pointe vers le même **inode** | Pointe vers le **chemin** |
| **Si l'original est supprimé** | Le lien continue de fonctionner | Le lien devient **cassé** |
| **Entre partitions** | ❌ Impossible | ✅ Possible |
| **Dossiers** | ❌ Interdit (généralement) | ✅ Possible |
| **Analogie** | Copie miroir en temps réel | Raccourci Windows |

```bash
ln fichier.txt hardlink.txt         # Créer un hard link
ln -s fichier.txt symlink.txt       # Créer un lien symbolique
ls -la                              # Voir les liens (l = symlink)
```

> 💡 Un symlink cassé = pointeur vers un fichier qui n'existe plus. Il apparaît en rouge dans `ls`.

---

## 5. Le fichier /etc/fstab

`/etc/fstab` est le fichier de configuration des **montages automatiques au démarrage**.

### Structure d'une ligne (6 champs)

```
UUID=a1b2-c3d4   /home   ext4   defaults   0   2
     [1]         [2]     [3]      [4]      [5] [6]
```

| Champ | Description | Exemple |
|-------|-------------|---------|
| 1 — Périphérique | Identifiant du disque/partition | `UUID=xxxx` ou `/dev/sda1` |
| 2 — Point de montage | Où il sera accessible | `/`, `/home`, `/mnt/data` |
| 3 — Type FS | Système de fichiers | `ext4`, `xfs`, `swap`, `vfat` |
| 4 — Options | Options de montage | `defaults`, `ro`, `noatime` |
| 5 — Dump | Sauvegarde (0 = non) | `0` |
| 6 — Pass | Ordre vérification fsck | `1` (racine), `2` (autres), `0` (aucun) |

```bash
# Toujours préférer l'UUID plutôt que /dev/sda1 :
# → L'UUID est stable (ne change pas si on branche un disque supplémentaire)
# → /dev/sda1 peut changer selon l'ordre de détection

blkid                  # Afficher les UUID de toutes les partitions
mount -a               # Monter tout ce qui est dans fstab
mount /dev/sdb1 /mnt   # Montage manuel temporaire
```

---

## 6. Commandes de gestion des fichiers

```bash
# Navigation et listing
ls -l fichier.txt            # Droits, propriétaire, taille
ls -la                       # Inclut les fichiers cachés (.)
find /var/log -name "*.log"  # Chercher des fichiers

# Manipulation
cp source destination        # Copier
mv source destination        # Déplacer / renommer
rm fichier.txt               # Supprimer un fichier
rm -r dossier/               # Supprimer un dossier (récursif)
mkdir -p /opt/app/data       # Créer répertoire + parents si nécessaire

# Affichage de contenu
cat fichier.txt              # Afficher tout
less fichier.txt             # Afficher page par page
head -20 fichier.txt         # 20 premières lignes
tail -f fichier.txt          # Suivre en temps réel
grep "erreur" fichier.txt    # Chercher un motif

# Archivage
tar -czf archive.tar.gz dossier/    # Compresser
tar -xzf archive.tar.gz             # Décompresser
```

---

## 7. Points clés à retenir

```
✅ Racine unique /  →  tout part de là
✅ /etc            →  configuration
✅ /var/log        →  logs
✅ /proc           →  processus en cours (pseudo-fichiers)
✅ /dev            →  périphériques (pseudo-fichiers)
✅ UUID > /dev/sda →  stable pour fstab
✅ Hard link       →  même inode, survit à la suppression de l'original
✅ Symlink         →  raccourci, peut devenir cassé
✅ pwd             →  où suis-je ?
✅ cd ~            →  retour au home
```
