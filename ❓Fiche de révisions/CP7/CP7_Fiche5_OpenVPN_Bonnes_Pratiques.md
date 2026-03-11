# CP7 – Fiche de Révision 5 : OpenVPN et Bonnes Pratiques de Sécurité

> **Formation** : TSSR | **Compétence** : CP7  
> **Thème** : OpenVPN, PKI, révocation de certificats, sécurisation avancée

---

## 1. OpenVPN

### Présentation

**OpenVPN** est un logiciel VPN **libre et open-source** basé sur TLS/DTLS.

| Caractéristique | Détail |
|---|---|
| Première version | 2001 |
| Licence | Open-source (GPL) |
| Base cryptographique | TLS / DTLS |
| Protocole réseau | UDP (recommandé) ou TCP |
| Port par défaut | **1194** |
| Plateformes | Multiplateforme (Windows, Linux, macOS, Android, iOS) |

---

### Les deux modes de fonctionnement

#### Mode Routeur – tun (niveau 3) ✅ Recommandé

- Crée des interfaces **tun** (tunnel)
- Les interfaces se comportent comme des **ports de routeur**
- Transporte des **paquets IP**

```
┌────────────┐     ┌─────────────────┐     ┌────────────┐
│  Paquet IP │ --> │ TLS → UDP → IP  │ --> │  Paquet IP │
└────────────┘     └─────────────────┘     └────────────┘
```

**Avantages** : Plus simple, plus efficace, idéal pour les VPN d'accès distant.

#### Mode Pont – tap (niveau 2)

- Crée des interfaces **tap** (bridge)
- Les interfaces se comportent comme des **ports de switch**
- Transporte des **trames Ethernet** (niveau 2)

**Usage** : Cas rares nécessitant le transport de broadcast L2 (ex : environnements legacy avec NetBIOS).

| Mode | Interface | Niveau OSI | Transport | Recommandation |
|---|---|---|---|---|
| **Routeur** | tun | 3 | Paquets IP | ✅ Recommandé |
| **Pont** | tap | 2 | Trames Ethernet | Usage spécifique |

---

### Méthodes d'authentification

| Méthode | Description | Recommandation |
|---|---|---|
| Login/Mot de passe | Identifiants classiques | ❌ Insuffisant seul |
| PSK (Pre-Shared Key) | Secret partagé | ⚠️ Acceptable pour petits déploiements |
| **Certificats X.509** | Infrastructure PKI | ✅ Configuration classique et recommandée |
| **MFA** (Multi-Factor Authentication) | Authentification à plusieurs facteurs | ✅ Fortement recommandé |

---

## 2. Infrastructure PKI pour OpenVPN

Pour un déploiement professionnel, on met en place une **PKI (Public Key Infrastructure)**.

### Composants d'une PKI

| Composant | Rôle |
|---|---|
| **AC (Autorité de Certification)** | Signe et certifie les certificats. C'est la racine de confiance. |
| **Certificat X.509** | Fichier associant une clé publique à une identité, signé par l'AC |
| **CSR (Certificate Signing Request)** | Demande de signature envoyée à l'AC |
| **Clé privée** | Secrète, ne doit jamais être transmise |

### Étapes de mise en place (avec Easy-RSA)

1. **Créer l'AC** : générer le certificat racine et la clé privée de l'AC  
   → Stocker sur une machine **hors ligne** (sécurité maximale)

2. **Pour chaque serveur/client** :
   - Générer un CSR
   - Faire signer par l'AC
   - Déployer le certificat + le certificat de l'AC
   - ⚠️ Protéger la clé privée (permissions strictes, passphrase)

```bash
# Exemple Easy-RSA
easyrsa init-pki
easyrsa build-ca
easyrsa gen-req serveur nopass
easyrsa sign-req server serveur
```

### Ce que vérifie OpenVPN à la connexion

- Le certificat client est-il **signé par la bonne AC** ?
- Le certificat est-il **dans sa période de validité** ?
- Le certificat figure-t-il sur la **liste de révocation** ?

---

## 3. Révocation de certificats

En cas de compromission d'un certificat (vol de clé privée, départ d'un employé…) :

### Mécanismes disponibles

| Mécanisme | Type | Description |
|---|---|---|
| **CRL** (Certificate Revocation List) | Manuel | Liste de révocation distribuée par l'AC. Le serveur vérifie la CRL lors de chaque connexion. |
| **OCSP** (Online Certificate Status Protocol) | Automatique | Vérification en temps réel de l'état d'un certificat auprès de l'AC. |

