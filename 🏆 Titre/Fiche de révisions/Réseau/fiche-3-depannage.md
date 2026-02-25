# 🔧 Fiche 3 — Dépannage Réseau

> *"Un bon technicien ne devine pas — il élimine. Toujours du bas vers le haut."*

La méthode OSI de dépannage consiste à vérifier les couches dans l'ordre, de la plus basse à la plus haute, pour isoler rapidement la source du problème.

---

## La méthode en 5 étapes

```
① Physique     → Câble branché ? Wi-Fi associé ? Voyant actif ?
② Liaison      → La carte réseau détecte-t-elle un lien ?
③ Réseau       → IP attribuée ? Passerelle correcte ? (ipconfig / ip a)
④ Transport    → ping passerelle → ping 8.8.8.8
⑤ Application  → ping google.com → nslookup → test du service
```

---

## 🚨 Scénarios classiques et leur diagnostic

| Symptôme | Couche coupable | Cause typique |
|----------|-----------------|---------------|
| Rien ne fonctionne | **L1** | Câble débranché, port switch mort |
| IP en 169.254.x.x | **L7** | Serveur DHCP injoignable |
| Ping passerelle OK, 8.8.8.8 KO | **L3 WAN** | Route par défaut manquante, NAT, lien opérateur |
| IP OK mais pas de nom de domaine | **L7** | Serveur DNS en panne ou mal configuré |
| Ping OK, service inaccessible | **L4 / L7** | Port bloqué (firewall) ou service arrêté |
| Accessible en local, pas depuis l'extérieur | **L4** | Service écoute sur `127.0.0.1` au lieu de `0.0.0.0` |
| Inter-VLAN inaccessible | **L3** | Routage inter-VLAN non configuré |
| SSL handshake failed | **L6** | Certificat expiré, versions TLS incompatibles |

---

## 🛠️ Boîte à outils du technicien

| Commande | Ce qu'elle teste |
|----------|-----------------|
| `ipconfig` / `ip a` | IP, masque, passerelle |
| `ping 8.8.8.8` | Connectivité IP (L3) |
| `ping google.com` | Résolution DNS (L7) |
| `nslookup` / `dig` | Serveur DNS ciblé |
| `telnet IP port` | Disponibilité d'un port (L4) |
| `tracert` / `traceroute` | Chemin réseau saut par saut |
