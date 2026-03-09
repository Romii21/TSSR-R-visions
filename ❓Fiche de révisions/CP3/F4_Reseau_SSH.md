# 📘 Fiche 4 — Réseau & SSH
### Formation TSSR | CP3 — Exploiter des serveurs Linux

---

## 1. Commandes réseau essentielles

### Afficher la configuration réseau

```bash
ip a                           # Interfaces + adresses IP (remplace ifconfig)
ip link                        # État des interfaces (up/down)
ip addr show eth0              # Infos d'une interface spécifique
ip route                       # Table de routage
ip route show default          # Voir la passerelle par défaut
ip neigh                       # Table ARP (voisins connus)
```

> ⚠️ `ifconfig` est **déprécié**. En 2025, on utilise `ip` (paquet `iproute2`).

### Tests de connectivité

```bash
ping 192.168.1.1               # Test joignabilité (ICMP — couche réseau)
ping -c 4 8.8.8.8              # 4 paquets seulement

# Tester un port spécifique
nc -zv 192.168.1.10 80         # Test TCP port 80 (netcat)
nc -zv 192.168.1.10 443        # Test HTTPS
telnet 192.168.1.10 22         # Test SSH (basique)

# Voir les ports ouverts sur la machine locale
ss -tulnp                      # Remplace netstat (moderne)
netstat -tulnp                 # Ancienne commande (net-tools)

# Options de ss :
# -t = TCP  -u = UDP  -l = listening  -n = numérique  -p = processus
```

### Table de routage

```bash
ip route
# Exemple de sortie :
# default via 192.168.1.254 dev eth0       ← passerelle par défaut
# 192.168.1.0/24 dev eth0 proto kernel     ← réseau local direct
```

`default via 192.168.1.254` = tout trafic inconnu est envoyé à **192.168.1.254** (le routeur).

---

## 2. Configuration réseau avec Netplan

Netplan est la couche de configuration réseau sur **Ubuntu 20.04+**.

- **Fichiers** : `/etc/netplan/*.yaml`
- **Format** : YAML

```yaml
# /etc/netplan/01-config.yaml
network:
  version: 2
  ethernets:
    eth0:
      dhcp4: false                        # IP statique
      addresses:
        - 192.168.1.10/24
      routes:
        - to: default
          via: 192.168.1.254              # Passerelle
      nameservers:
        addresses: [8.8.8.8, 1.1.1.1]   # DNS
```

```bash
netplan try           # Tester la configuration (retour auto si problème)
netplan apply         # Appliquer définitivement
```

---

## 3. Gestion des paquets APT

```bash
apt update                     # Mettre à jour la liste des paquets disponibles
apt upgrade                    # Installer les mises à jour
apt install nginx              # Installer un paquet
apt remove nginx               # Désinstaller (garde la config)
apt purge nginx                # Désinstaller + supprimer la config
apt search "web server"        # Rechercher un paquet par mot-clé
apt show nginx                 # Infos détaillées sur un paquet
apt autoremove                 # Supprimer les dépendances inutiles
```

### Dépôts APT

| Fichier | Rôle |
|---------|------|
| `/etc/apt/sources.list` | Dépôts principaux |
| `/etc/apt/sources.list.d/` | Dépôts supplémentaires (un fichier par dépôt) |

```bash
add-apt-repository "deb http://..."   # Ajouter un dépôt
apt update                             # Toujours après ajout
```

---

## 4. SSH — Protocole et connexion

### Caractéristiques
- **Port** : 22 (TCP)
- **Protocole** : TCP (fiable, pas UDP)
- **Chiffrement** : asymétrique + symétrique
- **Usage** : administration distante sécurisée, transfert de fichiers

### Authentification par clé publique/privée

```
┌─────────────────────────────────────────────────────┐
│                                                     │
│  1. Générer une paire de clés (côté client)         │
│     → clé PRIVÉE  : ~/.ssh/id_ed25519  (secrète)   │
│     → clé PUBLIQUE: ~/.ssh/id_ed25519.pub           │
│                                                     │
│  2. Déposer la clé publique sur le serveur          │
│     → ~/.ssh/authorized_keys  (sur le serveur)      │
│                                                     │
│  3. Connexion                                       │
│     → Serveur envoie un challenge chiffré           │
│     → Seule la clé PRIVÉE peut le déchiffrer        │
│     → Identité prouvée SANS envoyer de mot de passe │
│                                                     │
└─────────────────────────────────────────────────────┘
```

