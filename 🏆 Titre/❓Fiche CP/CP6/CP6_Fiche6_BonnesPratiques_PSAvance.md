# CP6 — Fiche de Révision 6 : Bonnes Pratiques & PowerShell Avancé

> **Bash & PowerShell | TSSR**

---

## BASH — Bonnes Pratiques

### 1. Toujours ajouter le shebang portable

```bash
#!/usr/bin/env bash
```

### 2. Structurer l'en-tête du script

```bash
#!/usr/bin/env bash
# ============================================
# Nom    : backup.sh
# Utilité: Sauvegarde automatique de /home
# Usage  : sudo ./backup.sh /chemin/source
# Auteur : Jean DOE
# Màj    : 2025-01-15
# ============================================
```

### 3. Toujours quoter les variables

```bash
# ❌ Risqué — problème si la variable contient des espaces
rm $fichier

# ✅ Recommandé
rm "$fichier"
```

**Exemple de problème évité :**

```bash
fichier="mon fichier.txt"
rm $fichier       # Bash voit : rm mon fichier.txt → tente de supprimer "mon" ET "fichier.txt"
rm "$fichier"     # Bash voit : rm "mon fichier.txt" → supprime le bon fichier ✅
```

### 4. Vérifier les droits root

Ne jamais mettre `sudo` dans le script. Vérifier les droits en début de script :

```bash
#!/usr/bin/env bash

if [[ $EUID -ne 0 ]]
then
    echo "Ce script doit être exécuté en root (sudo)" >&2
    exit 1
fi

# Suite du script...
```

> **`$EUID`** = Effective User ID. L'UID de root est toujours **0**.

### 5. Gestion d'erreurs avec && et ||

```bash
# Pattern classique : succès ou message d'erreur
cd /dossier 2>/dev/null && echo "Succès" || echo "Erreur : dossier introuvable"

# Redirection des erreurs dans /dev/null pour ne pas polluer la sortie
commande 2>/dev/null

# Rediriger stderr vers un fichier de log
commande 2>> /var/log/erreurs.log
```

**Que fait `2>/dev/null` ?**
- `2` = flux d'erreur standard (stderr)
- `>` = rediriger vers
- `/dev/null` = "poubelle" système (tout ce qui y va est supprimé)

### 6. Logs et traçabilité

```bash
#!/usr/bin/env bash

LOG_FILE="/var/log/mon_script.log"

log_message() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $1" >> "$LOG_FILE"
}

log_message "Script démarré"
mkdir /test && log_message "Dossier créé" || log_message "Erreur création dossier"
log_message "Script terminé"
```

### 7. Redirection `>` vs `>>`

| Opérateur | Comportement |
|---|---|
| `>` | Écrase le fichier (recrée à zéro) |
| `>>` | Ajoute à la fin du fichier |

```bash
echo "Nouveau log" > /var/log/app.log     # ❌ Efface l'historique
echo "Nouveau log" >> /var/log/app.log    # ✅ Conserve l'historique
```

> Pour un fichier de log, utiliser toujours `>>` pour ne pas perdre l'historique.

### 8. Utiliser ShellCheck pour déboguer

**ShellCheck** est un outil d'analyse statique de scripts Bash qui détecte les erreurs courantes :

```bash
shellcheck mon_script.sh
```

Erreurs typiques détectées :

- Variables non quotées
- Comparaisons dangereuses
- Shebang manquant
- Syntaxes dépréciées

---

## Structure d'un Script Bash Professionnel

```bash
#!/usr/bin/env bash

# ============================================
# Nom    : backup.sh
# Utilité: Sauvegarde horodatée d'un répertoire
# Usage  : sudo ./backup.sh /chemin/source
# Auteur : Jean DOE
# ============================================

# --- Variables ---

SOURCE="$1"
BACKUP_DIR="/backups"
LOG_FILE="/var/log/backup.log"
DATE=$(date '+%Y-%m-%d_%H-%M-%S')

# --- Fonctions ---

log() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $1" >> "$LOG_FILE"
}

# --- Vérifications ---

if [[ $EUID -ne 0 ]]; then
    echo "Exécuter avec sudo" >&2
    exit 1
fi

if [ -z "$SOURCE" ]; then
    echo "Usage : $0 /chemin/source" >&2
    exit 1
fi

if [ ! -d "$SOURCE" ]; then
    echo "Erreur : $SOURCE n'existe pas" >&2
    exit 1
fi

# --- Corps du script ---

mkdir -p "$BACKUP_DIR"
ARCHIVE="$BACKUP_DIR/backup_${DATE}.tar.gz"

tar -czf "$ARCHIVE" "$SOURCE" 2>/dev/null \
    && log "Sauvegarde réussie : $ARCHIVE" \
    || log "ERREUR lors de la sauvegarde de $SOURCE"

exit 0
```

