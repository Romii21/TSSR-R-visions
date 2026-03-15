```bash
#! /bin/bash
#
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

boisson1="café"
boisson2="thé"
boisson3="chocolat"

clear

# Message d'acceuil

echo "Bienvenue"
sleep 1

# Affichage des choix
  
echo ""
echo "Quelle boisson souhaitez-vous choisir ?"
echo ""
echo "1. $boisson1"
echo "2. $boisson2"
echo "3. $boisson3"
echo "x. Sortie"
echo ""

# Lecture du choix

read -p "Votre choix : " choix
echo
sleep 1

# stucture case pour la sortie du choix

case $choix in
  
    1)
        echo "Vous avez choisi $boisson1. Bonne dégustation !"
        exit
        ;;
  
    2)
        echo "Vous avez choisi $boisson2. Bonne dégustation !"
        exit
        ;;

    3)
        echo "Vous avez choisi $boisson3. Bonne dégustation !"
        exit
        ;;

    x|X)
        echo "Au revoir"
        exit
        ;;

    *)
        echo "Choix invalide"
        exit
        ;;
esac
```