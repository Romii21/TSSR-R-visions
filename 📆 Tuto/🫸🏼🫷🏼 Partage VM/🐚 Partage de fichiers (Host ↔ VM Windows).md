## 🛠 Étape 1 : Installation des Guest Additions

Le partage ne peut pas fonctionner sans les pilotes "Guest Additions".

1. Dans la fenêtre de la VM, allez dans le menu **Périphériques** > **Insérer l'image CD des Additions invité...**.
    
2. Ouvrez l'explorateur de fichiers dans la VM et allez sur le **Lecteur CD (D:)**.
    
3. Lancez l'installateur **VBoxWindowsAdditions-amd64**.
    
4. Suivez l'assistant (Next > Install) et **redémarrez la VM** à la fin.
    

---

## 📂 Étape 2 : Configuration du dossier partagé

Dans les paramètres de la VM (icône roue crantée) ou via l'onglet **Expert** de la fenêtre active :

1. Allez dans la section **Shared Folders** (Dossiers partagés).
    
2. Cliquez sur l'icône **"+" (Dossier avec un plus vert)**.
    
3. Remplissez les champs comme suit :
    
    - **Folder Path :** Sélectionnez le dossier sur votre PC physique.
        
    - **Folder Name :** Donnez-lui un nom simple (ex: `Partage_VM`).
        
    - **Mount Point :** Laissez **vide** (pour éviter les conflits de lettres de lecteur).
        
    - **Options à cocher :**
        
        - ✅ **Montage automatique** (pour qu'il apparaisse tout seul).
            
        - ✅ **Configuration permanente** (Make Machine-permanent).
            
4. Validez par **OK**.
    

---

## 💻 Étape 3 : Accéder aux fichiers

Une fois la VM redémarrée :

1. Ouvrez l'**Explorateur de fichiers** de Windows Server.
    
2. Allez dans **Ce PC** (ou _This PC_).
    
3. Le dossier apparaîtra sous **Emplacements réseau** (souvent avec la lettre **Z:**).
    

> 💡 **Astuce :** Si le dossier n'apparaît pas, tapez manuellement `\\VBOXSVR` dans la barre d'adresse de l'explorateur de fichiers de la VM.

---

Souhaitez-vous que je vous explique comment faire la même chose si vous utilisez une VM Linux ?