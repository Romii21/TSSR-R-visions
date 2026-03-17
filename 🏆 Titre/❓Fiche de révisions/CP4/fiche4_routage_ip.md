# FICHE 4 — Routage IP
### CP4 : Exploiter un réseau IP | Formation TSSR

---

## 1. Principe du routage IP

Le routage IP consiste à **acheminer les paquets IP d'un réseau à un autre** en prenant des décisions basées sur l'adresse IP de destination.

### Switch vs Routeur — différence fondamentale

| | **Switch** | **Routeur** |
|-|-----------|-------------|
| Couche OSI | **2** (Liaison) | **3** (Réseau) |
| Identifiant utilisé | **Adresse MAC** | **Adresse IP** |
| Table de référence | **Table CAM** | **Table de routage** |
| Portée | **Un seul réseau** (LAN) | **Plusieurs réseaux** |
| Broadcast | Transmet les broadcasts | **Bloque** les broadcasts |

---

## 2. La table de routage

### Structure d'une entrée

| Champ | Description | Exemple |
|-------|-------------|---------|
| **Réseau de destination** | Adresse du réseau cible | 192.168.5.0 |
| **Masque** | Définit la plage du réseau | /24 ou 255.255.255.0 |
| **Passerelle (next-hop)** | IP du routeur suivant | 10.0.0.1 |
| **Interface** | Interface locale à utiliser | eth0, enp0s3 |
| **Métrique** | Coût de la route (plus bas = préféré) | 100 |
| **Source** | Comment la route a été apprise | C, S, O, R… |

### Codes de source dans la table de routage

| Code | Signification |
|------|--------------|
| **C** | Connected — réseau directement connecté à l'interface |
| **L** | Local — adresse de l'interface elle-même |
| **S** | Static — route configurée manuellement |
| **O** | OSPF — route apprise via le protocole OSPF |
| **R** | RIP — route apprise via le protocole RIP |
| **B** | BGP — route apprise via le protocole BGP |

### La route par défaut
```
0.0.0.0/0 via 192.168.1.254
```
- Correspond à **toutes les destinations** non listées dans la table
- Utilisée pour aller vers Internet
- Appelée "**gateway of last resort**"
- Si aucune autre route ne correspond → on envoie vers la route par défaut

---

## 3. La métrique

La métrique est une **valeur numérique représentant le coût d'une route**.

**Règle : plus la métrique est basse, plus la route est préférée.**

### Utilité
Si deux routes existent vers la même destination, la plus faible métrique gagne.

```
Route 1 : 192.168.5.0/24 via 10.0.0.1  métrique 10  ← PRÉFÉRÉE
Route 2 : 192.168.5.0/24 via 10.0.0.2  métrique 100 ← route de secours
```

Si Route 1 tombe, Route 2 prend automatiquement le relais → **redondance et basculement automatique**.

### Métrique selon les protocoles
| Protocole | Critère de métrique |
|-----------|-------------------|
| RIP | Nombre de **sauts** |
| OSPF | **Coût** (basé sur la bande passante) |
| EIGRP | Combinaison bande passante + délai |
| Statique | Valeur **configurée manuellement** |

---

## 4. Routage statique vs dynamique

### Routage statique
Routes **configurées manuellement** par l'administrateur.

