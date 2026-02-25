# 📦 Fiche 2 — Encapsulation & Équipements

> *"Quand tu envoies un message, il est emballé comme une poupée russe — chaque couche ajoute sa propre enveloppe."*

L'encapsulation est le processus par lequel chaque couche OSI enveloppe les données de la couche supérieure avec ses propres informations avant de les transmettre.

---

## L'ordre d'encapsulation (émission)

```
Données  →  Segment (L4)  →  Paquet (L3)  →  Trame (L2)  →  Bits (L1)
```

| Unité | Couche | Ajoute |
|-------|--------|--------|
| **Segment** | L4 | Ports source et destination, numéros de séquence TCP |
| **Paquet** | L3 | Adresses IP source et destination, TTL |
| **Trame** | L2 | Adresses MAC source et destination |
| **Bits** | L1 | Conversion en signal physique |

---

## 🔄 Le rôle de chaque équipement

**Switch L2** — *"Je ne connais que les prénoms"*
- Lit uniquement les **adresses MAC**
- Consulte sa **table CAM** pour décider sur quel port envoyer la trame
- Si la MAC destination est inconnue → **il inonde (flood) sur tous les ports** sauf celui d'entrée
- Ne modifie **rien** dans la trame

**Routeur** — *"Je lis les adresses complètes et je trace le chemin"*
- Lit l'**adresse IP destination** et consulte sa table de routage
- **Remplace les MAC** à chaque saut : il met la sienne en source, celle du prochain équipement en destination
- **Décrémente le TTL** de 1 à chaque passage
- Ne modifie **jamais** les adresses IP

---

## ⚠️ Détail important sur TLS
> Le chiffrement TLS s'applique **avant** l'encapsulation réseau.  
> Les routeurs et switches intermédiaires ne voient jamais le contenu en clair.  
> Seuls l'émetteur et le destinataire final peuvent déchiffrer les données.
