📚 Fiche de révision — Encapsulation réseau & Wireshark

🔑 C'est quoi l'encapsulation ?
L'encapsulation est le processus par lequel chaque couche OSI ajoute un en-tête (header) aux données avant de les transmettre à la couche inférieure. À la réception, chaque couche retire son en-tête : c'est la désencapsulation.

📦 Schéma d'encapsulation
ÉMETTEUR                                  RÉCEPTEUR
─────────────────────────────────────────────────────
[7] Application  →  Données              Données
[6] Présentation →  Données              Données
[5] Session      →  Données              Données
                         ↓                    ↑
[4] Transport    →  [En-tête TCP/UDP | Données ]        = Segment / Datagramme
                         ↓                    ↑
[3] Réseau       →  [En-tête IP | Segment ]             = Paquet
                         ↓                    ↑
[2] Liaison      →  [En-tête ETH | Paquet | FCS ]       = Trame
                         ↓                    ↑
[1] Physique     →  01001011010100110...                 = Bits

🔁 Encapsulation à l'envoi | Désencapsulation à la réception


🧩 Anatomie des en-têtes
En-tête TCP (Couche 4)
 0      7 8     15 16    23 24    31
┌─────────┬─────────┬──────────────────┐
│ Port src│ Port dst│  Numéro de séq.  │
├─────────┴─────────┼──────────────────┤
│  Numéro d'acquit. │  Flags (SYN/ACK) │
├───────────────────┼──────────────────┤
│   Fenêtre (flow)  │    Checksum      │
└───────────────────┴──────────────────┘
ChampRôlePort source / destIdentifier l'applicationNuméro de séquenceOrdonner les segmentsNuméro d'acquittementConfirmer la réceptionFlagsSYN, ACK, FIN, RST, PSH, URGFenêtreContrôle de fluxChecksumVérification d'intégrité
En-tête UDP (Couche 4)
┌────────────┬────────────┬───────────┬──────────┐
│  Port src  │  Port dst  │ Longueur  │ Checksum │
└────────────┴────────────┴───────────┴──────────┘

UDP = 4 champs seulement → rapide, léger, sans garantie

En-tête IP (Couche 3)
ChampRôleVersionIPv4 (4) ou IPv6 (6)TTLTime To Live — évite les boucles (décrémenté à chaque routeur)ProtocoleTCP (6), UDP (17), ICMP (1)IP source / destAdresses logiques de l'émetteur et du destinataireChecksumVérification d'intégrité de l'en-tête
Trame Ethernet (Couche 2)
┌──────────┬──────────┬────────┬─────────┬─────┐
│ MAC dst  │ MAC src  │ Type   │ Données │ FCS │
│ 6 octets │ 6 octets │2 octets│46-1500o │4 o  │
└──────────┴──────────┴────────┴─────────┴─────┘
ChampRôleMAC destinationAdresse physique du destinataireMAC sourceAdresse physique de l'émetteurType/EtherTypeIPv4 (0x0800), ARP (0x0806), IPv6 (0x86DD)FCSFrame Check Sequence — détection d'erreurs

🔄 TCP — 3-Way Handshake
Client                    Serveur
  │──── SYN ────────────────▶│   (je veux me connecter)
  │◀─── SYN + ACK ───────────│   (ok, je suis prêt)
  │──── ACK ────────────────▶│   (c'est parti !)
  │                          │
  │════ Données échangées ═══│
  │                          │
  │──── FIN ────────────────▶│   (je ferme)
  │◀─── FIN + ACK ───────────│   (fermeture confirmée)

🦈 Wireshark — L'essentiel
C'est quoi ?
Wireshark est un analyseur de protocoles réseau (sniffer) open-source. Il capture et affiche en temps réel les trames qui transitent sur une interface réseau.

Interface — Les 3 zones principales
┌─────────────────────────────────────────────┐
│  Barre de filtre                            │  ← Saisir les filtres ici
├─────────────────────────────────────────────┤
│  Liste des paquets capturés                 │  ← Vue globale (n°, temps, IP, proto...)
├─────────────────────────────────────────────┤
│  Détail du paquet sélectionné               │  ← Couches OSI dépliables
├─────────────────────────────────────────────┤
│  Données brutes (hexadécimal + ASCII)       │  ← Les octets réels
└─────────────────────────────────────────────┘

🔍 Filtres de capture (avant capture)
Réduisent les données capturées — moins gourmand en ressources.
bashhost 192.168.1.1          # Trafic vers/depuis cette IP
port 80                   # Trafic sur le port 80
tcp                       # Uniquement TCP
udp                       # Uniquement UDP
not port 22               # Exclure SSH

🔍 Filtres d'affichage (après capture)
Filtrent les paquets déjà capturés — plus puissants et précis.
bash# Protocoles
ip                        # Tout le trafic IP
tcp                       # Tout le trafic TCP
udp                       # Tout le trafic UDP
http                      # Trafic HTTP
dns                       # Requêtes DNS
arp                       # Trafic ARP
icmp                      # Ping / messages ICMP

# Adresses IP
ip.addr == 192.168.1.1    # IP source OU destination
ip.src == 192.168.1.1     # IP source uniquement
ip.dst == 192.168.1.1     # IP destination uniquement

# Ports
tcp.port == 443           # Port TCP 443 (HTTPS)
udp.port == 53            # Port UDP 53 (DNS)
tcp.dstport == 80         # Port destination 80

# Flags TCP
tcp.flags.syn == 1        # Paquets SYN
tcp.flags.ack == 1        # Paquets ACK
tcp.flags.reset == 1      # Paquets RST (connexion reset)
tcp.flags.fin == 1        # Paquets FIN (fermeture)

# Combinaisons (opérateurs)
ip.src == 10.0.0.1 && tcp.port == 80     # ET logique
ip.src == 10.0.0.1 || ip.src == 10.0.0.2 # OU logique
!arp                                      # Exclure ARP

# Contenu
http.request.method == "GET"             # Requêtes HTTP GET
http.response.code == 200               # Réponses HTTP 200 OK
dns.qry.name contains "google"          # DNS contenant "google"

🎨 Code couleur par défaut
CouleurSignification🟢 Vert clairTrafic HTTP🔵 Bleu clairDNS⬛ NoirErreurs TCP (retransmissions, RST)🟡 Jaune/VertTrafic ARP / divers🔴 RougeErreurs, anomalies

📋 Cas d'usage courants
ObjectifFiltre à utiliserVoir un 3-way handshaketcp.flags.syn == 1Analyser une navigation webhttp ou tcp.port == 443Voir les résolutions DNSdnsDétecter un scan de portstcp.flags.syn == 1 && tcp.flags.ack == 0Trouver des erreurs réseautcp.analysis.flagsSuivre une conversation TCPClic droit → Follow > TCP Stream

📝 À retenir absolument

Encapsulation = ajout d'en-têtes couche par couche à l'envoi | Désencapsulation = retrait à la réception
L'unité change à chaque couche : Données → Segment → Paquet → Trame → Bits
TTL décrémenté à chaque routeur → atteint 0 = paquet détruit (évite les boucles)
FCS (Ethernet) et Checksum (IP/TCP) servent à détecter les erreurs de transmission
Dans Wireshark : filtre de capture = avant | filtre d'affichage = après
Follow > TCP Stream dans Wireshark permet de reconstituer une session complète lisiblement
