**Formation TSSR | Niveau 5 (Bac+2)**

---

> **Consignes** : Répondez aux questions de façon complète et rédigée. Aucune réponse à choix multiple. Justifiez vos réponses lorsque c'est demandé.

---

## 🔐 Partie 1 – VPN et sécurisation des accès distants

**Question 1** Expliquez ce qu'est un VPN et le concept de « tunnel ». Quels sont les trois objectifs principaux qu'un VPN doit garantir lors d'une communication ? Donnez un exemple concret d'utilisation en entreprise.

---

**Question 2** Décrivez les trois types de VPN selon leur topologie (Site à Site, Accès Distant, Point à Point). Pour chaque type, précisez les extrémités impliquées et un cas d'usage typique en entreprise.

---

**Question 3** Dans le protocole IPsec, deux protocoles de sécurité coexistent : **AH** et **ESP**.

- Quelle est la différence fondamentale entre ces deux protocoles ?
- Lequel est recommandé en production et pourquoi ?
- Quelle est la différence entre le mode **Transport** et le mode **Tunnel** d'IPsec ?

---

**Question 4** OpenVPN repose sur le protocole TLS. Expliquez :

- La différence entre le mode **TUN** (routeur) et le mode **TAP** (pont) d'OpenVPN.
- Lequel est recommandé pour un VPN d'accès distant professionnel et pourquoi ?
- Quel port est utilisé par défaut et quel protocole de transport est privilégié ?

---

## 🧱 Partie 2 – Filtrage réseau et pare-feu

**Question 5** Un pare-feu peut appliquer deux politiques de filtrage opposées. Décrivez ces deux approches, précisez laquelle est recommandée en sécurité et expliquez pourquoi l'autre est considérée comme risquée.

---

**Question 6** Comparez les trois types de pare-feux suivants en précisant leur niveau OSI de fonctionnement, leur capacité d'analyse et un cas d'usage adapté à chacun :

- Pare-feu **Stateless** (sans état)
- Pare-feu **Stateful** (avec état)
- Pare-feu **DPI / Applicatif** (Deep Packet Inspection)

---

**Question 7** Qu'est-ce qu'une **DMZ** (zone démilitarisée) dans une architecture réseau ? Expliquez pourquoi on l'utilise, quels types de serveurs on y place habituellement, et décrivez schématiquement le flux de trafic entre Internet, la DMZ et le réseau interne.

---

## ☁️ Partie 3 – Cloud Computing

**Question 8** Expliquez les trois modèles de service Cloud : **IaaS**, **PaaS** et **SaaS**. Pour chacun :

- Donnez la définition et un exemple concret de service.
- Précisez ce que gère le fournisseur et ce que gère le client.

---

**Question 9** Une entreprise hésite entre un Cloud **public**, **privé** et **hybride** pour héberger son infrastructure.

- Décrivez les caractéristiques de chaque modèle.
- Proposez un scénario d'usage justifié pour chaque modèle en fonction du type d'entreprise ou de données.

---

**Question 10** Dans une infrastructure Cloud, deux mécanismes de contrôle d'accès sont essentiels : le **MFA** et le **RBAC**.

- Définissez chacun de ces mécanismes.
- Expliquez le principe du moindre privilège et comment il s'applique dans le cadre du RBAC.
- Donnez un exemple concret de politique d'accès conditionnel.

---

## 🖥️ Partie 4 – Déploiement automatisé des postes de travail

**Question 11** Pourquoi ne peut-on pas simplement copier un poste de travail Windows configuré sur d'autres machines ? Quel outil Microsoft est indispensable avant toute capture d'image, et quelle est la commande à exécuter ? Expliquez ce que fait cet outil concrètement.

---

**Question 12** Comparez **WDS**, **MDT** et **SCCM** selon les critères suivants : coût, support du boot PXE natif, niveau de personnalisation et taille de parc recommandée. Pour quel contexte professionnel choisiriez-vous MDT + WDS plutôt que SCCM ?

---

**Question 13** Décrivez le processus de **démarrage PXE** (Pre-boot eXecution Environment) étape par étape, depuis l'allumage du poste jusqu'au démarrage de l'environnement de déploiement. Quels services réseau sont indispensables pour que ce mécanisme fonctionne ?

---

## 📋 Partie 5 – Journalisation et supervision

**Question 14** Sous Linux, deux systèmes de journalisation coexistent souvent : **syslog/rsyslog** et **systemd-journald**.

- Quelle est la différence fondamentale entre ces deux systèmes concernant le format et la persistance des logs ?
- Quelle commande permet de consulter les journaux systemd en temps réel ?
- Dans quel fichier les logs du noyau sont-ils généralement stockés sous syslog ?

---

**Question 15** Sous Windows, l'**Observateur d'événements** (Event Viewer) est l'outil central de journalisation.

- Citez et décrivez les trois journaux Windows principaux (Application, Sécurité, Système).
- Quels niveaux de gravité des événements connaissez-vous ?
- Dans un contexte de sécurité, quels types d'événements dans le journal **Sécurité** faut-il surveiller en priorité ? Donnez au moins deux exemples concrets.

---

_Questionnaire réalisé dans le cadre de la formation TSSR – CP4_ _Basé sur le référentiel RNCP37682 – Bloc 2 : Maintenir l'infrastructure et contribuer à son évolution et à sa sécurisation_