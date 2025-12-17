# DHCP - Dynamic Host Configuration Protocol

_Synthèse de cours_

---

## 📑 Sommaire

1. [Introduction et problématique](https://claude.ai/chat/11af8af6-f3e5-47d5-96f5-f4b72d45ad8f#1-introduction-et-probl%C3%A9matique)
2. [Définition et objectifs](https://claude.ai/chat/11af8af6-f3e5-47d5-96f5-f4b72d45ad8f#2-d%C3%A9finition-et-objectifs)
3. [Fonctionnement général](https://claude.ai/chat/11af8af6-f3e5-47d5-96f5-f4b72d45ad8f#3-fonctionnement-g%C3%A9n%C3%A9ral)
4. [La séquence DORA](https://claude.ai/chat/11af8af6-f3e5-47d5-96f5-f4b72d45ad8f#4-la-s%C3%A9quence-dora)
5. [Les différents scénarios de connexion](https://claude.ai/chat/11af8af6-f3e5-47d5-96f5-f4b72d45ad8f#5-les-diff%C3%A9rents-sc%C3%A9narios-de-connexion)
6. [Fonctionnalités avancées](https://claude.ai/chat/11af8af6-f3e5-47d5-96f5-f4b72d45ad8f#6-fonctionnalit%C3%A9s-avanc%C3%A9es)
7. [Points clés à retenir](https://claude.ai/chat/11af8af6-f3e5-47d5-96f5-f4b72d45ad8f#7-points-cl%C3%A9s-%C3%A0-retenir)
8. [Ressources complémentaires](https://claude.ai/chat/11af8af6-f3e5-47d5-96f5-f4b72d45ad8f#8-ressources-compl%C3%A9mentaires)

---

## 1. Introduction et problématique

### Le problème de l'adressage statique

En IPv4, il n'existe pas de mécanisme automatique de configuration des interfaces réseau. La configuration manuelle (adressage statique) pose plusieurs problèmes majeurs :

**Limites techniques :**

- ❌ Erreurs humaines fréquentes lors de la saisie
- ❌ Temps d'administration important
- ❌ Difficulté d'éviter les **doublons d'adresses IP** (conflits)
- ❌ Risque majeur lors d'un changement de plan d'adressage
- ❌ Complexité de mise à jour de la documentation

**Problème d'échelle :**

Cette méthode devient impossible à gérer avec un parc important de machines, surtout si elles sont nomades (ordinateurs portables, smartphones, etc.).

> **Exemple concret :** Un réseau d'entreprise en `192.168.0.0/24` avec 50 machines existantes. L'ajout de 10 nouvelles machines nécessite de vérifier manuellement toutes les adresses déjà utilisées pour éviter les conflits.

---

## 2. Définition et objectifs

### Qu'est-ce que le DHCP ?

Le **DHCP** (Dynamic Host Configuration Protocol) est un protocole défini par les **RFC 2131** et **RFC 2132** qui permet l'attribution automatique de paramètres réseau IP aux machines.

### Objectifs principaux

#### 🎯 Centralisation de la configuration

- Un seul serveur gère l'ensemble des paramètres réseau
- Facilitation de la gestion d'un parc informatique
- Mise à jour simple et rapide (passerelle, DNS, etc.)

#### 🎯 Configuration automatique

- Paramétrage IP automatique et dynamique (adresse IP, masque, etc.)
- Élimination des doublons d'adresses
- Réduction des erreurs humaines
- Gestion automatique des entrées/sorties du réseau via les **baux**

#### 🎯 Économie de ressources

- **Économie d'adresses IP** : seules les machines actives reçoivent une configuration
- Utilisation du système de **baux IP** (durée limitée d'attribution)
- Économie de temps pour l'administration

#### 🎯 Support du "diskless"

- Support des hôtes sans disque dur
- Héritage de BOOTP pour le démarrage réseau
- Évolution : RARP → BOOTP → DHCP

---

## 3. Fonctionnement général

### Architecture client/serveur

DHCP fonctionne selon une architecture client/serveur avec des ports UDP spécifiques :

```
Serveur DHCP : UDP port 67
Client DHCP  : UDP port 68
```

### Le client DHCP

**Caractéristiques :**

- ✓ Pas de configuration IP initiale (adresse source : `0.0.0.0`)
- ✓ S'identifie via son **adresse MAC**
- ✓ Envoie une demande en **broadcast** (`255.255.255.255`)
- ✓ Ne peut pas traverser un routeur (limité au domaine de collision)
    - _Exception : configuration possible avec `ip-helper`_
- ✓ Peut renouveler son bail

### Identification des clients

Le serveur utilise l'adresse MAC pour identifier les clients, mais cette méthode n'est **pas totalement fiable** (MAC spoofing, machines virtuelles).

**Solutions plus sûres :**

|Version|Solution|Description|
|---|---|---|
|**IPv4**|Option DHCP 61 "Client Identifier" (RFC 2132)|Adresse MAC, chaîne aléatoire, ou ID généré par l'OS|
|**IPv6**|DUID (DHCP Unique Identifier) (RFC 8415)|Généré à partir du matériel ou d'un ID logiciel|

### Le serveur DHCP

**Rôles principaux :**

- Écoute les requêtes des clients
- Propose une configuration IP disponible
- Gère les **étendues d'adresses** (plages IP)
- Attribue les paramètres IP via les **Options DHCP**
- Gère les **baux** et les **réservations**
- Peut imposer des options spécifiques

### Paramètres transmis

#### Obligatoires ⚠️

- Adresse IP
- Masque de sous-réseau (CIDR)
- Bail (durée de validité)

#### Optionnels

- Passerelle par défaut
- Serveurs DNS (adresse IP / nom)
- Nom de domaine
- Serveur PXE
- Serveur NTP (temps)
- Serveur TFTP (boot)
- Et bien d'autres...

---

## 4. La séquence DORA

### Les 4 étapes essentielles

La séquence **DORA** représente le fonctionnement optimal du protocole DHCP :

```
┌─────────┐                                    ┌─────────┐
│ Client  │                                    │ Serveur │
│  DHCP   │                                    │  DHCP   │
└────┬────┘                                    └────┬────┘
     │                                              │
     │  1. DHCPDISCOVER (broadcast)                │
     │─────────────────────────────────────────────>│
     │     "Qui peut me donner une config IP ?"    │
     │                                              │
     │  2. DHCPOFFER                                │
     │<─────────────────────────────────────────────│
     │     "Voici une proposition : 192.168.1.10"  │
     │                                              │
     │  3. DHCPREQUEST                              │
     │─────────────────────────────────────────────>│
     │     "J'accepte, je réserve cette IP"        │
     │                                              │
     │  4. DHCPACK                                  │
     │<─────────────────────────────────────────────│
     │     "OK, IP réservée + config complète"     │
     │                                              │
```

### Détail des messages

|Message|Direction|Description|
|---|---|---|
|**DHCPDISCOVER**|Client → Broadcast|Demande de configuration IP (recherche de serveur)|
|**DHCPOFFER**|Serveur → Client|Proposition de configuration (IP, masque, etc.)|
|**DHCPREQUEST**|Client → Serveur|Demande de réservation de la configuration proposée|
|**DHCPACK**|Serveur → Client|Confirmation et envoi du paramétrage complet|

### Les autres messages DHCP

|Message|Direction|Utilisation|
|---|---|---|
|**DHCPNACK**|Serveur → Client|Refus de réservation|
|**DHCPDECLINE**|Client → Serveur|Le client refuse l'offre (IP déjà utilisée détectée via ARP)|
|**DHCPRELEASE**|Client → Serveur|Résiliation du bail par le client|
|**DHCPINFORM**|Client → Serveur|Demande de paramètres sans réservation d'IP (client ayant déjà une adresse)|

---

## 5. Les différents scénarios de connexion

### Cas 1 : Première connexion 🆕

**Séquence complète DORA**

```
DHCPDISCOVER → DHCPOFFER → DHCPREQUEST → DHCPACK
```

Le client n'a aucune information préalable et effectue la séquence complète.

### Cas 2 : Redémarrage avec IP valide 🔄

**Séquence raccourcie**

```
DHCPREQUEST → DHCPACK
```

Le client peut envoyer directement un DHCPREQUEST uniquement s'il connaît :

- L'adresse IP qu'il utilisait
- L'identifiant du serveur qui lui a attribué cette IP

**Réponses possibles :**

- ✅ `DHCPACK` : si l'IP est toujours valide et dans l'étendue
- ❌ `DHCPNACK` : si l'IP n'est plus valable → retour au cas 1

### Cas 3 : Expiration du bail ⏰

**Séquence :**

```
(tentative de renouvellement)
DHCPNACK (refus) → DHCPDISCOVER (nouvelle demande)
```

Lorsque le bail expire, le serveur refuse le renouvellement et le client doit recommencer la procédure complète.

### Cas 4 : Changement de réseau 🌐

**Problème :** L'adresse IP n'est plus dans le bon sous-réseau

**Séquence :**

```
DHCPREQUEST → DHCPNACK → DHCPDISCOVER
```

Le serveur détecte l'incompatibilité et force le client à redemander une nouvelle configuration.

---

## 6. Fonctionnalités avancées

### Paramètres distribués

Le serveur DHCP peut distribuer de nombreux paramètres au-delà des paramètres de base :

- 🌐 Adresses IP et masques de réseau
- 🔀 Informations de routage (route par défaut)
- 📛 Adresses de serveurs DNS récursifs
- 💿 Adresses de serveurs d'images de boot (TFTP)
- ⏰ Adresses de serveurs de temps (NTP)
- 📋 Et de nombreuses autres options...

### Fonctionnalités avancées à explorer

Pour aller plus loin dans la maîtrise du DHCP :

#### Tolérance de pannes

- Mise en place de **multiples serveurs DHCP coordonnés**
- Assure la continuité du service en cas de panne

#### Relais DHCP (DHCP Relay)

- Permet de **franchir les routeurs**
- Un agent relais transmet les requêtes broadcast entre différents sous-réseaux

#### Démarrage PXE (Preboot Execution Environment)

- Démarrage de machines via le réseau
- Utile pour l'installation d'OS ou le déploiement massif

---

## 7. Points clés à retenir

### 🔑 Les essentiels

|Élément|Information clé|
|---|---|
|**Acronyme DORA**|**D**iscover → **O**ffer → **R**equest → **A**ck|
|**Ports UDP**|Serveur: **67** / Client: **68**|
|**Fonction principale**|Centralisation de l'attribution automatique de configuration IP|
|**Versions**|IPv4 (RFC 2131/2132) et IPv6 (RFC 8415)|
|**Type de protocole**|Client/Serveur sur UDP|
|**Identification**|Par adresse MAC (ou Client Identifier/DUID)|
|**Portée**|Limitée au domaine de broadcast (sauf avec relais)|

### ⚠️ Points d'attention

- Les adresses MAC ne sont **pas des identifiants totalement fiables**
- Le DHCP ne franchit **pas les routeurs** sans configuration spécifique
- Un bail a une **durée limitée** et doit être renouvelé
- Les paramètres obligatoires sont : **IP, Masque, Bail**

---

## 8. Ressources complémentaires

### Documentation officielle

- **RFC 2131** : DHCP (Dynamic Host Configuration Protocol) pour IPv4
- **RFC 2132** : Options DHCP et extensions BOOTP
- **RFC 8415** : DHCPv6 (Dynamic Host Configuration Protocol for IPv6)

### Pour approfondir

- Format détaillé des paquets DHCP
- Liste exhaustive des options DHCP
- Configuration de serveurs DHCP (ISC DHCP, dnsmasq, Windows Server)
- Sécurisation du DHCP (DHCP Snooping, DAI)

---

_Synthèse réalisée à partir du cours DHCP et des RFC officielles_