```bash
# Générer une paire de clés
ssh-keygen -t ed25519 -C "alice@entreprise.com"
# → ~/.ssh/id_ed25519       (privée — NE JAMAIS partager)
# → ~/.ssh/id_ed25519.pub   (publique — à déposer sur les serveurs)

# Copier la clé publique sur le serveur
ssh-copy-id -i ~/.ssh/id_ed25519.pub alice@192.168.1.10
# → Ajoute automatiquement dans ~/.ssh/authorized_keys sur le serveur

# Se connecter
ssh alice@192.168.1.10
ssh -p 2222 alice@192.168.1.10    # Port personnalisé
```

### Permissions obligatoires des fichiers SSH

```bash
chmod 700 ~/.ssh/                  # Dossier .ssh
chmod 600 ~/.ssh/id_ed25519        # Clé privée
chmod 644 ~/.ssh/id_ed25519.pub    # Clé publique
chmod 600 ~/.ssh/authorized_keys   # Clés autorisées
```

---

## 5. Configuration du serveur SSH

Fichier : **`/etc/ssh/sshd_config`**

```bash
# Directives de sécurité essentielles en production
PermitRootLogin no              # ← CRITIQUE : interdire connexion root directe
PasswordAuthentication no       # ← Forcer l'authentification par clé
Port 2222                       # Changer le port (sécurité par obscurité)
MaxAuthTries 3                  # Limiter les tentatives par connexion
AllowUsers alice bob thomas     # Whitelist des utilisateurs autorisés
X11Forwarding no                # Désactiver le forwarding graphique
```

```bash
# Après modification de sshd_config
systemctl restart sshd          # ou sshd selon la distro
sshd -t                         # Tester la syntaxe avant redémarrage
```

---

## 6. Transfert de fichiers via SSH

```bash
# SCP — copie simple
scp fichier.txt alice@serveur:/home/alice/          # Envoyer
scp alice@serveur:/home/alice/fichier.txt ./        # Recevoir
scp -r /dossier/ alice@serveur:/destination/        # Récursif

# RSYNC — synchronisation incrémentale (recommandé)
rsync -avz /source/ alice@serveur:/destination/
# -a = archive (récursif + droits + timestamps)
# -v = verbose
# -z = compression pendant le transfert
# --delete = supprimer côté destination ce qui n'est plus dans la source
```

> 💡 Préférer `rsync` pour les sauvegardes : il ne retransfère que les fichiers modifiés.

---

## 7. Fail2Ban — Protection brute force

**Fail2Ban** surveille les logs et bannit automatiquement les IP avec trop d'échecs d'authentification.

```
Tentatives SSH échouées dans /var/log/auth.log
                    ↓
           Fail2Ban détecte le seuil
                    ↓
     Crée une règle iptables/nftables
                    ↓
      IP bannie pendant X minutes
```

```bash
# Vérifier l'état
fail2ban-client status              # Vue générale
fail2ban-client status sshd         # Jail SSH (IPs bannies)

# Débannir une IP
fail2ban-client unban 192.168.1.100

# Configuration
/etc/fail2ban/jail.conf             # Config de base (ne pas modifier)
/etc/fail2ban/jail.local            # Surcharge personnalisée
```

Fichier de log SSH surveillé : **`/var/log/auth.log`** (Debian/Ubuntu)

---

## 8. Points clés à retenir

```
✅ ip a           →  remplace ifconfig
✅ ss -tulnp      →  ports en écoute (remplace netstat)
✅ ip route       →  table de routage
✅ default via    →  la passerelle par défaut
✅ SSH port 22    →  TCP (pas UDP)
✅ Clé privée     →  ne JAMAIS partager
✅ authorized_keys→  fichier côté serveur qui autorise les clés
✅ PermitRootLogin no  →  directive SSH critique en production
✅ scp            →  copie simple
✅ rsync -avz     →  synchronisation incrémentale (sauvegardes)
✅ Fail2Ban       →  protection brute force SSH
✅ /var/log/auth.log →  logs SSH et authentification
```
