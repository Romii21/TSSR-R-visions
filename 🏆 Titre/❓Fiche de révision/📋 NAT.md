## 🔑 Définition

Le **NAT** traduit des adresses IP (et éventuellement des ports) à la frontière entre un réseau **privé (LAN)** et un réseau **public (WAN)**. Implémenté sur un routeur ou un pare-feu. Mécanisme quasi-exclusif à **IPv4** (RFC 3022).

### Plages d'adresses privées (RFC 1918)

|Plage|Étendue|
|---|---|
|`10.0.0.0/8`|10.0.0.0 → 10.255.255.255|
|`172.16.0.0/12`|172.16.0.0 → 172.31.255.255|
|`192.168.0.0/16`|192.168.0.0 → 192.168.255.255|

> ⚠️ Ces adresses ne sont **pas routables** sur Internet.

---

## ⚙️ Les 3 critères du NAT

|Critère|Options|
|---|---|
|**Sens de traduction**|Source (sortie) / Destination (entrée)|
|**Mode d'association**|Statique (fixe) / Dynamique (temporaire)|
|**Niveau de traduction**|@IP seule / @IP + Port|

---

## 🔄 Les 3 types de NAT

### 1. PAT / NAPT _(le plus courant)_

> **Port Address Translation** — aussi appelé _NAT overload_ ou _masquerade_

- **Sens** : Source | **Mode** : Dynamique | **Niveau** : @IP + Port
- Plusieurs machines internes → **1 seule IP publique** (différenciées par les ports)
- Usages : accès Internet des clients, box domestiques
- ⚠️ Incompatible avec FTP actif, certains contrôles d'intégrité ; recalcul des checksums nécessaire

### 2. DNAT _(publication de services)_

> **Destination NAT** — aussi appelé _port forwarding_ ou _static DNAT_

- **Sens** : Destination | **Mode** : Statique | **Niveau** : @IP (+ port optionnel)
- Redirige le trafic entrant vers un serveur interne
- Plusieurs services publiables sur la même IP publique **avec des ports différents**
- ⚠️ Expose les services → pare-feu strict obligatoire

### 3. NAT 1:1 _(Static NAT)_

> Une IP privée ↔ une IP publique dédiée, dans les deux sens

- **Sens** : Source + Destination | **Mode** : Statique | **Niveau** : @IP
- Les ports **ne changent pas**, tous les ports sont accessibles
- Usage : serveur en DMZ
- ⚠️ Consomme une IP publique par machine

---

## 🌐 Ports courants à connaître

|Service|Protocole|Port|
|---|---|---|
|HTTP|TCP|**80**|
|HTTPS|TCP|**443**|
|RDP|TCP|**3389**|
|FTP (contrôle)|TCP|**21**|
|SSH|TCP|**22**|

---

## 📊 Tableau comparatif

|Type|Sens|Mode|Niveau|Usage|
|---|---|---|---|---|
|**PAT/NAPT**|Source|Dynamique|@IP + Port|Accès Internet clients|
|**DNAT**|Destination|Statique|@IP (+ Port)|Publication de services|
|**NAT 1:1**|Src + Dst|Statique|@IP|Serveur en DMZ|

---

## ⚡ Points clés à retenir

- Le NAT est une **solution transitoire** à la pénurie d'IPv4 (≈ 4,3 milliards d'adresses)
- Il **casse le principe de bout en bout** (end-to-end)
- Pour héberger un serveur derrière un NAT → **port forwarding** (DNAT)
- Si une IP publique est bloquée (spam, abus), **tous les utilisateurs** derrière ce NAT sont bloqués
- Avec **IPv6**, NAT n'est plus indispensable → argument pour accélérer la migration
- Le **Carrier-grade NAT** déploie le NAT directement au niveau de l'opérateur