# CP6 — Fiche de Révision 1 : Fondamentaux du Scripting

> **Bash & PowerShell | TSSR**

---

## 1. Programme vs Script

| Critère | Programme | Script |
|---|---|---|
| Compilation | Oui (binaire) | Non (interprété) |
| Exécution | Directe par l'OS | Via un interpréteur |
| Exemples | `/bin/ls`, `notepad.exe` | `backup.sh`, `deploy.ps1` |

Un **script** est un fichier texte contenant des instructions lues et exécutées ligne par ligne par un interpréteur (bash, python, powershell...).

---

## 2. Le Shebang Bash

La **première ligne** d'un script indique à l'OS quel interpréteur utiliser. On appelle cette ligne le **shebang**.

```bash
#!/bin/bash              # Chemin absolu vers bash
#!/usr/bin/env bash      # ✅ Recommandé — cherche bash dans le PATH
```

**Pourquoi `#!/usr/bin/env bash` est préférable ?**
Sur certains systèmes, bash n'est pas dans `/bin/bash` (macOS, certaines distributions). `env` recherche l'interpréteur dynamiquement dans le `$PATH`, ce qui rend le script **portable**.

---

## 3. Rendre un Script Exécutable

Un script Bash doit avoir le bit d'exécution activé avant d'être lancé :

```bash
chmod +x backup.sh      # Ajouter le droit d'exécution
./backup.sh             # Exécuter le script
```

Sans `chmod +x`, le shell refusera d'exécuter le fichier directement.

---

## 4. Les Codes de Sortie (Exit Codes)

Chaque commande retourne un **code numérique** à sa fin :

| Code | Signification |
|---|---|
| `0` | ✅ Succès |
| `1` à `255` | ❌ Erreur (type dépend du programme) |

```bash
ls /tmp         # Succès
echo $?         # Affiche : 0

ls /inexistant  # Erreur
echo $?         # Affiche : 2
```

> **`$?`** est la variable spéciale qui contient le code de sortie de la dernière commande exécutée.

---

## 5. Les Guillemets en Bash

Les guillemets ont un comportement différent selon leur type :

| Type               | Syntaxe   | Variables interprétées ? |
| ------------------ | --------- | ------------------------ |
| Guillemets doubles | `"texte"` | ✅ Oui                    |
| Guillemets simples | `'texte'` | ❌ Non (texte brut)       |

```bash
USER="Alice"

echo "Bonjour $USER"    # Affiche : Bonjour Alice
echo 'Bonjour $USER'    # Affiche : Bonjour $USER
```

---

## 6. Introduction à PowerShell

### Qu'est-ce que PowerShell ?

PowerShell est un **interpréteur de commandes** ET un **langage de script** développé par Microsoft.

| Critère | PowerShell | Bash |
|---|---|---|
| Données manipulées | **Objets .NET** | Texte brut |
| Extension script | `.ps1` | `.sh` |
| Exécutable | `powershell.exe` / `pwsh.exe` | `/bin/bash` |
| Sensibilité casse | ❌ Non sensible | ✅ Sensible |

### La convention Verbe-Nom

Toutes les commandes PowerShell (**cmdlets**) respectent la convention `Verbe-Nom` :

```powershell
Get-Service          # Récupère des services
Set-Content          # Définit un contenu
New-Item             # Crée un élément
Remove-Item          # Supprime un élément
Invoke-Command       # Exécute une commande
```

> Utiliser `Get-Verb` pour voir la liste des verbes approuvés par Microsoft.

---

## 7. La Politique d'Exécution PowerShell (Execution Policy)

Par défaut, PowerShell peut bloquer l'exécution de scripts. La politique d'exécution contrôle ce comportement :

| Politique | Description |
|---|---|
| `Restricted` | Aucun script autorisé (défaut Windows) |
| `AllSigned` | Seuls les scripts signés sont autorisés |
| `RemoteSigned` | Scripts locaux OK, scripts distants doivent être signés |
| `Unrestricted` | Tous les scripts autorisés |
| `Bypass` | Aucune restriction (usage temporaire) |

```powershell
Get-ExecutionPolicy                        # Voir la politique actuelle
Set-ExecutionPolicy RemoteSigned           # Recommandé en production
```

---

## 8. Schéma Récapitulatif

```
BASH                              POWERSHELL
─────────────────────────────     ─────────────────────────────
#!/usr/bin/env bash               (pas de shebang)
chmod +x script.sh                Set-ExecutionPolicy RemoteSigned
./script.sh                       .\script.ps1

Manipule : TEXTE                  Manipule : OBJETS .NET
Sensible à la casse : OUI         Sensible à la casse : NON
Extension : .sh                   Extension : .ps1
$? = code de sortie               $? = True / False
```

---

## Points Clés à Retenir ✅

- Le **shebang** (`#!/usr/bin/env bash`) est indispensable en première ligne d'un script Bash
- `chmod +x` est nécessaire avant d'exécuter un script Bash
- `$?` retourne le code de sortie : **0 = succès**, **non-zéro = erreur**
- Les guillemets doubles `"..."` interprètent les variables, les simples `'...'` non
- PowerShell manipule des **objets .NET**, pas du texte brut
- Les cmdlets PowerShell suivent la convention **Verbe-Nom**
- La politique `RemoteSigned` est recommandée en production
