# 📋 Questionnaire — CP7 : Maintenir et sécuriser les accès à Internet et les interconnexions des réseaux

**Niveau** : TSSR  
**Nombre de questions** : 15  
**Thèmes couverts** : NAT/PAT, Pare-feu, DMZ, VPN, IPsec, TLS, OpenVPN, Sécurisation des accès distants

---

## 🔹 Partie 1 — NAT et gestion des adresses

**Question 1**  
Expliquez la différence entre le **SNAT (PAT/NAPT)** et le **DNAT**. Dans quel cas d'usage concret utilise-t-on chacun d'eux ?

---

**Question 2**  
Un administrateur souhaite publier un serveur web interne (192.168.1.10:80) vers l'extérieur. Quelle technique NAT doit-il mettre en place et sur quel équipement est-elle généralement configurée ?

---

**Question 3**  
Citez **deux inconvénients techniques** du NAT qui peuvent poser problème dans un environnement d'entreprise, et expliquez pourquoi.

---

## 🔹 Partie 2 — Pare-feu et politique de filtrage

**Question 4**  
Quelle est la différence entre un pare-feu **stateless** et un pare-feu **stateful** ? Quel est l'avantage principal du stateful pour l'écriture des règles ?

---

**Question 5**  
Qu'est-ce que l'inspection **DPI (Deep Packet Inspection)** ? Citez une limite importante de cette technique.

---

**Question 6**  
En matière de politique de filtrage, deux approches existent : la **liste de blocage** (_blacklist_) et la **liste d'autorisation** (_whitelist_). Laquelle est recommandée et pourquoi ?

---

**Question 7**  
Lors de la configuration d'un pare-feu, un paquet peut recevoir trois traitements différents : `Accept`, `Drop` ou `Reject`. Expliquez la différence entre `Drop` et `Reject`, et précisez lequel est généralement préféré en production.

---

**Question 8**  
Qu'est-ce qu'une **DMZ (Zone Démilitarisée)** dans une architecture réseau ? Quel type de serveurs y place-t-on et pourquoi ?

---

**Question 9**  
Expliquez le principe de **défense en profondeur** (_defense in depth_). En quoi est-il supérieur à une simple défense périmétrique ?

---

## 🔹 Partie 3 — VPN et protocoles de sécurité

**Question 10**  
Quels sont les **trois types de VPN** selon les cas d'usage ? Donnez un exemple concret pour chacun.

---

**Question 11**  
IPsec repose sur deux protocoles de protection des données : **AH** et **ESP**. Quelle est la différence entre ces deux protocoles, et lequel est recommandé dans la pratique ?

---

**Question 12**  
IPsec peut fonctionner en **mode transport** ou en **mode tunnel**. Décrivez la différence et indiquez lequel est utilisé pour les VPN site-à-site.

---

**Question 13**  
Quelles sont les versions de **TLS** considérées comme **obsolètes** et ne devant plus être utilisées ? Quelle version est aujourd'hui recommandée et pourquoi ?

---

**Question 14**  
**OpenVPN** peut fonctionner en mode **tun** ou en mode **tap**. Expliquez la différence entre ces deux modes et précisez lequel est recommandé pour un VPN d'accès distant classique.

---

## 🔹 Partie 4 — Sécurisation globale des accès

**Question 15**  
Qu'est-ce qu'un **serveur bastion** (ou jump server) ? Quel est son rôle dans la sécurisation des accès d'administration, et quels risques doit-on surveiller lors de sa mise en place ?

---

_Bonne chance ! Renvoyez vos réponses pour la correction._