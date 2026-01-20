* Un VPN permet de connecter deux réseaux distant via une connexion sécurisé dans un milieu qui ne l'est pas.
* Types de VPN :
	* Site à site :
		* Interconnexion de réseaux physiques
		* Les extrémités sont des passerelles 
		* C'est comme si les réseaux étaient les mêmes
	* Accès distant :
		* Une machine accède à un réseau
		* C'est comme si elle est directement dans le réseau
	* Point à point :
		* Créer une communications entre deux machines 
		* Comme une connexion SSH entre deux machines qui ne sont pas sur le même réseau
* tout les tunnels ne pas des VPN sécurisés !
* VPN sécurisés :
	* IPsec :
		* (Internet Protocol Security)
		* IPv6 de base l'IPv4 est rajouté après
		* Cryptographie :
			* De préférence des clés asymétrique 
		* AH et ESP sont des options d'IP
		* Mode Transport : 
			* Couche 4
			* TCP, UDP
		* Mode Tunnel :
			* Couche 3
			* IP
	* TLS :
		* (Transport Layer Security)
		* TLS 1.3 (RFC 8446) et DTLS 1.3 (RFC 9147) Versions recommandées
		* TLS 1.2 (RFC 5246) et DTLS 1.2 (RFC 6347) Utilisable pour sa compatibilité
	* OpenVPN : 
		* Logicel VPN libre client/serveur
		* Basé sur TLS
		* UDP TCP ports 1194
		* versions : 2.6.14
		* Authentification :
			* MFA
* Serveur Bastion :
	* Point d'accès unique et contrôlé vers les ressources internes
	* Uniquement pour les Administrateurs
* Jump serveur :
	* Bastion mais simplifié
	* Le bastion est un principe et le Jump est le serveur
		* Serveur SSH
		* Serveur RDP
	* 