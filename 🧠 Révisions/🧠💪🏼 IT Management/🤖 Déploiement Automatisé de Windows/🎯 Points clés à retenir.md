### Les 4 acteurs principaux

|Acronyme|Nom complet|Rôle résumé|
|---|---|---|
|**WDS**|Windows Deployment Services|Rôle serveur — déploie via PXE/TFTP|
|**MDT**|Microsoft Deployment Toolkit|Outil gratuit — automatise et personnalise|
|**WADK**|Windows Assessment and Deployment Kit|Suite d'outils — requis par MDT|
|**SCCM**|System Center Configuration Manager|Solution complète pour grands parcs (payant)|

### Les éléments clés du déploiement

|Élément|Rôle essentiel|
|---|---|
|**Master / Image de référence**|Le modèle qui sera dupliqué sur tous les postes|
|**Sysprep**|Obligatoire avant capture — généralise l'image|
|**WinPE**|Mini-OS qui démarre le poste avant installation|
|**PXE**|Permet le boot réseau sans USB ni CD|
|**WIM**|Format de fichier image utilisé par tout l'écosystème Microsoft|
|**DISM**|Outil de gestion des images WIM (montage, modification, export…)|
|**DHCP**|Indispensable pour le boot PXE|

### Règles à ne pas oublier

> ✅ **MDT + WADK** = déploiement gratuit et personnalisable  
> ✅ **MDT + WDS** = nécessaire pour activer le boot PXE avec MDT  
> ✅ **SCCM** = uniquement pour les grands parcs, solution payante  
> ⚠️ **Sysprep** doit être fait avant toute capture d'image  
> ⚠️ **Ne jamais rallumer le PC** après un Sysprep avant capture  
> ⚠️ **DHCP obligatoire** pour tout déploiement réseau (PXE)

### Commande Sysprep à retenir absolument

```cmd
C:\Windows\System32\sysprep\sysprep.exe /oobe /generalize /shutdown
```

---
