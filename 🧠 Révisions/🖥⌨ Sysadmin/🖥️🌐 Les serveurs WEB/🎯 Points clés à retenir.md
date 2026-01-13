### Architecture du Web

|Concept|Points essentiels|
|---|---|
|**Web**|Système hypertexte basé sur HTTP, URL et HTML|
|**HTTP**|Protocole client-serveur, sans état, port 80|
|**HTTPS**|HTTP sécurisé par TLS, port 443, indispensable|
|**URL**|Identifiant de ressource : `schéma://autorité/chemin?requête#fragment`|

### Ports standards à retenir

|Service|Port|Protocole|
|---|---|---|
|HTTP|**80**|TCP|
|HTTPS|**443**|TCP/TLS|
|Apache backend|8080|TCP|

### Serveurs Web

|Type|Rôle|Exemples|
|---|---|---|
|**Serveur HTTP**|Servir des fichiers statiques|Apache, Nginx|
|**Backend**|Générer du contenu dynamique|PHP, Node.js, Django|
|**Proxy**|Intermédiaire client-serveur|Squid|
|**Reverse Proxy**|Frontal devant serveurs backend|Nginx, HAProxy|

### Sécurité

- **HTTPS est obligatoire** pour tout site web moderne
- Utiliser des certificats d'autorités publiques (Let's Encrypt)
- Activer HSTS, CSP pour renforcer la sécurité
- Les proxys nécessitent de casser TLS pour inspecter HTTPS

### Virtualisation

- Un serveur peut héberger **plusieurs sites**
- Différenciation par **nom d'hôte** (le plus courant)
- Virtual Host (Apache) / Server Block (Nginx)

### Reverse Proxy - Cas d'usage

```
Architecture optimale :
Nginx (frontal) ──┬──> Fichiers statiques
                  └──> Apache (backend dynamique)
                       
Avantages :
✓ Performances maximales
✓ Répartition de charge
✓ Sécurité renforcée
✓ Cache efficace
```

### Messages HTTP

**Requête** = Méthode + URL + Entêtes + Corps  
**Réponse** = Code statut + Entêtes + Corps

### Commandes essentielles

```bash
# Apache
sudo systemctl start|stop|restart apache2
sudo apache2ctl configtest

# Nginx  
sudo systemctl start|stop|restart nginx
sudo nginx -t
sudo nginx -s reload

# Test HTTP
curl -I https://example.com
wget https://example.com/file.pdf
```

---
