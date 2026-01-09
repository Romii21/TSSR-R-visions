

## 📋 Table des matières

```table-of-contents
title: 
style: nestedList # TOC style (nestedList|nestedOrderedList|inlineFirstLevel)
minLevel: 2 # Include headings from the specified level
maxLevel: 2 # Include headings up to the specified level
include: 
exclude: 
includeLinks: true # Make headings clickable
hideWhenEmpty: false # Hide TOC if no headings are found
debugInConsole: false # Print debug info in Obsidian console
```
---

## 🎯 Introduction aux ACL Standard

Les **ACL Standard** sont le type d'ACL le plus simple sur les routeurs Cisco. Elles permettent de filtrer le trafic en se basant **uniquement sur l'adresse IP source**.

> [!info] Caractéristiques principales
> 
> - Filtrage basé sur l'**adresse IP source** uniquement
> - Impossible de filtrer par destination, port, ou protocole
> - Moins flexibles que les ACL étendues, mais plus simples à configurer
> - Utilisées pour des besoins de filtrage basiques

> [!warning] Limitation importante Les ACL standard ne peuvent pas distinguer les types de trafic (HTTP, FTP, SSH, etc.). Si vous bloquez une source, TOUT son trafic est bloqué.

**Quand utiliser une ACL standard ?**

- Contrôle d'accès simple basé sur la source
- Restriction d'accès à des services d'administration (Telnet, SSH vers le routeur)
- Filtrage pour des protocoles de routage (OSPF, EIGRP)
- NAT (définir les réseaux à traduire)

---

## 🔢 Numéros d'ACL Standard

Les ACL standard sont identifiées par des **numéros spécifiques** :

|Plage|Type|Utilisation|
|---|---|---|
|**1-99**|Standard|Plage classique originale|
|**1300-1999**|Standard|Plage étendue ajoutée ultérieurement|

> [!tip] Astuce de numérotation
> 
> - Utilisez les numéros **1-99** pour rester compatible avec d'anciens équipements
> - Utilisez les numéros **1300-1999** si vous avez besoin de plus de 99 ACL standard
> - Ou utilisez des **ACL nommées** pour une meilleure lisibilité

**Exemple de choix de numéro :**

```bash
# ACL numérotée classique
Router(config)# access-list 10 ...

# ACL numérotée étendue (si besoin de plus de 99)
Router(config)# access-list 1350 ...

# ACL nommée (recommandé pour la clarté)
Router(config)# ip access-list standard BLOCAGE_VLAN20
```

---

## 📝 Syntaxe de base

### Création d'une ACL standard numérotée

```bash
Router(config)# access-list [numéro] {permit|deny} [source] [wildcard-mask]
```

**Paramètres :**

- `[numéro]` : 1-99 ou 1300-1999
- `{permit|deny}` : Autoriser ou refuser le trafic
- `[source]` : Adresse IP source ou mot-clé (`any`, `host`)
- `[wildcard-mask]` : Masque inversé (optionnel selon contexte)

### Création d'une ACL standard nommée

```bash
Router(config)# ip access-list standard [nom]
Router(config-std-nacl)# {permit|deny} [source] [wildcard-mask]
Router(config-std-nacl)# exit
```

> [!example] Exemples concrets
> 
> **Autoriser une seule IP :**
> 
> ```bash
> Router(config)# access-list 10 permit host 192.168.1.50
> # Ou syntaxe équivalente :
> Router(config)# access-list 10 permit 192.168.1.50 0.0.0.0
> ```
> 
> **Autoriser un réseau entier :**
> 
> ```bash
> Router(config)# access-list 10 permit 192.168.10.0 0.0.0.255
> ```
> 
> **Bloquer une IP spécifique, autoriser tout le reste :**
> 
> ```bash
> Router(config)# access-list 20 deny host 10.0.0.100
> Router(config)# access-list 20 permit any
> ```
> 
> **ACL nommée :**
> 
> ```bash
> Router(config)# ip access-list standard ADMIN_ONLY
> Router(config-std-nacl)# permit 172.16.5.0 0.0.0.255
> Router(config-std-nacl)# deny any
> Router(config-std-nacl)# exit
> ```

### Règle implicite "deny any"

> [!warning] Deny implicite Toute ACL se termine par un **`deny any`** implicite. Si aucune règle ne correspond, le trafic est **bloqué par défaut**.

```bash
# Cette ACL bloque TOUT sauf 192.168.1.10
Router(config)# access-list 30 permit host 192.168.1.10
# deny any est implicite (tout le reste est bloqué)

# Pour autoriser le reste du trafic, ajoutez :
Router(config)# access-list 30 permit any
```

---

