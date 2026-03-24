# CORRECTION — CP4 : Exploiter un réseau IP

### Formation TSSR

> **Légende :**
> 
> - ✅ Bonne réponse
> - ⚠️ Réponse partiellement correcte — à compléter
> - ❌ Réponse incorrecte ou manquante

---

## PARTIE 1 — Fondamentaux réseau

---

### Q1 — Différence LAN / WAN

**Statut : ⚠️ Partiellement correct**

Ta réponse confond LAN/WAN avec adresses privées/publiques. Ce n'est pas la même chose.

**Correction :**

- **LAN (Local Area Network)** : réseau local, limité géographiquement à un bâtiment ou un site. Les machines communiquent directement entre elles via un switch. _Exemple : le réseau d'une entreprise, avec des PC, imprimantes, serveurs reliés en interne._
- **WAN (Wide Area Network)** : réseau étendu qui interconnecte plusieurs LAN à grande distance, via des opérateurs télécom. _Exemple : Internet, ou la liaison entre deux agences d'une entreprise via une ligne MPLS._

> ⚠️ Les plages 10.x, 172.16.x, 192.168.x sont des **adresses privées**, mais elles définissent l'espace d'adressage utilisé à l'intérieur d'un LAN, pas le LAN lui-même.

---

### Q2 — Couches OSI des équipements

**Statut : ✅ Correct**

Bonne réponse.

- Hub → Couche 1 (Physique)
- Switch → Couche 2 (Liaison de données)
- Routeur → Couche 3 (Réseau)

> **Complément :** Le switch L3 peut aussi travailler en couche 3 (routage inter-VLAN). Les pare-feux modernes travaillent jusqu'en couche 7 (Application).

---

### Q3 — Adresse MAC

**Statut : ✅ Correct**

Bonne réponse. L'adresse MAC est bien codée sur **6 octets (48 bits)**, en hexadécimal, sous la forme `FF:FF:FF:FF:FF:FF`.

> **Complément :** Les 3 premiers octets identifient le **fabricant (OUI — Organizationally Unique Identifier)**, les 3 derniers sont l'identifiant unique de l'interface. Ex : `00:1A:2B` peut identifier Intel ou Cisco.

---

### Q4 — Protocole ARP

**Statut : ❌ Réponse incomplète**

Ta réponse s'arrête au milieu et n'explique pas le mécanisme.

**Correction :** ARP (Address Resolution Protocol) permet de **résoudre une adresse IP en adresse MAC**. Il est déclenché lorsqu'une machine veut communiquer avec une autre sur le **même réseau local** et qu'elle ne connaît pas encore son adresse MAC.

**Mécanisme :**

1. La machine source envoie un **broadcast ARP** : _"Qui a l'IP 192.168.1.10 ? Réponds-moi à 00:AA:BB:CC:DD:EE"_
2. La machine cible répond en **unicast** avec son adresse MAC
3. La source stocke cette correspondance dans sa **table ARP** (cache)

> **Commande utile :** `arp -a` (Windows/Linux) pour afficher la table ARP locale.

---

### Q5 — Encapsulation TCP/IP

**Statut : ⚠️ Idée correcte mais trop vague**

**Correction :** L'encapsulation est le principe par lequel chaque couche **ajoute son propre en-tête (header)** aux données transmises par la couche supérieure.

|Couche|PDU|En-tête ajouté|
|---|---|---|
|Application|Données|—|
|Transport|Segment|Port source/destination (TCP/UDP)|
|Réseau (Internet)|Paquet|IP source/destination|
|Liaison (MAC)|Trame|MAC source/destination|
|Physique|Bits|Signal électrique/optique|

À la réception, chaque couche **désencapsule** (retire son en-tête) et passe les données à la couche supérieure.

---

### Q6 — Protocoles applicatifs courants

**Statut : ⚠️ Partiellement correct — erreur sur DHCP**

**Correction :**

|Protocole|Port|Transport|Rôle|
|---|---|---|---|
|**DHCP**|UDP 67 (serveur) / UDP 68 (client)|**UDP** ❌ (tu as dit TCP)|Attribution d'adresses IP|
|**DNS**|UDP/TCP 53|UDP (principalement)|Résolution de noms|
|**SSH**|TCP 22|TCP|Accès distant sécurisé|
|**RDP**|TCP 3389|TCP|Bureau à distance Windows|

> ⚠️ **DHCP utilise UDP**, pas TCP. Le client n'a pas encore d'IP pour établir une connexion TCP fiable. C'est pour ça qu'il utilise le broadcast en UDP.

