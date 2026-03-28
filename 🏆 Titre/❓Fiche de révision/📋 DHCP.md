## Définition

**DHCP** (_Dynamic Host Configuration Protocol_) — RFC 2131 & RFC 2132 (IPv4) / RFC 8415 (IPv6)  
Protocole permettant l'**attribution automatique et centralisée** des paramètres réseau IP aux machines.

---

## Ports à retenir

|Port|Rôle|
|---|---|
|**UDP 67**|Serveur DHCP|
|**UDP 68**|Client DHCP|

---

## Paramètres distribués

|Type|Paramètres|
|---|---|
|**Obligatoires**|Adresse IP, Masque sous-réseau (CIDR), Bail|
|**Optionnels**|Passerelle par défaut, DNS, Nom de domaine, Serveur PXE/TFTP, NTP|

---

## La séquence DORA ⭐

> Séquence normale lors d'une **première connexion**

```
Client  ──DHCPDISCOVER──►  broadcast (255.255.255.255)
Serveur ──DHCPOFFER──────►  Client  (propose une config)
Client  ──DHCPREQUEST────►  Serveur (accepte l'offre)
Serveur ──DHCPACK─────────►  Client  (confirme & envoie la config)
```

---

## Les 8 messages DHCP

|Message|Sens|Description|
|---|---|---|
|`DHCPDISCOVER`|Client → Broadcast|Recherche d'un serveur DHCP|
|`DHCPOFFER`|Serveur → Client|Proposition de configuration|
|`DHCPREQUEST`|Client → Serveur|Demande de réservation|
|`DHCPACK`|Serveur → Client|Confirmation + envoi config|
|`DHCPNACK`|Serveur → Client|Refus de réservation|
|`DHCPDECLINE`|Client → Serveur|Adresse déjà utilisée (vérifié via ARP)|
|`DHCPRELEASE`|Client → Serveur|Résiliation du bail|
|`DHCPINFORM`|Client → Serveur|Demande de paramètres sans réservation d'IP|

---

## Cas d'usage

|Cas|Séquence|
|---|---|
|Première connexion|DISCOVER → OFFER → REQUEST → ACK|
|Redémarrage, IP encore valide|REQUEST → ACK|
|Redémarrage, IP invalide|REQUEST → NACK → DISCOVER|
|Bail expiré|NACK → DISCOVER|
|Changement de réseau|NACK → DISCOVER|

---

## Points clés

- Le client s'identifie via son **adresse MAC** (peu fiable : MAC spoofing)
    - IPv4 : option **61 Client Identifier**
    - IPv6 : **DUID** (DHCP Unique Identifier)
- Le broadcast DHCP **ne passe pas les routeurs** (sauf configuration `ip-helper`)
- Le serveur gère des **plages d'adresses** et des **baux** (durée limitée)
- Héritage : **RARP → BOOTP → DHCP**

---

## À retenir absolument

> **D.O.R.A.** — **UDP 67/68** — Centralisation de la configuration IP (v4 & v6)