### Procédure en cas de compromission d'une AC

1. Isoler immédiatement l'infrastructure
2. **Révoquer tous les certificats** émis par l'AC compromise
3. Recréer une nouvelle AC
4. Réémettre tous les certificats (serveurs et clients)
5. Redéployer les nouveaux certificats sur tous les équipements
6. Mettre à jour les CRL/OCSP
7. Analyser les logs pour identifier l'impact

---

## 4. Sécurisation avancée

### PFS – Perfect Forward Secrecy

Le **PFS** garantit que la compromission d'une clé long terme ne permet pas de déchiffrer les communications passées.

- Basé sur des **clés de session éphémères** (renouvelées à chaque session)
- Mécanisme : **Diffie-Hellman éphémère** (ECDHE)
- Appliqué dans : TLS 1.3 (obligatoire), TLS 1.2 (ECDHE/DHE), IKEv2

### Tunnel VPN ≠ Sécurité totale

Un VPN chiffre le tunnel mais **ne protège pas contre** :
- Les attaques depuis l'intérieur du tunnel (compromission de la machine distante)
- L'accès non autorisé à des ressources internes une fois connecté
- Les malwares présents sur la machine cliente

**Approche complémentaire recommandée** :
- Segmentation du réseau (zones de confiance)
- Contrôle d'accès basé sur les rôles (RBAC)
- Surveillance des connexions VPN (logs, alertes)
- **Zero Trust** : ne jamais faire confiance implicitement, même sur le VPN

### Tunneling de protocoles non autorisés

Un attaquant peut faire passer du trafic non autorisé via un port autorisé (ex : port 443). Ce mécanisme s'appelle le **tunneling applicatif**.

**Solution** : le **DPI (Deep Packet Inspection)** permet d'identifier le vrai protocole applicatif au-delà du port.

---

## 5. Bonnes pratiques récapitulatives

### ✅ À faire

- Utiliser **TLS 1.3** (ou minimum TLS 1.2)
- Authentification par **certificats X.509 + MFA**
- Mode **tunnel** pour IPsec (VPN site-à-site)
- Mode **tun** pour OpenVPN (accès distant)
- Mettre en place un **bastion** pour l'administration
- Surveiller et loguer les connexions VPN
- Gérer la révocation des certificats (**CRL/OCSP**)
- Activer le **PFS** (Perfect Forward Secrecy)

### ❌ À éviter

- SSL, TLS 1.0, TLS 1.1 (obsolètes et vulnérables)
- Authentification par **simple mot de passe**
- Confiance aveugle dans le VPN
- Clés privées non protégées ou partagées
- AC stockée sur une machine connectée au réseau

---

## 6. Tableau comparatif final : IPsec vs OpenVPN

| Critère | IPsec | OpenVPN |
|---|---|---|
| **Couche OSI** | 3 (Réseau) | 2 ou 3 |
| **Usage principal** | VPN site-à-site, matériel réseau | VPN accès distant, multiplateforme |
| **Complexité de mise en œuvre** | Élevée | Modérée |
| **Compatibilité NAT** | ⚠️ Problèmes (NAT-T nécessaire) | ✅ Fonctionne nativement |
| **Support multiplateforme** | Variable | ✅ Excellent |
| **Base cryptographique** | IKEv2 + AH/ESP | TLS/DTLS |
| **Choix pour accès distant multiplateforme** | ⚠️ Moins adapté | ✅ **Recommandé** |

---

## 7. Points clés à retenir

1. **OpenVPN** = VPN open-source basé TLS, port 1194. Mode **tun** recommandé.
2. Pour un déploiement pro : authentification par **certificats X.509 + MFA**.
3. La **PKI** repose sur une AC qui signe les certificats. L'AC doit être **hors ligne et sécurisée**.
4. En cas de compromission : **révoquer** via CRL ou OCSP, recréer, redéployer.
5. Le **PFS** protège les sessions passées même si une clé est compromise plus tard.
6. Un VPN ne remplace pas la segmentation réseau ni le contrôle d'accès → approche **Zero Trust**.
7. Pour un VPN d'accès distant multiplateforme → **OpenVPN** est recommandé.

---

_Fiche CP7 – 5/5 | TSSR_
