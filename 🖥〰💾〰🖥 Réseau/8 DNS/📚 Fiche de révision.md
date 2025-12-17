# Synthèse du cours DNS (Domain Name System)

## Sommaire

1. [Introduction - Le problème fondamental](https://claude.ai/chat/fe41f7fd-8a1f-4e60-b288-a1a89cba1437#introduction)
2. [Les noms de domaine](https://claude.ai/chat/fe41f7fd-8a1f-4e60-b288-a1a89cba1437#noms-de-domaine)
3. [Le protocole DNS](https://claude.ai/chat/fe41f7fd-8a1f-4e60-b288-a1a89cba1437#protocole-dns)
4. [Les enregistrements DNS](https://claude.ai/chat/fe41f7fd-8a1f-4e60-b288-a1a89cba1437#enregistrements)
5. [Enregistrer un nom de domaine](https://claude.ai/chat/fe41f7fd-8a1f-4e60-b288-a1a89cba1437#enregistrement-domaine)
6. [Outils pratiques](https://claude.ai/chat/fe41f7fd-8a1f-4e60-b288-a1a89cba1437#outils)

---

## 1. Introduction - Le problème fondamental {#introduction}

### Le besoin humain vs la logique machine

Les ordinateurs communiquent sur Internet en utilisant des **adresses IP** :

- **IPv4** : 32 bits (ex : 192.168.1.1)
- **IPv6** : 128 bits (ex : 2a00:1450:4007:815::2013)

**Problème** : Ces adresses sont difficiles à mémoriser pour les humains.

**Solution** : Le DNS agit comme un **traducteur** entre les noms textuels (ex : google.com) et les adresses IP.

### L'évolution historique

**Avant le DNS (années 1970-1980)** :

- Un fichier texte appelé **HOSTS.TXT** (standardisé par la RFC 608 en 1974)
- Maintenu manuellement par le NIC (Network Information Center)
- Copié sur chaque machine via FTP
- **Limite** : Impossible à maintenir avec l'augmentation du nombre d'hôtes

**Naissance du DNS (1983)** :

- Créé par Paul V. Mockapetris
- Standardisé dans les RFC 882 et 883 (1983), puis RFC 1034 et 1035 (toujours en vigueur)
- **DNS = Base de données répartie et décentralisée** → robustesse et passage à l'échelle

### Caractéristiques clés du DNS

- **Technologie d'infrastructure** : invisible pour l'utilisateur final
- **Base distribuée** : pas de point unique de défaillance
- **Hiérarchique** : organisation en arbre pour une meilleure gestion
- **Robuste** : réplication des données sur plusieurs serveurs

---

## 2. Les noms de domaine {#noms-de-domaine}

### Définition

Un nom de domaine est :

- Un **identifiant textuel unique** (non sensible à la casse)
- **Hiérarchique** : structure arborescente
- **Plus stable** que les adresses IP
- Un **vecteur d'image** : choix important pour l'identité en ligne

### Structure hiérarchique

```
                    . (Racine)
                    |
        +-----------+-----------+----------+
        |           |           |          |
       FR          COM         ORG       香港 (TLD)
        |           |           |
     gouv.fr    google.com  wikipedia.org
        |           |
   ecologie.gouv.fr  www.google.com
```

**Composants d'un nom de domaine** :

1. **Racine** : point final `.` (souvent omis)
2. **TLD** (Top Level Domain) : `.com`, `.fr`, `.org`, etc.
3. **Domaine de second niveau** : `google`, `wikipedia`
4. **Sous-domaines** : `www`, `mail`, etc.
5. **FQDN** (Fully Qualified Domain Name) : nom complet avec le point final

**Exemple** : `www.wildcodeschool.com.`

- `.` = racine
- `com` = TLD
- `wildcodeschool` = domaine de second niveau
- `www` = sous-domaine

### Règles techniques

- **Labels** (composants) : nombre illimité (mais minimum 2 en pratique)
- **Longueur** : 63 caractères max par label, 255 max pour le nom complet
- **Caractères** : support des caractères internationaux (IDN - Internationalized Domain Name)
- **Exemples** :
    - `odyssey.wildcodeschool.com`
    - `fr.wikipedia.org`
    - `नेपाल.icom.museum` (caractères internationaux)

---

## 3. Le protocole DNS {#protocole-dns}

### Architecture client-serveur

**Caractéristiques techniques** :

- **Couche 7** (applicatif) du modèle OSI
- **Port 53** (port 853 pour DNS over TLS)
- **Protocoles** :
    - **UDP** : utilisé par défaut (rapide, moins de surcharge)
    - **TCP** : utilisé pour les transferts de zone ou les grandes réponses (> 512 octets)
- Communication simple : **une requête → une réponse → fin**

### Les différents types de serveurs

#### 1. Serveurs racines (Root Servers)

- **13 domaines** : `a.root-servers.net` à `m.root-servers.net`
- Gérés par **l'ICANN** et 12 organisations (dont RIPE NCC)
- En réalité **plus de 130 sites physiques** grâce à l'anycast
- Contiennent la liste des serveurs TLD
- Point de départ de toute résolution DNS

#### 2. Serveurs faisant autorité (Authoritative Servers)

**Rôle** : Contenir les informations définitives pour une ou plusieurs zones DNS

**Organisation** :

- **Serveur primaire** : contient les données originales
- **Serveurs secondaires** : copies pour la redondance
- **Synchronisation** : transfert de zone via DNS

**Logiciels** : NSD, Knot, BIND, PowerDNS, Microsoft DNS

#### 3. Résolveurs DNS (Recursive Resolvers)

**Rôle** : Interroger les serveurs faisant autorité pour le compte des clients

**Fonctionnement** :

- Démarrent avec la liste des 13 serveurs racines
- **Mettent en cache** les réponses (TTL - Time To Live)
- Souvent chez les **FAI** ou sur les réseaux privés

**Logiciels** : Unbound, BIND, PowerDNS, Microsoft DNS

**Résolveurs publics populaires** :

- **Quad9** (Suisse) : `9.9.9.9`
- **Cloudflare** (US) : `1.1.1.1`
- **Google** (US) : `8.8.8.8`
- **FDN** (FR) : `80.67.169.12`

#### 4. Stub Resolver (Résolveur local)

**Rôle** : Interface DNS sur le système d'exploitation client

**Caractéristiques** :

- Ne gère **pas la récursivité** (délègue au résolveur récursif)
- Gère un **cache local**
- Consulte le fichier **hosts** (`/etc/hosts` sous Unix)
- Intégré au système (ex : `systemd-resolved`)

### Processus de résolution DNS

```
Client                  Stub         Résolveur      Serveur       Serveur      Serveur
(Navigateur)          Resolver      Récursif        Racine        .com      wildcodeschool.com
    |                     |              |              |            |              |
    |--1. odyssey.wildcodeschool.com?-->|              |            |              |
    |                     |--2.--------->|              |            |              |
    |                     |              |--3. Qui gère .com?------->|              |
    |                     |              |<--NS de .com--|            |              |
    |                     |              |--4. Qui gère wildcodeschool.com?-------->|
    |                     |              |<--NS de wildcodeschool.com--|            |
    |                     |              |--5. odyssey.wildcodeschool.com?--------->|
    |                     |              |<--6. IP: 2a00:1450...---------------------|
    |                     |<--7. IP------|              |            |              |
    |<--8. IP-------------|              |              |            |              |
```

**Étapes de résolution** :

1. Le client demande l'adresse de `odyssey.wildcodeschool.com`
2. Le stub resolver transmet au résolveur récursif
3. Le résolveur interroge le serveur racine → obtient les serveurs `.com`
4. Le résolveur interroge le serveur `.com` → obtient les serveurs `wildcodeschool.com`
5. Le résolveur interroge le serveur `wildcodeschool.com` → obtient l'adresse IP
6. L'adresse IP remonte au client

### Configuration DNS d'un hôte

**Informations fournies** (généralement via DHCP) :

- Adresse(s) du/des résolveur(s) DNS
- Domaine de recherche par défaut

**Fichier hosts** :

- Permet des résolutions locales sans DNS
- Consulté **avant** les requêtes DNS
- Localisation : `/etc/hosts` (Unix/Linux) ou `C:\Windows\System32\drivers\etc\hosts` (Windows)

---

## 4. Les enregistrements DNS {#enregistrements}

### Qu'est-ce qu'un Resource Record (RR) ?

Un **RR** est une entrée dans la base DNS qui associe un nom de domaine à des données.

**Tous les RR ont un TTL** (Time To Live) : durée maximale de conservation dans un cache.

### Types d'enregistrements principaux

|Type|Nom complet|Rôle|
|---|---|---|
|**A**|Address|Adresse **IPv4** (32 bits)|
|**AAAA**|IPv6 Address|Adresse **IPv6** (128 bits)|
|**NS**|Name Server|Serveur faisant autorité pour la zone|
|**CNAME**|Canonical Name|Alias pointant vers un autre nom|
|**SOA**|Start of Authority|Serveur primaire et paramètres de la zone|
|**MX**|Mail Exchange|Serveur de courrier pour le domaine|
|**PTR**|Pointer|Résolution inverse (IP → nom)|
|**TXT**|Text|Informations textuelles diverses|

### Résolution inverse

**Objectif** : Trouver le nom de domaine à partir d'une adresse IP

**Mécanisme** :

- Utilise des **pseudo-domaines** :
    - `.in-addr.arpa` pour IPv4
    - `.ip6.arpa` pour IPv6
- L'adresse IP est **inversée** :
    - **IPv4** : `172.67.146.155` → `155.146.67.172.in-addr.arpa`
    - **IPv6** : chaque chiffre hexadécimal est séparé par un point

**Enregistrement** : PTR (pointer record)

**Utilité** :

- Vérification de l'identité des serveurs
- Lutte contre le spam (vérification des serveurs mail)
- Logs et audit

### DNS Round-Robin (Tourniquet)

**Technique** : Associer plusieurs adresses IP à un même nom de domaine

**Objectif** : Répartition de charge (load balancing basique)

**Fonctionnement** :

- À chaque requête, le serveur **fait tourner** l'ordre des adresses
- Les clients reçoivent des réponses différentes
- Distribution automatique du trafic

**Exemple** :

```
google.com → 142.250.74.206
google.com → 142.250.74.238
google.com → 142.250.74.174
```

---

## 5. Enregistrer un nom de domaine {#enregistrement-domaine}

### Hiérarchie de gestion

#### 1. ICANN (Internet Corporation for Assigned Names and Numbers)

- Gère la **racine DNS** via sa composante IANA
- Enregistre et délègue les **TLD**
- Autorité mondiale sur les noms de domaine

#### 2. Les TLD (Top Level Domains)

**Types de TLD** :

- **ccTLD** (Country Code TLD) : domaines nationaux
    - Exemples : `.fr`, `.uk`, `.de`, `.jp`
- **gTLD** (Generic TLD) : domaines génériques ouverts
    - Exemples : `.com`, `.org`, `.net`, `.info`
- **sTLD** (Sponsored TLD) : domaines réservés à des activités spécifiques
    - Exemples : `.edu` (éducation), `.gov` (gouvernement US), `.mil` (militaire US)
- **Domaines spéciaux** :
    - `.arpa` (infrastructure, résolution inverse)
    - `.example`, `.invalid`, `.localhost`, `.test` (réservés pour tests/documentation)

**Évolution** : Depuis 2012, de nombreux nouveaux TLD ont été créés (`.app`, `.blog`, `.tech`, etc.)

#### 3. Registre (Registry)

**Définition** : Organisme gérant la base de données d'un domaine

**Exemples** :

- **AFNIC** : gère `.fr` (et `.pm`, `.re`, `.tf`, `.wf`, `.yt`)
- **Verisign** : gère `.com` et `.net`
- Chaque TLD a son registre désigné par l'ICANN

**Rôle** :

- Maintient la base de données des noms de domaine
- Gère les serveurs DNS faisant autorité pour le TLD
- Définit les règles d'enregistrement

#### 4. Bureau d'enregistrement (Registrar)

**Définition** : Interface entre le client et le registre

**Rôle** :

- Vendre/réserver des noms de domaine pour les clients
- Parfois **héberger la zone DNS** (hébergeur DNS)
- Communiquer les serveurs NS au registre

**Exemples** : OVH, Gandi, GoDaddy (383 accrédités par l'AFNIC pour `.fr`)

### Processus d'enregistrement

1. **Choisir** un nom de domaine disponible
2. **Vérifier** la disponibilité auprès d'un bureau d'enregistrement
3. **Acheter** le nom pour une durée déterminée (généralement 1 an, renouvelable)
4. **Configurer** les serveurs DNS (NS) :
    - Soit utiliser ceux du registrar
    - Soit indiquer ses propres serveurs
5. **Créer les enregistrements DNS** (A, AAAA, MX, etc.)

---

## 6. Outils pratiques {#outils}

### dig (Domain Information Groper)

**Plateforme** : Unix/Linux (fait partie de la suite BIND)

**Utilité** : Interroger les serveurs DNS et diagnostiquer

**Avantages** : Plus complet que `nslookup` ou `host`

**Exemples d'utilisation** :

```bash
# Requête simple (enregistrement A par défaut)
dig google.com

# Requête d'un type spécifique
dig google.com AAAA          # IPv6
dig google.com MX            # Serveurs mail
dig google.com NS            # Serveurs de noms

# Interroger un serveur DNS spécifique
dig @8.8.8.8 google.com

# Résolution inverse
dig -x 142.250.74.206

# Affichage court
dig +short google.com
```

### nslookup

**Plateforme** : Windows (disponible par défaut), Unix/Linux

**Utilité** : Outil de base pour interroger DNS

**Exemple** :

```cmd
nslookup google.com
```

### host

**Plateforme** : Unix/Linux

**Utilité** : Outil simple pour résolution DNS

**Exemple** :

```bash
host google.com
```

---

## Points clés à retenir

### Le DNS est essentiel au fonctionnement d'Internet

- Traduit les noms en adresses IP
- Infrastructure invisible mais critique
- Base distribuée = robustesse

### Architecture hiérarchique

- Racine → TLD → Domaines → Sous-domaines
- Délégation de l'autorité à chaque niveau
- Réplication pour la fiabilité

### Deux types de serveurs principaux

- **Serveurs faisant autorité** : détiennent les données officielles
- **Résolveurs récursifs** : interrogent pour le compte des clients

### Le cache est fondamental

- Réduit la charge sur les serveurs
- Accélère les résolutions
- Contrôlé par le TTL

### Résolution en cascade

- Client → Stub resolver → Résolveur récursif → Racine → TLD → Autoritaire
- Chaque niveau connaît le suivant
- Cache à chaque étape

---

## Sources

### Documents du cours

- DNS.pdf (cours principal)
- Note.md (notes personnelles)

### Sources complémentaires consultées

- IBM - "What Is the DNS Protocol?" (décembre 2024)
- DNSFilter - "A guide to DNS"
- Microsoft Learn - "Domain Name System (DNS) in Windows"
- FreeCodeCamp - "What is DNS? Basics for Beginners" (juillet 2023)
- Firewall.cx - "The DNS Protocol - Part 1: Introduction"
- Oracle - "DNS Basics (System Administration Guide)"
- Null Hardware - "DNS Fundamentals and Packet Structure" (octobre 2022)
- Grandmetric - "DNS – What Is It & What Is Its Use?" (janvier 2025)
- Medium - "DNS Protocol explained" par Grzegorz Piechnik (décembre 2023)
- Wikipédia (FR) - "Domain Name System" (décembre 2024)