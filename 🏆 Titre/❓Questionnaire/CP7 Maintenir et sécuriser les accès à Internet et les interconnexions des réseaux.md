> **Formation** : TSSR  
> **Compétence** : CP7  
> **Thèmes couverts** : NAT/PAT, Pare-feu, DMZ, VPN (IPsec, TLS, OpenVPN), sécurisation des accès  
> **Consigne** : Répondez sans support dans un premier temps, puis nous corrigerons ensemble.

---

## Partie 1 – Notions fondamentales

> Niveau : Débutant

**1.** Qu'est-ce que le NAT ? Quel problème principal résout-il dans les réseaux IPv4 ?

---

**2.** Un pare-feu peut prendre trois décisions face à un paquet réseau. Citez-les et expliquez brièvement la différence entre chacune.

---

**3.** À quelle couche du modèle OSI fonctionne au minimum un pare-feu ? Que cela implique-t-il sur ce qu'il est capable d'analyser ?

---

**4.** Qu'est-ce qu'un VPN ? Quel est l'objectif principal de la mise en place d'un tunnel VPN ?

---

**5.** Quelle est la différence entre un réseau LAN et un réseau WAN ? Donnez un exemple concret pour chacun.

---

**6.** Citez les trois grandes familles de types de pare-feu vues en cours, en donnant pour chacune une caractéristique principale.

---

## Partie 2 – NAT et translation d'adresses

> Niveau : Intermédiaire

**7.** Expliquez la différence entre le NAT, le PAT/NAPT et le DNAT. Pour chaque type, donnez un cas d'usage concret en entreprise.

---

**8.** Dans le cadre du PAT/NAPT, une machine interne `192.168.1.50` souhaite accéder à un serveur web public `93.184.216.34:443`. Décrivez les deux grandes étapes (requête sortante et réponse entrante) en précisant ce que modifie le routeur NAT à chaque fois.

---

**9.** Le NAT introduit plusieurs limitations techniques. Citez-en au moins trois et expliquez pourquoi chacune est problématique.

---

**10.** Qu'est-ce que le DNAT (Destination NAT) ? Dans quel cas concret l'utilise-t-on ? Quel risque de sécurité implique-t-il ?

---

**11.** Pourquoi le NAT n'est-il pas nécessaire en IPv6 ? Qu'est-ce qui change fondamentalement entre IPv4 et IPv6 sur ce point ?

---

## Partie 3 – Filtrage réseau et architecture sécurisée

> Niveau : Intermédiaire

**12.** Quelle est la politique de filtrage recommandée pour un pare-feu : la liste d'autorisation ou la liste de blocage ? Justifiez votre réponse.

---

**13.** Définissez la notion de « défense en profondeur ». Pourquoi est-elle préférable à une simple défense périmétrique ?

---

**14.** Qu'est-ce qu'une DMZ ? Quels types de serveurs y place-t-on habituellement et pourquoi ?

---

**15.** Qu'est-ce qu'une matrice des flux ? À quoi sert-elle dans la gestion des zones de confiance d'un réseau d'entreprise ?

---

**16.** Quelle est la différence entre un pare-feu stateless et un pare-feu stateful ? Donnez un exemple de situation où le stateful apporte un avantage concret par rapport au stateless.

---

**17.** Qu'est-ce que le DPI (Deep Packet Inspection) ? Quel est son avantage majeur et sa limite principale ?

---

## Partie 4 – VPN : protocoles et mise en œuvre

> Niveau : Intermédiaire à avancé

**18.** Quels sont les trois types de VPN selon leur usage ? Pour chacun, précisez les extrémités du tunnel et un exemple d'utilisation typique.

---

**19.** IPsec est composé de trois protocoles. Nommez-les, précisez le rôle de chacun et indiquez lequel est généralement recommandé et pourquoi.

---

**20.** Expliquez la différence entre le mode transport et le mode tunnel dans IPsec. Dans quel contexte utilise-t-on chacun ?

---

**21.** Qu'est-ce qu'une SA (Security Association) dans IPsec ? Quel est le rôle du SPI ?

---

**22.** Quelles sont les versions de TLS actuellement recommandées ? Pourquoi les versions SSL et TLS 1.0/1.1 ne doivent-elles plus être utilisées ?

---

**23.** Quels sont les deux modes de fonctionnement d'OpenVPN (tun / tap) ? Quelle est la différence entre eux et lequel est recommandé ?

---

**24.** Quelle méthode d'authentification est recommandée pour un déploiement professionnel d'OpenVPN ? Expliquez brièvement comment elle fonctionne (PKI, certificats, AC).

---

## Partie 5 – Sécurisation avancée et bonnes pratiques

> Niveau : Avancé

**25.** Qu'est-ce que le PFS (Perfect Forward Secrecy) ? Dans quel contexte est-il appliqué et pourquoi est-ce important ?

---

**26.** Un administrateur réseau configure un pare-feu pour autoriser uniquement le trafic HTTP (port 80) et HTTPS (port 443) sortant depuis le réseau utilisateurs. Un employé parvient néanmoins à faire passer du trafic non autorisé via le port 443. Quel mécanisme cela exploite-t-il ? Quelle solution technique peut y remédier ?

---

**27.** Vous devez publier un serveur web interne sur Internet tout en limitant les risques. Décrivez l'architecture que vous mettriez en place (DMZ, pare-feu, NAT, règles de filtrage) en justifiant chaque choix.

---

**28.** Pourquoi ne faut-il pas faire confiance aveuglément à un VPN pour sécuriser entièrement un réseau ? Quelle approche complémentaire est recommandée ?

---

**29.** Un certificat d'une autorité de certification (AC) est compromis dans votre infrastructure OpenVPN. Quelles actions devez-vous entreprendre ? Quels mécanismes permettent de gérer la révocation de certificats ?

---

**30.** Comparez IPsec et OpenVPN sur les points suivants : couche OSI d'opération, usage principal, complexité de mise en œuvre, compatibilité NAT. Lequel choisiriez-vous pour un VPN d'accès distant multiplateforme et pourquoi ?

---

_Questionnaire réalisé dans le cadre de la préparation au titre TSSR – CP7_