> **Bonus à connaître :** HTTP=TCP 80, HTTPS=TCP 443, FTP=TCP 21, SMTP=TCP 25

---

## PARTIE 2 — Adressage IPv4

---

### Q7 — Codage de l'adresse IPv4

**Statut : ⚠️ Partiellement correct — erreur de calcul**

**Correction :** Une adresse IPv4 est codée sur **32 bits**.  
Le nombre d'adresses théoriques est **2³² = 4 294 967 296** (environ 4,3 milliards).

> ❌ Tu as écrit "32²" ce qui donnerait seulement 1024. La bonne formule est **2 puissance 32**.

---

### Q8 — Notation CIDR

**Statut : ✅ Correct**

Bonne réponse. Le CIDR (Classless Inter-Domain Routing) indique le nombre de bits à 1 dans le masque. `/24` = `255.255.255.0` = 256 adresses totales, dont 254 utilisables (on retire l'adresse réseau et le broadcast).

---

### Q9 — Réseau 192.168.10.0/24

**Statut : ✅ Correct**

Toutes les réponses sont correctes :

- Réseau : 192.168.10.0
- Masque : 255.255.255.0
- Broadcast : 192.168.10.255
- Plage hôtes : 192.168.10.1 → 192.168.10.254
- Nombre d'hôtes : 254 (2⁸ - 2)

---

### Q10 — Réseau 172.16.0.0/20

**Statut : ❌ Broadcast incorrect**

**Correction :**

- Masque /20 = 255.255.**240**.0 ✅
- Nombre d'hôtes = 2¹² - 2 = 4094 ✅
- **Broadcast = 172.16.15.255** ❌ (tu as mis 172.16.255.255 qui serait le broadcast d'un /16)

**Calcul :**

```
172.16.0.0  en binaire pour le 3ème octet : 00000000
Masque /20  → les 20 premiers bits sont réseau
3ème octet du masque : 11110000 = 240

Bits hôte disponibles dans le 3ème octet : 0000 (4 bits)
→ max = 1111 = 15

Broadcast = 172.16.15.255
```

---

### Q11 — Plages d'adresses privées (RFC 1918)

**Statut : ⚠️ Erreur sur la plage 172.16.0.0**

**Correction :**

|Plage|CIDR correct|
|---|---|
|10.0.0.0|**/8** ✅|
|172.16.0.0|**/12** ❌ (tu as écrit /20)|
|192.168.0.0|**/16** ✅|

> ⚠️ La plage 172.16.0.0/**12** couvre de 172.16.0.0 à 172.31.255.255. Le /20 serait un sous-réseau à l'intérieur de cette plage, pas la plage complète.

---

### Q12 — Adresse 127.0.0.1

**Statut : ✅ Correct — à compléter**

Bonne réponse. C'est l'adresse de **loopback**, aussi appelée **localhost**, qui représente la machine elle-même.

> **Complément :** On peut tester la pile réseau avec `ping 127.0.0.1`. Si ça répond, la pile TCP/IP locale fonctionne. L'interface s'appelle **lo** (Linux) ou **Loopback Adapter** (Windows). La plage complète de loopback est `127.0.0.0/8`.

---

### Q13 — Machine 10.0.5.42/22

**Statut : ❌ Non répondu**

**Correction :**

**/22 = masque 255.255.252.0**

```
Adresse :  10.0.5.42
           00001010 . 00000000 . 00000101 . 00101010

Masque /22:
           11111111 . 11111111 . 11111100 . 00000000
           255      . 255      . 252      . 0

AND bit à bit :
           00001010 . 00000000 . 00000100 . 00000000
           = 10.0.4.0
```

- **Adresse réseau :** 10.0.4.0
- **Broadcast :** 10.0.7.255 (les 10 bits hôtes tous à 1 : 00000111.11111111)
- **Plage hôtes :** 10.0.4.1 → 10.0.7.254
- **Nombre d'hôtes :** 2¹⁰ - 2 = 1022

> **Astuce :** Pour un /22, le 3ème octet varie par blocs de 4 (256 - 252 = 4). Ici 10.0.**4**.0 → 10.0.**7**.255.

---

### Q14 — VLSM

**Statut : ❌ Non répondu**

**Correction :** Le **VLSM (Variable Length Subnet Masking)** est une technique de découpage en sous-réseaux où **chaque sous-réseau peut avoir un masque différent**, adapté au nombre réel d'hôtes nécessaires.

**Avantage vs découpage classique :** Le découpage classique attribue la même taille à tous les sous-réseaux, ce qui gaspille des adresses. Avec le VLSM, on optimise l'espace d'adressage en donnant à chaque réseau exactement la taille dont il a besoin.

_Exemple :_ Pour un lien WAN entre 2 routeurs, on utilise un /30 (2 hôtes) au lieu d'un /24 (254 hôtes).

---

## PARTIE 3 — Protocoles de support IPv4

---

### Q15 — Protocole ICMP

**Statut : ⚠️ Trop simplifié**

**Correction :** ICMP (Internet Control Message Protocol) est un **protocole de diagnostic et de signalisation réseau**. Il ne transporte pas de données applicatives mais des **messages de contrôle**.

**Exemples de messages ICMP :**

- **Echo Request / Echo Reply** : utilisés par `ping` pour tester la connectivité et mesurer la latence
- **Destination Unreachable** : informe l'émetteur qu'un hôte ou un réseau est inaccessible
- **Time Exceeded** : envoyé quand le TTL d'un paquet tombe à 0 (utilisé par `traceroute`)
- **Redirect** : signale à un hôte qu'il doit utiliser un autre routeur

---

### Q16 — Ping : "Destination host unreachable" vs "Request timed out"

**Statut : ⚠️ Partiellement correct**

**Correction :**

- **"Destination host unreachable"** : un équipement **intermédiaire** (souvent le routeur local ou la passerelle) répond qu'il ne sait pas comment atteindre la destination. Il génère lui-même ce message ICMP. → _La machine n'est pas joignable depuis ton réseau._
- **"Request timed out"** : le paquet est parti mais **aucune réponse n'est revenue** dans le délai imparti. La cible existe peut-être mais ne répond pas (firewall, machine éteinte, perte de paquet). → _Le paquet est envoyé mais aucun retour._

> ⚠️ Dans les deux cas, ça ne prouve pas forcément que la machine n'est pas connectée à Internet. Le problème peut venir d'un firewall, d'une route manquante, d'un host éteint, etc.

---

### Q17 — DHCP et séquence DORA

**Statut : ⚠️ Partiellement correct — erreur sur le "R"**

**Correction :**

- **D — Discover** : le client envoie un **broadcast** (255.255.255.255) pour trouver un serveur DHCP ✅
- **O — Offer** : le serveur propose une configuration IP disponible ✅
- **R — Request** : le client **demande officiellement** l'adresse proposée (encore en broadcast, car il peut y avoir plusieurs serveurs) ❌ _(tu as écrit "Receive" — le R est bien "Request")_
- **A — Acknowledge** : le serveur confirme l'attribution et envoie la configuration complète ✅

> ⚠️ Le **R** est **Request**, pas "Receive". C'est une étape active : le client choisit une offre et la réclame formellement.

---

### Q18 — Bail DHCP

**Statut : ✅ Correct — à compléter**

Bonne compréhension. Le bail DHCP est la **durée pendant laquelle une adresse IP est réservée** pour un client.

> **Complément :** La demande de renouvellement se fait généralement à **50% de la durée du bail** (T1), puis à **87,5%** (T2). Si le renouvellement échoue, le client tente de contacter n'importe quel serveur DHCP. Si aucun serveur ne répond, l'adresse est libérée et le client en 169.254.x.x (APIPA).

---

### Q19 — Réservation DHCP

**Statut : ✅ Correct**

Bonne réponse. La réservation DHCP lie une adresse MAC à une IP fixe, garantissant que la machine recevra toujours la même adresse. Utile pour les serveurs, imprimantes, équipements réseau qu'on veut joindre à une IP fixe sans configuration manuelle.

---

## PARTIE 4 — DNS

---

### Q20 — Rôle du DNS

**Statut : ⚠️ Trop vague — manque la description du processus complet**

**Correction :** Le DNS (Domain Name System) traduit des **noms de domaine lisibles** en **adresses IP**.

**Processus complet lors d'une saisie de www.google.com :**

1. Le navigateur vérifie son **cache DNS local**
2. Si absent, interroge le **fichier hosts** local
3. Si absent, interroge le **résolveur DNS** configuré (ex: 8.8.8.8)
4. Le résolveur interroge un **serveur DNS racine** (root server)
5. Le root server redirige vers le **serveur TLD** (.com)
6. Le serveur TLD redirige vers le **serveur DNS autoritaire** de google.com
7. Ce serveur retourne l'IP correspondant à www.google.com
8. Le résolveur renvoie l'IP au client et la met en cache
9. Le navigateur établit une connexion TCP (Three-Way Handshake) vers cette IP

---

### Q21 — Types d'enregistrements DNS

**Statut : ⚠️ Incomplet — manquent MX et PTR**

**Correction :**

|Type|Rôle|Exemple|
|---|---|---|
|**A**|Nom → IPv4|`www.exemple.com → 93.184.216.34`|
|**AAAA**|Nom → IPv6|`www.exemple.com → 2606:2800::1`|
|**CNAME**|Alias vers un autre nom|`mail.exemple.com → ghs.google.com`|
|**MX**|Serveur de messagerie du domaine|`exemple.com → mail.exemple.com (priorité 10)`|
|**PTR**|IP → Nom (résolution inverse)|`34.216.184.93.in-addr.arpa → www.exemple.com`|

---

### Q22 — Résolution récursive vs itérative

**Statut : ❌ Non répondu**

**Correction :**

- **Résolution récursive** : le client demande une résolution complète à son serveur DNS. Ce serveur prend en charge toutes les requêtes intermédiaires et retourne le résultat final. _C'est ce que fait votre box Internet ou le DNS de votre entreprise._
- **Résolution itérative** : le serveur DNS répond au client avec une référence vers un autre serveur à interroger. Le client effectue lui-même chaque étape jusqu'à obtenir la réponse. _C'est ce que font les serveurs DNS racines entre eux._

> En pratique : votre PC fait une requête **récursive** à son résolveur, qui effectue des requêtes **itératives** vers les serveurs DNS autoritaires.

---

### Q23 — Outils de requête DNS

**Statut : ✅ Correct — à compléter**

Bonne réponse.

- `nslookup google.com` (Windows et Linux)
- `dig google.com` (Linux, plus détaillé)

> **Exemples utiles :**
> 
> - `dig google.com A` → cherche l'enregistrement A
> - `dig -x 8.8.8.8` → résolution inverse (PTR)
> - `nslookup -type=MX google.com` → cherche les enregistrements MX

---

## PARTIE 5 — Routage IP

---

### Q24 — Switch vs Routeur

**Statut : ⚠️ Partiellement correct — erreur sur le switch**

**Correction :**

- **Switch (Couche 2)** : relie des machines **dans le même réseau local**. Il commute les trames en utilisant les **adresses MAC** et sa **table CAM** (Content Addressable Memory), pas ARP. _La table ARP est une table des hôtes, pas du switch._
- **Routeur (Couche 3)** : interconnecte **des réseaux différents** et achemine les paquets IP en utilisant sa **table de routage** en se basant sur les **adresses IP de destination**.

---

### Q25 — Table de routage

**Statut : ✅ Correct — à compléter**

Bonne réponse. La table de routage contient :

- **Destination** : réseau ou hôte cible
- **Masque** : pour identifier la plage réseau
- **Next-hop (passerelle)** : adresse du prochain routeur
- **Interface de sortie** : par quelle interface le paquet doit partir
- **Métrique** : coût de la route (plus la métrique est basse, plus la route est préférée)
- **Type de route** : C (Connected), S (Static), O (OSPF), R (RIP)...

---

### Q26 — Route par défaut

**Statut : ✅ Correct**

Bonne réponse. La route par défaut (0.0.0.0/0) est utilisée quand **aucune autre route** dans la table ne correspond à la destination. Elle pointe généralement vers le routeur de l'opérateur Internet (passerelle FAI).

---

### Q27 — Commandes Linux (table de routage)

**Statut : ❌ Incomplet**

**Correction :**

```bash
# Afficher la table de routage IPv4
ip route show
# ou : route -n (ancienne commande)

# Ajouter une route statique
ip route add 192.168.5.0/24 via 10.0.0.1

# Ajouter une route par défaut
ip route add default via 10.0.0.254

# Supprimer une route
ip route del 192.168.5.0/24
```

> **Note :** Ces commandes sont temporaires. Pour les rendre persistantes, il faut les écrire dans `/etc/network/interfaces` ou via `Netplan` (Ubuntu moderne).

---

### Q28 — Commandes Windows (table de routage)

**Statut : ❌ Non répondu**

**Correction :**

```cmd
# Afficher la table de routage
route print
# ou PowerShell : Get-NetRoute

# Ajouter une route statique
route add 192.168.5.0 mask 255.255.255.0 10.0.0.1

# Ajouter une route par défaut
route add 0.0.0.0 mask 0.0.0.0 10.0.0.254

# Supprimer une route
route delete 192.168.5.0

# Rendre une route persistante (survit au redémarrage)
route add -p 192.168.5.0 mask 255.255.255.0 10.0.0.1
```

---

### Q29 — Routage statique vs dynamique

**Statut : ❌ Non répondu**

**Correction :**

||**Routage statique**|**Routage dynamique**|
|---|---|---|
|**Principe**|Routes configurées manuellement|Routes apprises automatiquement via un protocole|
|**Avantage**|Simple, prévisible, économe en ressources, sécurisé|S'adapte automatiquement aux pannes et changements|
|**Inconvénient**|Ne s'adapte pas aux pannes, lourd à maintenir sur grands réseaux|Consomme CPU, RAM et bande passante|
|**Usage**|Petits réseaux stables, routes par défaut|Grands réseaux complexes (entreprises, ISP)|

---

### Q30 — Protocoles de routage dynamique

**Statut : ❌ Non répondu**

**Correction :**

|Protocole|Contexte|Métrique|Transport|
|---|---|---|---|
|**RIP** (Routing Info Protocol)|Petits réseaux, obsolète|Nombre de sauts (max 15)|UDP 520|
|**OSPF** (Open Shortest Path First)|Réseaux d'entreprise intra-domaine|Coût (basé sur bande passante)|IP direct (protocole 89)|
|**EIGRP** (Enhanced Interior Gateway RP)|Réseaux Cisco intra-domaine|Composite (bande passante + délai)|IP direct (protocole 88)|
|**BGP** (Border Gateway Protocol)|Interconnexion FAI/Internet (inter-domaine)|AS Path (chemin d'AS traversés)|TCP 179|

---

## PARTIE 6 — Protocoles TCP et UDP

---

### Q31 — Différences TCP / UDP

**Statut : ⚠️ Partiellement correct — erreur sur DHCP**

**Correction :**

|Critère|TCP|UDP|
|---|---|---|
|Connexion|Orienté connexion (3-Way Handshake)|Sans connexion|
|Fiabilité|Accusés de réception, retransmission|Aucune garantie|
|Ordre|Numéros de séquence garantissent l'ordre|Non garanti|
|Vitesse|Plus lent (overhead)|Plus rapide|

- **TCP :** HTTP/HTTPS, SSH, FTP, SMTP, RDP
- **UDP :** DNS, DHCP, streaming vidéo, VoIP, jeux en ligne

> ❌ **DHCP utilise UDP** (ports 67/68), pas TCP. Tu l'avais aussi dit en Q6.

---

### Q32 — Three-Way Handshake TCP

**Statut : ✅ Correct — à compléter**

Les 3 flags sont corrects : SYN → SYN-ACK → ACK.

**Complément :**

```
Client                          Serveur
  │                                │
  │  1. SYN (Seq=x)               │  → "Je veux me connecter"
  │───────────────────────────────>│
  │                                │
  │  2. SYN+ACK (Seq=y, Ack=x+1) │  → "OK, je suis prêt, à toi"
  │<───────────────────────────────│
  │                                │
  │  3. ACK (Ack=y+1)             │  → "Connexion établie"
  │───────────────────────────────>│
```

Ce mécanisme garantit que **les deux parties sont prêtes** à communiquer et **synchronise les numéros de séquence** avant l'échange de données.

---

### Q33 — Numéros de port

**Statut : ❌ Incomplet**

**Correction :**

|Plage|Type|Exemples|
|---|---|---|
|**0 – 1023**|Ports système / Well Known|HTTP (80), HTTPS (443), SSH (22), DNS (53)|
|**1024 – 49151**|Ports enregistrés / Registered|RDP (3389), MySQL (3306), PostgreSQL (5432)|
|**49152 – 65535**|Ports dynamiques / Éphémères|Ports attribués temporairement aux clients lors d'une connexion|

> Le port **source** d'une requête client est généralement tiré aléatoirement dans la plage dynamique. Le port **destination** est le port bien connu du service (ex: 443 pour HTTPS).

---

### Q34 — Fenêtre TCP (Window Size)

**Statut : ❌ Non répondu**

**Correction :** La **fenêtre TCP (Window Size)** définit le nombre d'octets que l'émetteur peut envoyer **sans attendre un accusé de réception**.

**Problème résolu :** Sans fenêtre, l'émetteur devrait attendre un ACK après chaque segment, ce qui serait très lent sur des liaisons à haute latence. La fenêtre permet d'envoyer **plusieurs segments en rafale** et d'optimiser l'utilisation de la bande passante.

Si le récepteur est saturé, il **réduit la taille de fenêtre annoncée** pour ralentir l'émetteur → c'est le **contrôle de flux TCP**.

---

### Q35 — Fermeture d'une connexion TCP

**Statut : ❌ Non répondu**

**Correction :** La fermeture TCP utilise **4 échanges (4-Way Teardown)** avec les flags **FIN** et **ACK** :

```
Client                          Serveur
  │  1. FIN                        │  → "Je n'ai plus rien à envoyer"
  │───────────────────────────────>│
  │  2. ACK                        │  → "Reçu, attends que je finisse"
  │<───────────────────────────────│
  │  3. FIN                        │  → "Moi aussi j'ai fini"
  │<───────────────────────────────│
  │  4. ACK                        │  → "OK, fermeture confirmée"
  │───────────────────────────────>│
```

> Chaque sens de communication se ferme indépendamment → c'est une **fermeture gracieuse**. Le flag **RST** provoque quant à lui une **fermeture immédiate (brutale)**.

---

## PARTIE 7 — NAT

---

### Q36 — Pourquoi le NAT ?

**Statut : ✅ Correct**

Bonne réponse. Le NAT (Network Address Translation) a été créé pour pallier la **pénurie d'adresses IPv4 publiques**. Il permet à tout un réseau privé (utilisant des adresses RFC 1918) de partager **une seule adresse IP publique**.

---

### Q37 — PAT/NAPT vs DNAT vs NAT statique 1:1

**Statut : ❌ Non répondu**

**Correction :**

|Type|Mécanisme|Cas d'usage|
|---|---|---|
|**PAT / NAPT** (masquerade)|Plusieurs IP privées → 1 IP publique, différenciées par les **ports**|Accès Internet des postes internes (box FAI)|
|**DNAT** (Destination NAT)|Redirige les connexions entrantes vers un serveur interne|Publier un serveur web interne sur Internet|
|**NAT 1:1** (statique)|1 IP privée ↔ 1 IP publique fixe|Serveur nécessitant une IP publique dédiée (VoIP, partenaire)|

---

### Q38 — Flux NAT/PAT

**Statut : ⚠️ Bonne idée générale mais quelques imprécisions**

**Correction détaillée :**

1. PC interne (192.168.1.10, port source 50432) envoie une requête vers 142.250.74.46:443 (Google)
2. Le routeur NAT **traduit** : remplace 192.168.1.10:50432 par son **IP publique** (ex: 82.64.31.5:50432) et note la correspondance dans sa **table NAT**
3. Le paquet arrive chez Google qui voit 82.64.31.5:50432
4. Google répond à 82.64.31.5:50432
5. Le routeur reçoit la réponse, consulte sa table NAT, **retrouve** que ce port correspond à 192.168.1.10:50432 et retransmet le paquet en interne

> ⚠️ C'est le **port source** du client qui permet au NAT d'identifier quel hôte interne a initié la connexion, pas un port fixe "laissé ouvert".

---

## PARTIE 8 — IPv6

---

### Q39 — Adresse IPv6

**Statut : ✅ Correct**

IPv6 est codée sur **128 bits**, représentée en **hexadécimal** par groupes de 16 bits séparés par `:`.

**Règles de simplification :**

1. Supprimer les **zéros non significatifs** dans un groupe : `0042` → `42`
2. Remplacer **une seule séquence** de groupes nuls consécutifs par `::` : `2001:0db8:0000:0000:0000:0000:0000:0001` → `2001:db8::1`

---

### Q40 — Types d'adresses IPv6

**Statut : ❌ Non répondu**

**Correction :**

|Type|Préfixe|Usage|
|---|---|---|
|**Link-local**|`FE80::/10`|Communication sur le lien local uniquement (non routable)|
|**Global Unicast**|`2000::/3` (souvent `2001::/32`)|Adresses publiques routables sur Internet|
|**Multicast**|`FF00::/8`|Envoi à un groupe de machines (remplace le broadcast)|
|**Unique Local**|`FC00::/7`|Équivalent IPv6 des adresses privées RFC 1918|

---

### Q41 — SLAAC

**Statut : ❌ Non répondu**

**Correction :** **SLAAC (Stateless Address Autoconfiguration)** permet à une machine de **se configurer automatiquement** en IPv6 **sans serveur DHCP**.

**Mécanisme :**

1. La machine génère une **adresse link-local** (FE80::/10) à partir de son adresse MAC (méthode EUI-64)
2. Elle envoie un **Router Solicitation (RS)** en multicast
3. Le routeur répond avec un **Router Advertisement (RA)** contenant le préfixe réseau
4. La machine construit son **adresse globale** : préfixe (64 bits) + identifiant d'interface (64 bits, dérivé de la MAC via EUI-64)

---

### Q42 — Remplacement d'ARP en IPv6

**Statut : ❌ Non répondu**

**Correction :** En IPv6, ARP est remplacé par le **NDP (Neighbor Discovery Protocol)**, qui utilise des messages **ICMPv6**.

Il utilise notamment :

- **Neighbor Solicitation (NS)** : équivalent de l'ARP Request
- **Neighbor Advertisement (NA)** : équivalent de l'ARP Reply
- **Router Advertisement (RA)** / **Router Solicitation (RS)** : pour la découverte des routeurs et SLAAC

---

### Q43 — Avantages d'IPv6 vs IPv4

**Statut : ❌ Non répondu**

**Correction :**

1. **Espace d'adressage quasi illimité** : 2¹²⁸ adresses (≈ 340 sextillions), plus besoin de NAT
2. **Autoconfiguration (SLAAC)** : les hôtes peuvent se configurer sans DHCP
3. **En-tête simplifié** : traitement plus rapide par les routeurs
4. **Sécurité intégrée** : IPsec est natif en IPv6
5. **Multicast natif** : remplace le broadcast, plus efficace
6. **Meilleure mobilité** : protocoles de mobilité intégrés

---

## PARTIE 9 — Mise en situation et diagnostic

---

### Q44 — Diagnostic : accès Internet impossible, LAN OK

**Statut : ⚠️ Trop vague — manque une méthodologie structurée**

**Correction — Méthodologie OSI (du bas vers le haut) :**

1. Vérifier la configuration IP du poste : `ipconfig` / `ip addr` (IP, masque, gateway, DNS)
2. Pinger la **passerelle locale** : `ping 192.168.x.1` → si KO, problème réseau local ou config IP
3. Pinger une **IP publique connue** : `ping 8.8.8.8` → si KO, problème de routage ou de NAT au niveau du routeur
4. Pinger un **nom de domaine** : `ping google.com` → si KO alors que l'IP publique répond, problème **DNS**
5. Vérifier si le problème est **isolé à ce poste** ou général (tester depuis une autre machine)
6. Vérifier les logs du routeur/pare-feu pour des blocages

---

### Q45 — Navigation web lente, LAN OK

**Statut : ❌ Non répondu**

**Correction — Causes possibles :**

- **Problème DNS** : le DNS primaire (192.168.1.1) est interne et lent à répondre → tester avec `nslookup google.com 8.8.8.8`
- **Saturation de la bande passante** : une autre machine ou application consomme le lien Internet → vérifier avec un outil de monitoring
- **Problème de MTU** : des paquets trop grands sont fragmentés ou perdus → tester avec `ping -f -l 1400 8.8.8.8`
- **Proxy mal configuré** : vérifier les paramètres proxy dans le navigateur
- **Latence élevée vers Internet** : `tracert google.com` pour identifier où se situe le ralentissement
- **Serveur DNS secondaire (8.8.8.8) utilisé** : si le primaire ne répond pas, le basculement vers le secondaire ajoute un délai

---

### Q46 — Plan d'adressage VLSM sur 10.10.0.0/16

**Statut : ❌ Non répondu**

**Correction — Méthodologie VLSM (toujours commencer par le plus grand) :**

|Site|Hôtes requis|Masque choisi|Adresse réseau|Broadcast|Hôtes dispo|
|---|---|---|---|---|---|
|Site A|500|/23 (512-2=510)|10.10.0.0/23|10.10.1.255|510|
|Site B|250|/24 (256-2=254)|10.10.2.0/24|10.10.2.255|254|
|Site C|100|/25 (128-2=126)|10.10.3.0/25|10.10.3.127|126|
|Lien WAN|2|/30 (4-2=2)|10.10.3.128/30|10.10.3.131|2|

> **Règle clé du VLSM :** Toujours partir du **plus grand réseau** et attribuer en ordre décroissant pour éviter les chevauchements.

---

### Q47 — Serveur DHCP et DHCP Relay

**Statut : ✅ Correct**

Bonne réponse sur les deux points.

> **Complément paramètres scope :**
> 
> - Plage d'adresses (début et fin)
> - Masque de sous-réseau
> - Passerelle par défaut
> - Serveurs DNS
> - Durée du bail
> - Exclusions éventuelles (adresses déjà attribuées statiquement)

> **DHCP Relay Agent :** configuré sur le routeur avec la commande `ip helper-address <IP_serveur_DHCP>`. Il transforme le broadcast DHCP du client en unicast vers le serveur DHCP centralisé.

---

### Q48 — Publication d'un serveur web interne (DNAT)

**Statut : ❌ Confusion DMZ / DNAT**

**Correction :** La technique utilisée est le **DNAT (Destination NAT)**, pas uniquement la DMZ.

**Configuration sur le routeur/pare-feu :**

- Règle DNAT : toute connexion entrante sur l'IP publique port **80** → redirige vers l'IP interne du serveur port **80**
- Règle DNAT : toute connexion entrante sur l'IP publique port **443** → redirige vers l'IP interne du serveur port **443**

**La DMZ** est une zone réseau isolée où on place ce serveur pour qu'une compromission ne mette pas en danger le LAN. C'est une **bonne pratique de sécurité** qui **complète** le DNAT, mais ce n'est pas la technique réseau demandée ici.

---

### Q49 — Métrique dans la table de routage

**Statut : ❌ Non répondu**

**Correction :** La **métrique** est un coût attribué à une route. Quand plusieurs routes vers la même destination existent, **la route avec la métrique la plus basse** est préférée.

**Utilité :** Permet de définir des **routes préférentielles** et des **routes de secours (backup)**. Si la route principale tombe, le routeur bascule automatiquement sur la route de secours (métrique plus haute).

_Exemple :_

- Route A vers 10.5.0.0/24 via 192.168.1.1 → métrique 10 → **préférée**
- Route B vers 10.5.0.0/24 via 192.168.2.1 → métrique 100 → **backup**

---

### Q50 — Traceroute / Tracert

**Statut : ✅ Correct — à compléter**

Bonne réponse. `traceroute` (Linux) / `tracert` (Windows) affiche tous les routeurs traversés entre la source et la destination.

**Mécanisme TTL :**

- Le premier paquet est envoyé avec TTL=1 → le 1er routeur répond avec un message ICMP "Time Exceeded" et s'identifie
- Puis TTL=2 → le 2ème routeur répond, etc.
- Jusqu'à atteindre la destination qui répond avec "Echo Reply"

**Ce qu'on peut tirer du résultat :**

- Identifier où se situe une **latence anormale** (un saut avec un temps élevé)
- Identifier un routeur **qui ne répond pas** (asterisques `***`) → firewall ou problème réseau
- Vérifier le **chemin emprunté** par les paquets

---

## RÉSUMÉ DES RÉSULTATS

|Partie|Questions|✅ Correct|⚠️ Partiel|❌ Incorrect/Manquant|
|---|---|---|---|---|
|Fondamentaux|Q1–Q6|Q2, Q3|Q1, Q5, Q6|Q4|
|Adressage IPv4|Q7–Q14|Q8, Q9, Q12|Q7, Q10, Q11|Q13, Q14|
|Protocoles support|Q15–Q19|Q18, Q19|Q15, Q16, Q17|—|
|DNS|Q20–Q23|Q23|Q20, Q21|Q22|
|Routage|Q24–Q30|Q25, Q26|Q24|Q27, Q28, Q29, Q30|
|TCP/UDP|Q31–Q35|Q32|Q31|Q33, Q34, Q35|
|NAT|Q36–Q38|Q36|Q38|Q37|
|IPv6|Q39–Q43|Q39|—|Q40, Q41, Q42, Q43|
|Mise en situation|Q44–Q50|Q47, Q50|Q44|Q45, Q46, Q48, Q49|

---

### Points forts 💪

- Bonne maîtrise des couches OSI et des équipements
- Calculs de base IPv4 (/24) maîtrisés
- Séquence DORA globalement comprise
- Three-Way Handshake connu

### Points à retravailler 📚

- **Calculs de sous-réseaux complexes** (/20, /22, VLSM) → pratiquer avec des exercices binaires
- **IPv6** dans son ensemble → à revoir en priorité
- **Commandes Linux/Windows** de routage → à pratiquer en atelier
- **Protocoles de routage dynamique** (RIP, OSPF, BGP) → mémoriser les métriques et contextes
- **Fermeture TCP et fenêtre TCP** → mécanismes à réviser

---

_Correction réalisée sur la base des cours TSSR et des référentiels techniques RFC._