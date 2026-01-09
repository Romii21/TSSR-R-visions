### Comparaison des 3 types de NAT

|Type|Sens|Mode|Niveau|Usage principal|
|---|---|---|---|---|
|**PAT/NAPT**|Source|Dynamique|IP + port|Accès Internet pour clients|
|**DNAT**|Destination|Statique|IP (+ port)|Publication de services|
|**NAT 1:1**|Source + Destination|Statique|IP|Serveurs en DMZ|

### À retenir absolument

1. **NAT = solution transitoire** à la pénurie d'IPv4
2. **PAT/NAPT** : le plus utilisé, une seule IP publique pour plusieurs machines
3. **DNAT** : permet de publier des services internes (port forwarding)
4. **NAT 1:1** : une IP publique dédiée par serveur
5. Le NAT introduit des **problèmes de compatibilité** avec certains protocoles
6. Avec **IPv6**, le NAT n'est plus indispensable

### NAT et IPv6

- En IPv6, le NAT existe encore (**NPTv6**) mais n'est **plus indispensable**
- IPv6 résout le problème de pénurie d'adresses
- Une raison supplémentaire d'accélérer le déploiement d'IPv6

---
