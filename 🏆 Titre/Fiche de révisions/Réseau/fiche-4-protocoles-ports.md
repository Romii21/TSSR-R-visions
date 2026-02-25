# 🔌 Fiche 4 — Protocoles & Ports Essentiels

> *"Les ports, c'est comme les numéros de bureau dans un immeuble — l'IP t'amène au bâtiment, le port t'indique à quelle porte frapper."*

Connaître les protocoles, leurs ports et leurs couches est indispensable pour diagnostiquer rapidement une panne ou configurer un firewall.

---

## Protocoles incontournables

| Protocole | Couche OSI | Port                       | Transport                     | Rôle                         |
| --------- | ---------- | -------------------------- | ----------------------------- | ---------------------------- |
| **HTTP**  | 7          | 80                         | TCP                           | Web non chiffré              |
| **HTTPS** | 7          | 443                        | TCP                           | Web chiffré (via TLS)        |
| **DNS**   | 7          | 53                         | **UDP** (TCP si > 512 octets) | Résolution de noms           |
| **DHCP**  | 7          | 67 (serveur) / 68 (client) | UDP                           | Attribution d'IP automatique |
| **SSH**   | 7          | 22                         | TCP                           | Accès distant sécurisé       |
| **FTP**   | 7          | 21                         | TCP                           | Transfert de fichiers        |
| **SMTP**  | 7          | 25                         | TCP                           | Envoi d'e-mails              |
| **TLS**   | 6          | —                          | —                             | Chiffrement des données      |

---

## 🔁 TCP vs UDP — quand utiliser quoi ?

**TCP** — *"Je m'assure que tout est bien arrivé"*
- Connexion établie (handshake en 3 étapes)
- Accusés de réception, retransmission si perte
- Utilisé quand la fiabilité prime : HTTP, SSH, FTP

**UDP** — *"J'envoie et j'oublie"*
- Pas de connexion, pas d'accusé de réception
- Plus rapide, moins de surcharge
- Utilisé quand la vitesse prime : DNS, streaming, jeux en ligne

---

## 💡 Astuces mémo
- **DNS utilise UDP** par défaut — pas TCP — c'est une source de confusion fréquente
- **DHCP utilise le broadcast** pour trouver un serveur (l'IP n'est pas encore connue)
- Un port **< 1024** est un port système (réservé aux services connus)
- Les ports **1024 – 65535** sont utilisés dynamiquement par les clients
