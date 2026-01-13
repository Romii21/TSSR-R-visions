## Commandes CLI - Configuration IP {#commandes-cli}

### Linux (commande `ip`)

|Action|Commande|
|---|---|
|Afficher config IPv4|`ip -4 address show` ou `ip -4 a`|
|Afficher config IPv6|`ip -6 address show` ou `ip -6 a`|
|Afficher une interface|`ip address show dev enp0s3`|
|Ajouter une IP|`sudo ip address add 192.168.1.10/24 dev enp0s3`|
|Supprimer une IP|`sudo ip address del 192.168.1.10/24 dev enp0s3`|
|Activer interface|`sudo ip link set enp0s3 up`|
|Désactiver interface|`sudo ip link set enp0s3 down`|
|Ajouter route par défaut|`sudo ip route add default via 192.168.1.1`|
|Afficher table de routage|`ip route` ou `ip -6 route`|

**Configuration persistante** : Modifier `/etc/network/interfaces` ou `/etc/netplan/` selon la distribution

### Windows (PowerShell)

|Action|Commande|
|---|---|
|Lister interfaces|`Get-NetAdapter`|
|Afficher config IP|`Get-NetIPAddress`|
|Ajouter une IP|`New-NetIPAddress -InterfaceIndex 2 -IPAddress 192.168.1.10 -PrefixLength 24`|
|Supprimer une IP|`Remove-NetIPAddress -IPAddress 192.168.1.10`|
|Définir passerelle|`New-NetRoute -DestinationPrefix 0.0.0.0/0 -InterfaceIndex 2 -NextHop 192.168.1.1`|
|Afficher routes|`Get-NetRoute`|
|Configurer DNS|`Set-DnsClientServerAddress -InterfaceIndex 2 -ServerAddresses 8.8.8.8,8.8.4.4`|

**Note** : `InterfaceIndex` s'obtient via `Get-NetAdapter`

### Windows (CMD - anciennes commandes)

|Action|Commande|
|---|---|
|Afficher config|`ipconfig` ou `ipconfig /all`|
|Ajouter IP statique|`netsh interface ip set address "Ethernet" static 192.168.1.10 255.255.255.0 192.168.1.1`|
|Passer en DHCP|`netsh interface ip set address "Ethernet" dhcp`|
|Afficher routes|`route print`|
|Ajouter route|`route add 192.168.2.0 mask 255.255.255.0 192.168.1.1`|
