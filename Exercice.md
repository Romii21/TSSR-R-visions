# Exercice IP
## 1. Différence IPv4 et IPv6


| Information                 | IPv4 | IPv6 |
| --------------------------- | ---- | ---- |
| Adresse réseau              | oui  | oui  |
| 1ere adresse disponible     | oui  | oui  |
| Dernière adresse disponible | oui  | oui  |
| Adresse de broadcast        | oui  | -    |
| Nombre d'hôtes              | oui  | oui  |

## 2. Une méthode - La méthode magique

- 4 étapes :

1. Trouver l'octet "actif" -> celui où le masque n'est ni 255 ni 0
2. Calculer la valeur magique = 256 - valeur de l'octet du masque
3. Trouver le bloc réseau pour avoir l'adresse de réseau -> multiple de la valeur magique juste inférieur ou égal à l'hôte
4. À partir de ce résultat en déduire tout le reste (broadcast, plage, hôtes, etc.)


## 3. Adresse IPv4

## a. 192.168.10.75/26

1. Trouver l'octet "actif"
=> CIDR = 26 => Masque : 255.255.255.192
Octet actif : le 4ème (192)

2. Calculer la valeur magique
256 - 192 = 64

3. Trouver le bloc réseau pour avoir l'adresse de réseau
Les blocs sont : 0, 64, 128, 192…
Adresse IP : 192.168.10.75 => octet 4 = 75
75 est entre 64 et 127 => adresse de réseau : 192.168.10.64

4. À partir de ce résultat en déduire tout le reste (broadcast, plage, hôtes, etc.)
Depuis adresse de réseau = 192.168.10.64
1ère adresse : 192.168.10.65

Adresse de broadcast :
Prochain réseau = 64 + 64 -1 = 128 -1 => adresse de broadcast = 192.168.10.127
En enlevant 1 à l'octet actif :
Dernière adresse disponible : 192.168.10.126

Nombre d'hôtes : 
2 ^ (32-CIDR) -2 => 2 ^ (32-26) -2 = 2 ^ 6 -2 = 64-2 = 62

Résultat :
Pour 192.168.10.75/26 :
- Adresse de réseau : 192.168.10.64/26
- 1ère adresse disponible : 192.168.10.65
- Dernière adresse disponible : 192.168.10.126
- Adresse de broadcast : 192.168.10.127
- Nombre d'hôtes : 62

## b. 172.16.45.200/19

1. 224
2. 256 - 224 = 32
3. 172.16.32.0
4. Dans le réseau 172.16.32.0/19 :
	1. Première adresse : 172.16.32.1
	2. Dernière adresse : 172.16.63.254
	3. Broadcast : 172.16.63.255
	4. 2 ^ (32-19) - 2 = 2 ^ 13 - 2 = 8192 - 2 = 8190 hôtes

## c. 10.4.130.17/11

1. 224
2. 256 - 224 = 32
3. 10.0.0.0
4. Dans le réseau 10.0.0.0/11
	1. Première adresse : 10.0.0.1
	2. Dernière adresse : 10.31.255.254
	3. Broadcast : 10.31.255.255
	4. 2 ^ (32-11) - 2 = 2 ^ 21 - 2 = 2 097 152 - 2 = 2 097 150 hôtes
## 4. Adresse IPv6


> [!NOTE] Rappels
> Pas de broadcast en IPv6 (remplacé par le multicast).
> Adresses sur 128 bits en notation hexadécimale par groupes de 16 bits.
> La méthode magique s'applique sur l'octet hexadécimal actif.
> Le nombre d'hôtes peut être très grand ! Donc on peut le laisser en puissance de 2 ou 10.


La méthode magique adaptée à l'IPv6 :

1. Trouver le groupe "actif" -> celui où le préfixe coupe
2. Convertir en binaire si besoin pour trouver les bits réseaux/hôtes
	1. Mettre tous les bits hôtes à 0 -> adresse réseau
	2. Mettre tous les bits hôtes à 1 → dernière adresse dispo (pas de broadcast !)
3. 1ère adresse = réseau + 1, dernière = last − 1
Rmq :
En IPv6 l'adresse réseau elle-même est souvent utilisable (selon les implémentations).

