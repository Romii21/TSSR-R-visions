# 🧠 Fiche 1 — Le Modèle OSI

> *"Comprendre OSI, c'est comprendre comment deux machines se parlent sans se connaître."*

Le modèle OSI découpe la communication réseau en **7 couches indépendantes**. Chaque couche a un rôle précis et ne communique qu'avec ses voisines directes. C'est la base de tout diagnostic réseau.

---

## Les 7 couches

| # | Nom | Rôle | Exemples |
|---|-----|------|----------|
| 7 | **Application** | Interface avec l'utilisateur | HTTP, DNS, FTP, DHCP |
| 6 | **Présentation** | Chiffrement, encodage, formatage | TLS/SSL |
| 5 | **Session** | Ouverture et maintien des sessions | NetBIOS, RPC |
| 4 | **Transport** | Fiabilité, segmentation, ports | TCP, UDP |
| 3 | **Réseau** | Adressage logique et routage | IP, ICMP |
| 2 | **Liaison** | Accès au réseau local (MAC) | Ethernet, Wi-Fi |
| 1 | **Physique** | Transmission brute des bits | Câbles, signaux radio |

---

## ⚡ Mnémotechnique (couche 1 → 7)
> **P**our **L**e **R**éseau **T**out **S**e **P**asse **A**insi
> Physique · Liaison · Réseau · Transport · Session · Présentation · Application

---

## 🔑 Points clés
- **TLS = couche 6**, pas 7 — il chiffre les données avant transmission, c'est le rôle de la Présentation
- **DNS et DHCP = couche 7** — ce sont des services applicatifs, même s'ils utilisent UDP en couche 4
- Chaque équipement "monte" jusqu'à la couche dont il a besoin : un switch s'arrête à la couche 2, un routeur à la couche 3, un firewall applicatif monte jusqu'à la couche 7
