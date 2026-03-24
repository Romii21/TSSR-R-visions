
**Niveau :** TSSR  
**Durée estimée :** 45 minutes  
**Consigne :** Répondez avec précision, en développant vos réponses lorsque cela est demandé. Aucun QCM, des réponses rédigées sont attendues.

---

## Partie 1 – Adressage IP et sous-réseaux

**Question 1**  
Une machine possède l'adresse IP `172.16.45.200/20`.

- Calculez l'adresse réseau, l'adresse de broadcast, et la plage d'adresses hôtes utilisables.
- Combien d'hôtes peut-on adresser sur ce réseau ?

**REP** :
* **172.16.45.200/20**
	- Adresse de réseau = 172.16.32.0/20
	- Plage d'adresses = 172.16.32.1 à 172.16.47.254
	- Broadcast = 172.16.47.255
	- Hôtes = 4094

---

**Question 2**  
Vous devez découper le réseau `10.0.0.0/8` en sous-réseaux contenant chacun au minimum **500 hôtes**.

- Quel masque de sous-réseau utiliserez-vous ?
- Combien de sous-réseaux pouvez-vous créer ?

**REP** :
- J'utiliserai un CIDR de /23 soit un masque de sous-réseau de 255.255.254.0

---

**Question 3**  
Expliquez la différence entre une adresse IP **publique** et une adresse IP **privée**.  
Citez les trois plages d'adresses privées définies par la RFC 1918 et indiquez dans quel contexte chacune est typiquement utilisée.

**REP** : 
Une adresse IP privé est un adresse non routable sur internet. Elle représente les réseaux des entreprise ou des domiciles. 
On peut retrouver :
- 10.0.0.0/8 pour les grandes entreprises
- 172.16.0.0/12 pour les réseaux de tailles moyennes
- 192.168.0.0/16 pour les réseaux domestiques

---

## Partie 2 – Routage IP

**Question 4**  
Expliquez le **processus de décision de routage** d'un hôte lorsqu'il souhaite envoyer un paquet vers une destination.  
Quels critères lui permettent de savoir s'il doit passer par un routeur ou envoyer directement ?

**REP** :
- L'hôte va vérifié via le protocole ARP si il connait l'hôte et si il est dans sa plage réseau, si le destinataire ne fais pas partie de son réseau le paquet part à sa passerelle. Si la table de routage est bien configurée le routeur envoie le paquet au bon réseau si il est mitoyen ou à son Next-hop si le paquet doit passer par un autre routeur.

---

**Question 5**  
Voici une table de routage simplifiée d'un routeur Linux :

```
Destination        Passerelle        Interface
192.168.10.0/24    0.0.0.0           eth0
10.0.0.0/8         192.168.10.254    eth0
0.0.0.0/0          192.168.10.1      eth0
```

Un paquet à destination de `10.5.3.12` arrive sur ce routeur.

- Quelle route sera utilisée et pourquoi ?
- Que signifie l'entrée `0.0.0.0/0` dans cette table ?

**REP** :


---

**Question 6**  
Quelle est la différence entre le **routage statique** et le **routage dynamique** ?  
Donnez un exemple de cas d'usage pour chacun, et citez deux protocoles de routage dynamique en précisant leur champ d'application.

**REP** : 
- Le routage statique est le faite de faire la table de routage manuellement quand la dynamique permet qu'elle soit faite automatiquement 

---

**Question 7**  
Vous êtes technicien et devez ajouter une route statique persistante sur un serveur Linux pour atteindre le réseau `192.168.50.0/24` via la passerelle `10.0.0.1`.

- Donnez la commande permettant d'ajouter cette route **temporairement**.
- Comment rendre cette route **permanente** sur une distribution Linux moderne (type Debian/Ubuntu) ?

**REP** :


---

## Partie 3 – Protocoles de transport

**Question 8**  
Décrivez le mécanisme du **Three-Way Handshake TCP**.  
Quels sont les trois échanges réalisés, et quel est leur rôle dans l'établissement d'une connexion ?

---

**Question 9**  
Dans quels cas est-il préférable d'utiliser **UDP plutôt que TCP** ?  
Justifiez votre réponse en vous appuyant sur les caractéristiques techniques de chaque protocole.

---

## Partie 4 – Protocoles associés au réseau IP

**Question 10**  
Expliquez le rôle du protocole **ARP** dans la communication réseau.  
Quel problème résout-il, et comment fonctionne le mécanisme de requête/réponse ARP ?  
Quelle est l'équivalence de ce protocole en IPv6 ?

---

**Question 11**  
Un technicien utilise la commande `ping 8.8.8.8` et obtient un message `Destination Host Unreachable`.

- Quel protocole génère ce message ?
- Que peut indiquer ce type de message, et quelles étapes de diagnostic pourriez-vous effectuer ?

---

## Partie 5 – NAT et DHCP

**Question 12**  
Expliquez le fonctionnement du **PAT (Port Address Translation)**, également appelé NAPT ou NAT overload.  
Comment un routeur NAT est-il capable de faire correspondre les réponses aux bonnes machines internes lorsque plusieurs hôtes partagent la même IP publique ?

---

**Question 13**  
Décrivez les **4 étapes de la séquence DORA** utilisée par le protocole DHCP lors de l'attribution d'une adresse IP.  
Précisez pour chaque étape : l'émetteur, le destinataire, et le rôle du message.

---

## Partie 6 – IPv6 et DNS

**Question 14**  
Donnez l'écriture **simplifiée** de l'adresse IPv6 suivante :  
`2001:0DB8:0000:0000:0000:0000:0000:0001`  
Expliquez les deux règles de simplification applicables.  
De quel type d'adresse IPv6 s'agit-il (unicast, multicast, lien-local...) ? Justifiez votre réponse.

---

**Question 15**  
Expliquez le rôle du **DNS** dans l'exploitation d'un réseau IP.  
Décrivez le processus de résolution DNS lorsqu'un client tape `www.exemple.fr` dans son navigateur, en détaillant les différents acteurs impliqués (résolveur, serveur racine, TLD, serveur autoritaire).

---

_Fin du questionnaire – CP4 : Exploiter un réseau IP_