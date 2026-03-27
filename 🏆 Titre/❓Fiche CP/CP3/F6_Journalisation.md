# 📘 Fiche 6 — Journalisation
### Formation TSSR | CP3 — Exploiter des serveurs Linux

---

## 1. Pourquoi les logs ?

Les logs (journaux) sont les **traces de l'activité** d'un système, d'un service ou d'une application.

| Qui ? | Usage |
|-------|-------|
| **Admin système** | Diagnostiquer les pannes, analyser les comportements |
| **Cybersécurité** | Détecter des intrusions, retracer des attaques |
| **Développeur** | Déboguer une application |
| **Conformité légale** | RGPD, LPM — conservation obligatoire des traces |

---

## 2. Les niveaux de gravité Syslog (à connaître par cœur)

| Code | Niveau | Mot-clé | Description |
|------|--------|---------|-------------|
| **0** | Emergency | `emerg` | 🔴 Système inutilisable |
| **1** | Alert | `alert` | 🔴 Intervention immédiate nécessaire |
| **2** | Critical | `crit` | 🟠 Erreur critique système |
| **3** | Error | `err` | 🟠 Erreur de fonctionnement |
| **4** | Warning | `warn` | 🟡 Avertissement |
| **5** | Notice | `notice` | 🔵 Événement normal notable |
| **6** | Informational | `info` | ⚪ Information |
| **7** | Debug | `debug` | ⚪ Débogage |

> 💡 Moyen mnémotechnique : **E**very **A**dmin **C**an **E**asily **W**atch **N**ew **I**nteresting **D**ata
> → **E**merg / **A**lert / **C**rit / **E**rr / **W**arn / **N**otice / **I**nfo / **D**ebug

> En production : on configure généralement à partir du niveau **Warning (4)** pour ne pas générer trop de logs.

---

## 3. Fichiers de logs principaux dans /var/log/

| Fichier | Contenu |
|---------|---------|
| `/var/log/auth.log` | Authentifications, SSH, sudo, su — **le plus important en sécurité** |
| `/var/log/syslog` | Messages système généraux (tous services confondus) |
| `/var/log/kern.log` | Messages du noyau Linux |
| `/var/log/dpkg.log` | Installations/suppressions de paquets APT |
| `/var/log/nginx/access.log` | Toutes les requêtes HTTP reçues par nginx |
| `/var/log/nginx/error.log` | Erreurs nginx |
| `/var/log/mail.log` | Serveur de messagerie |
| `/var/log/faillog` | Échecs de connexion |

---

## 4. rsyslog vs journal systemd

| | rsyslog | Journal systemd |
|--|---------|----------------|
| **Type de stockage** | Texte clair dans `/var/log/` | Binaire dans `/run/systemd/journal/` |
| **Outil de lecture** | `cat`, `grep`, `tail`… | `journalctl` |
| **Envoi distant** | ✅ Natif (syslog) | ❌ Non natif |
| **Indexé/filtrable** | ❌ Recherche textuelle | ✅ Très filtrable |
| **Sur distros modernes** | Les deux coexistent | journald capte en premier |

> Sur Ubuntu/Debian modernes : **journald** capture les logs en premier, **rsyslog** peut les relayer vers `/var/log/` et vers un serveur distant.

---

## 5. Lire les logs avec journalctl

```bash
journalctl                           # Tous les logs (depuis le début)
journalctl -f                        # Suivre en temps réel (comme tail -f)
journalctl -b                        # Logs depuis le dernier démarrage
journalctl -b -1                     # Logs du démarrage précédent

# Filtrer par service
journalctl -u nginx                  # Logs du service nginx
journalctl -u sshd -f               # Logs SSH en temps réel

# Filtrer par niveau
journalctl -p err                    # Erreurs et plus grave
journalctl -p 3                      # Équivalent (code numérique)

# Filtrer par date
journalctl --since "2024-01-01"
journalctl --since "1 hour ago"
journalctl --since "2024-01-01" --until "2024-01-31"

# Chercher dans les logs
journalctl -u sshd | grep "Failed"   # Filtrer avec grep
```

---

## 6. Lire les logs classiques

```bash
# Lecture simple
cat /var/log/syslog                  # Tout afficher (peu pratique si gros)
less /var/log/syslog                 # Page par page (q pour quitter)
tail -20 /var/log/auth.log           # 20 dernières lignes
tail -f /var/log/auth.log            # Suivre en temps réel

# Recherche
grep "Failed" /var/log/auth.log      # Chercher les échecs SSH
grep "error" /var/log/syslog         # Chercher les erreurs

# Logs compressés (fichiers archivés)
zcat /var/log/syslog.1.gz            # Lire un log compressé
zgrep "error" /var/log/syslog.2.gz   # Chercher dans un log compressé

# Connexions
last                                 # Historique des connexions réussies
lastb                                # Historique des connexions ÉCHOUÉES
```

---

## 7. logrotate — Rotation des logs

**Problème** : les logs grossissent sans arrêt → risque de saturation du disque.

**Solution** : **logrotate** archive, compresse et supprime les anciens logs automatiquement.

```
Chaque jour :
/var/log/syslog (actuel)
    ↓ rotation
/var/log/syslog.1             ← déplacé
/var/log/syslog.2.gz          ← compressé
...
/var/log/syslog.7.gz          ← supprimé si > 7 jours
```

```bash
# Configuration
cat /etc/logrotate.conf              # Config globale
ls /etc/logrotate.d/                 # Config par service (nginx, apt, etc.)

# Forcer une rotation manuellement
logrotate -f /etc/logrotate.conf

# Déclenchement automatique
# → Via cron : /etc/cron.daily/logrotate (chaque nuit)
```

Exemple de configuration `/etc/logrotate.d/nginx` :
```
/var/log/nginx/*.log {
    daily                  # Rotation quotidienne
    missingok              # Pas d'erreur si le fichier manque
    rotate 14              # Garder 14 archives
    compress               # Compresser les anciennes
    postrotate
        systemctl reload nginx
    endscript
}
```

---

## 8. Centralisation des logs

**Pourquoi centraliser sur un serveur distant ?**

1. Si la machine est compromise/éteinte, les **logs sont préservés** ailleurs
2. Un attaquant peut **effacer les logs locaux** — pas les logs distants
3. **Corrélation multi-machines** pour détecter des patterns (SIEM)
4. **Obligations légales** : RGPD, LPM imposent la conservation

### Méthode : rsyslog vers serveur distant

```bash
# Sur le CLIENT — ajouter dans /etc/rsyslog.conf
*.* @@syslog.exemple.com:514    # @@ = TCP (recommandé), @ = UDP
# Avec TLS (recommandé en production) : port 6514

# Redémarrer
systemctl restart rsyslog
```

> ✅ Toujours préférer **TCP** à UDP (garantie de livraison)
> ✅ Utiliser **TLS** (port 6514) pour chiffrer les logs en transit

---

## 9. Points clés à retenir

```
✅ 8 niveaux syslog  →  0 (emerg) à 7 (debug), du + au - critique
✅ /var/log/auth.log →  SSH, sudo, connexions — le + important sécurité
✅ tail -f           →  suivre en temps réel
✅ journalctl -f     →  idem pour systemd
✅ journalctl -u     →  logs d'un service spécifique
✅ lastb             →  échecs de connexion
✅ logrotate         →  rotation auto via cron (évite saturation disque)
✅ rsyslog           →  texte dans /var/log/ + envoi distant possible
✅ journald          →  binaire, très filtrable, systemd
✅ Centraliser logs  →  sécurité + continuité + conformité légale
```