## a. 2001:db8:0:a3::1/48

1. Trouver le groupe "actif" -> celui où le préfixe coupe
/48 = 48 bits de réseau
Les groupes font 16 bits chacun -> 48 bits = 3x16 => 3 groupes complets

2. Partie réseau : 2001 : db8 : 0
Partie hôte : a3 : :1

Donc adresse de réseau : 2001 : db8 : :/48
Et dernière adresse de la plage (mais non-utilisable) : 2001 : db8 : :ffff:ffff:ffff:ffff:ffff

3. 1ère adresse dispo : 2001:db8::1
Dernière adresse dispo : 2001:db8::ffff:ffff:ffff:ffff:fffe

Nb d'hôtes : 2 ^ (128−48) − 2 = 2 ^ 80 égal environ 1,2 × 10 ^ 24 hôtes

Résultat :
Pour 2001:db8:0:a3::1/48 :
- Adresse de réseau : 2001 : db8 : :/48
- 1ère adresse disponible : 2001:db8::1
- Dernière adresse disponible : 2001 : db8 : :ffff:ffff:ffff:ffff:fffe
- Nombre d'hôtes : 1,2 × 10 ^ 24

## b. 2001:db8:abcd:12::80/60

1. 2001:0db8:0000:0000:000 0:0000:0000:0001
   
## c. fe80::1a2b:3c4d:5e6f:1/64


Voici 3 exercices à résoudre avec la méthode magique :

---

## 5. Exo Claude

## Exercice 1

**192.168.50.200/28**

Adresse de réseau = 192.168.50.192/28
1ere dispo = 192.168.50.193
Last dispo = 192.168.50.206
Broadcast = 192.168.50.207
Nb d'hôtes = 14

---

## Exercice 2

**172.31.18.77/21**

Adresse de réseau = 172.31.16.0/21
1ere dispo = 172.31.16.1
Last dispo = 172.31.23.254
Broadcast = 172.31.23.255
Nb d'hôtes = 2046

---

## Exercice 3

**10.0.145.33/13**

Adresse de réseau = 10.0.0.0/13
1ere dispo = 10.0.0.1
Last dispo = 10.7.255.254
Broadcast = 10.7.255.255
Nb d'hôtes = 524 286


---

Pour chaque exercice trouve :

- Adresse de réseau
- 1ère adresse disponible
- Dernière adresse disponible
- Adresse de broadcast
- Nombre d'hôtes

Envoie tes réponses et je corrige !

# Exercice Routage

## 1. Exercice 1

### Des ordinateurs sont connectés sur un switch avec un seul VLAN.

| PC  | Adresse IP   | Masque        |
| --- | ------------ | ------------- |
| PC1 | 192.168.1.10 | 255.255.255.0 |
| PC2 | 192.168.1.20 | 255.255.255.0 |
| PC3 | 192.168.2.30 | 255.255.255.0 |
| PC4 | 192.168.1.40 | 255.255.255.0 |

### Pour chaque ordinateur, indique les communications ICMP réussies.

- Le PC 1 communique avec tout les PC sauf PC3 car il n'est pas sur le même réseau. (/24)
- Le PC 2 communique avec tout les PC sauf PC3 car il n'est pas sur le même réseau. (/24)
- Le PC 4 communique avec tout les PC sauf PC3 car il n'est pas sur le même réseau. (/24)
- Le PC 3 ne communique avec aucune autre machine car il n'est pas sur le même réseau.

## 2. Exercice 2

### Des ordinateurs sont connectés sur un switch avec un seul VLAN.

| PC  | Adresse IP     | Masque        |
| --- | -------------- | ------------- |
| PC1 | 192.168.10.8   | 255.255.255.0 |
| PC2 | 192.168.10.12  | 255.255.0.0   |
| PC3 | 192.168.11.9   | 255.255.255.0 |
| PC4 | 192.168.10.50. | 255.255.255.0 |

### Pour chaque ordinateur, indique les communications ICMP réussies.

- Le PC 1 communique avec le PC 4, ils sont sur le même réseau, il ne peut communiquer avec PC 2 et PC 3 ne font pas partis de sa plage réseau
- Le PC2