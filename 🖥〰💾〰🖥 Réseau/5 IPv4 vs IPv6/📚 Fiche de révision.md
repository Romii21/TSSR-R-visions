### Comparatif IPv4 vs IPv6

|Aspect|IPv4|IPv6|
|---|---|---|
|**Taille adresse**|32 bits (4,3 milliards d'adresses)|128 bits (3,4 × 10³⁸ adresses)|
|**Notation**|Décimale pointée (ex: 192.168.0.1)|Hexadécimale avec ":" (ex: 2001:db8::1)|
|**Découpage**|Classes (A/B/C) puis CIDR (/s)|CIDR-like, préfixe variable (ex: /64)|
|**Entête paquet**|Variable (20-60 octets), inclut checksum, fragmentation, TTL|Fixe (40 octets), simplifié : Traffic Class, Flow Label, Hop Limit, Next Header ; pas de checksum|
|**Fragmentation**|Gérée par routeurs (champs ID, Flags, Offset)|Optionnelle ; évitée via PMTUD (découverte MTU)|
|**Configuration**|Manuelle, DHCP ; adresses privées (RFC 1918)|SLAAC (auto), DHCPv6 optionnel ; adresses lien-local (fe80::/10), uniques locales (fc00::/7), globales (2000::/3)|
|**Broadcast**|Oui (ex: 255.255.255.255)|Non ; remplacé par multicast/anycast|
|**Sécurité**|IPsec optionnel|IPsec intégré ; mobilité (MIPv6)|
|**Protocoles associés**|ICMP (erreurs), ARP (résolution MAC)|ICMPv6 (inclut ARP/IGMP), NDP (découverte voisins)|
|**Objectifs**|Routage basique, QoS, TTL pour vie paquet|Étendre adresses, simplifier, autoconfig, end-to-end sans NAT|

IPv6 résout la pénurie d'adresses IPv4, simplifie les routeurs et améliore l'autoconfiguration, mais coexiste via dual-stack.

### Méthodes de Conversion/Transition

- **Dual-Stack** : Machines/routeurs supportent IPv4 et IPv6 simultanément ; trafic choisit la version disponible.
- **Tunneling** : Encapsule IPv6 dans IPv4 (ex: 6to4) pour traverser réseaux IPv4-only.
- **Translation** : Convertit paquets (ex: NAT64 pour traduire IPv6 en IPv4) ; utilisé pour compatibilité apps/serveurs.

Transition progressive : IPv6 natif préféré, fallback IPv4.