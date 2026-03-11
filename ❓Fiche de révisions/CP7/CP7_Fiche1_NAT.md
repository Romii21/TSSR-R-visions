# CP7 – Fiche de Révision 1 : NAT et Translation d'Adresses

> **Formation** : TSSR | **Compétence** : CP7  
> **Thème** : NAT, PAT/NAPT, DNAT, NAT 1:1

---

## 1. Contexte et définition

Le **NAT (Network Address Translation)** est un mécanisme qui traduit des adresses IP à la frontière entre un réseau privé (LAN) et un réseau public (WAN/Internet).

- Défini dans la **RFC 3022**
- Mis en œuvre sur un **routeur** ou un **pare-feu**
- Problématique **exclusive à IPv4** (en IPv6, le NAT existe sous la forme NPTv6 mais n'est plus indispensable)

### Pourquoi le NAT existe-t-il ?

| Raison historique | Raison actuelle |
|---|---|
| Cacher son plan d'adressage interne | Compenser la **pénurie d'adresses IPv4 publiques** |

> Le NAT est une **solution transitoire** en attendant la généralisation d'IPv6.

---

## 2. Les trois critères de fonctionnement

Tout mécanisme NAT se définit selon 3 axes :

| Critère | Options |
|---|---|
| **Sens** | Source (sortant) ou Destination (entrant) |
| **Mode** | Statique (fixe) ou Dynamique (temporaire) |
| **Niveau** | Adresse IP seule, ou Adresse IP + Port |

---

## 3. Les trois types de NAT

### 3.1 PAT / NAPT – Translation avec ports

**Définition** : Traduction **dynamique** de plusieurs flux internes vers **une seule IP publique**, en utilisant les **ports** pour différencier les flux.

- Aussi appelé : NAT overload / NAT masquerade (Linux) / SNAT avec ports
- Défini dans la **RFC 2663**

**Caractéristiques** :

| Sens | Mode | Niveau |
|---|---|---|
| Source | Dynamique | IP + port |

**Fonctionnement** :

```
[Machine interne]           [Routeur NAT]              [Internet]
10.1.1.11:52369    →    203.1.113.123:52369    →    216.58.214.83:443
                         (IP publique)
```

- **Requête sortante** : Le routeur remplace l'IP source privée par l'IP publique et enregistre la correspondance dans sa table NAT.
- **Réponse entrante** : Le routeur identifie la machine interne grâce au port, et retransmet le paquet vers elle.
- **Collision de ports** : Si le port est déjà utilisé, le routeur en choisit un autre disponible.

**Usage** : Le plus courant. Donne accès à Internet à tous les clients internes avec une seule IP publique.

---

### 3.2 DNAT – Destination NAT (Port Forwarding)

**Définition** : Traduction **statique** de l'adresse de **destination** pour rediriger une communication entrante vers une machine interne.

**Caractéristiques** :

| Sens | Mode | Niveau |
|---|---|---|
| Destination | Statique | IP (+ port optionnel) |

**Fonctionnement** :

```
[Internet]              [Routeur NAT]              [Serveur interne]
204.1.97.10:57221  →  203.1.113.123:80  →  172.16.1.15:80
```

- **Requête entrante** : Le routeur traduit l'IP de destination publique vers l'IP privée du serveur interne.
- **Réponse sortante** : Le routeur remplace l'IP source du serveur interne par l'IP publique.

**Multiplexage** : Plusieurs services sur une même IP publique, différenciés par les ports :

```
203.1.113.123:80    →  172.16.1.15:80   (HTTP)
203.1.113.123:3389  →  172.16.1.15:3389 (RDP)
```

**Usage** : Publication de services internes (HTTP, HTTPS, RDP, FTP…).  
**Risque** : Expose des services vers l'extérieur → nécessite un pare-feu strict et un durcissement des serveurs.

---

### 3.3 NAT 1:1 – Traduction statique

**Définition** : Association **permanente** d'une IP privée à une IP publique dédiée, dans les **deux sens**.

**Caractéristiques** :

| Sens | Mode | Niveau |
|---|---|---|
| Source ET Destination | Statique | IP uniquement |

**Fonctionnement** :

```
[Serveur interne]        [Routeur NAT]           [Internet]
172.16.1.20      ↔    203.1.113.50      ↔    216.58.214.83
```

- **Les ports ne changent pas**
- **Tous les ports** sont accessibles (pas de port forwarding nécessaire)
- Consomme **1 IP publique par machine**

**Usage** : Serveurs en DMZ, rendre un serveur joignable sur Internet avec une identité IP fixe.

---

## 4. Tableau comparatif des 3 types

| Type | Sens | Mode | Niveau | Usage principal |
|---|---|---|---|---|
| **PAT/NAPT** | Source | Dynamique | IP + port | Accès Internet pour les clients |
| **DNAT** | Destination | Statique | IP (+ port) | Publication de services |
| **NAT 1:1** | Source + Destination | Statique | IP | Serveurs en DMZ |

---

## 5. Avantages et inconvénients du NAT

### Avantages

- ✅ Économise les adresses IPv4 publiques
- ✅ Masque le plan d'adressage interne
- ✅ Ajoute une couche de sécurité (les machines internes ne sont pas directement accessibles)

### Inconvénients

| Catégorie | Problème |
|---|---|
| **Technique** | Modification des paquets (L3 et L4) → recalcul des checksums |
| **Compatibilité** | Incompatible avec certains protocoles (ex : FTP actif, IPsec AH) |
| **Architecture** | Brise le principe de **bout en bout** (end-to-end) d'Internet |
| **Intégrité** | Incompatible avec certains contrôles d'intégrité |
| **Services** | Impossible d'héberger un serveur en PAT/NAPT sans DNAT |
| **Blocage collectif** | Si une IP publique est bannie (spam…), **tous les utilisateurs** derrière le NAT sont bloqués |

---

## 6. NAT et IPv6

En **IPv6**, le NAT n'est plus nécessaire car :

- L'espace d'adressage est immense (2¹²⁸ adresses) → plus de pénurie
- Chaque machine peut avoir une adresse publique unique
- Le principe d'**end-to-end** est restauré

> Le **NPTv6** (Network Prefix Translation) existe en IPv6 mais est considéré comme une pratique déconseillée, et non comme une nécessité.

---

## 7. Points clés à retenir

1. Le NAT résout la **pénurie d'adresses IPv4** mais reste une solution transitoire.
2. **PAT/NAPT** : le plus utilisé → une seule IP publique pour de nombreuses machines.
3. **DNAT** : permet de **publier des services** internes depuis Internet (port forwarding).
4. **NAT 1:1** : une IP publique dédiée par serveur, utile en DMZ.
5. Le NAT introduit des **problèmes de compatibilité** avec certains protocoles.
6. Avec **IPv6**, le NAT n'est plus indispensable.

---

_Fiche CP7 – 1/5 | TSSR_
