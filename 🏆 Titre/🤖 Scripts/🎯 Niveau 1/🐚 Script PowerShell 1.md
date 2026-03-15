``` PowerShell
# ===============================================
#
# Scipt machine à café de Niveau 1
#
# ===============================================
#
# Consignes :
# - Afficher un menu avec trois choix : café, thé, chocolat
# - Choisir une boisson avec un chiffre (de 1 à 3 par exemple)
# - Afficher un message `Vous avez choisi <la boisson>. Bonne dégustation !`
# - Sortir du script
#
# ===============================================

$boisson1 = "Café"
$boisson2 = "Thé"
$boisson3 = "Chocolat"

Clear-Host
  
Write-Host "Bienvenue"
Start-Sleep -Seconds 1
  
Write-Host ""
Write-Host "Quelle boisson souhaitez-vous choisir ?"
Write-Host ""
Write-Host "1. $boisson1"
Write-Host "2. $boisson2"
Write-Host "3. $boisson3"
Write-Host "x. Sortie"
Write-Host ""

$choix = Read-Host "Votre choix"
Write-Host ""
Start-Sleep -Seconds 1

switch ($choix) {

    "1" {
        Write-Host "Vous avez choisi $boisson1. Bonne dégustation !"
        exit
    }

    "2" {
        Write-Host "Vous avez choisi $boisson2. Bonne dégustation !"
        exit
    }

    "3" {
        Write-Host "Vous avez choisi $boisson3. Bonne dégustation !"
        exit
    }

    { $_ -in "x", "X" } {
        Write-Host "Au revoir"
        exit
    }

    default {
        Write-Host "Choix invalide."
        exit
    }
}
```