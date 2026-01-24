## 🛠 Étape 1 : Installation des Guest Additions

C'est l'étape indispensable pour que Linux reconnaisse le système de fichiers de VirtualBox.

1. Dans le menu de la fenêtre VM : **Périphériques** > **Insérer l'image CD des Additions invité...**.
    
2. Ouvrez un terminal dans Linux et tapez les commandes suivantes :
    
    Bash
    
    ```
    # Créer un point de montage pour le CD
    sudo mount /dev/cdrom /mnt
    
    # Lancer l'installation
    sudo /mnt/VBoxLinuxAdditions.run
    ```
    
3. **Redémarrez** la VM une fois l'installation terminée.
    

---

## 📂 Étape 2 : Configuration du dossier partagé

Dans les paramètres de la VM (icône roue crantée) :

1. Allez dans **Shared Folders** (Dossiers partagés).
    
2. Ajoutez un nouveau partage (icône `+`) :
    
    - **Folder Path :** Chemin sur votre PC (ex: `C:\MonPartage`).
        
    - **Folder Name :** `Partage_Linux` (notez bien ce nom).
        
    - ✅ Cochez **Montage automatique**.
        
    - ✅ Cochez **Configuration permanente**.
        
    - ⚠️ **Mount Point :** Laissez **vide**.
        

---

## 🔑 Étape 3 : Débloquer l'accès (Permissions)

Sous Linux, par sécurité, les dossiers partagés appartiennent au groupe `vboxsf`. Votre utilisateur n'y est pas inscrit par défaut, ce qui cause souvent une erreur "Permission refusée".

1. Ouvrez un terminal et ajoutez votre utilisateur au groupe :
    
    Bash
    
    ```
    sudo usermod -aG vboxsf $USER
    ```
    
2. **Redémarrez la session** (déconnexion/reconnexion) ou la VM pour appliquer les droits.
    

---

## 📍 Étape 4 : Où trouver mes fichiers ?

Par défaut, VirtualBox monte le dossier dans le répertoire `/media/` avec le préfixe `sf_`.

- **Chemin d'accès :** `/media/sf_Partage_Linux`
    
- **Via Terminal :** `cd /media/sf_Partage_Linux`
    

### 💡 Astuce : Créer un raccourci dans votre dossier Home

Pour ne pas avoir à aller dans `/media/` à chaque fois, créez un lien symbolique :

Bash

```
ln -s /media/sf_Partage_Linux ~/Bureau/Partage
```

_Cela créera un dossier "Partage" directement sur votre bureau Linux._

---

Souhaitez-vous que je vous montre comment monter le dossier manuellement avec la commande `mount` si le montage automatique échoue ?