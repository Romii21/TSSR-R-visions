# FICHE 1 — Fondamentaux de la Virtualisation
### Formation TSSR — CP5 : Maintenir des serveurs dans une infrastructure virtualisée

---

## 1. Définition et principe

La **virtualisation** consiste à faire fonctionner plusieurs systèmes d'exploitation (OS) sur une seule machine physique grâce à un logiciel appelé **hyperviseur**. Chaque OS tourne dans une **machine virtuelle (VM)** isolée qui emprunte les ressources matérielles de la machine hôte (CPU, RAM, stockage).

```
┌─────────────────────────────────────────────┐
│              Machine Physique               │
│                                             │
│  ┌──────────────────────────────────────┐   │
│  │           Hyperviseur                │   │
│  │  ┌────────┐  ┌────────┐  ┌────────┐  │   │
│  │  │  VM 1  │  │  VM 2  │  │  VM 3  │  │   │
│  │  │ Linux  │  │Windows │  │ Debian │  │   │
│  │  └────────┘  └────────┘  └────────┘  │   │
│  └──────────────────────────────────────┘   │
│         CPU     RAM     Disques             │
└─────────────────────────────────────────────┘
```

---

## 2. Hyperviseur Type 1 vs Type 2

| Critère | Type 1 — Bare Metal | Type 2 — Hosted |
|---|---|---|
| **Installation** | Directement sur le matériel | Sur un OS existant |
| **Performance** | Excellente (accès direct) | Réduite (overhead de l'OS hôte) |
| **Usage** | Serveurs de production | Tests, développement, formation |
| **Exemples** | Proxmox VE, VMware ESXi, Hyper-V, KVM | VirtualBox, VMware Workstation |

```
TYPE 1                          TYPE 2
┌──────────────┐                ┌──────────────┐
│ Hyperviseur  │                │     OS Hôte  │
│  (direct)    │                │  ┌─────────┐ │
│  ┌────┐┌────┐│                │  │Hyperv.  │ │
│  │VM1 ││VM2 ││                │  │┌────┐   │ │
│  └────┘└────┘│                │  ││VM1 │   │ │
├──────────────┤                │  │└────┘   │ │
│   Matériel   │                │  └─────────┘ │
└──────────────┘                ├──────────────┤
                                │   Matériel   │
                                └──────────────┘
```

---

## 3. Proxmox VE — L'hyperviseur de la formation

**Proxmox VE** est une plateforme open-source de Type 1 qui combine deux technologies :

| Technologie | Rôle | Type d'objet créé |
|---|---|---|
| **KVM** (Kernel-based Virtual Machine) | Virtualisation complète | Machines Virtuelles (VM) |
| **LXC** (Linux Containers) | Conteneurisation légère | Conteneurs |

**Caractéristiques de Proxmox :**
- Interface web intuitive (port 8006)
- Gratuit et open-source
- Support du clustering et de la haute disponibilité (HA)
- Gestion avancée du stockage (LVM, NFS, Ceph, iSCSI)
- Migration à chaud des VMs entre nœuds

---

## 4. Concepts essentiels à maîtriser

### L'Overhead
Ressources consommées par **l'hyperviseur lui-même** pour fonctionner, et donc non disponibles pour les VMs.

> Exemple : 32 Go RAM physique → Proxmox en consomme ~2 Go → 30 Go disponibles pour les VMs.

⚠️ Ne pas confondre avec **l'overcommit** (surallocation) : allouer en théorie plus de ressources que la machine n'en a physiquement.

### Le SPOF (Single Point of Failure)
Risque principal d'un seul serveur physique : **si la machine tombe, toutes les VMs s'arrêtent**.
Solution : clustering, haute disponibilité, redondance matérielle.

### Snapshot vs Sauvegarde complète

| | Snapshot | Sauvegarde complète |
|---|---|---|
| **Définition** | Photo de l'état à un instant T | Copie intégrale des données |
| **Espace disque** | Faible (stocke les différences) | Important |
| **Création** | Quasi-instantanée | Longue |
| **Indépendance** | Non (lié au disque source) | Oui (stockable ailleurs) |
| **Usage** | Avant une MAJ ou un test | Sauvegarde de production régulière |

> ⚠️ **Un snapshot n'est PAS une sauvegarde.** Si le disque physique tombe, le snapshot est perdu avec lui.

---

## 5. Formats de disques virtuels — À mémoriser absolument

| Format | Hyperviseur associé | Particularités |
|---|---|---|
| **QCOW2** | KVM / Proxmox / QEMU | Natif Proxmox, supporte snapshots, compression, thin provisioning |
| **VMDK** | VMware (ESXi, Workstation) | Format propriétaire VMware |
| **VHD / VHDX** | Microsoft Hyper-V | VHDX = version améliorée (plus de 2 To, journalisation) |
| **VDI** | Oracle VirtualBox | Format natif VirtualBox |

---

## 6. KVM vs LXC — Tableau comparatif

| Critère | VM KVM | Conteneur LXC |
|---|---|---|
| **Kernel** | Propre kernel virtuel | Partagé avec l'hôte |
| **Isolation** | Forte (matérielle) | Légère (logicielle - namespaces) |
| **Performance** | Très bonne | Quasi-native |
| **Mémoire utilisée** | Plus importante | Très faible |
| **Démarrage** | 30s à plusieurs minutes | Quelques secondes |
| **OS supportés** | Tous (Windows, Linux, BSD...) | Linux uniquement |
| **Sécurité** | Isolation maximale | Isolation plus faible |

**Quand utiliser KVM :** OS non-Linux, sécurité maximale, kernel spécifique requis.
**Quand utiliser LXC :** Services Linux légers, performances critiques, nombreux environnements identiques.

---

## 7. Avantages et inconvénients de la virtualisation

### ✅ Avantages
1. **Consolidation** → Moins de serveurs physiques, réduction des coûts et de la consommation électrique
2. **Isolation des services** → Une VM en panne n'impacte pas les autres
3. **Déploiement rapide** → Clonage, templates, création en minutes
4. **Flexibilité** → Snapshots, migration, redimensionnement des ressources
5. **Haute disponibilité** → Clustering, migration à chaud, HA automatisée

### ⚠️ Inconvénients
1. **SPOF** → Un seul serveur physique = point de défaillance unique
2. **Matériel puissant requis** → CPU multicœurs, grande RAM, stockage rapide
3. **Overhead** → Légère pénalité de performances (~1-5% sur matériel moderne)
4. **Complexité** → Diagnostics plus difficiles (OS invité ? Hyperviseur ? Matériel ?)
5. **Inadaptation** → Certaines applications nécessitant un accès matériel direct ne peuvent pas être virtualisées (GPU dédié, cartes réseau haute vitesse)

---

## 8. Mots-clés à retenir

| Terme | Définition courte |
|---|---|
| **Hyperviseur** | Logiciel qui crée et gère les VMs |
| **OS Hôte (Host)** | Système de la machine physique |
| **OS Invité (Guest)** | Système d'une VM |
| **Overhead** | Ressources consommées par l'hyperviseur lui-même |
| **Overcommit** | Allouer plus de ressources que disponibles physiquement |
| **SPOF** | Single Point of Failure — défaillance unique qui arrête tout |
| **Snapshot** | Photo instantanée de l'état d'une VM |
| **Template** | VM convertie en modèle non démarrable pour clonage rapide |
| **Migration à chaud** | Déplacement d'une VM sans interruption de service |
| **HA** | High Availability — redémarrage automatique en cas de panne |

---
*CP5 — Fiche 1/5 — Fondamentaux de la virtualisation*
