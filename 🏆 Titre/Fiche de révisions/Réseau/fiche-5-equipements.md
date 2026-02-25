# 🖧 Fiche 5 — Équipements Réseau

> *"Chaque équipement a sa propre vision du réseau — certains ne voient que le voisinage immédiat, d'autres tracent la carte du monde entier."*

---

## Comparatif des équipements

| Équipement | Couche | Lit | Modifie | Rôle principal |
|------------|--------|-----|---------|----------------|
| **Hub** | L1 | Rien | Rien | Répète le signal sur tous les ports (obsolète) |
| **Switch L2** | L2 | MAC | Rien | Commute les trames dans un réseau local |
| **Switch L3** | L3 | MAC + IP | Rien | Commutation + routage inter-VLAN |
| **Routeur** | L3 | IP | MAC (à chaque saut) | Interconnecte des réseaux différents |
| **Firewall** | L3-L4 (L7) | IP + Port (+ contenu) | Rien | Filtre et sécurise le trafic |
| **Proxy** | L7 | Tout | Tout | Intermédiaire applicatif |

---

## 🔍 Zoom sur le Switch L2

Le switch apprend les adresses MAC des machines connectées et les stocke dans sa **table CAM** (Content Addressable Memory).

```
Trame reçue
    ↓
La MAC destination est-elle dans la table CAM ?
    ├── OUI → Envoie sur le port correspondant
    └── NON → Flood sur tous les ports (sauf port source)
               ↓
         Hôte répond → Switch apprend et met à jour sa table
```

> ⚠️ Le switch ne fait **jamais** d'ARP. C'est le rôle des hôtes.

---

## 🔍 Zoom sur le Routeur

À chaque saut, le routeur effectue systématiquement ces 3 actions :

1. **Lit** l'IP destination et consulte sa table de routage
2. **Remplace** la MAC source (la sienne) et la MAC destination (prochain saut)
3. **Décrémente** le TTL de 1 — si TTL = 0, le paquet est détruit

> ✅ L'adresse IP de destination ne change **jamais** tout au long du trajet.