| Avantages | Inconvénients |
|-----------|--------------|
| Simple et prédictible | Lourd à maintenir sur grands réseaux |
| Pas de consommation CPU/bande passante | Ne s'adapte pas aux pannes automatiquement |
| Sécurisé (pas d'annonces réseau) | Configuration à reprendre à chaque changement |

**Usage :** Petits réseaux, routes par défaut, liens WAN simples.

### Routage dynamique
Les routeurs **échangent automatiquement** leurs informations de routage via un protocole.

| Avantages | Inconvénients |
|-----------|--------------|
| S'adapte automatiquement aux pannes | Plus complexe à configurer |
| Idéal pour les grands réseaux | Consomme CPU, RAM et bande passante |
| Convergence automatique | Risque de boucles de routage si mal configuré |

**Usage :** Grandes infrastructures, réseaux redondants, Internet.

---

## 5. Les protocoles de routage dynamique

### Vue d'ensemble

| Protocole | Type | Métrique | Transport | Usage |
|-----------|------|----------|-----------|-------|
| **RIP** | IGP — vecteur de distance | Nb de sauts (max **15**) | UDP/**520** | Petits réseaux simples |
| **OSPF** | IGP — état de lien | Coût (bande passante) | IP direct (**protocole 89**) | Réseaux moyens/grands |
| **EIGRP** | IGP — hybride (Cisco) | Bande passante + délai | IP direct (**protocole 88**) | Réseaux Cisco complexes |
| **BGP** | EGP — entre AS | Attributs de chemin (AS-PATH…) | TCP/**179** | **Internet** (entre FAI) |

### Terminologie
- **IGP** (Interior Gateway Protocol) = protocole utilisé **à l'intérieur** d'un même système autonome (entreprise)
- **EGP** (Exterior Gateway Protocol) = protocole utilisé **entre systèmes autonomes** (Internet)
- **AS** (Autonomous System) = ensemble de réseaux sous une même administration

### Détails par protocole

**RIP (Routing Information Protocol)**
- Simple à configurer
- Limite de **15 sauts** maximum → inutilisable sur grands réseaux
- Convergence lente
- 3 versions : RIPv1, RIPv2 (IPv4), RIPng (IPv6)

**OSPF (Open Shortest Path First)**
- Algorithme de Dijkstra (chemin le plus court)
- Converge rapidement
- Supporte le VLSM et CIDR
- Très utilisé en entreprise
- Organisé en **zones** (area) pour optimiser les échanges

**BGP (Border Gateway Protocol)**
- Protocole de routage d'Internet
- Gère les routes entre les FAI et les grands réseaux
- Très complexe, basé sur des politiques
- Utilise TCP/179

---

## 6. Commandes Linux — ip route

```bash
# Afficher la table de routage IPv4
ip route
ip route show
route -n   # ancienne commande (encore présente)

# Afficher la table de routage IPv6
ip -6 route

# Ajouter une route statique vers 192.168.5.0/24 via 10.0.0.1
ip route add 192.168.5.0/24 via 10.0.0.1

# Ajouter une route statique avec interface spécifiée
ip route add 192.168.5.0/24 via 10.0.0.1 dev eth0

# Ajouter une route par défaut
ip route add default via 10.0.0.254

# Supprimer une route
ip route del 192.168.5.0/24

# Supprimer la route par défaut
ip route del default

# Vérifier par quelle route passe un paquet
ip route get 8.8.8.8
```

> **Attention :** Ces routes sont **temporaires** (perdues au redémarrage). Pour les rendre persistantes, il faut les ajouter dans `/etc/network/interfaces` ou `/etc/netplan/` selon la distribution.

---

## 7. Commandes Windows — route

``` powershell
# Afficher la table de routage
route print

# Afficher uniquement IPv4
route print -4

# Ajouter une route statique vers 192.168.5.0/24 via 10.0.0.1
route add 192.168.5.0 mask 255.255.255.0 10.0.0.1

# Ajouter une route persistante (survit au redémarrage)
route add -p 192.168.5.0 mask 255.255.255.0 10.0.0.1

# Ajouter une route par défaut
route add 0.0.0.0 mask 0.0.0.0 10.0.0.254

# Supprimer une route
route delete 192.168.5.0

# Modifier une route existante
route change 192.168.5.0 mask 255.255.255.0 10.0.0.2
```

> **Sans le `-p`**, la route est **temporaire** et disparaît au redémarrage.

---

## 8. Lecture d'une table de routage Linux

```bash
$ ip route
default via 192.168.1.1 dev eth0 proto dhcp metric 100
192.168.1.0/24 dev eth0 proto kernel scope link src 192.168.1.50
10.0.0.0/8 via 192.168.1.254 dev eth0

   ↑                    ↑         ↑          ↑
Destination          Passerelle  Interface  Source/Infos
```

| Ligne | Signification |
|-------|--------------|
| `default via 192.168.1.1` | Route par défaut → tout ce qui ne correspond pas va vers 192.168.1.1 |
| `192.168.1.0/24 dev eth0` | Réseau directement connecté sur eth0 |
| `10.0.0.0/8 via 192.168.1.254` | Route statique ou dynamique vers le réseau 10.x.x.x |

---

## 9. Points clés à retenir ✅

- Le **routeur bloque les broadcasts** — le switch les transmet
- La table de routage contient : destination, masque, **next-hop**, interface, métrique
- La **route par défaut** = `0.0.0.0/0` → utilisée si aucune autre route ne correspond
- **Métrique basse = route préférée**
- Routage statique : manuel, simple, **ne s'adapte pas aux pannes**
- Routage dynamique : automatique, s'adapte, **consomme des ressources**
- **RIP** = max 15 sauts, UDP/520
- **OSPF** = coût bande passante, IP protocole 89, très utilisé en entreprise
- **BGP** = entre FAI/AS sur Internet, TCP/179
- Linux : `ip route add/del` | Windows : `route add/delete (-p pour persistant)`

---

*Fiche 4/6 — CP4 Exploiter un réseau IP*
