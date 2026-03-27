# CP7 – Fiche de Révision 3 : Architecture Réseau Sécurisée

> **Formation** : TSSR | **Compétence** : CP7  
> **Thème** : Défense en profondeur, zones de confiance, DMZ, matrice des flux

---

## 1. Pourquoi sécuriser l'architecture réseau ?

Tous les systèmes présentent des **failles de sécurité**, des **configurations par défaut peu sécurisées** et des **services inutiles activés**. Il est donc indispensable de concevoir une architecture qui :

- **Résiste** aux attaques
- **Détecte** les menaces
- **Contient** les incidents (évite leur propagation)
- **Se relève** après une compromission

L'approche recommandée est le **"Security by Design"** : la sécurité est pensée dès la conception du réseau, et non ajoutée après coup.

---

## 2. Défense en profondeur

### Principe

**Ne jamais faire confiance à une seule ligne de défense.**

Au lieu d'une simple **défense périmétrique** (un seul pare-feu à l'entrée du réseau), on applique plusieurs **couches de sécurité successives** :

```
Internet
   │
[Pare-feu périmétrique]        ← 1ère ligne de défense
   │
[DMZ – serveurs exposés]       ← zone tampon surveillée
   │
[Pare-feu interne]             ← 2ème ligne de défense
   │
[Réseau interne segmenté]      ← zones de confiance distinctes
   │
[Pare-feux personnels]         ← dernière ligne de défense
```

**Avantages** :
- Un attaquant qui franchit une couche est bloqué par la suivante
- Limite la propagation latérale d'une attaque
- Donne du temps pour détecter et réagir

---

## 3. Zones de confiance

### Principe

Segmenter le réseau en **zones regroupant les équipements ayant les mêmes besoins en sécurité**. Le filtrage s'effectue entre ces zones.

La segmentation peut être physique (réseaux séparés) ou logique (VLANs).

### Exemples de zones typiques en entreprise

| Zone | Description | Niveau de filtrage |
|---|---|---|
| **Utilisateurs** | Postes de travail internes | Fortement filtré (sortie uniquement) |
| **Serveurs** | Serveurs internes (fichiers, AD…) | Filtrage modéré |
| **Administration** | Zone d'administration réseau/système | Très fortement filtré |
| **DMZ** | Serveurs accessibles depuis l'extérieur | Surveillance et filtrage spécifiques |
| **Visiteurs/WiFi** | Réseau invités | Isolé totalement du réseau interne |

### Architecture schématique

```
                         Internet
                            │
                       [Pare-feu]
                            │
         +------------------+------------------+
         │                  │                  │
        DMZ             Visiteurs          Réseau
    (Web, Mail)           (WiFi)           interne
                                               │
                                    +----------+----------+
                                    │                     │
                               Serveurs             Utilisateurs
                                                          │
                                                   Administration
```

---

## 4. La DMZ (DeMilitarized Zone)

### Définition

La **DMZ** (Zone Démilitarisée) est une zone réseau intermédiaire, placée entre Internet et le réseau interne. Elle contient les serveurs qui doivent être **accessibles depuis l'extérieur**.

### Ce qu'on y place

- Serveur **web** (HTTP/HTTPS)
- Serveur **mail** (SMTP)
- Serveur **DNS** public
- Serveur **FTP** public
- **Reverse proxy**

> ⚠️ On ne place en DMZ **que** les machines qui doivent communiquer avec l'extérieur.  
> Les serveurs internes (bases de données, annuaires LDAP…) restent dans le réseau interne.

### Caractéristiques de la DMZ

- **Surveillance renforcée** (logs, alertes)
- **Filtrage spécifique** : seuls les flux nécessaires sont autorisés
- **Point d'entrée potentiel** : en cas de compromission d'un serveur DMZ, le réseau interne doit rester protégé

### Règles de filtrage typiques en DMZ

| Flux | Autorisé ? |
|---|---|
| Internet → DMZ (port 80, 443) | ✅ Oui |
| DMZ → Internet | ✅ Limité (ex : mises à jour) |
| DMZ → Réseau interne | ❌ Interdit (ou très restreint) |
| Réseau interne → DMZ | ⚠️ Administration uniquement |

---

## 5. La matrice des flux

### Définition

Document qui recense **toutes les communications légitimes** autorisées entre les différentes zones du réseau.

### Structure type

| Source | Destination | Port | Protocole | Justification |
|---|---|---|---|---|
| Utilisateurs | Internet | 80, 443 | TCP | Navigation web |
| Utilisateurs | Serveurs | 445 | TCP | Partage fichiers SMB |
| DMZ (web) | Serveurs BDD | 3306 | TCP | Requêtes base de données |
| Administration | Tous | 22 | TCP | SSH administration |
| Internet | DMZ (web) | 443 | TCP | HTTPS public |

### Utilité

- Sert de **référence** pour l'écriture des règles du pare-feu
- Permet d'auditer les règles existantes
- Documente les décisions de sécurité
- Doit être **tenue à jour** lors de chaque modification

---

## 6. Publication de services avec NAT + DMZ

Lorsqu'on publie un service interne sur Internet, la bonne pratique est de combiner **DNAT** et **DMZ** :

```
Internet → [Pare-feu avec DNAT] → [Serveur en DMZ] → (si besoin) → Serveur BDD interne
```

**Principe** :
1. Le pare-feu externe reçoit les connexions entrantes et les redirige (DNAT) vers le serveur en DMZ
2. Le serveur en DMZ traite les requêtes publiques
3. Si ce serveur doit accéder au réseau interne (ex : BDD), c'est filtré strictement par un deuxième pare-feu

---

## 7. Points clés à retenir

1. La **défense en profondeur** = plusieurs couches de sécurité, pas une seule barrière.
2. Les **zones de confiance** segmentent le réseau selon les besoins de sécurité.
3. La **DMZ** isole les serveurs exposés pour protéger le réseau interne en cas de compromission.
4. On ne place en DMZ **que** les machines qui communiquent avec l'extérieur.
5. La **matrice des flux** est l'outil de référence pour gérer les règles de filtrage.
6. Un pare-feu seul ne suffit pas : la sécurité réseau nécessite une **architecture globale pensée**.

---

_Fiche CP7 – 3/5 | TSSR_
