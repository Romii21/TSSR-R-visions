## Pourquoi le DNS ?

Les adresses IP sont difficiles à mémoriser → le DNS fait correspondre des **noms lisibles** à des **adresses IP**.  
Avant DNS, un fichier `HOSTS.TXT` était maintenu manuellement et distribué en FTP (RFC 608, 1974). L'explosion d'Internet a rendu cette approche obsolète.

DNS est standardisé depuis **1983** par Paul V. Mockapetris (RFC 882/883, puis **RFC 1034/1035**).

---

## Caractéristiques de DNS

- **Base de données répartie et décentralisée** → robustesse, passage à l'échelle
- **Technologie d'infrastructure** → invisible pour l'utilisateur
- Associe des **noms de domaine** à des données (adresses IP, serveurs mail, etc.)

---

## Protocole

|Paramètre|Valeur|
|---|---|
|Couche OSI|Applicative (couche 7)|
|Transport|**UDP** (en général) ou **TCP**|
|Port standard|**53**|
|DNS over TLS (DoT)|**853**|

---

## Structure d'un nom de domaine

```
www . wildcodeschool . com .
 │         │            │   └── racine (.)
 │         │            └────── TLD (Top Level Domain)
 │         └─────────────────── domaine de 2nd niveau
 └───────────────────────────── sous-domaine
```

- **FQDN** (Fully Qualified Domain Name) : nom complet avec le point final (souvent omis)
- Chaque composant (_label_) : max **63 caractères** / total : max **255 caractères**
- Insensible à la casse, supporte les IDN (caractères internationaux)

### Types de TLD

|Type|Exemples|
|---|---|
|ccTLD (national)|.fr, .de, .jp|
|gTLD (générique)|.com, .org, .net, .info|
|Commandités|.edu, .gov, .mil|
|Technique|.arpa|

---

## Les acteurs du DNS

### Serveurs faisant autorité (_Authoritative servers_)

- Contiennent les enregistrements d'une **zone**
- En général : 1 primaire + plusieurs secondaires (tolérance de panne)
- Synchronisation par **transfert de zone**
- Logiciels : BIND, NSD, Knot, PowerDNS, Microsoft DNS

### Résolveurs récursifs (_Resolvers_)

- Interrogent les serveurs faisant autorité pour le compte du client
- Mettent en **cache** les réponses (durée limitée par le **TTL**)
- Généralement chez les FAI ou en réseau privé
- Logiciels : Unbound, BIND, PowerDNS

#### Résolveurs publics connus

|Fournisseur|IPv4|
|---|---|
|Quad9 (🇨🇭)|`9.9.9.9`|
|Cloudflare (🇺🇸)|`1.1.1.1`|
|Google (🇺🇸)|`8.8.8.8`|
|FDN (🇫🇷)|`80.67.169.12`|

### Stub resolver (DNS local)

- Intégré à l'OS (ex. `systemd-resolved`)
- Gère un cache local, consulte aussi `/etc/hosts`
- Délègue les requêtes à un résolveur récursif configuré (souvent via **DHCP**)

### Serveurs racines

- **13 domaines** (`a.root-servers.net` → `m.root-servers.net`), gérés par **12 organisations**
- Gérés par l'**ICANN** → connus de tous les résolveurs
- En réalité **+130 sites** dans le monde via **anycast**

---

## Résolution de nom (étapes)

```
Client → Stub resolver → Résolveur récursif → Serveur racine (.)
                                            → Serveur TLD (.com)
                                            → Serveur autoritaire (wildcodeschool.com)
                                            ← Réponse IP
```

---

## Enregistrements DNS (Resource Records)

|Type|Rôle|
|---|---|
|**A**|Adresse IPv4|
|**AAAA**|Adresse IPv6|
|**NS**|Serveur faisant autorité pour la zone|
|**CNAME**|Alias vers un nom canonique|
|**MX**|Serveur de messagerie|
|**PTR**|Résolution inverse (IP → nom)|
|**SOA**|Serveur primaire + infos de zone|
|**TXT**|Données textuelles libres (SPF, DKIM…)|

> Chaque RR possède un **TTL** (Time To Live) : durée de validité en cache.

### Résolution inverse

- IPv4 : domaine `.in-addr.arpa` (octets inversés) → ex. `172.67.146.155.in-addr.arpa`
- IPv6 : domaine `.ip6.arpa` (chiffres hex inversés)
- Stockée dans des enregistrements **PTR**

### DNS round-robin

Associer **plusieurs adresses** à un même nom → répartition de charge simple.

---

## Gouvernance

|Entité|Rôle|
|---|---|
|**ICANN / IANA**|Gère la racine et délègue les TLD|
|**Registre** (ex. AFNIC pour .fr)|Gère un TLD|
|**Bureau d'enregistrement** (ex. OVH, Gandi)|Interface client pour réserver un domaine|

---

## Outils

|Outil|OS|Usage|
|---|---|---|
|`dig`|Unix/Linux|Interrogation DNS avancée (recommandé)|
|`nslookup`|Windows/Unix|Interrogation DNS basique|
|`host`|Unix/Linux|Interrogation DNS simple|