---

## POWERSHELL — Fonctionnalités Avancées

### Le Pipeline PowerShell

Le pipeline `|` transmet des **objets** d'une cmdlet à une autre (et non du texte brut comme en Bash) :

```powershell
# Récupérer les services arrêtés et les trier par nom
Get-Service | Where-Object { $_.Status -eq "Stopped" } | Sort-Object Name

# Chaîne : Get-Service → filtrage → tri → affichage
Get-Process | Where-Object { $_.CPU -gt 10 } | Select-Object Name, CPU
```

**Différence avec le pipe Bash :**

| Pipe | Ce qui est transmis |
|---|---|
| Bash `\|` | Texte brut (sortie standard) |
| PowerShell `\|` | Objets .NET avec propriétés et méthodes |

### Les Jobs (Exécution Parallèle)

Un **Job** permet d'exécuter une tâche en **arrière-plan** sans bloquer le terminal :

```powershell
# Démarrer un job
$Job = Start-Job -ScriptBlock {
    Start-Sleep -Seconds 10
    Get-Service
}

# Gérer les jobs
Get-Job          # Lister tous les jobs
Wait-Job $Job    # Attendre la fin du job
Receive-Job $Job # Récupérer le résultat
Remove-Job $Job  # Supprimer le job
```

**Les 5 cmdlets des Jobs :**

| Cmdlet | Action |
|---|---|
| `Start-Job` | Démarre un job en arrière-plan |
| `Get-Job` | Liste les jobs et leur état |
| `Wait-Job` | Bloque jusqu'à la fin du job |
| `Receive-Job` | Récupère la sortie du job |
| `Remove-Job` | Supprime le job de la mémoire |

### PowerShell à Distance (Remoting)

**Prérequis :** Le service **WinRM** doit être démarré sur la machine cible.

```powershell
# Activer WinRM sur la cible
Enable-PSRemoting -Force
```

**Session interactive — `Enter-PSSession`**

Pour travailler interactivement sur une machine distante :

```powershell
Enter-PSSession -ComputerName Serveur01
# Vous êtes maintenant sur Serveur01
Get-Service         # S'exécute sur Serveur01
Exit-PSSession      # Retour en local
```

**Exécution de commandes — `Invoke-Command`**

Pour exécuter un bloc de commandes sur une ou plusieurs machines :

```powershell
Invoke-Command -ComputerName Serveur01 -ScriptBlock {
    Get-Service | Where-Object { $_.Status -eq "Running" }
}

# Sur plusieurs machines simultanément
Invoke-Command -ComputerName Serveur01, Serveur02, Serveur03 -ScriptBlock {
    Get-Disk
}
```

**Comparaison `Enter-PSSession` vs `Invoke-Command` :**

| Cmdlet | Usage |
|---|---|
| `Enter-PSSession` | Session interactive, commandes une par une |
| `Invoke-Command` | Exécution de scripts/blocs, plusieurs machines |

---

## Récapitulatif des Bonnes Pratiques

```
SCRIPT BASH PROFESSIONNEL
├── 1. Shebang          #!/usr/bin/env bash
├── 2. En-tête          # Nom, utilité, usage, auteur
├── 3. Vérif. droits    if [[ $EUID -ne 0 ]]
├── 4. Variables        Tout en haut, quotées
├── 5. Fonctions        log(), afficher_aide()...
├── 6. Vérifications    Arguments, fichiers, dossiers
├── 7. Corps script     Logique principale
├── 8. Logs             >> /var/log/script.log
└── 9. exit 0           Toujours finir proprement
```

---

## Points Clés à Retenir ✅

- Toujours `"$variable"` avec guillemets pour éviter les bugs sur les espaces
- Ne jamais mettre `sudo` dans un script — vérifier `$EUID` en début de script
- `>` écrase ; `>>` ajoute — utiliser `>>` pour les logs
- `2>/dev/null` redirige les erreurs vers la poubelle
- `ShellCheck` analyse statiquement un script Bash et détecte les problèmes
- En PowerShell, le pipeline transmet des **objets**, pas du texte
- `WinRM` doit être actif pour que le remoting PowerShell fonctionne
- `Enter-PSSession` = session interactive ; `Invoke-Command` = exécution de scripts distants
- Les Jobs PowerShell permettent l'exécution **asynchrone** en arrière-plan
