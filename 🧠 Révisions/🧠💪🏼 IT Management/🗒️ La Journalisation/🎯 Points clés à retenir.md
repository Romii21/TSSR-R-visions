### Ce qu'il faut absolument savoir

|Thème|À retenir|
|---|---|
|**Définition**|Un log = trace de l'activité d'un système, application ou réseau|
|**Utilité**|Debug, administration, cybersécurité|
|**Syslog Linux**|Protocole standard, RFC 5424, port 514/UDP ou 6514/TCP+TLS|
|**Gravité syslog**|8 niveaux : 0 (Emergency) → 7 (Debug)|
|**Catégories syslog**|24 catégories (kern, auth, daemon, cron...)|
|**Stockage Linux**|`/var/log/` via rsyslog, format texte|
|**Rotation Linux**|`logrotate` + `cron` pour archiver et supprimer les vieux logs|
|**Systemd**|`journalctl` pour consulter les logs en format binaire|
|**Lire les logs Linux**|`tail -f`, `grep`, `journalctl -f`|
|**Windows Event Viewer**|Logs dans `C:\Windows\System32\winevt\Logs`|
|**Accès Windows**|`eventvwr` depuis Exécuter (Win+R)|
|**Centralisation Windows**|WEF via WinRM (port 5985/5986)|
|**Event IDs critiques**|4625 (échec connexion), 1102 (suppression logs), 4740 (compte bloqué)|
|**Obligations légales**|RGPD art. 32, LPM — respecter la durée de conservation|
|**Analyse avancée**|SIEM, HIDS, SOC pour détecter les menaces automatiquement|

### 🔑 Règles d'or

1. **Centraliser** les logs pour ne pas les perdre en cas d'incident
2. **Protéger** les logs (une suppression de logs = Event ID 1102 = alerte !)
3. **Chiffrer** les logs en transit (TLS/HTTPS)
4. **Archiver** régulièrement pour économiser l'espace disque
5. **Conserver** selon les obligations légales en vigueur

---
