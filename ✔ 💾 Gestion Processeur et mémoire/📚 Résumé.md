
```
╔════════════════════════════════════════════════════════════╗
║         GESTION DES PROCESSUS ET DE LA MÉMOIRE             ║
╚════════════════════════════════════════════════════════════╝

┌──────────────────────────────────────────────────────────┐
│  MATÉRIEL (Processeur + RAM)                             │
└──────────────────────────────────────────────────────────┘
                           ↓
┌──────────────────────────────────────────────────────────┐
│  SYSTÈME D'EXPLOITATION                                  │
├──────────────────────────────────────────────────────────┤
│  ┌─────────────────────────────────────────────────────┐ │
│  │ ORDONNANCEUR (Décide qui s'exécute)                │ │
│  │ • Politique : FIFO, Tourniquet, Priorités, CFS... │ │
│  └─────────────────────────────────────────────────────┘ │
│                        ↓                                  │
│  ┌─────────────────────────────────────────────────────┐ │
│  │ GESTIONNAIRE MÉMOIRE                               │ │
│  │ • Mémoire virtuelle (pages)                        │ │
│  │ • Swap (débordement disque)                        │ │
│  │ • Protection (droits d'accès)                      │ │
│  └─────────────────────────────────────────────────────┘ │
│                        ↓                                  │
│  ┌─────────────────────────────────────────────────────┐ │
│  │ TABLE DES PROCESSUS                                │ │
│  │ • ProcessA (PID 1000, priorité normal)             │ │
│  │ • ProcessB (PID 1001, priorité haute)              │ │
│  │ • Thread 1, Thread 2, ...                          │ │
│  └─────────────────────────────────────────────────────┘ │
└──────────────────────────────────────────────────────────┘
                           ↓
┌──────────────────────────────────────────────────────────┐
│  APPLICATIONS (Processus)                                │
│  • Navigateur, Éditeur, Serveur, etc.                   │
└──────────────────────────────────────────────────────────┘
```

---

## 🎯 Points Clés à Retenir

### ⭐ Concepts Fondamentaux

- **Programme** = code statique sur disque
- **Processus** = programme en cours d'exécution avec état
- **Multitâche préemptif** = OS bascule rapidement entre processus

### ⭐ Mémoire

- **Mémoire virtuelle** = chaque processus a son espace isolé
- **Pagination** = découpe la mémoire en petits blocs
- **Swap** = débordement sur disque quand RAM saturée
- **MMU** = traduit adresses virtuelles → physiques

### ⭐ Processus

- **Métadonnées** : PID, PPID, UID, priorité, état
- **Communication** : signaux entre processus
- **Threads** : processus légers partageant mémoire

### ⭐ Ordonnancement

- **Objectifs** : réactivité, efficacité, équité
- **CFS (Linux)** : ordonnanceur juste et moderne
- **Interruptions** : permettent de basculer entre processus

### ⭐ Démarrage

- **UEFI + Secure Boot** : démarrage moderne et sécurisé
- **GRUB (Linux)** : bootloader flexible
- **Systemd (Linux)** : démarrage moderne et parallélisé
- **Boot Manager (Windows)** : démarrage plus simple
