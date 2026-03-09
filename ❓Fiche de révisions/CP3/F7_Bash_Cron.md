# 📘 Fiche 7 — Scripting Bash & Planification
### Formation TSSR | CP3 — Exploiter des serveurs Linux

---

## 1. Structure d'un script Bash

```bash
#!/bin/bash
# ↑ Shebang : indique au système d'utiliser bash pour exécuter ce script

# Commentaire
echo "Bonjour $USER"           # Afficher un message
```

```bash
# Rendre le script exécutable et le lancer
chmod +x mon_script.sh
./mon_script.sh
```

> Le **shebang `#!/bin/bash`** détermine quel interpréteur utilise le script.
> Sans lui, l'exécution directe (`./script.sh`) peut utiliser n'importe quel shell.

---

## 2. Variables

```bash
# Déclaration (pas d'espaces autour du =)
nom="Alice"
age=25
fichier="/etc/passwd"

# Utilisation
echo "Bonjour $nom"
echo "Tu as $age ans"
echo "Fichier : ${fichier}"        # {} recommandé pour éviter les ambiguïtés

# Variables spéciales
$0          # Nom du script
$1, $2...   # Arguments positionnels (1er, 2ème argument...)
$@          # TOUS les arguments
$#          # NOMBRE d'arguments
$?          # Code de retour de la dernière commande (0=succès, ≠0=erreur)
$$          # PID du script lui-même
```

---

## 3. Redirections

```bash
commande > fichier.txt       # Redirige stdout → ÉCRASE le fichier
commande >> fichier.txt      # Redirige stdout → AJOUTE à la fin
commande 2> erreurs.txt      # Redirige stderr (erreurs) dans un fichier
commande 2>&1                # Redirige stderr VERS stdout (les deux ensemble)
commande > out.txt 2>&1      # Tout (stdout + stderr) dans le fichier
commande 2>/dev/null         # Jette les erreurs dans la poubelle
commande1 | commande2        # Pipe : sortie de cmd1 = entrée de cmd2

# /dev/null = trou noir, poubelle Linux
```

---

## 4. Tests et conditions

### Tests courants

```bash
# Fichiers
[ -f /etc/passwd ]       # Le fichier existe et est un fichier ordinaire
[ -d /home/alice ]       # Est un répertoire
[ -e /etc/nginx.conf ]   # Existe (fichier OU dossier)
[ -r fichier ]           # Est lisible
[ -w fichier ]           # Est modifiable
[ -x script.sh ]         # Est exécutable

# Comparaisons numériques
[ $a -eq $b ]    # Égal
[ $a -ne $b ]    # Différent
[ $a -lt $b ]    # Inférieur
[ $a -gt $b ]    # Supérieur
[ $a -le $b ]    # Inférieur ou égal

# Comparaisons de chaînes
[ "$a" = "$b" ]   # Égales
[ "$a" != "$b" ]  # Différentes
[ -z "$a" ]       # Chaîne vide
[ -n "$a" ]       # Chaîne non vide
```

### Structure if / elif / else

```bash
if [ -f /etc/passwd ]; then
    echo "Le fichier existe"
elif [ -d /etc/passwd ]; then
    echo "C'est un dossier"
else
    echo "N'existe pas"
fi
```

> ⚠️ Ne pas oublier : les **espaces** à l'intérieur des `[ ]`, le `;` avant `then`, et le `fi` à la fin.

---

## 5. Boucles

### Boucle for

```bash
# Itérer sur une liste
for fruit in pomme banane cerise; do
    echo "Fruit : $fruit"
done

# Itérer sur des fichiers
for fichier in /var/log/*.log; do
    echo "Fichier log : $fichier"
done

# Itérer sur une plage de nombres
for i in {1..10}; do
    echo "Numéro $i"
done
```

### Boucle while

```bash
compteur=0
while [ $compteur -lt 5 ]; do
    echo "Tour $compteur"
    compteur=$((compteur + 1))
done
```

---

## 6. Fonctions

```bash
# Déclarer une fonction
ma_fonction() {
    echo "Bonjour $1"      # $1 = premier argument de la fonction
    return 0               # Code de retour
}

# Appeler la fonction
ma_fonction "Alice"
```

---

