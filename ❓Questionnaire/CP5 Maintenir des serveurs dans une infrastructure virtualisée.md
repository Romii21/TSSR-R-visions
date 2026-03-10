### Questionnaire de préparation — Formation TSSR

> **Consignes** : Aucune réponse n'est fournie. Réponds par écrit à chaque question, en justifiant tes réponses lorsque c'est demandé. Les questions montent progressivement en difficulté.

---

## NIVEAU 1 — Fondamentaux de la virtualisation

**Q1.** Quelle est la différence fondamentale entre un hyperviseur de **Type 1** et un hyperviseur de **Type 2** ? Donne un exemple de chaque.

**R1.** Une hyperviseur de type 1 est une machine dédié à la virtualisation (barre-métal) plusieurs VM comme des serveurs sont dessus (Proxmox, VMWare EXI), l'hyperviseur de type 2 quand à lui marche comme un logiciel ou les VM sont stockées dedans (VMWare Workstation ou VirtualBox) dans les deux cas les VM consomme les ressources de la machines ou elles se trouves 

---

**Q2.** Cite trois avantages majeurs de la virtualisation de serveurs en entreprise.

**R2.** Moins de machines à entretenir, gestion rapide des ressources

---

**Q3.** Qu'est-ce qu'une **machine virtuelle (VM)** ? En quoi diffère-t-elle d'un serveur physique dédié ?

**R3.** C'est une machine qui fonctionne comme une machine réel mais sans corps physique, elle emprunte les ressources de la machine sur laquelle est tourne.

---

**Q4.** Qu'est-ce que **Proxmox VE** ? Sur quel(s) type(s) de virtualisation s'appuie-t-il ? Cite les deux technologies principales qu'il utilise.

**R4.** Il s'agit d'un hyperviseur de type 1 ou barre-métal

---

**Q5.** Quel est le **risque principal** lié au fait de faire tourner plusieurs VMs sur un seul serveur physique ? Comment peut-on atténuer ce risque ?

**R5.** Si le serveur tombe toutes les VMs tombent, il faut penser à la redondance 

---

**Q6.** Dans un environnement virtualisé, à quoi correspond le terme **"overhead"** ? Quel est son impact sur les performances ?

**R6.** C'est quand les VMs utilisent plus de ressource que ne le permette la machine sur laquelle elle se trouve, perte de performance principalement 

---

**Q7.** Cite deux formats de fichiers couramment utilisés pour les **disques virtuels** (images de disques). À quel hyperviseur chacun est-il associé ?

**R7.** 

---

**Q8.** Quelle est la différence entre un **snapshot** et une **sauvegarde complète** d'une machine virtuelle ? Ces deux mécanismes sont-ils interchangeables ?

**R8.** Le snapshot est une fonction qui permet de revenir en arrière à l'endroit où a été fait le snap, une sauvegarde complète permet d'enregistrer l'intégralité des donnés de la VM

---

## NIVEAU 2 — Administration d'une infrastructure virtualisée

**Q9.** Tu dois créer une nouvelle VM sous Proxmox VE. Cite au moins **cinq paramètres** que tu devras définir lors de la création.

**R9.** 

- Son nom
- Son ID
- La taille de son disque dur
- La RAM
- La carte réseau

---

**Q10.** Explique ce qu'est un **template (modèle)** de VM dans Proxmox. Quel est l'intérêt d'en utiliser plutôt que de créer chaque VM from scratch ?

**R10.** Cela permet d'avoir ces paramètre déjà prédéfini à chaque démarrage

---

**Q11.** Quelle est la différence entre un **clone complet** et un **clone lié** d'une VM ? Dans quel cas privilégieras-tu l'un ou l'autre ?

**R11.** 

---

**Q12.** Dans Proxmox, il existe plusieurs types de réseau virtuel. Explique la différence entre un **bridge Linux** et un **vSwitch**. Quel rôle joue le bridge `vmbr0` dans une configuration standard ?

**R12.** 

---

**Q13.** Qu'est-ce qu'un **VLAN** dans un contexte de virtualisation ? Comment permet-il d'isoler le trafic entre différentes VMs sur un même hyperviseur ?

**R13.** 

---

**Q14.** Tu constates qu'une VM consomme beaucoup plus de RAM que prévu et ralentit les autres VMs. Quels outils ou commandes utiliserais-tu pour **diagnostiquer** ce problème, et quelles actions correctives pourrais-tu envisager ?

**R14.** 

---

**Q15.** Explique le principe du **ballooning mémoire** dans la virtualisation. Quel en est l'intérêt dans la gestion des ressources partagées ?

