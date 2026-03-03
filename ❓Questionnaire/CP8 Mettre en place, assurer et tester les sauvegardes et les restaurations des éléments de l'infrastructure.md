> **Consignes** : Ce questionnaire est progressif. Les premières questions évaluent les connaissances fondamentales, les suivantes montent en complexité jusqu'au niveau TSSR. Répondez sans consulter vos cours dans un premier temps.

---

## PARTIE 1 – Concepts fondamentaux

_Niveau : Débutant_

**Question 1** Qu'est-ce qu'une sauvegarde ? Quelle est la différence fondamentale entre une sauvegarde et une simple copie de fichiers ?

---

**Question 2** Expliquez la règle **3-2-1** appliquée aux sauvegardes. Donnez un exemple concret d'application de cette règle dans une infrastructure d'entreprise.

---

**Question 3** Définissez les trois types de sauvegardes suivants et précisez pour chacun ses avantages et inconvénients :

- Sauvegarde complète (_full_)
- Sauvegarde incrémentale
- Sauvegarde différentielle

---

**Question 4** Qu'est-ce que le **RPO** (_Recovery Point Objective_) et le **RTO** (_Recovery Time Objective_) ? Illustrez chaque concept par un exemple chiffré.

---

**Question 5** Pourquoi dit-on que le RAID n'est **pas** une sauvegarde ? Citez au moins trois scénarios contre lesquels le RAID ne protège pas.

---

**Question 6** Qu'est-ce qu'un **snapshot** ? Dans quel(s) contexte(s) est-il utilisé, et quelle en est la limite principale en tant que solution de sauvegarde ?

---

## PARTIE 2 – Outils et mise en œuvre sous Linux

_Niveau : Intermédiaire_

**Question 7** Décrivez le fonctionnement de la commande `rsync`. Quelle option permet de ne transférer que les fichiers modifiés depuis la dernière synchronisation ? Donnez un exemple de commande pour sauvegarder `/var/www` vers un serveur distant `backup-srv` dans le répertoire `/sauvegardes/web`.

---

**Question 8** Vous devez créer une archive compressée de `/etc` avec `tar` et la stocker dans `/backup/etc_backup.tar.gz`. Écrivez la commande correspondante. Comment restaureriez-vous cette archive dans `/tmp/restauration` ?

---

**Question 9** Dans le cadre de LVM, expliquez le mécanisme **COW** (_Copy-On-Write_). Donnez la séquence de commandes permettant de :

1. Créer un snapshot de 5 Go du volume logique `/dev/vg_data/lv_data`
2. Monter ce snapshot en lecture seule sur `/mnt/snap`
3. Supprimer le snapshot une fois la sauvegarde terminée

---

**Question 10** Vous souhaitez automatiser une sauvegarde différentielle quotidienne avec `rsync`. Quelle approche et quels outils Linux utiliseriez-vous pour planifier cette tâche ? Citez la commande ou la configuration nécessaire.

---

**Question 11** Quelle est la différence entre une sauvegarde **à chaud** et une sauvegarde **à froid** ? Donnez un exemple d'utilisation justifiée pour chacune dans un environnement de production.

---

## PARTIE 3 – Outils et mise en œuvre sous Windows Server

_Niveau : Intermédiaire_

**Question 12** Qu'est-ce que le service **VSS** (_Volume Shadow Copy Service_) sous Windows ? Quel rôle joue-t-il dans le processus de sauvegarde ? Donnez un exemple de situation où il est indispensable.

---

**Question 13** Vous utilisez **Windows Server Backup** pour sauvegarder un serveur. Décrivez les étapes principales pour configurer une sauvegarde planifiée complète vers un partage réseau. Quelle commande `wbadmin` permet de lancer une sauvegarde complète manuellement ?

---

**Question 14** Un utilisateur a supprimé accidentellement un fichier sur un partage Windows géré par un serveur sous Windows Server. Quelle fonctionnalité Windows permet une restauration rapide sans passer par une sauvegarde complète ? Décrivez brièvement la procédure de restauration.

---

**Question 15** Dans le cadre d'une infrastructure **Hyper-V**, expliquez comment fonctionne le snapshot d'une machine virtuelle. Quels sont les risques liés à la conservation prolongée d'un snapshot de VM en production ?

---

## PARTIE 4 – Sauvegarde des services critiques

_Niveau : Avancé_

**Question 16** Expliquez pourquoi la sauvegarde d'un **Active Directory** est une opération critique et complexe. Quelle est la différence entre une restauration **authoritative** et une restauration **non-authoritative** d'un contrôleur de domaine ? Dans quel cas utilise-t-on chacune ?

---

**Question 17** Citez la commande `wbadmin` permettant de sauvegarder l'état du système (_System State_) sur un contrôleur de domaine Windows. Quels éléments sont inclus dans cette sauvegarde ?

---

**Question 18** Vous administrez un serveur Linux hébergeant une base de données **MySQL/MariaDB** en production. Expliquez pourquoi une copie directe des fichiers de la base n'est pas suffisante. Quelle commande ou outil utiliseriez-vous pour effectuer une sauvegarde cohérente de la base ? Donnez un exemple de commande.

---

**Question 19** Dans un environnement virtualisé avec **VMware ESXi**, citez deux méthodes pour sauvegarder une machine virtuelle. Quelle solution professionnelle est fréquemment utilisée en entreprise pour centraliser la gestion des sauvegardes de VMs ?

---

**Question 20** Un NAS est utilisé comme cible de sauvegarde dans votre infrastructure. Quels sont les avantages et les limites de cette approche ? Quand préférerait-on un **SAN** à un **NAS** pour le stockage des sauvegardes ?

---

## PARTIE 5 – Tests, validation et plan de reprise

_Niveau : Avancé_

**Question 21** Pourquoi est-il indispensable de **tester régulièrement** les restaurations ? Décrivez une procédure de test de restauration pour un serveur de fichiers Windows. Quels critères permettent de valider que la restauration est un succès ?

---

**Question 22** Quelle est la différence entre un **PCA** (_Plan de Continuité d'Activité_) et un **PRA** (_Plan de Reprise d'Activité_) ? Donnez un exemple concret pour chacun, en précisant les valeurs de RTO et RPO typiquement associées.

---

**Question 23** Vous êtes technicien TSSR dans une PME. Le responsable informatique vous demande de rédiger une **politique de sauvegarde**. Quels éléments essentiels doit-elle contenir ? (Listez et expliquez brièvement chaque point.)

---

**Question 24** Un serveur de production tombe en panne à 14h00. La dernière sauvegarde complète date de la veille à 23h00, et des sauvegardes incrémentales ont été effectuées à 6h00 et 12h00 le jour même. Décrivez l'ordre de restauration à suivre pour retrouver l'état le plus récent possible des données.

---

**Question 25** _(Mise en situation)_

Vous êtes appelé en urgence : un ransomware a chiffré les données d'un serveur de fichiers Windows. Les sauvegardes sont stockées sur un NAS connecté en permanence au réseau. Les sauvegardes sur le NAS sont également chiffrées.

- Quelles conclusions tirez-vous sur la politique de sauvegarde en place ?
- Quelles mesures auraient pu éviter cette situation ?
- Comment organisez-vous la reprise d'activité dans ce contexte ?

---

_Fin du questionnaire – CP8_

> **Note** : Ce questionnaire couvre les notions de sauvegarde et restauration au niveau TSSR : concepts fondamentaux, outils Linux et Windows, services critiques (AD, BDD, VMs), et démarche de reprise d'activité.