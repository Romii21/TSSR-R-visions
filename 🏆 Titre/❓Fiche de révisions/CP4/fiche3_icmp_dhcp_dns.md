# FICHE 3 — Protocoles de Support : ICMP, DHCP, DNS
### CP4 : Exploiter un réseau IP | Formation TSSR

---

## PARTIE A — ICMP

## 1. Définition et rôle

**ICMP** (Internet Control Message Protocol) est un protocole de **diagnostic et de contrôle** du réseau.

- Il ne transporte **pas de données applicatives**
- Il signale des erreurs et permet des tests de connectivité
- Il fonctionne au niveau de la **couche 3 (réseau)**

## 2. Messages ICMP importants

| Type | Message | Utilité |
|------|---------|---------|
| Echo Request / Echo Reply | **Ping** | Tester la joignabilité d'un hôte |
| Destination Unreachable | Hôte/réseau inaccessible | Diagnostic de routage |
| Time Exceeded (TTL=0) | TTL expiré | Utilisé par **traceroute** |
| Redirect | Redirection | Informe d'une meilleure route |

## 3. Interprétation des messages ping

| Message | Signification |
|---------|--------------|
| **Reply from X** | La machine cible répond — connexion OK |
| **Destination host unreachable** | Un routeur **sur le chemin** (ou la machine locale) ne sait pas comment atteindre la destination. Réponse **active** d'un équipement — route manquante ou problème de gateway. |
| **Request timed out** | Aucune réponse reçue dans le délai imparti. La cible est peut-être éteinte, son pare-feu bloque ICMP, ou les paquets sont perdus. Pas de réponse du tout. |

> **Différence clé :** "Unreachable" = quelqu'un a répondu pour dire "je ne sais pas". "Timed out" = silence total.

## 4. Traceroute / Tracert

### Fonctionnement
Traceroute envoie des paquets avec un **TTL croissant** (1, 2, 3…).

- Quand le TTL atteint **0** sur un routeur, ce routeur envoie un **ICMP Time Exceeded**
- Traceroute identifie ainsi chaque saut jusqu'à la destination

```
traceroute 8.8.8.8

1  192.168.1.1  (gateway locale)    2 ms
2  10.0.0.1    (routeur FAI)        8 ms
3  195.x.x.x  (réseau FAI)        12 ms
4  216.x.x.x  (réseau Google)     20 ms
5  8.8.8.8    (destination)       21 ms
```

Chaque ligne = un **routeur intermédiaire** avec son IP et le temps de réponse.

**Commandes :**
- Linux : `traceroute 8.8.8.8` (TTL par défaut : 64)
- Windows : `tracert 8.8.8.8` (TTL par défaut : 30)

---

## PARTIE B — DHCP

## 5. Rôle et définition

**DHCP** (Dynamic Host Configuration Protocol) attribue automatiquement une configuration réseau aux clients.

- Défini par les **RFC 2131 et 2132**
- Protocole **client/serveur** sur **UDP**
  - Serveur : port **67**
  - Client : port **68**
- Le broadcast DHCP ne franchit **pas les routeurs** (sans configuration spécifique)

## 6. Paramètres distribués par DHCP

| Paramètre | Obligatoire ? |
|-----------|--------------|
| Adresse IP | ✅ Oui |
| Masque de sous-réseau | ✅ Oui |
| Durée du bail | ✅ Oui |
| Passerelle par défaut | Non (mais quasi toujours inclus) |
| Serveurs DNS | Non (mais quasi toujours inclus) |
| Serveur NTP | Non |
| Serveur TFTP / PXE | Non |

## 7. La séquence DORA — À connaître par cœur

