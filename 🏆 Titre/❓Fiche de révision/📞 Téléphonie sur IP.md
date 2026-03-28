## 1. Définitions clés

|Terme|Définition|
|---|---|
|**RTC**|Réseau Téléphonique Commuté — réseau cuivre historique, fin prévue en 2030|
|**RNIS**|Réseau Numérique à Intégration de Services — hybride numérique/cuivre (marque : Numéris), fin de commercialisation 2021|
|**VoIP**|Transmission de la voix sur réseau IP (protocole) — commutation de **paquets**|
|**ToIP**|Gestion de la téléphonie d'entreprise sur réseau IP (infrastructure)|

> **Retenir :** VoIP = protocole / ToIP = infrastructure s'appuyant sur la VoIP

---

## 2. RTC vs VoIP

||RTC|VoIP|
|---|---|---|
|Transmission|Commutation de **circuits**|Commutation de **paquets**|
|Chemin|Unique, dédié|Multiple, variable|
|Réseau|Cuivre|IP (LAN/WAN, Wi-Fi, fibre…)|
|Stockage|Non|Oui (paquets)|

---

## 3. Équipements

|Équipement|Rôle|
|---|---|
|**Terminal IP**|Téléphone IP, smartphone, PC|
|**Softphone**|Logiciel remplaçant le téléphone physique (aussi appelé logiciel SIP)|
|**Adaptateur ATA**|Convertit un téléphone analogique (RTC) vers IP — ports RJ11 → RJ45|
|**Switch PoE**|Alimente les terminaux IP via le câble Ethernet (IEEE 802.3af/at/bt)|
|**PABX**|Autocommutateur privé — réseau RTC/RNIS|
|**IPBX**|Autocommutateur privé — réseau IP (remplace le PABX)|
|**Passerelle VoIP**|Convertit le trafic téléphonique en données IP|

---

## 4. Protocoles

### H.323

- Protocole historique (1996–2009), dérivé de H.320 (RNIS)
- Communication multimédia (voix, image, données) sur IP
- **Remplacé par SIP**

### SIP — Session Initiation Protocol (RFC 3261)

- Protocole de **signalisation** ouvert (couche 5 OSI)
- Utilisé en ToIP/VoIP, visio, IoT, réalité virtuelle

|Paramètre|Valeur|
|---|---|
|Transport|TCP ou **UDP**|
|Port standard|**5060**|
|Port sécurisé (SIPS/TLS)|**5061**|
|Format|Texte (similaire à HTTP)|
|Adressage|`sip:user@domain` (URI SIP)|

#### Composants SIP

|Composant|Rôle|
|---|---|
|**User Agent (UA)**|Client SIP présent dans les terminaux/softphones|
|**UAC**|Envoie une requête SIP|
|**UAS**|Répond à une requête SIP|
|**Registrar**|Enregistre les UA (couple URI / @IP) → base AOR|
|**Proxy SIP**|Route les messages SIP entre UA|
|**B2BUA**|Proxy avancé gérant toute la communication|

#### Méthodes SIP principales

|Requêtes|Réponses|
|---|---|
|INVITE, ACK, BYE|100 Trying|
|REGISTER, CANCEL|180 Ringing|
||200 OK, 401 Unauthorized|

### RTP / RTCP

- Transport du **flux média** (audio/vidéo) une fois la session établie

---

## 5. Canaux RNIS

|Canal|Communications simultanées|
|---|---|
|**T0** (accès de base)|2|
|**T2** (accès primaire)|15 à 30|

> Remplacés progressivement par le **Trunk SIP**

---

## 6. Évolutions réseau

### Trunk SIP

- Connexion IPBX ↔ opérateur via Internet (VoIP)
- Pas de limitation de canaux (contrairement à T0/T2)
- Nécessite une souscription auprès d'un opérateur IP

### Centrex IP

- Téléphonie hébergée dans le **cloud** (mutualisée)
- Simple et peu coûteux pour les PME
- Moins flexible qu'un IPBX on-premise

---

## 7. Avantages / Inconvénients ToIP

|✅ Avantages|❌ Inconvénients|
|---|---|
|Réseau unifié (data + voix)|Coût initial (migration matériel)|
|Fonctionnalités avancées (SVI, MEVO)|Qualité dépendante du réseau et de la QoS|
|Mobilité, télétravail|Vulnérabilité aux attaques réseau|
|Réduction des coûts inter-sites||

---

## 8. À retenir absolument

- **RTC → fin 2030** → migration obligatoire vers IP
- **VoIP ≠ ToIP** : protocole vs infrastructure
- **SIP port 5060** (UDP/TCP), **SIPS port 5061** (TLS)
- **PABX** (RTC) → **IPBX** (IP)
- Flux SIP = signalisation | Flux RTP = voix