**R15.** 

---

**Q16.** Dans le cadre de la gestion du stockage virtualisé, qu'est-ce que **LVM** ? Décris son architecture en trois couches (PV, VG, LV).

**R16** 

---

**Q17.** Un administrateur te demande de **réduire la taille d'un volume logique** (LV). Quels sont les risques liés à cette opération et quelle précaution absolue dois-tu prendre avant de la réaliser ?

---

**Q18.** Quelle est la différence entre un stockage **NAS** et un stockage **SAN** ? Lequel est le plus adapté pour héberger des disques de machines virtuelles et pourquoi ?

---

**Q19.** Tu dois mettre en place un **RAID 5** sur un serveur disposant de 4 disques de 2 To chacun. Quelle sera la **capacité utile** totale ? Combien de pannes disque ce niveau RAID peut-il tolérer ?

**R19.** Au total 8 To de capacité total comme il y a de la redondance et il pourra supporter une panne

---

**Q20.** Quelle commande Linux permet de **vérifier l'état d'une grappe RAID logicielle** gérée par `mdadm` ? Qu'est-ce que le fichier `/proc/mdstat` contient ?

---

## NIVEAU 3 — Haute disponibilité, maintenance et sécurité

**Q21.** Explique la différence entre la **migration à chaud (live migration)** et la **migration à froid** d'une VM. Quelles sont les conditions techniques nécessaires pour réaliser une migration à chaud ?

---

**Q22.** Qu'est-ce qu'un **cluster Proxmox** ? Combien de nœuds minimum faut-il pour former un cluster fonctionnel ? Quel rôle joue le **quorum** dans ce contexte ?

---

**Q23.** Dans un cluster Proxmox, qu'est-ce que la fonctionnalité **HA (High Availability)** ? Décris ce qui se passe concrètement lorsqu'un nœud du cluster tombe en panne.

**R23.** Un cluster permet la redondance sur les deux nœuds sont synchronisés, si le premier tombe le second prend le relais sans que l'architecture ne soit impacté

---

**Q24.** Explique les notions de **RTO** (Recovery Time Objective) et **RPO** (Recovery Point Objective). Comment ces deux indicateurs influencent-ils la stratégie de sauvegarde d'une infrastructure virtualisée ?

---

**Q25.** Quelle est la **règle 3-2-1** en matière de sauvegarde ? Comment l'appliquer dans un environnement virtualisé avec Proxmox ?

---

**Q26.** Dans Proxmox, qu'est-ce que **PBS (Proxmox Backup Server)** ? Quels avantages offre-t-il par rapport à une sauvegarde classique par copie de fichier ?

---

**Q27.** Un administrateur applique une mise à jour sur le nœud hyperviseur pendant les heures de production. Quels risques cela comporte-t-il et quelle procédure devrais-tu suivre pour minimiser l'impact sur les services ?

**R27.** 

---

**Q28.** Tu remarques qu'un disque d'une grappe **RAID 5** est signalé comme **dégradé**. Décris la procédure complète que tu dois suivre pour remplacer le disque défaillant et reconstruire la grappe, en précisant les commandes `mdadm` utilisées.

**R29.** je vérifie quel disque est endommagé, je le démonte, puis je créer un nouveau disque de mémoire équivalente, je le formate pour du RAID 5 puis je l'intègre dans la grappe 

---

**Q29.** Dans le contexte de la sécurité d'une infrastructure virtualisée, qu'est-ce que l'**isolation des VMs** ? Quelles menaces cette isolation permet-elle de contenir, et quelles sont ses limites ?

---

**Q30.** Explique ce qu'est un **conteneur LXC** et en quoi il diffère d'une **machine virtuelle KVM**. Dans quels cas est-il plus pertinent d'utiliser un conteneur plutôt qu'une VM complète, et dans quels cas vaut-il mieux éviter les conteneurs ?

---

## BONUS — Mise en situation

**Q31. — Scénario** Ton entreprise dispose d'un unique serveur physique sous Proxmox hébergeant 8 VMs de production (serveur AD, serveur de fichiers, serveur web, serveur de messagerie, etc.). Le DSI te demande un plan pour **améliorer la résilience** de cette infrastructure avec un budget limité.

Propose une architecture améliorée en justifiant chaque choix technique (clustering, stockage partagé, HA, sauvegarde). Quels seraient selon toi les premiers points à adresser en priorité ?

---

_Questionnaire réalisé dans le cadre de la préparation au titre TSSR — CP5_ _Corrigé à remettre lors de la séance suivante_