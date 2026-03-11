# FICHE 2 — Administration d'une infrastructure Proxmox
### Formation TSSR — CP5 : Maintenir des serveurs dans une infrastructure virtualisée

---

## 1. Création d'une VM dans Proxmox

Lors de la création d'une VM, les paramètres suivants doivent être définis :

| Paramètre | Description | Exemple |
|---|---|---|
| **ID** | Identifiant unique de la VM dans Proxmox | 101, 102... |
| **Nom** | Nom descriptif de la VM | srv-web, srv-ad |
| **Type d'OS** | Aide Proxmox à optimiser les pilotes | Linux 6.x, Windows 2022 |
| **ISO** | Image d'installation du système | debian-12.iso |
| **Disque** | Taille, format (QCOW2), stockage cible | 50 Go sur local-lvm |
| **CPU** | Nombre de sockets et de cœurs | 2 cœurs |
| **RAM** | Quantité allouée | 4096 Mo |
| **Réseau** | Bridge, modèle de carte (VirtIO recommandé) | vmbr0, VirtIO |
| **Boot order** | Ordre de démarrage | Disque, puis ISO |
| **BIOS/UEFI** | Type de firmware | SeaBIOS (BIOS legacy) ou OVMF (UEFI) |

---

## 2. Templates et clonage

### Template (Modèle)
Un **template** est une VM **convertie en modèle non démarrable** servant de base pour créer des VMs rapidement.

**Workflow de création :**
```
1. Créer une VM normale
2. Installer et configurer l'OS de base
3. Installer les outils communs (qemu-guest-agent, sudo, etc.)
4. Éteindre la VM
5. Clic droit → "Convertir en template"
6. Cloner le template pour chaque nouvelle VM
```

**Avantages :** Standardisation, gain de temps, cohérence des configurations.

---

### Clone complet vs Clone lié

| | Clone complet (Full) | Clone lié (Linked) |
|---|---|---|
| **Fonctionnement** | Copie indépendante et intégrale du disque | Partage le disque du template, stocke uniquement les différences |
| **Espace disque** | Autant que la VM source | Très faible au départ |
| **Indépendance** | Totale (fonctionne sans le template) | Dépend du template source |
| **Performance** | Optimale | Légèrement réduite |
| **Usage** | Production, déploiement permanent | Tests rapides, environnements temporaires |

> **Règle pratique :** En production → clone complet. Pour des labs ou tests → clone lié.

---

## 3. Le réseau virtuel dans Proxmox

### Bridge Linux (vmbr)
Un **bridge** est un switch virtuel logiciel intégré au kernel Linux. Il connecte les VMs entre elles et au réseau physique.

```
Réseau physique (LAN entreprise)
          |
     [Interface physique : eth0]
          |
        vmbr0  ← Bridge principal
       /     \
    VM1       VM2    ← Toutes ont accès au LAN
```

**`vmbr0`** est le bridge par défaut dans Proxmox. Il est relié à l'interface physique de l'hyperviseur et donne aux VMs un accès au réseau local et à Internet.

### Modes réseau

| Mode | Description | Usage |
|---|---|---|
| **Bridge (vmbr0)** | VM accessible sur le réseau local | Serveurs de production |
| **NAT** | VM derrière un NAT, non accessible depuis l'extérieur | Labs, isolation |
| **Host-only** | Communication uniquement avec l'hôte | Tests internes |
| **Aucun** | VM sans réseau | Isolation complète |

---

## 4. VLANs dans un environnement virtualisé

Un **VLAN** (Virtual LAN) segmente logiquement le réseau sans infrastructure physique supplémentaire.

**Exemple de segmentation :**
```
vmbr0 (bridge physique)
  ├── VLAN 10 → VM Serveur Web    (DMZ)
  ├── VLAN 20 → VM Active Directory (Interne)
  └── VLAN 30 → VM Tests          (Isolé)
```

**Principe :** Les trames sont **taguées 802.1Q** avec un VLAN ID. Deux VMs sur des VLANs différents ne peuvent pas communiquer directement, même sur le même hyperviseur.

**Prérequis :** Le switch physique connecté à l'hyperviseur doit être configuré en **mode trunk** pour laisser passer les trames taguées.

---

## 5. Gestion des ressources

### Diagnostiquer une VM trop gourmande en RAM

**Depuis l'interface Proxmox :**
- Onglet "Summary" → graphiques temps réel CPU/RAM

**Depuis la VM (Linux) :**
```bash
free -h                          # Vue générale de la mémoire
top / htop                       # Processus en temps réel
ps aux --sort=-%mem | head -10   # Top 10 processus par RAM
```

**Actions correctives :**
1. Identifier et redémarrer le processus fautif
2. Augmenter la RAM allouée via l'interface Proxmox
3. Activer le **ballooning** pour une gestion dynamique

### Memory Ballooning
Mécanisme permettant à l'hyperviseur de **récupérer dynamiquement la RAM inutilisée** dans les VMs pour la redistribuer.

```
Hyperviseur
  ├── VM1 (allouée : 4 Go | utilisée : 2 Go)  → ballon gonfle → libère 2 Go
  └── VM2 (allouée : 4 Go | manque de RAM)    → reçoit les 2 Go libérés
```

Nécessite l'installation du **driver balloon** dans la VM (inclus dans `qemu-guest-agent`).

---

## 6. Migration des VMs

### Migration à chaud (Live Migration)
Déplacement d'une VM **en cours de fonctionnement** vers un autre nœud, **sans interruption de service**.

**Conditions requises :**
- Stockage **partagé** entre les nœuds (NFS, SAN, Ceph) — la VM ne doit pas être sur un disque local
- Nœuds dans le **même cluster Proxmox**
- **Architecture CPU compatible** entre les deux nœuds
- Réseau dédié à la migration (bande passante suffisante)

### Migration à froid
Déplacement d'une VM **éteinte ou suspendue**. Plus simple mais entraîne une interruption de service.

| | À chaud | À froid |
|---|---|---|
| **État de la VM** | En fonctionnement | Éteinte |
| **Interruption** | Nulle (quelques ms) | Oui |
| **Complexité** | Élevée | Simple |
| **Stockage requis** | Partagé obligatoire | Local possible |

---

## 7. Surveillance et diagnostic

### Outils Proxmox intégrés
- **Graphiques de performance** → CPU, RAM, réseau, disque par VM
- **Journaux (logs)** → Historique des actions et erreurs
- **Tâches** → Suivi des opérations en cours (migrations, sauvegardes...)

### Commandes utiles depuis la VM (Linux)
```bash
# Ressources système
top / htop                    # CPU et RAM en temps réel
df -h                         # Espace disque
iostat -x 1                   # I/O disque

# Réseau
ip a                          # Interfaces réseau
ss -tulnp                     # Ports en écoute
ping <IP>                     # Test connectivité

# Logs
journalctl -xe                # Logs système récents
tail -f /var/log/syslog       # Logs en temps réel
```

---

## 8. Bonnes pratiques d'administration

| Pratique | Pourquoi |
|---|---|
| Nommer les VMs de façon cohérente (srv-web-01) | Lisibilité et organisation |
| Utiliser des templates standardisés | Uniformité et gain de temps |
| Installer `qemu-guest-agent` sur chaque VM | Snapshots cohérents, stats dans Proxmox |
| Attribuer les ressources avec discernement | Éviter la surcharge de l'hôte |
| Documenter les VMs et leur rôle | Maintenance facilitée |
| Ne jamais modifier une VM de prod sans snapshot préalable | Retour arrière possible |

---
*CP5 — Fiche 2/5 — Administration d'une infrastructure Proxmox*
