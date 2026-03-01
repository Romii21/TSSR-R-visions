### Concepts fondamentaux

|Concept|À retenir|
|---|---|
|**VPN**|Simule un réseau privé sur infrastructure publique via tunnel chiffré|
|**Tunnel**|Encapsulation + chiffrement + authentification des données|
|**Tous les tunnels ≠ VPN**|L2TP, GRE, SSH = tunnels mais pas VPN sécurisés complets|

### Choix de solution

|Solution|Niveau|Usage principal|Points clés|
|---|---|---|---|
|**IPsec**|3 (IP)|VPN site-à-site, matériel réseau|Transparent, mode tunnel recommandé, ESP > AH|
|**TLS**|5-6|Sécurisation applicative|TLS 1.3 obligatoire, handshake rapide|
|**OpenVPN**|2 ou 3|VPN accès distant, multi-plateforme|Basé TLS, mode routeur (tun) recommandé|

### Bonnes pratiques sécurité

✅ **À faire** :

- TLS 1.3 (ou minimum TLS 1.2)
- Authentification par certificats X.509 + MFA
- Mode tunnel IPsec pour VPN site-à-site
- Mise en place d'un bastion pour l'administration
- Surveillance et logs des connexions VPN
- Révocation de certificats compromis (CRL/OCSP)

❌ **À éviter** :

- SSL/TLS 1.0/1.1 (obsolètes et vulnérables)
- Authentification par simple mot de passe
- Confiance aveugle dans le VPN (toujours segmenter)
- Clés partagées non protégées

### Commandes et outils utiles

**Pour IPsec** :

```bash
# Vérifier les SA actives (Linux)
ip xfrm state
ip xfrm policy
```

**Pour TLS** :

```bash
# Tester une connexion TLS
openssl s_client -connect example.com:443 -tls1_3

# Vérifier un certificat
openssl x509 -in certificat.crt -text -noout
```

**Pour OpenVPN** :

```bash
# Démarrer un serveur OpenVPN
openvpn --config serveur.conf

# Se connecter en tant que client
openvpn --config client.ovpn

# Générer une PKI avec Easy-RSA
easyrsa init-pki
easyrsa build-ca
easyrsa gen-req serveur nopass
easyrsa sign-req server serveur
```

### Tableau récapitulatif des protocoles

|Protocole|Couche OSI|Port|Chiffrement|Authentification|Usage|
|---|---|---|---|---|---|
|**IPsec (ESP)**|3|IP proto 50|✅|✅|VPN site-à-site|
|**IPsec (AH)**|3|IP proto 51|❌|✅|Intégrité seule (rare)|
|**TLS 1.3**|5-6|Variable|✅|✅|Applications web/email|
|**OpenVPN**|2 ou 3|UDP/TCP 1194|✅ (TLS)|✅|VPN accès distant|

---
