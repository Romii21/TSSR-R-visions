# Gestion des Processus et de la Mémoire

## 📑 Sommaire

1. [Concepts Fondamentaux](https://claude.ai/chat/8187d618-3e80-49e5-9c95-bc086f7d6c76#concepts-fondamentaux)
2. [Le Problème du Partage du Processeur](https://claude.ai/chat/8187d618-3e80-49e5-9c95-bc086f7d6c76#le-probl%C3%A8me-du-partage-du-processeur)
3. [Notion de Processus](https://claude.ai/chat/8187d618-3e80-49e5-9c95-bc086f7d6c76#notion-de-processus)
4. [Gestion de la Mémoire](https://claude.ai/chat/8187d618-3e80-49e5-9c95-bc086f7d6c76#gestion-de-la-m%C3%A9moire)
5. [L'Ordonnancement](https://claude.ai/chat/8187d618-3e80-49e5-9c95-bc086f7d6c76#lordonnancement)
6. [Approche GNU/Linux](https://claude.ai/chat/8187d618-3e80-49e5-9c95-bc086f7d6c76#approche-gnulinux)
7. [Approche Windows PowerShell](https://claude.ai/chat/8187d618-3e80-49e5-9c95-bc086f7d6c76#approche-windows-powershell)
8. [Le Démarrage du Système](https://claude.ai/chat/8187d618-3e80-49e5-9c95-bc086f7d6c76#le-d%C3%A9marrage-du-syst%C3%A8me)

---


---



---





---

## 

---

## 

---

## 


---

## 

---

## 
### Métadonnées Windows

|Terme|Signification|
|---|---|
|**Id**|Process ID (équivalent PID)|
|**ParentId**|Identifiant du processus parent|
|**UserName**|Utilisateur propriétaire|
|**Group**|Groupe associé|
|**Handles**|Nombre de fichiers/ressources ouverts|
|**State**|État (Running, Stopped, etc.)|
|**Path**|Répertoire de travail|

### Cmdlets PowerShell

**Consultation** :

```powershell
Get-Process                  # Liste les processus
Get-Process -Name notepad    # Info sur un processus spécifique
Get-ComputerInfo            # Infos système globales
Get-CimInstance Win32_Process # Via WMI
```

**Gestion** :

```powershell
Start-Process notepad.exe               # Lancer un processus
Stop-Process -Id 1234                   # Arrêter un processus
Wait-Process -Name notepad              # Attendre la fin
Invoke-Command -ScriptBlock {...}       # Exécuter localement ou à distance
```

**Services** :

```powershell
Get-Service                 # Liste les services
Start-Service ServiceName   # Démarrer un service
Stop-Service ServiceName    # Arrêter un service
Restart-Service ServiceName # Redémarrer
Suspend-Service ServiceName # Mettre en pause
Set-Service -Name ServiceName -StartupType Automatic
```

---

## Le Démarrage du Système



---

## 📊 Résumé Graphique Global

---

## 📚 Sources

- Cours fourni : "Gestion processeur et mémoire.pdf"
- Notes personnelles fournies
- Documentation officielle GNU/Linux et Microsoft PowerShell

---

**Créé** : 2025 | **Niveau** : Bac+2 Informatique | **État** : Synthèse complète