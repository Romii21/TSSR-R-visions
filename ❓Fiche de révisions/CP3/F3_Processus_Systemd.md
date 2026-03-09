# 📘 Fiche 3 — Processus & Systemd
### Formation TSSR | CP3 — Exploiter des serveurs Linux

---

## 1. Programme vs Processus

| | Programme | Processus |
|--|-----------|-----------|
| **Définition** | Fichier exécutable **statique** sur le disque | Instance **en cours d'exécution** d'un programme |
| **Localisation** | `/usr/bin/nginx`, `script.sh` | En mémoire RAM, avec un PID |
| **État** | Inerte (du code) | Vivant (CPU, mémoire allouée) |
| **Multiplicité** | Un seul fichier | Plusieurs instances possibles simultanément |

---

## 2. Métadonnées d'un processus

Chaque processus possède :

| Donnée | Signification |
|--------|---------------|
| **PID** | Process ID — identifiant unique |
| **PPID** | Parent PID — identifiant du processus parent |
| **UID / GID** | Utilisateur et groupe propriétaire |
| **État** | Running (R), Sleeping (S), Stopped (T), Zombie (Z) |
| **Priorité** | Ordre de passage sur le CPU |
| **TTY** | Terminal de contrôle |

Tout est accessible dans **`/proc/[PID]/`** :
```
/proc/1234/
├── cmdline     ← Commande de lancement
├── status      ← État (nom, PID, PPID, état, mémoire)
├── fd/         ← Fichiers ouverts
└── environ     ← Variables d'environnement
```

---

## 3. Hiérarchie des processus

```
systemd (PID 1)      ← Premier processus lancé par le noyau
├── sshd             ← Serveur SSH
│   └── bash         ← Session SSH ouverte
│       └── vim      ← Éditeur lancé dans la session
├── nginx            ← Serveur web
├── cron             ← Planificateur de tâches
└── rsyslogd         ← Service de logs
```

> **PID 1 = systemd** sur les distributions modernes. Il démarre tous les autres services et ne s'arrête jamais.

---

## 4. Commandes de gestion des processus

### Consultation

```bash
ps aux                          # Liste tous les processus (snapshot)
ps aux --sort=-%cpu             # Triés par consommation CPU
ps -ef                          # Format étendu avec PPID
pstree                          # Vue arborescente
top                             # Vue dynamique (natif)
htop                            # Vue dynamique améliorée (interactif)
```

### Gestion des signaux

```bash
kill PID                        # Envoyer SIGTERM (arrêt gracieux) — par défaut
kill -15 PID                    # SIGTERM explicite
kill -9 PID                     # SIGKILL (force brutale — dernier recours)
killall nginx                   # Tuer tous les processus nommés "nginx"
pkill -f "mon_script"           # Tuer par nom de commande
```

### Signaux principaux

| Signal | Numéro | Effet | Interceptable ? |
|--------|--------|-------|----------------|
| **SIGTERM** | 15 | Arrêt gracieux (le processus peut nettoyer) | ✅ Oui |
| **SIGKILL** | 9 | Arrêt immédiat forcé par le noyau | ❌ Non |
| **SIGINT** | 2 | Interruption (= Ctrl+C) | ✅ Oui |
| **SIGTSTP** | 20 | Suspension (= Ctrl+Z) | ✅ Oui |
| **SIGHUP** | 1 | Rechargement de configuration | ✅ Oui |

> ⚠️ Toujours essayer **SIGTERM** en premier. SIGKILL ne laisse pas le temps de sauvegarder/nettoyer → risque de corruption.

---

## 5. Premier plan / Arrière-plan

```bash
./script.sh &                          # Lancer en arrière-plan
jobs                                   # Lister les jobs de la session
fg                                     # Ramener le dernier job au premier plan
fg %2                                  # Ramener le job n°2
bg                                     # Reprendre un job suspendu en arrière-plan

# Ctrl+Z → Suspendre le processus courant (SIGTSTP)
# Ctrl+C → Interrompre (SIGINT)
```

