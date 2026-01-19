## Définition

Le **Three-Way Handshake** (poignée de main en trois étapes) est le processus d'établissement d'une connexion TCP entre un client et un serveur. C'est un mécanisme qui garantit que les deux parties sont prêtes à communiquer de manière fiable.

## Pourquoi une poignée de main ?

TCP est un protocole **orienté connexion** et **fiable**. Avant d'échanger des données, client et serveur doivent :

- Synchroniser leurs numéros de séquence
- Confirmer leur capacité à communiquer
- Négocier les paramètres de connexion

## Les 3 étapes du handshake

|Étape|Message|Direction|Flags actifs|Description|
|---|---|---|---|---|
|**1**|SYN|Client → Serveur|SYN|Le client initie la connexion avec un numéro de séquence initial (Seq=x)|
|**2**|SYN-ACK|Serveur → Client|SYN + ACK|Le serveur accepte avec son propre numéro de séquence (Seq=y) et accuse réception (Ack=x+1)|
|**3**|ACK|Client → Serveur|ACK|Le client confirme la réception (Ack=y+1). La connexion est établie|

## Détail du processus

### Étape 1 : SYN (Synchronize)

- Le client choisit un numéro de séquence initial aléatoire (ex: Seq=1000)
- Il envoie un segment TCP avec le flag **SYN=1**
- Le client passe en état **SYN-SENT**

### Étape 2 : SYN-ACK

- Le serveur reçoit le SYN et choisit son propre numéro de séquence (ex: Seq=5000)
- Il envoie un segment avec **SYN=1** et **ACK=1**
- L'accusé de réception est : Ack=1001 (numéro du client + 1)
- Le serveur passe en état **SYN-RECEIVED**

### Étape 3 : ACK (Acknowledge)

- Le client accuse réception du SYN du serveur : Ack=5001
- Il envoie un segment avec **ACK=1**
- Les deux parties passent en état **ESTABLISHED**
- **La connexion est opérationnelle**, les données peuvent être échangées

## Acronymes

|Acronyme|Signification|Définition|
|---|---|---|
|**TCP**|Transmission Control Protocol|Protocole de transport fiable de la couche 4|
|**SYN**|Synchronize|Flag indiquant une demande de synchronisation|
|**ACK**|Acknowledge|Flag indiquant un accusé de réception|
|**Seq**|Sequence number|Numéro de séquence du segment|
|**Ack**|Acknowledgment number|Numéro d'accusé de réception|

## Exemple concret

```
Client (192.168.1.10)                    Serveur (192.168.1.100:80)

État: CLOSED                             État: LISTEN
    |                                           |
    | 1. SYN (Seq=1000)                        |
    |------------------------------------------>|
    |                                           |
État: SYN-SENT                           État: SYN-RECEIVED
    |                                           |
    |        2. SYN-ACK (Seq=5000, Ack=1001)   |
    |<------------------------------------------|
    |                                           |
    | 3. ACK (Ack=5001)                        |
    |------------------------------------------>|
    |                                           |
État: ESTABLISHED                        État: ESTABLISHED
    |                                           |
    |    === Échange de données ===            |
```

## Points clés à retenir

✅ Le handshake se fait **avant** tout échange de données  
✅ Il nécessite **3 segments TCP** (d'où "three-way")  
✅ Chaque partie choisit son propre numéro de séquence initial  
✅ Les numéros de séquence permettent de maintenir l'ordre des données  
✅ Sans handshake réussi, **aucune donnée ne peut être transmise**  
✅ Ce mécanisme garantit que les deux parties sont prêtes et synchronisées

## Fermeture de connexion

TCP utilise également un mécanisme en **4 étapes** pour fermer une connexion proprement :

1. **FIN** (Client → Serveur)
2. **ACK** (Serveur → Client)
3. **FIN** (Serveur → Client)
4. **ACK** (Client → Serveur)

Cela permet de s'assurer qu'aucune donnée n'est perdue lors de la fermeture.