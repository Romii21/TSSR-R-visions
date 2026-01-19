## Définition

**DORA** est l'acronyme des 4 étapes du protocole **DHCP** (Dynamic Host Configuration Protocol) permettant à un client d'obtenir automatiquement une configuration IP.

- **D**iscover
- **O**ffer
- **R**equest
- **A**cknowledge

## Les 4 Étapes

|Étape|Message|Direction|Description|
|---|---|---|---|
|1|**DISCOVER**|Client → Broadcast|Le client cherche un serveur DHCP sur le réseau|
|2|**OFFER**|Serveur → Client|Le serveur propose une configuration IP disponible|
|3|**REQUEST**|Client → Serveur|Le client accepte et réserve la configuration proposée|
|4|**ACK**|Serveur → Client|Le serveur confirme et envoie la configuration complète|

## Fonctionnement Détaillé

### 1. DISCOVER (Découverte)

- Le client n'a **aucune adresse IP** (source : `0.0.0.0`)
- Envoi en **broadcast** (`255.255.255.255`) sur le port UDP 67
- S'identifie par son **adresse MAC**
- Message : _"Qui peut me donner une configuration IP ?"_

### 2. OFFER (Offre)

- Le serveur répond avec une **proposition**
- Contient : adresse IP, masque, durée du bail
- Peut être envoyé par **plusieurs serveurs** DHCP
- Message : _"Voici une IP disponible : 192.168.1.10"_

### 3. REQUEST (Demande)

- Le client **choisit une offre** (généralement la première reçue)
- Demande officiellement cette configuration
- Informe les autres serveurs qu'il refuse leurs offres
- Message : _"J'accepte l'offre, je réserve cette IP"_

### 4. ACK (Acquittement)

- Le serveur **confirme l'attribution**
- Envoie la **configuration complète** :
    - Adresse IP
    - Masque de sous-réseau
    - Passerelle par défaut
    - Serveurs DNS
    - Durée du bail
- Message : _"OK, IP réservée + paramètres complets"_

## Paramètres Transmis

### Obligatoires

- Adresse IP
- Masque de sous-réseau
- Durée du bail (lease time)

### Optionnels

- Passerelle par défaut
- Serveurs DNS
- Nom de domaine
- Serveur NTP (temps)

## Ports UDP Utilisés

|Équipement|Port|
|---|---|
|Serveur DHCP|67|
|Client DHCP|68|

## Exemple Pratique

```
Client (MAC: 00:1A:2B:3C:4D:5E)
      ↓
1. DISCOVER en broadcast
   "Je cherche un serveur DHCP"
      ↓
2. Serveur répond OFFER
   "Je te propose 192.168.1.50/24"
      ↓
3. Client envoie REQUEST
   "J'accepte 192.168.1.50"
      ↓
4. Serveur répond ACK
   "Confirmé + passerelle: 192.168.1.1, DNS: 8.8.8.8"
```

## Cas Particuliers

### Renouvellement de bail

Si le client **redémarre** avec une IP encore valide :

- Passe directement de **REQUEST** à **ACK**
- Évite les étapes DISCOVER et OFFER

### Refus du serveur

Si la configuration n'est plus disponible :

- Le serveur envoie un **NACK** (Negative Acknowledgment)
- Le client doit recommencer avec DISCOVER

## Points Clés

✅ Séquence **automatique** et **transparente** pour l'utilisateur  
✅ Permet la **centralisation** de la gestion des adresses IP  
✅ Évite les **conflits d'adresses**  
✅ Fonctionne uniquement sur le **même domaine de broadcast**  
✅ Le bail a une **durée limitée** (renewal obligatoire)