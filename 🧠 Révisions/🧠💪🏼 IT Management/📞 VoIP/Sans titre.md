# 📞 Synthèse — La Téléphonie sur IP (VoIP / ToIP)

> **Cours : La téléphonie sur IP** — Synthèse simplifiée avec tableaux, définitions et points clés.

---

## 📑 Sommaire

1. [Introduction — Les bases](https://claude.ai/chat/b9aef2df-108d-4588-8591-c44d65ad7ff0#1-introduction--les-bases)
    - [1.1 Définitions rapides : RTC, VoIP, ToIP](https://claude.ai/chat/b9aef2df-108d-4588-8591-c44d65ad7ff0#11-d%C3%A9finitions-rapides--rtc-voip-toip)
    - [1.2 Historique de la téléphonie](https://claude.ai/chat/b9aef2df-108d-4588-8591-c44d65ad7ff0#12-historique-de-la-t%C3%A9l%C3%A9phonie)
    - [1.3 Le RTC — Réseau Téléphonique Commuté](https://claude.ai/chat/b9aef2df-108d-4588-8591-c44d65ad7ff0#13-le-rtc--r%C3%A9seau-t%C3%A9l%C3%A9phonique-commut%C3%A9)
    - [1.4 Le RNIS — Réseau Numérique à Intégration de Services](https://claude.ai/chat/b9aef2df-108d-4588-8591-c44d65ad7ff0#14-le-rnis--r%C3%A9seau-num%C3%A9rique-%C3%A0-int%C3%A9gration-de-services)
    - [1.5 La VoIP — Voice over IP](https://claude.ai/chat/b9aef2df-108d-4588-8591-c44d65ad7ff0#15-la-voip--voice-over-ip)
    - [1.6 La ToIP — Telephony over IP](https://claude.ai/chat/b9aef2df-108d-4588-8591-c44d65ad7ff0#16-la-toip--telephony-over-ip)
    - [1.7 VoIP vs ToIP : ne pas confondre !](https://claude.ai/chat/b9aef2df-108d-4588-8591-c44d65ad7ff0#17-voip-vs-toip--ne-pas-confondre-)
    - [1.8 Avantages et inconvénients de la ToIP](https://claude.ai/chat/b9aef2df-108d-4588-8591-c44d65ad7ff0#18-avantages-et-inconv%C3%A9nients-de-la-toip)
2. [Les Éléments de la ToIP](https://claude.ai/chat/b9aef2df-108d-4588-8591-c44d65ad7ff0#2-les-%C3%A9l%C3%A9ments-de-la-toip)
    - [2.1 Les terminaux IP](https://claude.ai/chat/b9aef2df-108d-4588-8591-c44d65ad7ff0#21-les-terminaux-ip)
    - [2.2 Les switchs PoE](https://claude.ai/chat/b9aef2df-108d-4588-8591-c44d65ad7ff0#22-les-switchs-poe)
    - [2.3 L'adaptateur ATA](https://claude.ai/chat/b9aef2df-108d-4588-8591-c44d65ad7ff0#23-ladaptateur-ata)
    - [2.4 Les softphones](https://claude.ai/chat/b9aef2df-108d-4588-8591-c44d65ad7ff0#24-les-softphones)
    - [2.5 Le commutateur (PBX)](https://claude.ai/chat/b9aef2df-108d-4588-8591-c44d65ad7ff0#25-le-commutateur-pbx)
    - [2.6 Le PABX](https://claude.ai/chat/b9aef2df-108d-4588-8591-c44d65ad7ff0#26-le-pabx)
    - [2.7 L'IPBX](https://claude.ai/chat/b9aef2df-108d-4588-8591-c44d65ad7ff0#27-lipbx)
    - [2.8 La passerelle VoIP](https://claude.ai/chat/b9aef2df-108d-4588-8591-c44d65ad7ff0#28-la-passerelle-voip)
    - [2.9 Le protocole H.323](https://claude.ai/chat/b9aef2df-108d-4588-8591-c44d65ad7ff0#29-le-protocole-h323)
3. [Le Protocole SIP](https://claude.ai/chat/b9aef2df-108d-4588-8591-c44d65ad7ff0#3-le-protocole-sip)
    - [3.1 Définition et objectifs](https://claude.ai/chat/b9aef2df-108d-4588-8591-c44d65ad7ff0#31-d%C3%A9finition-et-objectifs)
    - [3.2 Fonctionnement technique](https://claude.ai/chat/b9aef2df-108d-4588-8591-c44d65ad7ff0#32-fonctionnement-technique)
    - [3.3 Les messages SIP](https://claude.ai/chat/b9aef2df-108d-4588-8591-c44d65ad7ff0#33-les-messages-sip)
    - [3.4 Le User Agent (UA)](https://claude.ai/chat/b9aef2df-108d-4588-8591-c44d65ad7ff0#34-le-user-agent-ua)
    - [3.5 Le Registrar](https://claude.ai/chat/b9aef2df-108d-4588-8591-c44d65ad7ff0#35-le-registrar)
    - [3.6 Le Proxy SIP et B2BUA](https://claude.ai/chat/b9aef2df-108d-4588-8591-c44d65ad7ff0#36-le-proxy-sip-et-b2bua)
    - [3.7 Déroulement d'un appel SIP](https://claude.ai/chat/b9aef2df-108d-4588-8591-c44d65ad7ff0#37-d%C3%A9roulement-dun-appel-sip)
    - [3.8 Glossaire SIP](https://claude.ai/chat/b9aef2df-108d-4588-8591-c44d65ad7ff0#38-glossaire-sip)
4. [Les Évolutions Réseau](https://claude.ai/chat/b9aef2df-108d-4588-8591-c44d65ad7ff0#4-les-%C3%A9volutions-r%C3%A9seau)
    - [4.1 Le Trunk SIP](https://claude.ai/chat/b9aef2df-108d-4588-8591-c44d65ad7ff0#41-le-trunk-sip)
    - [4.2 Le Centrex IP](https://claude.ai/chat/b9aef2df-108d-4588-8591-c44d65ad7ff0#42-le-centrex-ip)
    - [4.3 Comparaison des architectures](https://claude.ai/chat/b9aef2df-108d-4588-8591-c44d65ad7ff0#43-comparaison-des-architectures)
5. [🎯 Points clés à retenir](https://claude.ai/chat/b9aef2df-108d-4588-8591-c44d65ad7ff0#-points-cl%C3%A9s-%C3%A0-retenir)
6. [📚 Sources](https://claude.ai/chat/b9aef2df-108d-4588-8591-c44d65ad7ff0#-sources)

---

## 1. Introduction — Les bases


## 2. Les Éléments de la ToIP


## 3. Le Protocole SIP

### 3.1 Définition et objectifs

**SIP** = _Session Initiation Protocol_ (Protocole d'Initiation de Session)

> 📄 Défini par l'**IETF** _(Internet Engineering Task Force)_ dans la **RFC 3261**.

C'est un protocole **ouvert** (sans restriction d'utilisation) de **signalisation** pour **établir, modifier et terminer des sessions multimédia**.

> ⚠️ SIP ne transporte PAS la voix. Il établit la session. C'est **RTP** qui transporte ensuite les flux audio/vidéo.

**Usages de SIP :**

- ToIP / VoIP ← usage principal
- Visioconférence
- IoT _(Internet of Things)_ / Domotique
- Réalité virtuelle

---

### 3.2 Fonctionnement technique

|Caractéristique|Détail|
|---|---|
|Architecture|Client / Serveur|
|Couche OSI|**Couche 5** (Session)|
|Transport|**UDP ou TCP** (IPv4 et IPv6)|
|Port SIP|**5060** (non chiffré)|
|Port SIPS|**5061** (SIP-TLS = SIP chiffré)|
|Format des messages|**Texte** (comme HTTP)|
|Adressage|Numéro de téléphone, adresse IP, email…|

---

### 3.3 Les messages SIP

Un message SIP contient :

- Une **méthode** (requête ou réponse)
- Une **en-tête** (header) avec des informations normalisées

**Requêtes principales :**

|Méthode|Rôle|
|---|---|
|`INVITE`|Demande d'établissement d'une nouvelle session|
|`ACK`|Confirme l'établissement de la session après INVITE|
|`BYE`|Fin d'appel / fin de dialogue|
|`CANCEL`|Annulation d'une requête en cours|
|`REGISTER`|Enregistrement d'un UA auprès du Registrar|

**Réponses principales :**

|Code|Signification|
|---|---|
|`100 Trying`|Requête en cours de traitement|
|`180 Ringing`|L'UA de destination a reçu INVITE, le téléphone sonne|
|`200 OK`|Succès|
|`486 Busy Here`|Occupé|
|`404 Not Found`|Destinataire non trouvé|

**Champs d'en-tête importants :**

- `From` : identification de l'appelant, avec l'**AOR** _(Address Of Record)_
- `Via` : route à emprunter pour le message

---

### 3.4 Le User Agent (UA)

Un **User Agent (UA)** est un **logiciel client** responsable de l'envoi et de la réception des messages SIP. On le trouve dans les téléphones IP et les softphones.

|Type|Sigle|Rôle|
|---|---|---|
|Agent Utilisateur Client|**UAC**|Envoie une requête SIP et reçoit une réponse SIP|
|Agent Utilisateur Serveur|**UAS**|Génère une réponse SIP à une requête SIP reçue|

> 📌 Un même terminal peut être UAC (quand il appelle) et UAS (quand il reçoit un appel).

---

### 3.5 Le Registrar

Le **Registrar** gère les **enregistrements SIP** des UA. Il permet d'associer une identité SIP à une adresse IP.

- Traite les requêtes de type `REGISTER`
- Stocke en base de données un **enregistrement AOR** _(Address of Record)_ :
    - Format : `sip:user@domain` (ex : `sip:alice@entreprise.fr`)
    - Associé à l'adresse IP de l'UA

**Processus d'enregistrement :**

```
UA  ──F1: REGISTER (demande initiale + AOR)──────────→  REGISTRAR
UA  ←─F2: 401 Unauthorized (infos de login demandées)──  REGISTRAR
UA  ──F3: REGISTER (avec login/mot de passe)─────────→  REGISTRAR
UA  ←─F4: 200 OK (enregistrement confirmé)────────────  REGISTRAR
```

---

### 3.6 Le Proxy SIP et B2BUA

**Proxy SIP :** intermédiaire entre deux UA qui ne connaissent pas leurs adresses IP mutuelles.

- Interroge la base du Registrar pour trouver le couple `URI SIP ↔ adresse IP`
- Route les messages SIP vers le bon destinataire

**B2BUA** _(Back to Back User Agent)_ : proxy avancé qui s'**interpose entre deux UA** et gère **toute la communication** (pas seulement le transfert des messages SIP).

||Proxy SIP|B2BUA|
|---|---|---|
|Rôle|Route les messages SIP|Gère l'intégralité de la session|
|Contrôle|Partiel (signalisation)|Total (signalisation + session)|
|Exemple|Routeur SIP|Asterisk, FreeSWITCH|

---

### 3.7 Déroulement d'un appel SIP

Voici les étapes d'un appel SIP complet entre Alice et Bob, via un Proxy SIP :

```
Alice (UAC)     Proxy SIP        Bob (UAS)
    │                │                │
    ├─── INVITE ────→│                │
    │                ├─── INVITE ────→│
    │                │←── 100 Trying ─┤
    │←── 100 Trying ─┤                │
    │                │←── 180 Ringing─┤  ← Bob's phone rings
    │←── 180 Ringing─┤                │
    │                │←── 200 OK ─────┤  ← Bob picks up
    │←── 200 OK ─────┤                │
    ├─── ACK ────────┼────────────────→│
    │                │                │
    │◄═══════════════ RTP/RTCP (flux voix) ════════════►│
    │                │                │
    ├─── BYE ────────┼────────────────→│  ← Alice hangs up
    │←──────────────200 OK ───────────┤
```

> 📌 **RTP** _(Real-time Transport Protocol)_ : protocole qui transporte les **flux voix et vidéo** pendant l'appel. **RTCP** _(RTP Control Protocol)_ : protocole de contrôle associé à RTP.

---

### 3.8 Glossaire SIP

|Terme|Définition|
|---|---|
|**Transaction SIP**|Un échange requête/réponse finalisé et complet entre deux entités|
|**Dialogue SIP**|Une série de transactions entre deux éléments (ex : toute la durée d'un appel)|
|**Session média**|Flux audio et/ou vidéo circulant entre deux éléments via RTP|
|**AOR** _(Address of Record)_|Identifiant SIP unique d'un utilisateur, format `sip:user@domain`|
|**URI SIP** _(Uniform Resource Identifier)_|Adresse unique identifiant une ressource SIP|
|**RFC 3261**|Document officiel définissant le protocole SIP (IETF)|

> 💡 Des **transactions et des dialogues** sont nécessaires pour obtenir une **session média**.

---

## 4. Les Évolutions Réseau

### 4.1 Le Trunk SIP

Le **Trunk SIP** est un **lien virtuel** permettant à un **IPBX** de gérer ses communications téléphoniques **via Internet** (VoIP), en remplacement des lignes physiques T0/T2 du RNIS.

**Comment ça marche :**

- On souscrit une offre auprès d'un **opérateur de téléphonie IP**
- L'IPBX se connecte à l'opérateur via Internet
- Les appels passent en **VoIP** sur la connexion SDSL/Fibre

**Avantages vs RNIS (T0/T2) :**

||RNIS (T0/T2)|Trunk SIP|
|---|---|---|
|Canaux simultanés|Limités (2 par T0, 15-30 par T2)|**Illimité** (selon abonnement)|
|Support physique|Câble cuivre dédié|Internet (SDSL, Fibre)|
|Scalabilité|Contrainte (ajout de lignes physiques)|Flexible (ajout de canaux logiciels)|
|Coût|Plus élevé|Moins élevé|
|Disponibilité|Fin de commercialisation 2021|Technologie actuelle|

```
Architecture avec Trunk SIP :
[IPBX] ──(VoIP/SIP)──→ [Opérateur IP] ──→ [Internet] ──→ [Destinataire]
        ←─(SDSL/Fibre)──────────────────────────────────
```

---

### 4.2 Le Centrex IP

Le **Centrex IP** est une solution de téléphonie **entièrement hébergée dans le cloud** par l'opérateur. L'entreprise n'a pas à gérer d'IPBX en local.

> 📌 "Centrex" vient de _CENTRal EXchange_ — le central téléphonique est chez l'opérateur.

**Principe :**

- L'opérateur **centralise** les serveurs téléphoniques de tous ses clients sur une plateforme unique mutualisée.
- Les terminaux (téléphones IP, softphones) se connectent directement à cette plateforme via Internet.

```
Architecture Centrex :
[Téléphone IP] ──→ [LAN] ──→ [Internet] ──→ [Plateforme Centrex (Opérateur)]
```

**Pour qui ? Avantages et inconvénients :**

||Centrex IP|
|---|---|
|✅ Idéal pour|Petites et moyennes entreprises|
|✅ Avantages|Simple, peu coûteux, rapide à déployer, sécurisé, souvent en package équipements inclus|
|❌ Inconvénients|Niveau de service limité, moins de flexibilité, dépendant de l'opérateur|

---

### 4.3 Comparaison des architectures

|Critère|PABX (RTC)|IPBX (sur site)|IPBX + Trunk SIP|Centrex IP|
|---|---|---|---|---|
|Réseau utilisé|Cuivre/RTC|IP (LAN)|IP + Internet|Internet seul|
|Équipement en entreprise|PABX physique|IPBX physique|IPBX physique|Aucun (cloud)|
|Gestion|En interne|En interne|En interne|Opérateur|
|Flexibilité|Faible|Bonne|Très bonne|Moyenne|
|Coût initial|Élevé|Moyen|Moyen|Faible|
|Coût de fonctionnement|Élevé|Moyen|Faible|Abonnement mensuel|
|Avenir|❌ En extinction|✅ Standard actuel|✅ Standard actuel|✅ En croissance|

---

## 🎯 Points clés à retenir

### 🔤 Acronymes essentiels

|Acronyme|Signification|
|---|---|
|**RTC**|Réseau Téléphonique Commuté|
|**RNIS**|Réseau Numérique à Intégration de Services|
|**VoIP**|Voice over IP|
|**ToIP**|Telephony over IP|
|**PABX**|Private Automatic Branch EXchange|
|**IPBX**|Internet Protocol Branch EXchange|
|**SIP**|Session Initiation Protocol|
|**ATA**|Adaptateur Téléphone Analogique|
|**PoE**|Power over Ethernet|
|**UA**|User Agent|
|**UAC**|User Agent Client|
|**UAS**|User Agent Server|
|**AOR**|Address of Record|
|**URI**|Uniform Resource Identifier|
|**RTP**|Real-time Transport Protocol|
|**RTCP**|RTP Control Protocol|
|**SVI**|Serveur Vocal Interactif|
|**MEVO**|Messagerie Vocale|
|**QoS**|Quality of Service|
|**FXS**|Foreign eXchange Subscriber|
|**FXO**|Foreign eXchange Office|
|**B2BUA**|Back to Back User Agent|
|**IETF**|Internet Engineering Task Force|
|**RFC**|Request For Comments (documents de standardisation)|
|**ARCEP**|Autorité de Régulation des Communications Électroniques et des Postes|

---

### 📌 Les 10 points les plus importants

1. **Le RTC sera éteint entre 2023 et 2030** → les entreprises doivent migrer vers la VoIP/ToIP.
    
2. **VoIP ≠ ToIP** : VoIP est un protocole de transport de la voix ; ToIP est une infrastructure complète de téléphonie d'entreprise.
    
3. **PABX = téléphonie RTC (cuivre)** / **IPBX = téléphonie IP** : même rôle, mais technologies différentes.
    
4. **Le PoE** permet d'alimenter les téléphones IP via le câble RJ45 (un seul câble = données + alimentation).
    
5. **SIP** est le protocole incontournable de la ToIP : il établit, gère et termine les sessions (mais ne transporte pas la voix — c'est **RTP** qui le fait).
    
6. **SIP fonctionne sur le port 5060** (UDP/TCP), **SIPS (chiffré) sur le port 5061**.
    
7. **Un appel SIP implique 3 composants** : le **User Agent** (terminal), le **Registrar** (annuaire) et le **Proxy SIP** (routeur d'appels).
    
8. **Le Trunk SIP remplace les lignes T0/T2 du RNIS** : plus de limites sur les canaux simultanés, connexion via Internet.
    
9. **Le Centrex IP** = téléphonie dans le cloud, idéale pour les PME : simple à déployer mais moins flexible.
    
10. **La QoS** est indispensable en ToIP pour prioriser les paquets voix (très sensibles aux délais et aux pertes) sur le réseau IP.
    

---

## 📚 Sources

- **Cours principal** : _La téléphonie sur IP_ (document de cours fourni)
- **RFC 3261** — Définition officielle du protocole SIP : [https://datatracker.ietf.org/doc/html/rfc3261](https://datatracker.ietf.org/doc/html/rfc3261)
- **ARCEP** — Calendrier de fermeture du réseau cuivre : [https://www.arcep.fr](https://www.arcep.fr/)
- **IEEE 802.3 PoE** — Standard Power over Ethernet : [https://standards.ieee.org/ieee/802.3af/1090/](https://standards.ieee.org/ieee/802.3af/1090/)
- **Wikipedia — SIP** : [https://fr.wikipedia.org/wiki/Session_Initiation_Protocol](https://fr.wikipedia.org/wiki/Session_Initiation_Protocol)
- **Wikipedia — RNIS** : [https://fr.wikipedia.org/wiki/R%C3%A9seau_num%C3%A9rique_%C3%A0_int%C3%A9gration_de_services](https://fr.wikipedia.org/wiki/R%C3%A9seau_num%C3%A9rique_%C3%A0_int%C3%A9gration_de_services)
- **Wikipedia — VoIP** : [https://fr.wikipedia.org/wiki/Voix_sur_IP](https://fr.wikipedia.org/wiki/Voix_sur_IP)

---

_Synthèse réalisée à partir du cours "La téléphonie sur IP" — Mars 2026_