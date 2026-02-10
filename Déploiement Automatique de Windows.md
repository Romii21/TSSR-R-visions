- Permet de préparer un pc (OS, Logiciel, Personnalisation, etc...) depuis un serveur distant
- AD-DS non obligatoire
- DHCP nécessaire
- MDT (Microsoft Deployment Toolkit):
	- Gratuit
	- Nécessite WADK
	- Utilité :
		- Automatise la fabrication d'images
		- Automatise l'installation d'images
	- Fonctionnalités :
		- Personnalisation possible pour beaucoup de points 
- WADK (Windows Assessment and Deployement Kit)
	- Suite d'outils pour le déploiement d'OS
- SCCM (System Center Configuration Manager) :
	- Contient WSD et MDT
	- Utile pour les très grands parcs informatiques

- Master :
	- Image de disque de référence :
	- 3 types de masters :
		- Thin Images :
			- Image légère avec l'OS
			- MAJ : 1 à 2 ans
		- Thick image :
			- Contient l'OS et les logiciels nécessaires
			- MAJ : 6 Moiss grand maximum
		- Hybrid image :
			- Contient l'OS avec les logiciels de base
			- MAJ : 

- WinPE (Windows Preinstallation Environment) :
	- Noyau léger d'un OS Windows
	- Disponible pour MSD et WSD

- PXE (Preboot execution Environnement)
	- Permet de démarrer avec les informations réseaux

