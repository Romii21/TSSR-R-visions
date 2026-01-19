## Définition

Une **table de routage** est une structure de données présente dans chaque équipement réseau (routeur, ordinateur, serveur) qui détermine **comment acheminer les paquets IP** vers leur destination.

**Principe** : Quand un équipement doit envoyer un paquet, il consulte sa table de routage pour savoir par où l'envoyer.

---

## Structure d'une entrée

Chaque ligne de la table contient au minimum :

|Élément|Description|Exemple|
|---|---|---|
|**Destination**|Réseau de destination (adresse réseau + masque CIDR)|`192.168.1.0/24`|
|**Next hop**|Adresse IP de la passerelle (routeur suivant)|`10.0.0.1`|
|**Interface**|Interface réseau par laquelle envoyer le paquet|`eth0`, `enp0s3`|
|**Métrique**|Coût de la route (plus petit = meilleur)|`100`, `0`|

---

## Fonctionnement

### Processus de décision

1. L'équipement reçoit un paquet avec une adresse de destination
2. Il compare cette destination avec **chaque entrée** de sa table
3. Il applique le masque CIDR pour trouver la correspondance
4. Si plusieurs routes correspondent → il choisit la **plus spécifique** (masque le plus long)
5. Si aucune route ne correspond → il utilise la **route par défaut** (`0.0.0.0/0`)

### Exemple concret

Table de routage :

```
Destination      Next hop      Interface
192.168.1.0/24   -             eth0
10.0.0.0/8       10.0.0.254    eth1
0.0.0.0/0        203.1.113.1   eth2
```

**Cas 1** : Paquet vers `192.168.1.50`

- Correspond à `192.168.1.0/24`
- Envoi direct via `eth0` (même réseau)

**Cas 2** : Paquet vers `10.5.8.100`

- Correspond à `10.0.0.0/8`
- Envoi vers la passerelle `10.0.0.254` via `eth1`

**Cas 3** : Paquet vers `8.8.8.8`

- Ne correspond à aucune route spécifique
- Utilise la route par défaut (`0.0.0.0/0`)
- Envoi vers `203.1.113.1` via `eth2`

---

## Types de routes

|Type|Description|Exemple|
|---|---|---|
|**Route directe**|Réseau accessible sans routeur intermédiaire|Machines du même LAN|
|**Route statique**|Configurée manuellement par l'administrateur|Serveur interne spécifique|
|**Route dynamique**|Apprise automatiquement par protocoles (RIP, OSPF, BGP)|Routes Internet|
|**Route par défaut**|Utilisée quand aucune autre ne correspond (`0.0.0.0/0`)|Accès Internet|

---

## Agrégation de routes

**Problème** : Une table avec des millions d'entrées ralentit le routage

**Solution** : Regrouper plusieurs réseaux en un seul préfixe

### Exemple d'agrégation

Sans agrégation :

```
192.168.0.0/24   via R1
192.168.1.0/24   via R1
192.168.2.0/24   via R1
192.168.3.0/24   via R1
```

Avec agrégation :

```
192.168.0.0/22   via R1
```

→ Couvre les 4 réseaux avec une seule entrée

---

## Commandes pratiques

### Linux

```bash
# Afficher la table IPv4
ip route

# Afficher la table IPv6
ip -6 route

# Ajouter une route
ip route add 192.168.10.0/24 via 10.0.0.1

# Ajouter une route par défaut
ip route add default via 10.0.0.254

# Supprimer une route
ip route del 192.168.10.0/24
```

### Windows

```cmd
# Afficher toutes les tables
route print

# Ajouter une route
route add 192.168.10.0 mask 255.255.255.0 10.0.0.1

# Supprimer une route
route delete 192.168.10.0
```

---

## Points clés

✅ La table de routage indique **où envoyer chaque paquet**  
✅ Chaque équipement possède sa propre table  
✅ La **route la plus spécifique** est toujours choisie en priorité  
✅ La **route par défaut** (`0.0.0.0/0`) sert de filet de sécurité  
✅ L'**agrégation** optimise les performances  
✅ La **métrique** départage plusieurs routes vers la même destination

---

## Acronymes

- **CIDR** : Classless Inter-Domain Routing (notation avec `/24`, `/16`, etc.)
- **LAN** : Local Area Network (réseau local)
- **RIP** : Routing Information Protocol (protocole de routage dynamique)
- **OSPF** : Open Shortest Path First (protocole de routage dynamique)
- **BGP** : Border Gateway Protocol (protocole de routage Internet)