### Continuer après fermeture du terminal SSH

```bash
nohup ./script.sh > output.log 2>&1 &  # Ignore le signal HUP à la déconnexion
screen                                  # Multiplexeur — session persistante
tmux                                    # Multiplexeur moderne (recommandé)
disown %1                              # Détacher un job déjà lancé
```

---

## 6. Systemd — Gestion des services

### Commandes systemctl

```bash
systemctl start nginx          # Démarrer maintenant
systemctl stop nginx           # Arrêter maintenant
systemctl restart nginx        # Redémarrer (stop + start)
systemctl reload nginx         # Recharger la config SANS redémarrer
systemctl status nginx         # Voir l'état (actif, logs récents)
systemctl enable nginx         # Activer au démarrage (prochain boot)
systemctl disable nginx        # Désactiver du démarrage
systemctl enable --now nginx   # Activer ET démarrer immédiatement
systemctl is-active nginx      # Retourne "active" ou "inactive"
systemctl list-units --type=service   # Lister tous les services
```

### enable vs start : différence cruciale

```
systemctl start  → démarre MAINTENANT, ne change rien au démarrage
systemctl enable → active au PROCHAIN BOOT, ne démarre pas maintenant

Pour les deux : systemctl enable --now
```

### Fichiers Unit

| Emplacement | Usage |
|-------------|-------|
| `/lib/systemd/system/` | Fichiers fournis par les **paquets** (ne pas modifier) |
| `/etc/systemd/system/` | Fichiers **personnalisés** par l'admin — prioritaires |

Structure d'un fichier unit (`/etc/systemd/system/monservice.service`) :
```ini
[Unit]
Description=Mon service personnalisé
After=network.target

[Service]
ExecStart=/usr/bin/python3 /opt/monapp/app.py
Restart=always
User=www-data

[Install]
WantedBy=multi-user.target
```

Après modification : `systemctl daemon-reload`

### Targets (remplacent les runlevels)

| Target | Runlevel équivalent | Description |
|--------|-------------------|-------------|
| `poweroff.target` | 0 | Extinction |
| `rescue.target` | 1 | Mode maintenance (root seul) |
| `multi-user.target` | 3 | Multi-utilisateurs, sans GUI **(serveur)** |
| `graphical.target` | 5 | Multi-utilisateurs + interface graphique |
| `reboot.target` | 6 | Redémarrage |

```bash
systemctl get-default                         # Voir la target par défaut
systemctl set-default multi-user.target       # Définir le mode par défaut
```

---

## 7. Démons (daemons)

Un **démon** est un processus qui tourne en **arrière-plan en permanence**, sans interaction directe avec l'utilisateur. Convention : le nom finit souvent par `d`.

| Démon | Rôle |
|-------|------|
| `sshd` | Serveur SSH (accès distant) |
| `nginx` / `apache2` | Serveur web |
| `crond` | Planificateur de tâches |
| `rsyslogd` | Journalisation système |
| `networkd` | Gestion réseau |
| `mysqld` | Base de données MySQL |

---

## 8. Points clés à retenir

```
✅ PID 1          →  systemd (premier processus, parent de tout)
✅ /proc/[PID]/   →  infos sur un processus
✅ SIGTERM (15)   →  arrêt gracieux — toujours essayer en premier
✅ SIGKILL (9)    →  force brutale — dernier recours seulement
✅ &              →  lancer en arrière-plan
✅ nohup          →  survivre à la fermeture du terminal SSH
✅ jobs / fg / bg →  gérer les jobs de la session
✅ start ≠ enable →  maintenant ≠ au démarrage
✅ enable --now   →  les deux en une commande
✅ /etc/systemd/  →  mes fichiers unit (prioritaires sur /lib/systemd/)
✅ daemon-reload  →  après modification d'un fichier unit
```
