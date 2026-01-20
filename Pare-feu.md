* C'est quoi ? = Un filtre pour le réseau
* Emplacement = Intersection de réseaux
* Couche 4 minimum (TCP / IP) jusqu'à la couche 7
* Le Paquet est accept (Accept) ou refuse (drop ou reject) :
	* Drop : le paquet n'est pas accepter et l'expéditeur n'est pas au courant (le plus courant)
	* Reject : le paquet n'est pas accepter et l'expéditeur est au courant (le moins courant)
* Deux approches de filtrage :
	* Liste de blocage :
		* Tout accepter, puis filtrer.
		* Non recommandée
	* Liste d'autorisation : 
		* Tout bloquer, puis autoriser petit à petit
		* Recommandée
* Dans une DMZ = Uniquement les machines qui pourront communiquer avec l'extérieur (Web, Mail)

* Un pare-feu quel qu'il soit, si il est mal configuré, il ne sera pas efficace
* En entreprise le choix du pare-feu varie entre beaucoup de critères (prix, compétence, orientation de stockage)

* Pare-feu :
	* Check Point :
		* Expertise historique
		* Grandes entreprise
		* cher
	* Fortinet :
		* Rapport qualité/prix
		* Pour les PME
		* Solution complète sans trop de complexité
	* Cisco :
		* Très efficace en cas d'environnement globale Cisco
		* Assez cher

* La Cybersécurité n'est pas qu'une question de pare-feu et d'antivirus