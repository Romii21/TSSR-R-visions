# Les Conteneurs et Docker - Synthèse Complète

## 📑 Sommaire

1. [Introduction aux conteneurs](#1-introduction-aux-conteneurs)
2. [Comprendre l'isolation des processus](#2-comprendre-lisolation-des-processus)
3. [Conteneurs vs Machines Virtuelles](#3-conteneurs-vs-machines-virtuelles)
4. [Le point de vue développeur](#4-le-point-de-vue-développeur)
5. [Le point de vue infrastructure](#5-le-point-de-vue-infrastructure)
6. [Les solutions de conteneurisation](#6-les-solutions-de-conteneurisation)
7. [Docker : Concepts fondamentaux](#7-docker--concepts-fondamentaux)
8. [Docker en pratique](#8-docker-en-pratique)
9. [Cas d'usage et bonnes pratiques](#9-cas-dusage-et-bonnes-pratiques)
10. [Ressources et références](#10-ressources-et-références)
11. [Points clés à retenir](#-points-clés-à-retenir)

---

## 1. Introduction aux conteneurs

### Définition

**Un conteneur** est un ensemble de processus s'exécutant dans un environnement isolé qui dispose :
- De son propre système de fichiers (FS)
- De ressources limitées (CPU, RAM, disque)
- D'un accès au noyau du système hôte

### Problématique initiale

Les infrastructures modernes doivent déployer et maintenir des applications qui reposent sur :

| Élément | Description |
|---------|-------------|
| **Processus** | Programmes en cours d'exécution |
| **Dépendances** | Bibliothèques et composants nécessaires |
| **Configurations** | Paramètres spécifiques à l'environnement |

**La conteneurisation est une réponse système aux problématiques de déploiement applicatif.**

---

## 2. Comprendre l'isolation des processus

### Accès d'un processus sur un OS

Par défaut, un processus a accès à :

| Ressource | Problème potentiel |
|-----------|-------------------|
| **Fichiers** | Accès non restreint |
| **Bibliothèques** | Conflits de versions |
| **Interfaces réseau** | Exposition inutile |
| **CPU et mémoire** | Consommation non contrôlée |

> ⚠️ **Problème** : Un processus voit plus que ce qui lui est strictement nécessaire, posant des problèmes de sécurité, stabilité et isolation.

### Solutions de cloisonnement

#### Méthodes historiques

| Commande | Système | Description |
|----------|---------|-------------|
| `chroot` | Linux | Change la racine apparente du système de fichiers |
| `jails` | BSD | Crée des environnements isolés |

**Objectifs du cloisonnement :**
- Limiter ce qu'un processus peut voir
- Limiter l'impact d'un incident ou d'une brèche de sécurité
- Réduire la surface d'attaque

### Concept moderne : La conteneurisation

Extension moderne du concept de "prison logicielle" qui permet de :

1. **Cloisonner** un processus et ses processus fils
2. **Déclarer explicitement** les ressources accessibles
3. **Créer** des instances séparées, reproductibles et indépendantes

---

## 3. Conteneurs vs Machines Virtuelles

### Différences architecturales

| Caractéristique | Machine Virtuelle (VM) | Conteneur |
|-----------------|----------------------|-----------|
| **Noyau** | Un OS complet par VM | Un seul noyau partagé |
| **Isolation** | Via hyperviseur | Via namespaces et cgroups |
| **Taille** | Plusieurs Go | Quelques Mo à quelques centaines de Mo |
| **Démarrage** | Minutes | Secondes |
| **Consommation** | CPU, RAM, disque élevés | Léger et optimisé |
| **Portabilité** | Limitée | Excellente |

### Architecture comparative

```
┌─────────────────────────────┐  ┌─────────────────────────────┐
│     Machine Virtuelle       │  │        Conteneur            │
├─────────────────────────────┤  ├─────────────────────────────┤
│      Application            │  │      Application            │
│      bins/libs              │  │      bins/libs              │
│      OS (invité)            │  ├─────────────────────────────┤
├─────────────────────────────┤  │      Docker Engine          │
│      Hyperviseur            │  │      OS (Hôte)              │
│      OS (Hôte)              │  │      Matériel               │
│      Matériel               │  └─────────────────────────────┘
└─────────────────────────────┘
```

**Glossaire :**
- **Hyperviseur** : Couche logicielle permettant de virtualiser des machines
- **Namespaces** : Mécanisme Linux d'isolation des ressources système
- **Cgroups** (Control Groups) : Mécanisme Linux de limitation des ressources

---

## 4. Le point de vue développeur

### Problématique : "Ça marche sur ma machine..."

#### Diversité des environnements

Les développeurs font face à :

| Environnement | Problèmes courants |
|---------------|-------------------|
| **Poste de développement** | Versions spécifiques installées |
| **Serveur de test** | Configuration différente |
| **Pré-production** | Permissions différentes |
| **Production** | Réseau et sécurité renforcés |

#### Conséquences

- Test et debug compliqués
- Déploiement à risque
- Résultats incohérents entre environnements

### Solutions apportées par la conteneurisation

✅ **Avantages pour les développeurs :**

1. **Portabilité** : L'application est livrée avec ses dépendances
2. **Reproductibilité** : Comportement identique partout
3. **Configuration embarquée** : Pas de surprise de configuration
4. **Isolation** : Pas de conflit avec d'autres applications

---

## 5. Le point de vue infrastructure

### Rôle et missions d'un TSSR (Technicien Supérieur Systèmes et Réseaux)

| Mission | Objectif |
|---------|----------|
| **Sécurité** | Réduire la surface d'attaque |
| **Ressources** | Gérer CPU, RAM, espace disque |
| **Fiabilité** | Déploiement reproductible |
| **Maintenance** | MAJ sans problème |

### Problématiques sans conteneurs

❌ **Difficultés rencontrées :**

- Services liés à un OS spécifique
- Dépendances partagées entre services (conflits)
- Conflits de versions (OS/services)
- Difficulté d'isolation d'un service compromis
- Gaspillage de ressources

### Solutions apportées par la conteneurisation

✅ **Avantages pour l'infrastructure :**

1. **Cloisonnement strict** des processus
2. **Déploiements rapides** et clonage facile
3. **Indépendance de l'OS** hôte
4. **Pas de trace** sur les disques après suppression
5. **Alternative légère** aux VM

---

## 6. Les solutions de conteneurisation

### Vue d'ensemble

| Type | Solution | Usage |
|------|----------|-------|
| **Bas niveau** | chroot, namespaces, cgroups, jails | Cloisonnement manuel |
| **Système** | LXC/LXD | Conteneurs système complets |
| **Applicatif** | Docker, Podman | Un service par conteneur |
| **Orchestration** | Docker Swarm, Kubernetes | Gestion multi-conteneurs |

### 6.1 Conteneurisation bas niveau

#### Commandes principales

```bash
# chroot - Change la racine du système de fichiers
sudo chroot /chemin/nouvelle/racine /bin/bash

# Namespaces - Isolation des ressources système
unshare --pid --fork --mount-proc /bin/bash

# Cgroups - Limitation des ressources
cgcreate -g cpu,memory:/mongroupe
```

### 6.2 Conteneurisation système : LXC/LXD

**LXC** (Linux Containers) crée des conteneurs système ressemblant à une VM complète.

#### Caractéristiques d'un conteneur LXC

| Élément | Description |
|---------|-------------|
| **Init system** | systemd ou autre |
| **Services** | Plusieurs services simultanés |
| **Utilisateurs** | Plusieurs comptes utilisateurs |
| **Arborescence** | Linux complète (/bin, /etc, /usr...) |

**LXD** : Couche de gestion des conteneurs LXC (images, réseaux, stockage)

```bash
# Commandes LXC/LXD
lxc launch ubuntu:22.04 mon-conteneur
lxc list
lxc exec mon-conteneur -- /bin/bash
```

> 💡 **Usage** : Parfait pour l'approche infrastructure/système/DevOps/cybersécurité

### 6.3 Conteneurisation applicative : Docker

**Principe** : Un processus principal par conteneur

#### Caractéristiques

- ✅ Isolation application/service
- ✅ Très léger (pas d'OS complet)
- ✅ Démarrage rapide
- ✅ Idéal pour les microservices

### 6.4 Orchestration de conteneurs

#### Problématique

```
1 conteneur + 1 machine + 1 application = Simple
N conteneurs + N machines + N applications = Complexe
```

#### Solutions d'orchestration

| Solution | Complexité | Usage |
|----------|------------|-------|
| **Docker Compose** | Faible | Multi-conteneurs sur une machine |
| **Docker Swarm** | Moyenne | Cluster de Docker Engine |
| **Kubernetes** | Élevée | Orchestration poussée, production |

**Fonctionnalités de l'orchestration :**
- Gestion du cycle de vie des conteneurs
- Répartition de charge (load balancing)
- Scalabilité horizontale et verticale
- Haute disponibilité
- Auto-réparation (self-healing)

---

## 7. Docker : Concepts fondamentaux

### Vue d'ensemble

Docker structure le déploiement autour de 4 concepts clés :

```
Dockerfile → build → Image → run → Conteneur
                       ↓
                    Repository
```

### 7.1 Les Images Docker

#### Définition

Une **image Docker** est :
- L'ensemble des dossiers et fichiers constituant le système de fichiers racine
- Les informations de configuration (commande à exécuter, variables d'environnement, etc.)
- Une pile de couches (layers) en lecture seule

#### Architecture en couches

```
┌─────────────────────────┐
│   Application Layer     │  ← Votre code
├─────────────────────────┤
│   Dependencies Layer    │  ← npm install, pip install...
├─────────────────────────┤
│   Runtime Layer         │  ← Node.js, Python...
├─────────────────────────┤
│   Base OS Layer         │  ← Alpine, Ubuntu...
└─────────────────────────┘
```

**Avantages des couches :**
- Partage entre images (économie d'espace)
- Cache lors de la construction
- Téléchargement optimisé

#### Commandes images

```bash
# Lister les images
docker images
docker image ls

# Télécharger une image
docker pull nginx:latest
docker pull ubuntu:22.04

# Supprimer une image
docker rmi nginx:latest
docker image rm nginx:latest

# Inspecter une image
docker image inspect nginx:latest

# Historique des couches
docker image history nginx:latest
```

### 7.2 Les Conteneurs

#### Définition

Un **conteneur** est :
- Une instance d'une image en cours d'exécution
- Comparable à un processus pour un programme
- Éphémère par nature (peut être supprimé et recréé)

#### Relation Image ↔ Conteneur

| Image | Conteneur |
|-------|-----------|
| Programme | Processus |
| Lecture seule | Lecture/Écriture |
| Partagée | Isolé |
| Modèle | Instance |

#### Commandes conteneurs

```bash
# Créer et démarrer un conteneur
docker run nginx:latest
docker run -d nginx:latest              # En arrière-plan (detached)
docker run -it ubuntu:22.04 /bin/bash  # Mode interactif

# Lister les conteneurs
docker ps                    # Conteneurs actifs
docker ps -a                # Tous les conteneurs
docker container ls         # Équivalent à docker ps

# Démarrer/Arrêter
docker start mon-conteneur
docker stop mon-conteneur
docker restart mon-conteneur

# Exécuter une commande dans un conteneur actif
docker exec -it mon-conteneur /bin/bash

# Voir les logs
docker logs mon-conteneur
docker logs -f mon-conteneur  # Mode suivi (follow)

# Supprimer un conteneur
docker rm mon-conteneur
docker rm -f mon-conteneur    # Force la suppression

# Statistiques en temps réel
docker stats
```

### 7.3 Les Dépôts (Repositories)

#### Docker Hub

**Docker Hub** est le registre public officiel de Docker Inc.

| Caractéristique | Description |
|-----------------|-------------|
| **Images officielles** | nginx, postgres, redis, ubuntu... |
| **Images publiques** | Partagées par la communauté |
| **Images privées** | Nécessitent authentification |

#### Commandes dépôts

```bash
# Se connecter à Docker Hub
docker login

# Télécharger une image
docker pull nginx:latest
docker pull mon-utilisateur/mon-image:v1.0

# Publier une image
docker tag mon-image:latest mon-utilisateur/mon-image:v1.0
docker push mon-utilisateur/mon-image:v1.0

# Rechercher une image
docker search nginx
```

#### Dépôt privé

Il est possible d'héberger son propre registre :

```bash
# Démarrer un registre local
docker run -d -p 5000:5000 --name registry registry:2

# Taguer pour le registre local
docker tag mon-image:latest localhost:5000/mon-image:latest

# Pousser vers le registre local
docker push localhost:5000/mon-image:latest
```

### 7.4 Les Volumes

#### Définition

Un **volume Docker** est un espace de stockage persistant géré par Docker.

**Problématique** : Les données dans un conteneur sont perdues à sa suppression.

**Solution** : Les volumes permettent :
- Le stockage de données persistantes
- Le partage de fichiers entre conteneurs
- Le partage de fichiers entre l'hôte et les conteneurs

#### Types de montage

| Type | Description | Usage |
|------|-------------|-------|
| **Volume** | Géré par Docker | Données persistantes |
| **Bind mount** | Dossier de l'hôte | Développement, configuration |
| **tmpfs mount** | En mémoire | Données temporaires sensibles |

#### Commandes volumes

```bash
# Créer un volume
docker volume create mon-volume

# Lister les volumes
docker volume ls

# Inspecter un volume
docker volume inspect mon-volume

# Utiliser un volume avec un conteneur
docker run -d -v mon-volume:/app/data nginx:latest

# Bind mount (monter un dossier de l'hôte)
docker run -d -v /chemin/hote:/chemin/conteneur nginx:latest
docker run -d -v $(pwd):/app nginx:latest  # Dossier courant

# Supprimer un volume
docker volume rm mon-volume

# Nettoyer les volumes inutilisés
docker volume prune
```

### 7.5 Les Réseaux

#### Définition

Docker crée des **réseaux virtuels** permettant la communication entre conteneurs.

#### Types de réseaux

| Type | Description | Usage |
|------|-------------|-------|
| **bridge** | Réseau par défaut | Communication inter-conteneurs |
| **host** | Utilise le réseau de l'hôte | Performance maximale |
| **none** | Pas de réseau | Isolation totale |
| **overlay** | Multi-hôtes | Docker Swarm, Kubernetes |

#### Commandes réseaux

```bash
# Lister les réseaux
docker network ls

# Créer un réseau
docker network create mon-reseau
docker network create --driver bridge mon-reseau-bridge

# Inspecter un réseau
docker network inspect mon-reseau

# Connecter un conteneur à un réseau
docker network connect mon-reseau mon-conteneur

# Déconnecter un conteneur d'un réseau
docker network disconnect mon-reseau mon-conteneur

# Utiliser un réseau au lancement
docker run -d --network mon-reseau nginx:latest

# Publier des ports
docker run -d -p 8080:80 nginx:latest      # hôte:8080 → conteneur:80
docker run -d -p 127.0.0.1:8080:80 nginx  # Seulement localhost

# Supprimer un réseau
docker network rm mon-reseau

# Nettoyer les réseaux inutilisés
docker network prune
```

---

## 8. Docker en pratique

### 8.1 Composants Docker

| Composant | Description |
|-----------|-------------|
| **Docker Engine** | Serveur de gestion (dockerd) + CLI (docker) |
| **Docker Desktop** | Interface graphique (obligatoire sous Windows) |
| **Docker Compose** | Gestion d'applications multi-conteneurs |

### 8.2 Le Dockerfile

#### Définition

Le **Dockerfile** est un fichier texte contenant les instructions pour construire une image.

#### Instructions principales

| Instruction | Description | Exemple |
|-------------|-------------|---------|
| `FROM` | Image de base | `FROM node:18-alpine` |
| `WORKDIR` | Définit le répertoire de travail | `WORKDIR /app` |
| `COPY` | Copie des fichiers locaux | `COPY . .` |
| `ADD` | Copie + extraction archives | `ADD app.tar.gz /app` |
| `RUN` | Exécute une commande (build) | `RUN npm install` |
| `CMD` | Commande par défaut (run) | `CMD ["node", "app.js"]` |
| `ENTRYPOINT` | Point d'entrée | `ENTRYPOINT ["python"]` |
| `EXPOSE` | Ports en écoute | `EXPOSE 3000` |
| `ENV` | Variables d'environnement | `ENV NODE_ENV=production` |
| `VOLUME` | Points de montage | `VOLUME /data` |
| `USER` | Utilisateur d'exécution | `USER node` |
| `ARG` | Arguments de build | `ARG VERSION=1.0` |

#### Exemple pratique : Application Node.js

```dockerfile
# Image de base légère
FROM node:18-alpine

# Définir le répertoire de travail
WORKDIR /app

# Copier les fichiers de dépendances
COPY package*.json ./

# Installer les dépendances
RUN npm install --production

# Copier le code source
COPY . .

# Exposer le port
EXPOSE 3000

# Définir l'utilisateur non-root
USER node

# Commande de démarrage
CMD ["node", "src/index.js"]
```

#### Bonnes pratiques Dockerfile

✅ **Recommandations :**

1. **Utiliser des images officielles** légères (alpine)
2. **Minimiser le nombre de couches** (regrouper les RUN)
3. **Ordonner les instructions** (du moins au plus changeant)
4. **Utiliser .dockerignore** (exclure node_modules, .git...)
5. **Ne pas exécuter en root** (USER)
6. **Un seul processus** par conteneur
7. **Nettoyer les caches** (apt clean, npm cache clean)

#### Exemple .dockerignore

```
node_modules
npm-debug.log
.git
.gitignore
.env
*.md
.DS_Store
```

#### Construire une image

```bash
# Construction basique
docker build -t mon-app:latest .

# Spécifier un Dockerfile
docker build -f Dockerfile.prod -t mon-app:prod .

# Avec arguments
docker build --build-arg VERSION=2.0 -t mon-app:2.0 .

# Sans cache
docker build --no-cache -t mon-app:latest .

# Multi-stage build
docker build --target production -t mon-app:prod .
```

### 8.3 Docker Compose

#### Définition

**Docker Compose** permet de définir et gérer des applications multi-conteneurs via un fichier YAML.

#### Fichier docker-compose.yml

```yaml
version: '3.8'

services:
  # Service web
  web:
    build: .
    ports:
      - "8080:80"
    volumes:
      - ./app:/app
    environment:
      - NODE_ENV=production
    depends_on:
      - db
    networks:
      - app-network

  # Service base de données
  db:
    image: postgres:15-alpine
    volumes:
      - db-data:/var/lib/postgresql/data
    environment:
      POSTGRES_PASSWORD: secret
      POSTGRES_DB: myapp
    networks:
      - app-network

  # Service Redis
  redis:
    image: redis:7-alpine
    networks:
      - app-network

volumes:
  db-data:

networks:
  app-network:
    driver: bridge
```

#### Commandes Docker Compose

```bash
# Démarrer les services
docker-compose up
docker-compose up -d              # En arrière-plan
docker-compose up --build         # Rebuild les images

# Arrêter les services
docker-compose stop
docker-compose down               # Arrête et supprime
docker-compose down -v            # + supprime les volumes

# Voir les logs
docker-compose logs
docker-compose logs -f web        # Suivre les logs du service web

# Lister les services
docker-compose ps

# Exécuter une commande
docker-compose exec web /bin/bash

# Redémarrer un service
docker-compose restart web

# Construire les images
docker-compose build
```

### 8.4 Gestion globale Docker

#### Commandes system

```bash
# Informations système
docker system info
docker system df              # Utilisation disque

# Événements en temps réel
docker system events

# Nettoyage global
docker system prune           # Nettoie tout l'inutilisé
docker system prune -a        # + images non utilisées
docker system prune --volumes # + volumes
```

#### Nettoyage ciblé

```bash
# Nettoyer les conteneurs arrêtés
docker container prune

# Nettoyer les images non utilisées
docker image prune
docker image prune -a         # Toutes les images non utilisées

# Nettoyer les volumes
docker volume prune

# Nettoyer les réseaux
docker network prune
```

---

## 9. Cas d'usage et bonnes pratiques

### 9.1 Cas d'usage typiques

| Cas d'usage | Bénéfice |
|-------------|----------|
| **Développement local** | Environnement identique pour toute l'équipe |
| **CI/CD** | Tests dans des environnements propres |
| **Microservices** | Isolation et scalabilité |
| **Applications legacy** | Isolation des anciennes versions |
| **Tests de charge** | Déploiement rapide de multiples instances |

### 9.2 Recommandations Docker

#### Philosophie "un conteneur = un service"

✅ **Bonnes pratiques :**

```
✓ 1 conteneur = 1 processus principal
✓ Conteneurs éphémères (redéployés fréquemment)
✓ Configuration via variables d'environnement
✓ Logs vers stdout/stderr
✓ Données persistantes dans des volumes
```

❌ **À éviter :**

```
✗ Plusieurs services dans un conteneur
✗ SSH dans les conteneurs
✗ Données importantes sans volume
✗ Mots de passe en dur dans les images
✗ Exécution en root
```

### 9.3 Sécurité

#### Checklist sécurité

| Aspect | Action |
|--------|--------|
| **Images** | Utiliser des images officielles et vérifiées |
| **Utilisateur** | Ne jamais exécuter en root (USER) |
| **Secrets** | Utiliser Docker secrets ou variables d'environnement |
| **Réseau** | Limiter l'exposition des ports |
| **Mises à jour** | Maintenir les images à jour |
| **Scan** | Scanner les vulnérabilités (docker scan) |

#### Commandes sécurité

```bash
# Scanner une image
docker scan mon-image:latest

# Vérifier l'utilisateur
docker inspect mon-conteneur | grep User

# Limiter les ressources
docker run --memory="512m" --cpus="1.5" nginx
```

#### Guide ANSSI

📚 L'ANSSI (Agence Nationale de la Sécurité des Systèmes d'Information) propose un guide de sécurisation Docker :
- Recommandations sur la configuration
- Bonnes pratiques de déploiement
- Gestion des secrets
- Isolation réseau

> 🔗 Disponible sur le site de l'ANSSI : https://www.ssi.gouv.fr/

---

## 10. Ressources et références

### Documentation officielle Docker

| Ressource | Description | Lien |
|-----------|-------------|------|
| **Documentation complète** | Référence exhaustive | https://docs.docker.com/ |
| **Guides** | Tutoriels pas à pas | https://docs.docker.com/guides/ |
| **Reference** | API et CLI | https://docs.docker.com/reference/ |
| **Docker 101 Tutorial** | Introduction interactive | https://www.docker.com/101-tutorial |

### Ressources complémentaires

- **Docker Compose** : https://docs.docker.com/compose/
- **Docker Swarm** : https://docs.docker.com/engine/swarm/
- **Kubernetes** : https://kubernetes.io/fr/docs/home/
- **Site d'Hadrien Pelissier** : Ressources francophones
- **Site de Stéphane Robert** : Tutoriels avancés

### Outils et extensions

| Outil | Usage |
|-------|-------|
| **Portainer** | Interface web pour gérer Docker |
| **Lazydocker** | TUI (Terminal UI) pour Docker |
| **Dive** | Analyser les couches d'images |
| **Hadolint** | Linter pour Dockerfile |

---

## 🎯 Points clés à retenir

### Concepts fondamentaux

| Concept | À retenir |
|---------|-----------|
| **Conteneur** | Environnement isolé partageant le noyau de l'hôte |
| **Image** | Modèle en lecture seule pour créer des conteneurs |
| **Volume** | Stockage persistant géré par Docker |
| **Réseau** | Communication entre conteneurs |

### Différences Conteneur vs VM

```
Conteneur              │  Machine Virtuelle
─────────────────────  │  ─────────────────────
✓ Léger (Mo)           │  ✗ Lourd (Go)
✓ Démarrage rapide (s) │  ✗ Démarrage lent (min)
✓ Partage le noyau     │  ✗ OS complet
✓ Haute densité        │  ✗ Basse densité
```

### Commandes essentielles

```bash
# Images
docker pull nginx:latest        # Télécharger
docker build -t app:1.0 .      # Construire
docker images                   # Lister

# Conteneurs
docker run -d -p 8080:80 nginx # Lancer
docker ps                       # Lister actifs
docker exec -it app bash       # Exécuter commande
docker logs -f app             # Voir logs
docker stop app                # Arrêter

# Volumes
docker volume create data      # Créer
docker run -v data:/app nginx  # Utiliser

# Réseaux
docker network create net      # Créer
docker run --network net nginx # Utiliser

# Nettoyage
docker system prune -a         # Tout nettoyer
```

### Workflow type

```
1. Écrire le Dockerfile
2. Construire l'image : docker build -t app:1.0 .
3. Tester localement : docker run -p 8080:80 app:1.0
4. Pousser vers le registre : docker push mon-user/app:1.0
5. Déployer en production : docker pull + docker run
```

### Bonnes pratiques essentielles

✅ **À faire absolument :**

1. **Un service par conteneur** (séparation des responsabilités)
2. **Images légères** (alpine, slim)
3. **Utilisateur non-root** (USER dans Dockerfile)
4. **Variables d'environnement** pour la configuration
5. **Volumes** pour les données persistantes
6. **Logs vers stdout/stderr**
7. **Conteneurs éphémères** (faciles à recréer)
8. **.dockerignore** pour exclure les fichiers inutiles

❌ **À éviter absolument :**

1. Mots de passe en dur dans les images
2. Exécution en root
3. Installation SSH dans les conteneurs
4. Plusieurs services dans un conteneur
5. Images non maintenues

### Avantages de la conteneurisation

| Acteur | Bénéfices |
|--------|-----------|
| **Développeurs** | Environnements cohérents, "ça marche partout" |
| **Ops/TSSR** | Déploiement rapide, isolation, sécurité |
| **Entreprise** | Coûts réduits, scalabilité, agilité |

### Pour aller plus loin

1. **Docker Compose** : Orchestrer plusieurs conteneurs
2. **Docker Swarm** : Cluster de conteneurs
3. **Kubernetes** : Orchestration avancée (production)
4. **Sécurité** : Guide ANSSI, scan de vulnérabilités
5. **CI/CD** : Intégration dans GitLab CI, GitHub Actions, Jenkins

---

## Glossaire rapide

| Terme | Définition |
|-------|------------|
| **ANSSI** | Agence Nationale de la Sécurité des Systèmes d'Information |
| **API** | Application Programming Interface |
| **Bind mount** | Montage d'un dossier de l'hôte dans un conteneur |
| **Bridge** | Type de réseau Docker par défaut |
| **Build** | Construction d'une image Docker |
| **Cgroups** | Control Groups - Limitation des ressources Linux |
| **CLI** | Command Line Interface - Interface en ligne de commande |
| **Container** | Conteneur - Instance d'une image en exécution |
| **DHCP** | Dynamic Host Configuration Protocol |
| **Dockerfile** | Fichier de construction d'une image Docker |
| **FS** | File System - Système de fichiers |
| **Image** | Modèle en lecture seule pour créer des conteneurs |
| **Layer** | Couche d'une image Docker |
| **LXC** | Linux Containers - Conteneurisation système |
| **LXD** | Gestionnaire de conteneurs LXC |
| **Namespace** | Mécanisme d'isolation Linux |
| **NAT** | Network Address Translation |
| **OS** | Operating System - Système d'exploitation |
| **Pull** | Télécharger une image depuis un registre |
| **Push** | Publier une image vers un registre |
| **Registry** | Dépôt d'images Docker |
| **Repository** | Dépôt - Ensemble d'images versionnées |
| **Run** | Créer et démarrer un conteneur |
| **TSSR** | Technicien Supérieur Systèmes et Réseaux |
| **VM** | Virtual Machine - Machine virtuelle |
| **Volume** | Espace de stockage persistant géré par Docker |

---

**Document créé le** : Janvier 2026  
**Basé sur** : Cours "Conteneurs et Docker" + Documentation officielle Docker  
**Dernière mise à jour** : Janvier 2026

---

*Cette synthèse couvre les concepts essentiels de la conteneurisation et de Docker. Pour approfondir, consultez la documentation officielle et les guides de sécurité de l'ANSSI.*
