### 🔑 Les essentiels

|Élément|Information clé|
|---|---|
|**Acronyme DORA**|**D**iscover → **O**ffer → **R**equest → **A**ck|
|**Ports UDP**|Serveur: **67** / Client: **68**|
|**Fonction principale**|Centralisation de l'attribution automatique de configuration IP|
|**Versions**|IPv4 (RFC 2131/2132) et IPv6 (RFC 8415)|
|**Type de protocole**|Client/Serveur sur UDP|
|**Identification**|Par adresse MAC (ou Client Identifier/DUID)|
|**Portée**|Limitée au domaine de broadcast (sauf avec relais)|

### ⚠️ Points d'attention

- Les adresses MAC ne sont **pas des identifiants totalement fiables**
- Le DHCP ne franchit **pas les routeurs** sans configuration spécifique
- Un bail a une **durée limitée** et doit être renouvelé
- Les paramètres obligatoires sont : **IP, Masque, Bail**

---