## 🎭 Wildcard Mask

Le **wildcard mask** (masque générique) est l'**inverse du masque de sous-réseau**. Il indique quels bits de l'adresse IP doivent être vérifiés.

### Principe de fonctionnement

|Bit du wildcard|Signification|
|---|---|
|**0**|Le bit DOIT correspondre (vérification stricte)|
|**1**|Le bit est ignoré (n'importe quelle valeur)|

> [!info] Calcul du wildcard mask **Wildcard = 255.255.255.255 - Masque de sous-réseau**
> 
> Exemples :
> 
> - Masque `/24` (255.255.255.0) → Wildcard : **0.0.0.255**
> - Masque `/16` (255.255.0.0) → Wildcard : **0.0.255.255**
> - Masque `/30` (255.255.255.252) → Wildcard : **0.0.0.3**

### Exemples pratiques

```bash
# Réseau 192.168.10.0/24 (255.255.255.0)
Router(config)# access-list 10 permit 192.168.10.0 0.0.0.255
# 0.0.0.255 = les 3 premiers octets doivent correspondre, le dernier est libre

# Réseau 10.0.0.0/8 (255.0.0.0)
Router(config)# access-list 10 permit 10.0.0.0 0.255.255.255
# Premier octet = 10, les autres sont libres

# Hôte unique 172.16.5.100
Router(config)# access-list 10 permit 172.16.5.100 0.0.0.0
# Tous les bits doivent correspondre (équivaut à "host 172.16.5.100")
```

### Mots-clés spéciaux

|Mot-clé|Équivalent|Utilisation|
|---|---|---|
|`host [IP]`|`[IP] 0.0.0.0`|Désigne une seule adresse IP|
|`any`|`0.0.0.0 255.255.255.255`|N'importe quelle adresse IP|

```bash
# Ces deux lignes sont identiques :
Router(config)# access-list 10 permit host 192.168.1.1
Router(config)# access-list 10 permit 192.168.1.1 0.0.0.0

# Ces deux lignes sont identiques :
Router(config)# access-list 10 permit any
Router(config)# access-list 10 permit 0.0.0.0 255.255.255.255
```

> [!tip] Astuce pratique Utilisez toujours les mots-clés `host` et `any` pour plus de clarté dans vos configurations.

### Wildcards non-contigus (avancé)

Les wildcard masks peuvent être **non-contigus**, permettant des filtrages très spécifiques :

```bash
# Autoriser uniquement les adresses IP paires dans 192.168.1.0/24
Router(config)# access-list 10 permit 192.168.1.0 0.0.0.254
# Le dernier bit du dernier octet doit être 0 (nombres pairs)

# Autoriser toutes les IP se terminant par .1, .5, .9, .13, etc. (multiples de 4 + 1)
Router(config)# access-list 10 permit 192.168.1.1 0.0.0.252
```

> [!warning] Attention Les wildcards non-contigus sont rarement utilisés et peuvent rendre la configuration difficile à comprendre. Utilisez-les avec précaution.

---

## 🔌 Application sur les interfaces

Une ACL seule ne fait rien. Elle doit être **appliquée sur une interface** dans une direction spécifique.

### Syntaxe d'application

```bash
Router(config)# interface [type] [numéro]
Router(config-if)# ip access-group [numéro-ou-nom] {in|out}
```

**Paramètres :**

- `[numéro-ou-nom]` : Numéro (1-99, 1300-1999) ou nom de l'ACL
- `{in|out}` : Direction du filtrage

### Directions : IN vs OUT

|Direction|Filtrage|Quand utiliser|
|---|---|---|
|**in**|Trafic **entrant** dans l'interface|Filtrer le trafic provenant du réseau connecté|
|**out**|Trafic **sortant** de l'interface|Filtrer le trafic allant vers le réseau connecté|

> [!info] Visualisation
> 
> ```
> Réseau A ----[in]----> [ROUTEUR] ----[out]----> Réseau B
>              ↑ Filtre              ↑ Filtre
>              avant routing         après routing
> ```
> 
> - **IN** : Filtre AVANT que le routeur ne prenne une décision de routage
> - **OUT** : Filtre APRÈS la décision de routage, juste avant l'envoi

> [!example] Exemple d'application
> 
> **Scénario :** Bloquer le réseau 192.168.20.0/24 d'accéder au réseau 10.0.0.0/8
> 
> ```bash
> # Étape 1 : Créer l'ACL
> Router(config)# access-list 15 deny 192.168.20.0 0.0.0.255
> Router(config)# access-list 15 permit any
> 
> # Étape 2 : Appliquer sur l'interface appropriée
> # Option A : Sur l'interface connectée au réseau 192.168.20.0/24 (IN)
> Router(config)# interface GigabitEthernet0/0
> Router(config-if)# ip access-group 15 in
> Router(config-if)# exit
> 
> # Option B : Sur l'interface connectée au réseau 10.0.0.0/8 (OUT)
> Router(config)# interface GigabitEthernet0/1
> Router(config-if)# ip access-group 15 out
> Router(config-if)# exit
> ```

### Règles importantes

> [!warning] Limitations par interface
> 
> - **Une seule ACL par interface et par direction**
> - Vous pouvez avoir une ACL IN et une ACL OUT sur la même interface
> - Si vous appliquez une nouvelle ACL, elle remplace l'ancienne

```bash
# Interface Gi0/0 peut avoir :
Router(config-if)# ip access-group 10 in   # ACL pour trafic entrant
Router(config-if)# ip access-group 20 out  # ACL pour trafic sortant
# Mais pas deux ACL "in" en même temps
```

### Retrait d'une ACL

```bash
# Retirer l'ACL d'une interface
Router(config)# interface GigabitEthernet0/0
Router(config-if)# no ip access-group 10 in

# Supprimer complètement l'ACL
Router(config)# no access-list 10

# Pour les ACL nommées
Router(config)# no ip access-list standard ADMIN_ONLY
```

---

## 📍 Placement recommandé

Le placement des ACL standard est **crucial** pour leur efficacité.

### Règle d'or

> [!tip] Règle de placement des ACL standard **Placez les ACL standard le plus PRÈS possible de la DESTINATION**

**Pourquoi ?**

- Les ACL standard filtrent uniquement par IP source
- Si placée trop près de la source, elle bloque TOUT le trafic de cette source, même vers des destinations légitimes
- Placée près de la destination, elle filtre juste avant l'accès au réseau protégé

### Illustration

```
[Réseau A] ----R1---- [Réseau B] ----R2---- [Réseau C]
 192.168.1.0/24       10.0.0.0/24           172.16.0.0/16

Objectif : Bloquer 192.168.1.0/24 d'accéder à 172.16.0.0/16
```

> [!example] Bon vs Mauvais placement
> 
> **❌ MAUVAIS : Près de la source (R1)**
> 
> ```bash
> # Sur R1, interface vers Réseau B (OUT)
> R1(config)# access-list 10 deny 192.168.1.0 0.0.0.255
> R1(config)# access-list 10 permit any
> R1(config)# interface Gi0/1
> R1(config-if)# ip access-group 10 out
> ```
> 
> **Problème :** Cela bloque aussi l'accès de 192.168.1.0/24 vers 10.0.0.0/24 !
> 
> **✅ BON : Près de la destination (R2)**
> 
> ```bash
> # Sur R2, interface vers Réseau C (OUT)
> R2(config)# access-list 10 deny 192.168.1.0 0.0.0.255
> R2(config)# access-list 10 permit any
> R2(config)# interface Gi0/1
> R2(config-if)# ip access-group 10 out
> ```
> 
> **Avantage :** 192.168.1.0/24 peut toujours accéder à 10.0.0.0/24

### Cas particulier : VTY (accès au routeur)

Pour sécuriser l'accès Telnet/SSH au routeur, l'ACL est appliquée sur les **lignes VTY** :

```bash
# Autoriser uniquement le réseau admin à se connecter au routeur
Router(config)# access-list 50 permit 172.16.100.0 0.0.0.255
Router(config)# line vty 0 4
Router(config-line)# access-class 50 in
Router(config-line)# exit
```

> [!info] Note Sur les VTY, on utilise `access-class` au lieu de `ip access-group`

---

## ⚠️ Pièges courants et bonnes pratiques

### Pièges courants

> [!warning] Erreur n°1 : Oublier le "permit any"
> 
> ```bash
> # Ceci bloque TOUT sauf 192.168.1.10
> Router(config)# access-list 10 permit host 192.168.1.10
> # deny any implicite !
> 
> # Correction : ajouter permit any si nécessaire
> Router(config)# access-list 10 permit host 192.168.1.10
> Router(config)# access-list 10 permit any
> ```

> [!warning] Erreur n°2 : Mauvais ordre des règles Les ACL sont traitées **dans l'ordre**, du haut vers le bas. La première règle qui correspond est appliquée.
> 
> ```bash
> # ❌ MAUVAIS : "permit any" en premier rend le deny inutile
> Router(config)# access-list 20 permit any
> Router(config)# access-list 20 deny host 10.0.0.50
> # Le deny ne sera JAMAIS atteint !
> 
> # ✅ BON : deny spécifique avant permit général
> Router(config)# access-list 20 deny host 10.0.0.50
> Router(config)# access-list 20 permit any
> ```

> [!warning] Erreur n°3 : Impossible de modifier une ACL numérotée Une fois créée, vous ne pouvez pas insérer ou supprimer une seule ligne d'une ACL numérotée standard. Vous devez la recréer entièrement.
> 
> ```bash
> # Pour modifier l'ACL 10, il faut :
> Router(config)# no access-list 10  # Supprime toute l'ACL
> Router(config)# access-list 10 deny host 10.0.0.100
> Router(config)# access-list 10 permit 192.168.1.0 0.0.0.255
> Router(config)# access-list 10 permit any
> ```
> 
> **Solution :** Utilisez des **ACL nommées** qui permettent l'insertion et la suppression de lignes individuelles.

> [!warning] Erreur n°4 : Placement incorrect Placer une ACL standard trop près de la source bloque tout le trafic de cette source, pas seulement vers la destination voulue.

> [!warning] Erreur n°5 : Inverser le wildcard mask
> 
> ```bash
> # ❌ ERREUR : utiliser le masque de sous-réseau au lieu du wildcard
> Router(config)# access-list 10 permit 192.168.1.0 255.255.255.0
> # Ceci ne correspondra probablement pas à ce que vous voulez !
> 
> # ✅ CORRECT : wildcard mask
> Router(config)# access-list 10 permit 192.168.1.0 0.0.0.255
> ```

### Bonnes pratiques

> [!tip] Bonne pratique n°1 : Utilisez des ACL nommées Elles sont plus lisibles et modifiables :
> 
> ```bash
> Router(config)# ip access-list standard BLOCAGE_VLAN20
> Router(config-std-nacl)# 10 deny 192.168.20.0 0.0.0.255
> Router(config-std-nacl)# 20 permit any
> Router(config-std-nacl)# exit
> 
> # Insertion d'une nouvelle règle entre les deux
> Router(config)# ip access-list standard BLOCAGE_VLAN20
> Router(config-std-nacl)# 15 deny host 192.168.10.50
> Router(config-std-nacl)# exit
> ```

> [!tip] Bonne pratique n°2 : Documentez vos ACL
> 
> ```bash
> Router(config)# access-list 10 remark === Filtrage réseau invités ===
> Router(config)# access-list 10 deny 192.168.100.0 0.0.0.255
> Router(config)# access-list 10 remark === Autoriser le reste ===
> Router(config)# access-list 10 permit any
> ```

> [!tip] Bonne pratique n°3 : Testez avant la mise en production
> 
> - Vérifiez avec `show access-lists`
> - Testez la connectivité avec `ping`, `traceroute`
> - Gardez un accès console en cas d'erreur de configuration

> [!tip] Bonne pratique n°4 : Planifiez vos numéros d'ACL Adoptez une convention :
> 
> - 1-19 : ACL pour VTY/administration
> - 20-39 : ACL pour contrôle inter-VLAN
> - 40-59 : ACL pour services spécifiques
> - etc.

> [!tip] Bonne pratique n°5 : Vérifiez régulièrement
> 
> ```bash
> # Voir les ACL configurées
> Router# show access-lists
> Router# show access-lists 10
> 
> # Voir où elles sont appliquées
> Router# show ip interface GigabitEthernet0/0
> 
> # Voir les statistiques de correspondance
> Router# show access-lists
> # (affiche le nombre de paquets correspondant à chaque ligne)
> ```

### Commandes de vérification

```bash
# Afficher toutes les ACL
Router# show access-lists

# Afficher une ACL spécifique
Router# show access-lists 10
Router# show ip access-list standard ADMIN_ONLY

# Voir les ACL appliquées sur une interface
Router# show ip interface GigabitEthernet0/0

# Voir un résumé des interfaces et leurs ACL
Router# show ip interface brief

# Réinitialiser les compteurs d'ACL
Router# clear access-list counters
Router# clear access-list counters 10
```

---

## 📊 Récapitulatif rapide

|Aspect|Détail|
|---|---|
|**Numéros**|1-99, 1300-1999|
|**Filtrage**|IP source uniquement|
|**Wildcard**|Inverse du masque de sous-réseau|
|**Application**|`ip access-group [ACL] {in\|out}`|
|**Placement**|Près de la **destination**|
|**Ordre**|Du haut vers le bas, première correspondance|
|**Règle finale**|`deny any` implicite|

> [!tip] Méthode de configuration en 3 étapes
> 
> 1. **Créer l'ACL** avec les règles appropriées
> 2. **Déterminer le placement** (près de la destination)
> 3. **Appliquer sur l'interface** dans la bonne direction (in/out)

---

**✨ Fin du cours sur les ACL Standard**