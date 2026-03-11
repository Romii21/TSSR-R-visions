# CP7 – Fiche de Révision 2 : Pare-feu et Politique de Filtrage

> **Formation** : TSSR | **Compétence** : CP7  
> **Thème** : Définition, fonctionnement, types de pare-feux, politique de filtrage

---

## 1. Définition et rôle du pare-feu

Un **pare-feu** (firewall, garde-barrière, coupe-feu) est un dispositif de sécurité réseau qui :

- Se situe à l'**intersection de réseaux** (en coupure)
- Fonctionne **au minimum au niveau 4** (couche Transport : TCP/UDP) du modèle OSI
- **Contrôle et filtre** les paquets qui le traversent
- Est souvent associé à des **fonctions de routage**

**Objectif principal** : Fournir une connectivité **contrôlée et maîtrisée** entre des réseaux ayant des niveaux de confiance différents.

---

## 2. Principe de fonctionnement

Le pare-feu inspecte les paquets dans trois directions :

- **Entrants** : depuis l'extérieur vers le réseau interne
- **Sortants** : depuis le réseau interne vers l'extérieur
- **Traversants** : d'un réseau à un autre via le pare-feu

Pour chaque paquet, le pare-feu :

1. Déroule la pile protocolaire (au moins jusqu'au niveau 4)
2. Analyse les informations : IP source/destination, ports, protocoles, contenu (selon le type)
3. Prend **une des trois décisions** :

| Décision | Comportement | Usage |
|---|---|---|
| **Accept** | Le paquet est autorisé à passer | Trafic légitime |
| **Drop** | Le paquet est bloqué silencieusement (l'expéditeur n'est pas informé) | ✅ Le plus courant |
| **Reject** | Le paquet est bloqué ET l'expéditeur est notifié | ❌ Le moins courant (révèle qu'un pare-feu existe) |

---

## 3. Les types de pare-feux

### 3.1 Pare-feu sans état (Stateless)

**Principe** : Filtrage basé sur des **règles simples** appliquées à chaque paquet indépendamment, sans mémoire des connexions précédentes.

**Ce qu'il analyse** : IP source/destination, ports, protocoles.

| Avantages | Inconvénients |
|---|---|
| ✅ Très rapide et efficace | ❌ Chaque paquet traité indépendamment (pas de contexte) |
| ✅ Simple à configurer | ❌ Règles nombreuses et complexes nécessaires |

**Niveau OSI** : 3-4

---

### 3.2 Pare-feu à états (Stateful)

**Principe** : Le pare-feu possède une **mémoire des connexions actives**. Il vérifie la **conformité d'un paquet dans son contexte**.

**Ce qu'il apporte en plus** :
- Suivi des connexions TCP (état SYN, SYN-ACK, ESTABLISHED…)
- **Autorisation implicite des réponses** : pas besoin de règle pour le trafic retour
- Détection des paquets hors contexte (ex : RST ou FIN sans session établie)

| Avantages | Inconvénients |
|---|---|
| ✅ Filtrage plus intelligent | ❌ Consomme plus de ressources (RAM, CPU) |
| ✅ Allège l'écriture des règles | |
| ✅ Détecte les paquets invalides | |

**Niveau OSI** : 3-4  
> Check Point a été le pionnier du stateful inspection dans les années 90.

---

### 3.3 Pare-feu applicatif (DPI – Deep Packet Inspection)

**Principe** : Analyse l'**intégralité de la pile protocolaire**, jusqu'à la **couche 7 (Application)**. Il examine les données, pas seulement les en-têtes.

**Ce qu'il apporte en plus** :
- Analyse du contenu des paquets
- Vérification de la **conformité avec les protocoles applicatifs**
- Détection de malwares, attaques applicatives, contenus malveillants
- Gestion de protocoles complexes comme le FTP

| Avantages | Inconvénients |
|---|---|
| ✅ Analyse approfondie | ❌ Nécessite la connaissance du protocole applicatif |
| ✅ Détection d'attaques niveau 7 | ❌ **Impossible avec le chiffrement de bout en bout** |
| ✅ Filtrage fin | ❌ Consomme beaucoup de ressources |

**Niveau OSI** : 3-7  
> Les **WAF (Web Application Firewall)** sont une spécialisation du DPI pour HTTP/HTTPS.

---

### 3.4 Pare-feu personnel

Logiciel installé **sur une machine individuelle** pour filtrer son propre trafic.

| Avantages | Inconvénients |
|---|---|
| ✅ Filtrage interactif (alertes utilisateur) | ❌ Moins sûr (si la machine est compromise, le pare-feu l'est aussi) |
| ✅ Protection même sur réseau non sécurisé | ❌ Dépend entièrement de la machine hôte |

---

### 3.5 Tableau récapitulatif des types

| Type | Niveau OSI | Mémoire | Performances | Sécurité | Cas d'usage |
|---|---|---|---|---|---|
| **Stateless** | 3-4 | Non | ⚡⚡⚡ | ⭐⭐ | Filtrage simple, haute performance |
| **Stateful** | 3-4 | Oui | ⚡⚡ | ⭐⭐⭐ | Usage général, équilibre perf/sécu |
| **DPI/Applicatif** | 3-7 | Oui | ⚡ | ⭐⭐⭐⭐ | Sécurité maximale, analyse contenu |
| **Personnel** | 3-7 | Oui | ⚡⚡ | ⭐⭐ | Poste individuel |

---

## 4. Politique de filtrage

### 4.1 Les deux approches

| Approche | Principe | Recommandation |
|---|---|---|
| **Liste de blocage** (blacklist) | Tout autoriser par défaut, puis bloquer ce qui est dangereux | ❌ **Non recommandée** – risque d'oubli, peu d'alertes |
| **Liste d'autorisation** (whitelist) | **Tout bloquer par défaut** (deny all), puis autoriser uniquement le légitime | ✅ **Recommandée** – principe de base |

> La règle d'or : **"Ce qui n'est pas explicitement autorisé est interdit."**

### 4.2 Matrice des flux

Document qui définit l'ensemble des **communications légitimes** entre chaque zone du réseau.

- Source, destination, port, protocole
- Sert de base à l'écriture des règles du pare-feu
- Doit être tenue à jour

---

## 5. Solutions du marché

Il n'existe **pas de solution universellement meilleure**. Le choix dépend du contexte :

| Solution | Profil cible | Point fort principal |
|---|---|---|
| **Check Point** | Grandes entreprises | Expertise historique, conformité |
| **Fortinet (FortiGate)** | PME/ETI | Rapport qualité/prix, suite unifiée |
| **Cisco ASA/FirePower** | Environnements Cisco | Intégration écosystème Cisco |
| **Palo Alto Networks** | Tous types | Visibilité applicative (App-ID) |

> ⚠️ **Un pare-feu mal configuré est inefficace.** La configuration est aussi importante que le matériel.

---

## 6. Points clés à retenir

1. Un pare-feu fonctionne **au minimum en couche 4**, jusqu'à la couche 7 pour le DPI.
2. Trois décisions possibles : **Accept / Drop / Reject** (Drop est le plus courant).
3. **Stateless** = rapide, sans contexte. **Stateful** = intelligent, avec mémoire des connexions.
4. Le **DPI** analyse le contenu applicatif, mais est limité par le **chiffrement de bout en bout**.
5. La politique recommandée : **liste d'autorisation** (deny all par défaut).
6. Un pare-feu seul ne suffit pas : il faut une **approche globale de sécurité**.

---

_Fiche CP7 – 2/5 | TSSR_
