# 🧠 Quiz 1 — Bases OSI & Encapsulation

  
## 🔹 Partie 1 — Identification de couche

  
Donner le **numéro de couche OSI** correspondant :

  

1. Adresse MAC =  couche 2

2. Port TCP =  couche 4

3. Adresse IP =  couche 3

4. Signal électrique =  couche 1

5. TLS =  couche 7

6. HTTP =  couche 7

  

---

  

## 🔹 Partie 2 — Équipements

  

Répondre par **l’équipement réseau concerné** :

  

7. Quel équipement regarde l’adresse IP pour décider où envoyer un paquet ?  

Le routeur

8. Quel équipement lit uniquement les adresses MAC ?  

Le switch

9. Quel équipement peut filtrer selon les ports ?  

  

---

  

## 🔹 Partie 3 — Encapsulation

  

10. Ordre correct d’encapsulation :

  

- A — Trame → Segment → Paquet → Bits  

- B — Segment → Paquet → Trame → Bits  

- C — Paquet → Segment → Bits → Trame  

Réponse A  

---

  

11. Quand un paquet arrive sur un routeur, il lit principalement :

  

- A — MAC  

- B — IP  

- C — Port  

  Réponse B

---

  

12. Quand un paquet arrive sur le serveur final, la première information exploitée par l’application est :

  

- A — MAC  

- B — Port  

- C — IP  

  La réponse C

---

  

## 🔹 Partie 4 — Vrai / Faux

  

13. Un switch L2 peut router des paquets  

FAUX

14. TCP fonctionne en couche 4  

VRAI

15. DNS utilise uniquement TCP  

FAUX

16. La couche physique comprend les câbles  

VRAI

---

  

# 🧪 Quiz 2 — Cas pratiques (niveau technicien)

  

## Situation 1

Un PC :

- reçoit une IP automatiquement  

- peut ping sa passerelle  

- ne peut pas ping 8.8.8.8  

  

Questions :

- Quelle couche suspecter en premier ?  

Couche 1 

- Quelle cause probable ?

Les machines ne sont pas connectées physiquement

  
---

## Situation 2

Un utilisateur :

- accède à un site par IP  

- mais pas par nom de domaine  

Questions :

- Quelle couche est concernée ?  

Couche 4

- Quel service est probablement en panne ?

Le service DNS à probablement un problème 

---
## Situation 3

Un serveur web :

- répond au ping  

- répond sur port 22  

- mais pas sur 443  

Questions :

- Quelle couche fonctionne ?  

Couche 3 et couche 4 : ICMP et SSH

- Quelle couche pose problème ?

Couche 7 

---

  

## Situation 4

  

Deux machines sur le même réseau :

- IP OK  

- masque OK  

- ping impossible  


Questions :

- Quelle couche inspecter en priorité ?  

Couche 1 

- Donner 2 causes possibles.

Les machines ne sont pas reliées ou l'une des deux n'est pas allumées

---

## Situation 5

Une machine Wi-Fi :

- connectée au réseau  

- IP correcte  

- Internet inaccessible  

Questions :

- Quelle couche investiguer en premier ?  

Couche 3

- Quel équipement suspecter ?

Le NAT n'est pas configuré ou il n'y a pas de route vers l'extérieur 

---

## Situation 6

Un switch L2 reçoit une trame avec une MAC destination inconnue.

Question :

- Que fait-il ?

Il lance une requête ARP en broadcast, pour trouver une réponse.

---

  

## Situation 7

Un paquet arrive sur un routeur.

- MAC destination = routeur  

- IP destination = serveur distant  

Question :

- Que fait le routeur ?

Si la route est dans sa table de routage, il peut envoyer le paquet au serveur

---

## Situation 8

- Un firewall bloque une connexion web.

Question :

- Quelle information regarde-il en priorité ?

Son adresse IP, si elle fait partis du pool des adresses à bannir il rejette la demande

---

## Situation 9

Un utilisateur dit :

* "Internet ne marche pas"

Question :

- Quelle est la première chose à vérifier ? (réponse opérationnelle)

---

## Situation 10

Un serveur répond :

- ping OK  

- DNS OK  

- telnet port 80 OK  

Mais le navigateur affiche :

-  connexion refusée  

Question :

- Quelle est l’hypothèse la plus probable ?



---

# 🔥 Quiz 3 — Diagnostic avancé (niveau entretien)

## Cas 1

Un poste :

- IP correcte  

- passerelle correcte  

- DNS correct  

- ping 1.1.1.1 OK  

- ping google.com KO  

Questions :

- Couche suspectée ?  

Couche 4 TCP DNS

- Test de confirmation ?



---
## Cas 2

Un serveur interne :

- répond localement  

- inaccessible depuis un autre VLAN  

Questions :

- Quelle couche est probablement en cause ?  

Couche 3

- Élément à vérifier sur le switch ?

Le routage si il s'agit d'un switch L3

---

## Cas 3

Un site HTTPS affiche :

- SSL handshake failed  

Questions :

- Couche concernée ?  

- Causes possibles ?



---
## Cas 4

Un client DHCP n’obtient pas d’adresse IP.

Questions :

- Quelle couche ?  

Couche 

- Protocole concerné ?  

- Ports utilisés ?



---

  

## Cas 5

  

Un service fonctionne en local (`localhost`) mais pas depuis une autre machine.

  

Questions :

- Couche suspecte ?  

- Cause typique ?

  

---

  

# 🧩 Exercice spécial — Encapsulation logique

  

On envoie une requête HTTPS vers un serveur distant.

  

Décrire :

  

1. Ce que contient le segment en couche 4  

2. Ce que contient le paquet en couche 3  

3. Ce que contient la trame en couche 2  

4. Ce que modifie le routeur lors du passage  

5. Ce que fait le switch  

6. À quel moment le chiffrement TLS intervient