```
CLIENT                                          SERVEUR
  │                                               │
  │  1. DHCP DISCOVER  (broadcast)                │
  │──────────────────────────────────────────────>│
  │  "Qui peut me donner une config IP ?"         │
  │                                               │
  │  2. DHCP OFFER                                │
  │<──────────────────────────────────────────────│
  │  "Je te propose : 192.168.1.10, bail 24h"     │
  │                                               │
  │  3. DHCP REQUEST   (broadcast)                │
  │──────────────────────────────────────────────>│
  │  "J'accepte cette offre, je la réserve"       │
  │                                               │
  │  4. DHCP ACK                                  │
  │<──────────────────────────────────────────────│
  │  "OK, configuration confirmée !"              │
```

| Lettre | Message | Émetteur | Sens | Description |
|--------|---------|----------|------|-------------|
| **D** | DISCOVER | Client | Broadcast | Recherche d'un serveur DHCP |
| **O** | OFFER | Serveur | → Client | Proposition de configuration |
| **R** | **REQUEST** | Client | Broadcast | **Demande** officielle de réservation |
| **A** | ACK | Serveur | → Client | Confirmation et envoi de la config complète |

> ⚠️ Le **R** de DORA = **REQUEST** (pas "Receive"). Le client envoie encore un message au serveur après l'Offer.

## 8. Le bail DHCP et son renouvellement

```
Durée totale du bail : 24h (exemple)

 0h         12h (50%)       21h (87,5%)     24h (100%)
 │────────────│────────────────│───────────────│
 │            │                │               │
 Obtention   Renouvellement   Renouvellement  Expiration
 du bail     DHCP REQUEST     en broadcast    → DORA complète
             unicast vers     (n'importe quel  recommence
             le serveur       serveur)
```

- À **50%** : le client tente de renouveler en **unicast** vers son serveur DHCP
- À **87,5%** : si pas de réponse, il tente en **broadcast** vers n'importe quel serveur
- À **100%** : le bail expire → le client perd son IP → séquence **DORA complète**

## 9. Les autres messages DHCP

| Message | Émetteur | Utilisation |
|---------|----------|-------------|
| **DHCP NACK** | Serveur | Refus de la demande (IP plus valide) |
| **DHCP DECLINE** | Client | Refuse l'IP proposée (IP déjà utilisée détectée par ARP) |
| **DHCP RELEASE** | Client | Libère le bail volontairement |
| **DHCP INFORM** | Client | Demande d'options sans réservation d'IP (client avec IP fixe) |

## 10. Réservation DHCP

Permet d'attribuer **toujours la même IP** à une machine spécifique, tout en gardant la gestion centralisée.

**Mécanisme :** Association `adresse MAC → adresse IP fixe` dans la config du serveur DHCP.

**Cas d'usage :** Imprimantes réseau, serveurs, caméras IP, points d'accès Wi-Fi.

## 11. DHCP Relay Agent

**Problème :** Le DHCP utilise le broadcast, qui ne franchit pas les routeurs.

**Solution :** Le DHCP Relay Agent (configuré sur le routeur avec `ip helper-address`) **intercepte les requêtes DHCP locales** et les **transmet en unicast** vers un serveur DHCP centralisé sur un autre réseau.

```
VLAN 10 (192.168.10.0/24)        Routeur                 Serveur DHCP
       Client ──────────────> ip helper-address ──────> (192.168.1.100)
              DISCOVER (broadcast)         DISCOVER (unicast)
```

**Indispensable** dans les architectures avec plusieurs VLAN ou sites.

---

## PARTIE C — DNS

## 12. Rôle du DNS

**DNS** (Domain Name System) = système de résolution de noms permettant de **traduire un nom de domaine en adresse IP**.

- Protocole : **UDP/53** (requêtes classiques), **TCP/53** (transferts de zone)
- Système **décentralisé et hiérarchique**
- Défini par les RFC 1034 et 1035 (1987)

## 13. Processus de résolution complet

Exemple : l'utilisateur tape `www.google.com`

