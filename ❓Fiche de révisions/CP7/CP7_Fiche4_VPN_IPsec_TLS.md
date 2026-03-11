# CP7 – Fiche de Révision 4 : VPN – IPsec et TLS

> **Formation** : TSSR | **Compétence** : CP7  
> **Thème** : Définition du VPN, types, protocole IPsec, protocole TLS

---

## 1. Qu'est-ce qu'un VPN ?

Un **VPN (Virtual Private Network – Réseau Privé Virtuel)** permet de créer un réseau privé en utilisant une infrastructure publique (Internet).

**Objectifs principaux** :

| Objectif | Description |
|---|---|
| **Transparence** | L'utilisateur travaille comme s'il était sur le réseau local |
| **Sécurité** | Même niveau de sécurité malgré la traversée de réseaux publics |
| **Accessibilité** | Accès aux ressources du SI à distance |

### Le concept de tunnel

Un **tunnel** est un passage sécurisé à travers un réseau non sécurisé. Il fonctionne par **encapsulation** :
- Les données sont chiffrées et authentifiées
- Elles sont transmises dans un nouveau paquet (PDU)

> ⚠️ **Tous les tunnels ne sont pas des VPN sécurisés !**  
> L2TP, GRE, SSH sont des tunnels mais ne constituent pas des VPN sécurisés complets à eux seuls.

---

## 2. Les trois types de VPN

| Type | Extrémités du tunnel | Exemple d'usage |
|---|---|---|
| **VPN site-à-site** | Routeur/pare-feu ↔ Routeur/pare-feu | Relier deux sites d'entreprise |
| **VPN accès distant** | Client ↔ Passerelle VPN | Télétravailleur se connectant au SI de l'entreprise |
| **VPN hôte à hôte** | Machine ↔ Machine | Communication sécurisée entre deux serveurs |

---

## 3. Le protocole IPsec

### Présentation

**IPsec (Internet Protocol Security)** est un ensemble de standards IETF (1995) qui sécurise les communications directement au **niveau IP (couche 3)**.

- Développé initialement pour IPv6, adapté à IPv4
- **Transparent** pour les applications (opère sous la couche Transport)

### Les 3 protocoles d'IPsec

| Protocole | Rôle | Port/Numéro |
|---|---|---|
| **IKEv2** (Internet Key Exchange) | Négociation et établissement de la connexion | UDP 500 / 4500 |
| **AH** (Authentication Header) | Authentification et intégrité uniquement (pas de chiffrement) | IP proto 51 |
| **ESP** (Encapsulating Security Payload) | Confidentialité + Authentification + Intégrité | IP proto 50 |

#### IKEv2 – La négociation

- Établit un canal confidentiel via **Diffie-Hellman**
- Négocie les paramètres cryptographiques
- Authentification mutuelle (PSK ou certificats X.509)
- Communication sur UDP 500 (et **4500 pour le NAT traversal**)

#### AH – Authentification et intégrité

- Calcule un **ICV** (Integrity Check Value) de type HMAC
- Protège l'en-tête IP et le contenu
- Numéro de séquence → protection contre le **rejeu**
- ❌ **Pas de chiffrement** : les données restent lisibles

#### ESP – Protection complète (recommandé)

- **Chiffrement** du contenu (algorithme symétrique)
- **Intégrité** via ICV
- **Protection contre le rejeu** (numéro de séquence)
- ✅ **ESP est recommandé** car il offre à la fois confidentialité ET intégrité

---

### Les deux modes d'IPsec

#### Mode Transport

| Aspect | Description |
|---|---|
| **Usage** | Communication hôte à hôte (end-to-end) |
| **Protection** | Seule la couche Transport (TCP/UDP) est protégée |
| **En-têtes IP** | En clair (non chiffrés) |
| **Usage typique** | Communication directe entre deux machines |

#### Mode Tunnel ⭐ (recommandé)

| Aspect | Description |
|---|---|
| **Usage** | Passerelle à passerelle ou client à passerelle |
| **Protection** | Le **paquet IP complet** est encapsulé et protégé |
| **Avantage** | Tout le paquet original (y compris les IPs) est caché |
| **Usage typique** | **VPN site-à-site, VPN accès distant** |

---

### Security Association (SA)

