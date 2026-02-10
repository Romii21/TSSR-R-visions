# Les Réseaux Privés Virtuels (VPN)

## 📋 Sommaire

1. [Qu'est-ce qu'un VPN ?](https://claude.ai/chat/9444aff7-09de-46c6-8cec-28907becd044#1-quest-ce-quun-vpn)
2. [Les types de VPN](https://claude.ai/chat/9444aff7-09de-46c6-8cec-28907becd044#2-les-types-de-vpn)
3. [Le protocole IPsec](https://claude.ai/chat/9444aff7-09de-46c6-8cec-28907becd044#3-le-protocole-ipsec)
4. [Le protocole TLS](https://claude.ai/chat/9444aff7-09de-46c6-8cec-28907becd044#4-le-protocole-tls)
5. [OpenVPN](https://claude.ai/chat/9444aff7-09de-46c6-8cec-28907becd044#5-openvpn)
6. [Sécuriser l'infrastructure](https://claude.ai/chat/9444aff7-09de-46c6-8cec-28907becd044#6-s%C3%A9curiser-linfrastructure)
7. [🎯 Points clés à retenir](https://claude.ai/chat/9444aff7-09de-46c6-8cec-28907becd044#-points-cl%C3%A9s-%C3%A0-retenir)

---

## 1. Qu'est-ce qu'un VPN ?

### Définition

Un **VPN** (Virtual Private Network - Réseau Privé Virtuel) permet de créer un réseau privé en utilisant des infrastructures publiques comme Internet. L'objectif est de simuler une connexion directe entre deux points distants.

### Objectifs principaux

|Objectif|Description|
|---|---|
|**Transparence**|L'utilisateur travaille comme s'il était sur le réseau local|
|**Sécurité**|Même niveau de sécurité malgré la traversée de réseaux publics|
|**Accessibilité**|Accès aux ressources du système d'information à distance|

### Le concept de tunnel

Un **tunnel** est un passage sécurisé à travers un réseau non sécurisé. Il fonctionne grâce à l'**encapsulation** :

- Les données sont considérées comme un contenu à protéger
- Elles sont chiffrées et authentifiées (en général)
- Elles sont transmises dans un nouveau paquet (PDU)

Les VPN peuvent opérer à différents niveaux OSI :

- **Niveau 2** (couche liaison) : transport de trames Ethernet
- **Niveau 3** (couche réseau) : transport de paquets IP

> ⚠️ **Important** : Tous les tunnels ne sont pas des VPN sécurisés !

---

## 2. Les types de VPN

### VPN Site à Site

**Usage** : Interconnexion permanente de réseaux physiques distants

|Caractéristique|Description|
|---|---|
|**Extrémités**|Passerelles (routeurs, pare-feu)|
|**Exemple**|Entreprise à Paris connectée à une filiale à Montréal|
|**Transparence**|Totale pour les utilisateurs - ils travaillent sur le même intranet|

### VPN Accès Distant (Remote Access)

**Usage** : Une machine individuelle accède à un réseau distant

|Caractéristique|Description|
|---|---|
|**Extrémités**|Client (ordinateur) → Passerelle (serveur VPN)|
|**Exemple**|Employé en télétravail accédant au réseau de l'entreprise|
|**Connexion**|Activée/désactivée manuellement par l'utilisateur|
|**Sécurité**|Utilisable même depuis un WiFi non sécurisé|

### VPN Point à Point (Host-to-Host)

**Usage** : Communication sécurisée entre deux machines spécifiques

|Caractéristique|Description|
|---|---|
|**Extrémités**|Deux machines spécifiques|
|**Exemple**|Administrateur se connectant à un serveur précis|
|**Avantage**|N'expose pas le reste du réseau|

---

## 3. Le protocole IPsec

### Présentation

**IPsec** (Internet Protocol Security) est un ensemble de standards IETF créé en 1995.

- Développé initialement pour IPv6, adapté à IPv4
- Sécurisation directement au niveau IP (couche 3)
- Transparent pour les applications et utilisateurs

### Architecture d'IPsec

IPsec est constitué de **3 protocoles** :

|Protocole|Rôle|Numéro|
|---|---|---|
|**IKEv2** (Internet Key Exchange)|Négociation et établissement de la connexion|UDP 500/4500|
|**AH** (Authentication Header)|Authentification et intégrité|IP proto 51|
|**ESP** (Encapsulating Security Payload)|Confidentialité + Authentification + Intégrité|IP proto 50|

### IKEv2 - Négociation

**IKEv2** remplace IKE et ISAKMP pour négocier l'établissement de la connexion :

- Canal confidentiel via Diffie-Hellman
- Négociation des paramètres cryptographiques
- Authentification mutuelle (PSK ou certificats X.509)
- Communication sur UDP port 500 (et 4500 pour NAT traversal)

### AH - Authentification et Intégrité

**AH** (Authentication Header) protège l'intégrité du paquet :

- Calcul d'un **ICV** (Integrity Check Value) de type HMAC
- Protège : le contenu, l'entête AH, et les champs fixes de l'entête IP
- Numéro de séquence pour protection contre le rejeu
- **Limitation** : Pas de confidentialité (pas de chiffrement)

### ESP - Confidentialité complète

**ESP** (Encapsulated Security Payload) offre une protection complète :

- Chiffrement du contenu avec algorithme symétrique
- ICV calculé sur le contenu chiffré et l'entête ESP
- Protection contre le rejeu via numéro de séquence
- **Recommandé** : ESP est généralement préféré à AH car il offre plus de fonctionnalités

### Modes de fonctionnement

#### Mode Transport (couche 4)

|Aspect|Description|
|---|---|
|**Utilisation**|Communication hôte à hôte (end-to-end)|
|**Protection**|Protocole de transport (TCP, UDP, ICMP, etc.)|
|**Limite**|Entêtes IP en clair (non authentifiés avec AH)|
|**Usage**|Limité - surtout pour des communications directes|

#### Mode Tunnel (couche 3) ⭐

|Aspect|Description|
|---|---|
|**Utilisation**|Communication passerelle à passerelle ou client à passerelle|
|**Protection**|Paquet IP complet encapsulé|
|**Avantage**|Tout le paquet original est protégé|
|**Usage**|**Très utilisé** - standard pour les VPN site-à-site et accès distant|

### Security Association (SA)

Une **SA** définit tous les paramètres d'une connexion IPsec :

- Cryptosystèmes utilisés et leurs paramètres
- Clés cryptographiques
- Utilisation d'AH et/ou ESP
- Mode tunnel ou transport
- **SPI** (Security Parameter Index) : identifiant unique de la SA

---

## 4. Le protocole TLS

### Présentation

**TLS** (Transport Layer Security) est le successeur de SSL (Secure Sockets Layer).

|Aspect|Description|
|---|---|
|**Origine**|SSL développé par Netscape pour sécuriser HTTP|
|**Standardisation**|Repris par l'IETF en 1999 sous le nom TLS 1.0|
|**Niveau OSI**|Couche 5-6 (transport/présentation)|
|**Base**|Fonctionne au-dessus de TCP ou UDP (DTLS)|

### Versions de TLS

|Version|Status|Notes|
|---|---|---|
|SSL 1.0, 2.0, 3.0|❌ **Dépréciées**|Ne jamais utiliser|
|TLS 1.0, 1.1|❌ **Dépréciées** (RFC 8996)|Obsolètes|
|TLS 1.2 (RFC 5246)|⚠️ **Acceptable**|Pour compatibilité uniquement|
|**TLS 1.3** (RFC 8446)|✅ **Recommandé**|Version actuelle|
|DTLS 1.2, 1.3|✅ **Recommandé**|TLS sur UDP|

### Le Handshake TLS (Poignée de main)

Le **handshake** est la phase d'établissement de connexion TLS qui permet de :

- Établir une connexion confidentielle (via ECDH)
- Négocier la version TLS et les cryptosystèmes
- Authentifier le serveur (certificat X.509)
- Authentifier le client (optionnel)

#### Améliorations TLS 1.3 vs TLS 1.2

TLS 1.3 optimise le handshake pour être plus rapide et plus sécurisé :

|TLS 1.2|TLS 1.3|
|---|---|
|3 allers-retours (round-trips)|**1 seul aller-retour**|
|~0.25-0.5 secondes|Économie de centaines de millisecondes|
|37 cipher suites|**5 cipher suites** (plus sûres)|
|Chiffrement tardif|Chiffrement du handshake dès le départ|

**0-RTT Resumption** : TLS 1.3 permet même une reconnexion sans aucun aller-retour si le client s'est déjà connecté au serveur.

### Communication TLS

Une fois le handshake terminé :

- Échanges chiffrés avec algorithme symétrique (ex: AES 128 bits)
- Intégrité et authenticité via HMAC
- **PFS** (Perfect Forward Secrecy) : confidentialité persistante

### Implémentations TLS courantes

|Bibliothèque|Description|
|---|---|
|**OpenSSL**|Implémentation la plus répandue|
|**GnuTLS**|Alternative libre|
|**LibreSSL**|Fork d'OpenSSL axé sur la sécurité|
|**SChannel**|Implémentation Microsoft (Windows)|

### Utilisations de TLS

|Protocole|Port|Description|
|---|---|---|
|**HTTPS**|TCP 443|HTTP sécurisé (web)|
|**FTPS**|TCP 989/990|FTP sécurisé|
|**SMTPS**|TCP 465/587|Email sécurisé|
|**POP3S**|TCP 995|Réception email sécurisée|
|**IMAPS**|TCP 993|Réception email sécurisée|
|**DoT**|TCP/UDP 853|DNS sécurisé|
|**OpenVPN**|UDP/TCP 1194|VPN basé sur TLS|

---

## 5. OpenVPN

### Présentation

**OpenVPN** est un logiciel VPN libre basé sur TLS.

|Caractéristique|Détail|
|---|---|
|**Première version**|2001|
|**Licence**|Libre et open-source|
|**Base**|TLS/DTLS|
|**Protocole**|UDP ou TCP|
|**Port par défaut**|1194|
|**Version actuelle**|2.6.14 (cours mentionne 2.5.8)|
|**Plateformes**|Multiplateforme|

### Modes de fonctionnement

#### Mode Routeur (niveau 3) - Recommandé

- Création d'interfaces **tun** (tunnel)
- Les interfaces se comportent comme les ports d'un routeur
- Transport de **paquets IP**

```
┌────────────┐     ┌─────────────────┐     ┌────────────┐
│  Paquet IP │ --> │ TLS → UDP → IP  │ --> │  Paquet IP │
└────────────┘     └─────────────────┘     └────────────┘
```

#### Mode Pont (niveau 2)

- Création d'interfaces **tap** (bridge)
- Les interfaces se comportent comme les ports d'un switch
- Transport de **trames Ethernet**

```
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│  Trame Ethernet │ --> │ TLS → UDP → IP  │ --> │  Trame Ethernet │
└─────────────────┘     └─────────────────┘     └─────────────────┘
```

### Authentification OpenVPN

OpenVPN supporte plusieurs méthodes d'authentification :

|Méthode|Description|Recommandation|
|---|---|---|
|**Login/Mot de passe**|Identifiants classiques|Peu sécurisé seul|
|**PSK** (Pre-Shared Key)|Secret partagé|Acceptable pour petits déploiements|
|**Certificats X.509**|Infrastructure PKI|✅ **Configuration classique et recommandée**|
|**MFA**|Multi-Factor Authentication|✅ **Fortement recommandé**|

### Infrastructure PKI pour OpenVPN

Pour un déploiement professionnel, il est courant de créer sa propre PKI :

#### Étapes de mise en place

1. **Création de l'AC (Autorité de Certification)**
    
    - Générer le certificat et la clé privée de l'AC
    - **Recommandation** : Stocker sur machine hors ligne
    - Outil utile : **Easy-RSA** pour simplifier la gestion
2. **Pour chaque serveur/client**
    
    - Créer un **CSR** (Certificate Signing Request)
    - Faire signer par l'AC
    - Déployer le certificat + certificat de l'AC
    - ⚠️ **Attention** : Protéger la clé privée !

#### Révocation de certificats

En cas de compromission :

|Méthode|Type|Description|
|---|---|---|
|**CRL**|Manuel|Certificate Revocation List - Liste de révocation|
|**OCSP**|Automatique|Online Certificate Status Protocol - Vérification en temps réel|

---

## 6. Sécuriser l'infrastructure

### Serveur Bastion

Le **bastion** est un principe de sécurité, pas un protocole spécifique.

|Aspect|Description|
|---|---|
|**Définition**|Point d'accès unique et contrôlé vers les ressources internes|
|**Usage**|Uniquement pour les accès d'administration|
|**Objectifs**|Réduire la surface d'attaque, centraliser les accès, améliorer la traçabilité|

**Exemples de solutions bastion** :

- **Apache Guacamole** : Bastion web sans client
- **Azure Bastion** : Solution cloud Microsoft

### Jump Server

Le **Jump Server** est une implémentation simple de bastion :

|Aspect|Description|
|---|---|
|**Définition**|Serveur intermédiaire pour accéder aux serveurs internes|
|**Relation**|Tous les jump servers sont des bastions, mais tous les bastions ne sont pas des jump servers|
|**Avantage**|Accès indirect avec meilleure maîtrise|

**Exemples** :

- Serveur **SSH** : Connexion en ligne de commande
- Serveur **RDP** : Connexion graphique Windows

### ⚠️ Points de vigilance sécurité

|Risque|Recommandation|
|---|---|
|**Ouverture réseau**|Les VPN créent une brèche entre réseaux|
|**Authentification**|Utiliser une authentification robuste (MFA)|
|**Surveillance**|Monitoring particulier des connexions VPN|
|**Cloisonnement**|Éventuellement segmenter le réseau pour limiter l'accès|

---

## 🎯 Points clés à retenir

### Concepts fondamentaux

|Concept|À retenir|
|---|---|
|**VPN**|Simule un réseau privé sur infrastructure publique via tunnel chiffré|
|**Tunnel**|Encapsulation + chiffrement + authentification des données|
|**Tous les tunnels ≠ VPN**|L2TP, GRE, SSH = tunnels mais pas VPN sécurisés complets|

### Choix de solution

|Solution|Niveau|Usage principal|Points clés|
|---|---|---|---|
|**IPsec**|3 (IP)|VPN site-à-site, matériel réseau|Transparent, mode tunnel recommandé, ESP > AH|
|**TLS**|5-6|Sécurisation applicative|TLS 1.3 obligatoire, handshake rapide|
|**OpenVPN**|2 ou 3|VPN accès distant, multi-plateforme|Basé TLS, mode routeur (tun) recommandé|

### Bonnes pratiques sécurité

✅ **À faire** :

- TLS 1.3 (ou minimum TLS 1.2)
- Authentification par certificats X.509 + MFA
- Mode tunnel IPsec pour VPN site-à-site
- Mise en place d'un bastion pour l'administration
- Surveillance et logs des connexions VPN
- Révocation de certificats compromis (CRL/OCSP)

❌ **À éviter** :

- SSL/TLS 1.0/1.1 (obsolètes et vulnérables)
- Authentification par simple mot de passe
- Confiance aveugle dans le VPN (toujours segmenter)
- Clés partagées non protégées

### Commandes et outils utiles

**Pour IPsec** :

```bash
# Vérifier les SA actives (Linux)
ip xfrm state
ip xfrm policy
```

**Pour TLS** :

```bash
# Tester une connexion TLS
openssl s_client -connect example.com:443 -tls1_3

# Vérifier un certificat
openssl x509 -in certificat.crt -text -noout
```

**Pour OpenVPN** :

```bash
# Démarrer un serveur OpenVPN
openvpn --config serveur.conf

# Se connecter en tant que client
openvpn --config client.ovpn

# Générer une PKI avec Easy-RSA
easyrsa init-pki
easyrsa build-ca
easyrsa gen-req serveur nopass
easyrsa sign-req server serveur
```

### Tableau récapitulatif des protocoles

|Protocole|Couche OSI|Port|Chiffrement|Authentification|Usage|
|---|---|---|---|---|---|
|**IPsec (ESP)**|3|IP proto 50|✅|✅|VPN site-à-site|
|**IPsec (AH)**|3|IP proto 51|❌|✅|Intégrité seule (rare)|
|**TLS 1.3**|5-6|Variable|✅|✅|Applications web/email|
|**OpenVPN**|2 ou 3|UDP/TCP 1194|✅ (TLS)|✅|VPN accès distant|

---

## 📚 Sources et références

**Documents de cours** :

- VPN.pdf - Support de cours principal
- VPN.md - Notes de cours

**Standards et RFC** :

- RFC 7296 - IKEv2
- RFC 4302 - AH (Authentication Header)
- RFC 4303 - ESP (Encapsulating Security Payload)
- RFC 8446 - TLS 1.3
- RFC 9147 - DTLS 1.3
- RFC 5246 - TLS 1.2

**Ressources web consultées** :

- The Illustrated TLS 1.3 Connection - Explication détaillée du handshake TLS 1.3
- TLS 1.3 Handshake: Improvements over TLS 1.2 - Performance et améliorations
- ANSSI - Recommandations de sécurité relatives à TLS - Guide officiel français

**Documentation officielle** :

- OpenVPN Documentation - https://openvpn.net/
- ANSSI - Guide TLS - https://cyber.gouv.fr/publications/recommandations-de-securite-relatives-tls

---

_Synthèse créée le 20 janvier 2026 - Basée sur le cours VPN et les standards actuels_