```
1. Cache local + fichier hosts
   └─ Si trouvé → réponse immédiate

2. Résolveur DNS configuré (FAI ou entreprise)
   └─ Interroge les serveurs racines (root servers)

3. Serveur racine (.)
   └─ "Je ne connais pas l'IP mais voici le serveur TLD .com"

4. Serveur TLD .com
   └─ "Je ne connais pas l'IP mais voici les serveurs de google.com"

5. Serveur autoritaire de google.com
   └─ "www.google.com = 142.250.x.x" ← réponse finale

6. Mise en cache → renvoi au client
7. Le navigateur se connecte à 142.250.x.x sur TCP/443
```

## 14. Résolution récursive vs itérative

| Type | Qui fait le travail ? | Utilisé par |
|------|----------------------|-------------|
| **Récursive** | Le **résolveur DNS** interroge tout seul jusqu'à trouver la réponse et la renvoie au client | Les postes clients |
| **Itérative** | Chaque serveur répond avec la **meilleure info qu'il possède** (référence vers un autre serveur). Le client enchaîne les requêtes lui-même | Entre serveurs DNS |

## 15. Les enregistrements DNS

| Type | Nom | Rôle | Exemple |
|------|-----|------|---------|
| **A** | Address | Associe un nom à une **IPv4** | `www.exemple.fr → 93.184.216.34` |
| **AAAA** | IPv6 Address | Associe un nom à une **IPv6** | `www.exemple.fr → 2606:2800::1` |
| **CNAME** | Canonical Name | **Alias** vers un autre nom | `ftp.exemple.fr → www.exemple.fr` |
| **MX** | Mail Exchange | Serveur de **messagerie** du domaine | `exemple.fr → mail.exemple.fr (prio 10)` |
| **PTR** | Pointer | Résolution **inverse** (IP → nom) | `34.216.184.93.in-addr.arpa → www.exemple.fr` |
| **NS** | Name Server | Serveurs DNS **autoritaires** du domaine | `exemple.fr → ns1.hebergeur.fr` |
| **TXT** | Text | Informations textuelles (SPF, DKIM…) | Validation de domaine |
| **SOA** | Start of Authority | Informations sur la zone DNS | Serveur maître, TTL… |

## 16. Outils DNS en ligne de commande

### Linux — `dig`
```bash
dig www.google.com                    # Résolution A standard
dig www.google.com AAAA               # Résolution IPv6
dig -x 8.8.8.8                        # Résolution inverse (PTR)
dig @8.8.8.8 www.google.com           # Interroger un serveur DNS spécifique
dig www.google.com MX                 # Enregistrements MX
```

### Windows / Linux — `nslookup`
```cmd
nslookup www.google.com               # Résolution standard
nslookup www.google.com 8.8.8.8       # Via un serveur DNS spécifique
nslookup -type=MX google.com          # Enregistrements MX
nslookup 8.8.8.8                      # Résolution inverse
```

### Linux — `host`
```bash
host www.google.com
host -t MX google.com
```

---

## 17. Points clés à retenir ✅

**ICMP**
- Ping = ICMP Echo Request / Echo Reply
- "Destination unreachable" = réponse **active** d'un routeur (route inconnue)
- "Request timed out" = **silence** (hôte éteint, pare-feu, perte)
- Traceroute exploite **TTL + ICMP Time Exceeded**

**DHCP**
- Protocole **UDP** (67 serveur / 68 client) — jamais TCP !
- DORA = **D**iscover → **O**ffer → **R**equest → **A**ck
- Le **R** = REQUEST (le client envoie encore un message !)
- Renouvellement bail : à **50%** (unicast) puis **87,5%** (broadcast)
- DHCP Relay = `ip helper-address` → pour franchir les routeurs / VLAN

**DNS**
- UDP/53 (requêtes) et TCP/53 (transferts de zone)
- Enregistrements : **A** (IPv4), **AAAA** (IPv6), **CNAME** (alias), **MX** (mail), **PTR** (inverse)
- Récursive = le résolveur fait tout / Itérative = entre serveurs DNS
- Outils : `dig` (Linux) et `nslookup` (Windows/Linux)

---

*Fiche 3/6 — CP4 Exploiter un réseau IP*
