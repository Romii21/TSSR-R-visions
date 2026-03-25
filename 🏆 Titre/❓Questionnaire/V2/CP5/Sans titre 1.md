# Questionnaire – CP5 : Maintenir des serveurs dans une infrastructure virtualisée

**Formation** : TSSR  
**Niveau** : Bac+2 (Technicien Supérieur Systèmes et Réseaux)  
**Durée conseillée** : 45 à 60 minutes  
**Consigne** : Répondez aux questions de manière développée. Les réponses courtes ou non justifiées ne seront pas acceptées.

---

## Partie 1 – Concepts de la virtualisation

**Question 1**  
Expliquez la différence entre un hyperviseur de **type 1** et un hyperviseur de **type 2**. Donnez deux exemples de chaque type et précisez dans quel contexte chacun est utilisé en entreprise.

---

**Question 2**  
Qu'est-ce que la **consolidation de serveurs** ? Expliquez son intérêt économique et technique, et décrivez une limite importante à prendre en compte lors de sa mise en place.

---

**Question 3**  
Expliquez la différence entre une **machine virtuelle (VM)** et un **conteneur** (de type Docker ou LXC). Dans quel cas préfèrerait-on l'un ou l'autre dans un contexte professionnel ?

---

## Partie 2 – Stockage dans un environnement virtualisé

**Question 4**  
Vous devez mettre en place un système de stockage tolérant aux pannes pour un serveur hébergeant plusieurs machines virtuelles. Décrivez le principe du **RAID 5** : son fonctionnement, ses avantages, ses limites et le nombre minimum de disques requis.

---

**Question 5**  
Un collègue vous dit : _"J'ai mis en place un RAID 1 sur mon serveur, donc je n'ai plus besoin de faire de sauvegardes."_  
Que lui répondez-vous ? Justifiez précisément votre réponse en citant au moins deux situations que le RAID ne peut pas couvrir.

---

**Question 6**  
Expliquez l'architecture de **LVM** (Logical Volume Manager) en décrivant les trois niveaux qui la composent. Quel avantage majeur apporte LVM par rapport au partitionnement traditionnel dans un environnement de serveur virtualisé ?

---

**Question 7**  
Qu'est-ce qu'un **snapshot LVM** ? Quel mécanisme technique utilise-t-il pour fonctionner ? Décrivez un cas d'usage concret dans le cadre de la maintenance d'une VM.

---

## Partie 3 – Haute disponibilité et continuité de service

**Question 8**  
Définissez les notions de **RTO** (Recovery Time Objective) et de **RPO** (Recovery Point Objective). Expliquez en quoi ces deux indicateurs sont importants lors de la planification d'un plan de reprise d'activité (PRA) dans une infrastructure virtualisée.

---

**Question 9**  
Qu'est-ce que la **migration à chaud** (live migration) d'une machine virtuelle ? Expliquez son fonctionnement général et dans quel contexte cette fonctionnalité est particulièrement utile pour la maintenance des serveurs.

---

**Question 10**  
Décrivez le principe de la **haute disponibilité (HA)** dans un cluster d'hyperviseurs. Que se passe-t-il concrètement en cas de défaillance d'un nœud physique ? Quel est le rôle du stockage partagé dans ce mécanisme ?

---

## Partie 4 – Stockage réseau et administration

**Question 11**  
Quelle est la différence entre un **NAS** (Network Attached Storage) et un **SAN** (Storage Area Network) ? Lequel est le plus adapté pour héberger des disques virtuels de machines virtuelles, et pourquoi ?

---

**Question 12**  
Dans le cadre de Proxmox VE, comment sont gérés les **pools de stockage** ? Citez deux types de stockage supportés nativement par Proxmox et expliquez dans quel cas chacun est utilisé.

---

## Partie 5 – Sécurité et maintenance

**Question 13**  
Lors de la maintenance d'un serveur dans une infrastructure virtualisée, quelles **précautions** devez-vous prendre avant d'effectuer une mise à jour majeure de l'hyperviseur ? Décrivez la procédure à suivre.

---

**Question 14**  
Expliquez ce qu'est le principe du **moindre privilège** appliqué à l'administration d'une infrastructure virtualisée. Comment ce principe peut-il être mis en œuvre concrètement dans Proxmox ou dans un environnement VMware ?

---

**Question 15**  
Un disque appartenant à une grappe **RAID 5** tombe en panne sur un serveur de production hébergeant des VMs. Décrivez, étape par étape, la procédure de **reconstruction** du RAID et les vérifications à effectuer pour s'assurer que le service est rétabli correctement.

---

_Fin du questionnaire – Bonne chance !_