Une **SA** définit tous les paramètres d'une connexion IPsec :
- Cryptosystèmes utilisés et paramètres
- Clés cryptographiques
- Utilisation d'AH et/ou ESP
- Mode tunnel ou transport

Le **SPI (Security Parameter Index)** est l'identifiant unique d'une SA, inclus dans les en-têtes AH et ESP.

---

## 4. Le protocole TLS

### Présentation

**TLS (Transport Layer Security)** est le successeur de SSL. Développé par Netscape (SSL), standardisé par l'IETF en 1999.

- Niveau OSI : **couche 5-6** (Session/Présentation)
- Fonctionne au-dessus de **TCP** (ou UDP avec DTLS)

### Versions de TLS

| Version | Statut |
|---|---|
| SSL 1.0, 2.0, 3.0 | ❌ **Dépréciées** – Ne jamais utiliser |
| TLS 1.0, 1.1 | ❌ **Dépréciées** (RFC 8996) – Vulnérabilités connues |
| TLS 1.2 | ⚠️ **Acceptable** – Pour compatibilité uniquement |
| **TLS 1.3** | ✅ **Recommandé** – Version actuelle |
| DTLS 1.2, 1.3 | ✅ **Recommandé** – TLS sur UDP |

> SSL et TLS 1.0/1.1 ne doivent plus être utilisés car ils présentent des **vulnérabilités cryptographiques** (POODLE, BEAST, CRIME, etc.) et des algorithmes obsolètes.

### Le Handshake TLS (poignée de main)

Phase d'établissement de la connexion TLS :

1. Négociation de la version TLS et des algorithmes
2. Authentification du serveur (certificat X.509)
3. Échange de clé de session (ECDH)
4. Démarrage des communications chiffrées

**Améliorations de TLS 1.3 par rapport à TLS 1.2** :

| Critère | TLS 1.2 | TLS 1.3 |
|---|---|---|
| Allers-retours (round-trips) | 3 | **1** |
| Suites cryptographiques | 37 | **5** (plus sûres) |
| Chiffrement du handshake | Tardif | **Dès le départ** |
| Reconnexion (0-RTT) | Non | **Oui** (si déjà connecté) |

### PFS – Perfect Forward Secrecy

Le **PFS (Perfect Forward Secrecy)** garantit que la compromission d'une clé à long terme ne compromet pas les sessions passées.

- Utilisé dans TLS 1.3 (obligatoire) et TLS 1.2 (optionnel avec ECDHE/DHE)
- Basé sur des **clés éphémères** générées à chaque session
- **Si une clé est volée**, les sessions passées restent protégées

### Utilisations courantes de TLS

| Protocole | Port | Description |
|---|---|---|
| **HTTPS** | TCP 443 | Web sécurisé |
| **FTPS** | TCP 989/990 | FTP sécurisé |
| **SMTPS** | TCP 465/587 | Email sécurisé |
| **IMAPS** | TCP 993 | Réception email sécurisée |
| **DoT** | TCP/UDP 853 | DNS sécurisé |
| **OpenVPN** | UDP/TCP 1194 | VPN basé TLS |

---

## 5. Tableau comparatif IPsec vs TLS

| Critère | IPsec | TLS |
|---|---|---|
| **Couche OSI** | 3 (Réseau) | 5-6 (Session/Présentation) |
| **Transparent pour les applis** | ✅ Oui | ❌ Non (doit être intégré) |
| **Usage principal** | VPN site-à-site, matériel réseau | Sécurisation applicative (HTTPS, email) |
| **Compatibilité NAT** | ⚠️ Problèmes (NAT-T nécessaire pour UDP 4500) | ✅ Fonctionne nativement |
| **Complexité de mise en œuvre** | Élevée | Modérée |

---

## 6. Points clés à retenir

1. **IPsec** : 3 composants (IKEv2, AH, ESP). **ESP est recommandé** (chiffrement + intégrité). **Mode tunnel** pour les VPN.
2. Une **SA** définit les paramètres d'une connexion IPsec, identifiée par son **SPI**.
3. **TLS 1.3** est la version recommandée. TLS 1.0/1.1 et SSL sont **obsolètes et dangereux**.
4. Le **PFS** protège les sessions passées même si une clé est compromise ultérieurement.
5. IPsec opère en couche 3 (transparent pour les applications), TLS en couche 5-6 (doit être intégré dans les applications).

---

_Fiche CP7 – 4/5 | TSSR_
