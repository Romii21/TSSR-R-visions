## Définition

L'**encapsulation** est le processus par lequel chaque couche du modèle réseau ajoute ses propres informations (en-tête) aux données reçues de la couche supérieure.

## Principe de base

**PDU (Protocol Data Unit)** = Structure de données transmise par un protocole

```
PDU = EN-TÊTE (header) + CHARGE UTILE (payload)
```

- **En-tête** : informations de gestion du protocole
- **Payload** : données à transmettre (reçues de la couche supérieure)

## Le processus d'encapsulation

Chaque couche encapsule le PDU de la couche supérieure en y ajoutant son propre en-tête.

|Couche|Nom du PDU|Contenu|
|---|---|---|
|**Application (7)**|Données|Données brutes de l'application|
|**Transport (4)**|Segment (TCP) / Datagramme (UDP)|En-tête TCP/UDP + Données|
|**Réseau (3)**|Paquet|En-tête IP + Segment|
|**Liaison (2)**|Trame|En-tête Ethernet + Paquet + FCS|
|**Physique (1)**|Bits|Conversion en signal électrique|

## Exemple concret

Envoi du message "Bonjour" via HTTP :

1. **Application** : Données = "Bonjour"
2. **Transport (TCP)** : Ajout port source/destination → Segment
3. **Réseau (IP)** : Ajout adresses IP source/destination → Paquet
4. **Liaison (Ethernet)** : Ajout adresses MAC + contrôle d'erreur → Trame
5. **Physique** : Conversion en bits sur le câble

## Désencapsulation (réception)

À la réception, c'est **l'inverse** : chaque couche retire son en-tête pour transmettre les données à la couche supérieure.

## Pourquoi l'encapsulation ?

✅ **Modularité** : Chaque couche travaille indépendamment  
✅ **Séparation des responsabilités** : Chaque protocole gère sa fonction  
✅ **Évolutivité** : On peut changer une couche sans affecter les autres  
✅ **Interopérabilité** : Standards communs entre différents équipements

## Points clés à retenir

- Chaque couche ajoute **son propre en-tête**
- L'en-tête de la couche N devient le **payload** de la couche N-1
- Le processus est **réversible** (encapsulation → désencapsulation)
- L'encapsulation permet la **communication en couches indépendantes**