## 7. Exemples pratiques courants

### Vérifier qu'un service tourne

```bash
#!/bin/bash
SERVICE="nginx"
if systemctl is-active --quiet $SERVICE; then
    echo "$SERVICE est actif"
else
    echo "$SERVICE est INACTIF — tentative de redémarrage"
    systemctl start $SERVICE
fi
```

### Script de sauvegarde avec date

```bash
#!/bin/bash
# Compresser le dossier web avec la date du jour
DATE=$(date +%Y%m%d)
ARCHIVE="/backup/html_${DATE}.tar.gz"

tar -czf $ARCHIVE /var/www/html/
echo "Sauvegarde créée : $ARCHIVE"

# Supprimer les archives de plus de 30 jours
find /backup/ -name "html_*.tar.gz" -mtime +30 -delete
echo "Anciennes archives supprimées"
```

### Boucle sur des fichiers log

```bash
#!/bin/bash
for fichier in /var/log/*.log; do
    echo "=== $fichier ==="
    tail -5 "$fichier"
done
```

---

## 8. Planification avec cron

### Format d'une ligne cron

```
┌────────── minute         (0-59)
│ ┌──────── heure          (0-23)
│ │ ┌────── jour du mois   (1-31)
│ │ │ ┌──── mois           (1-12)
│ │ │ │ ┌── jour semaine   (0-7, 0 et 7 = dimanche)
│ │ │ │ │
* * * * *   /chemin/vers/commande
```

### Exemples à décoder

```bash
30 2 * * 1      /backup.sh    # Tous les lundis à 2h30
0  0 * * *      /clean.sh     # Tous les jours à minuit
*/5 * * * *     /check.sh     # Toutes les 5 minutes
0  8 1 * *      /rapport.sh   # Le 1er de chaque mois à 8h
0  6 * * 1-5    /backup.sh    # Du lundi au vendredi à 6h
```

### Gestion des crontabs

```bash
crontab -e              # Éditer sa crontab (ouvre un éditeur)
crontab -l              # Lister ses tâches cron
crontab -r              # Supprimer sa crontab (attention !)
crontab -u alice -l     # Voir la crontab d'alice (root uniquement)
```

### Crontabs système

| Emplacement | Usage |
|-------------|-------|
| `/etc/crontab` | Crontab système avec champ utilisateur |
| `/etc/cron.d/` | Fichiers cron par application |
| `/etc/cron.hourly/` | Scripts exécutés toutes les heures |
| `/etc/cron.daily/` | Scripts exécutés quotidiennement |
| `/etc/cron.weekly/` | Scripts exécutés chaque semaine |
| `/etc/cron.monthly/` | Scripts exécutés chaque mois |

### cron vs at

| | cron | at |
|--|------|-----|
| **Type** | Récurrent (planifié régulièrement) | Unique (une seule fois) |
| **Usage** | Sauvegardes, maintenance régulière | Tâche ponctuelle future |
| **Exemples** | Backup chaque nuit, logrotate | Redémarrer à 22h ce soir |

```bash
# at — exécution unique
at 22:00                 # Entrer les commandes, Ctrl+D pour valider
at now + 2 hours         # Dans 2 heures
atq                      # Lister les tâches at en attente
atrm 3                   # Supprimer la tâche n°3
```

---

## 9. Points clés à retenir

```
✅ #!/bin/bash      →  shebang : interpréteur du script
✅ chmod +x         →  rendre exécutable avant ./script.sh
✅ $1, $2...        →  arguments positionnels
✅ $@               →  TOUS les arguments
✅ $#               →  NOMBRE d'arguments
✅ $?               →  code retour (0 = succès)
✅ >                →  écrase le fichier
✅ >>               →  ajoute à la fin
✅ 2>&1             →  stderr → stdout (capturer les erreurs)
✅ 2>/dev/null      →  jeter les erreurs
✅ [ -f fichier ]   →  tester l'existence d'un fichier
✅ Espaces dans []  →  OBLIGATOIRES : [ -f /etc/passwd ]
✅ fi               →  ferme un if (toujours !)
✅ * * * * *        →  format cron : min heure j-mois mois j-semaine
✅ crontab -e       →  éditer sa crontab
✅ cron             →  récurrent | at = unique
```
