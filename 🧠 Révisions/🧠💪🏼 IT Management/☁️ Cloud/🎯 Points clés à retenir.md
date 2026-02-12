### Concepts fondamentaux

1. ☁️ **Le Cloud Computing** = Services informatiques accessibles via Internet (stockage, calcul, réseau, applications)
    
2. 🔄 **Modèles de service** :
    - **IaaS** : Infrastructure virtualisée (serveurs, stockage)
    - **PaaS** : Plateforme de développement
    - **SaaS** : Applications en ligne par abonnement

3. 🌐 **Modèles de déploiement** :
    - **Cloud Public** : AWS, Azure, Google Cloud (mutualisé)
    - **Cloud Privé** : Infrastructure dédiée
    - **Cloud Hybride** : Combinaison des deux

### Virtualisation

4. 💻 **Cloud = Virtualisation + Automatisation + Réseau**

5. 🔧 **Hyperviseurs** :
    - Type 1 (Bare Metal) : ESXi, Hyper-V → Production
    - Type 2 (Hosted) : VirtualBox, VMware Workstation → Développement

6. 📦 **Conteneurisation** : Docker + Kubernetes pour portabilité et scalabilité

### Économie et tarification

7. 💰 **Modèles économiques** :
    - **Forfait fixe** : Coûts prévisibles
    - **Pay-as-you-go** : Flexibilité maximale mais coûts variables

8. 📊 **Optimisation des coûts** :    
    - Éteindre les ressources inutilisées
    - Right-sizing (bon dimensionnement)
    - Reserved Instances pour charges stables
    - Nettoyage régulier

9. ⚠️ **Attention** : L'entrée de données est gratuite, la sortie est facturée !

### Sécurité

10. 🔐 **Authentification Multi-Facteurs (MFA)** : Obligatoire pour les comptes admin

11. 👤 **RBAC** (Role-Based Access Control) : Principe du moindre privilège

12. 🌍 **Conformité** :
    - **RGPD** : Données en Europe
    - ISO 27001, SOC 2, HDS

13. 🔒 **Chiffrement** : TLS obligatoire pour le transit des données

### Architecture et disponibilité

14. 📈 **Scalabilité** :
    - **Verticale** : Augmenter les ressources d'un serveur
    - **Horizontale** : Ajouter des serveurs

15. 🏗️ **Haute disponibilité** :
    - Zones de disponibilité (protection contre pannes locales)
    - Régions géographiques (protection contre catastrophes régionales)

16. ⚡ **RTO et RPO** :
    - RTO = Temps max de restauration
    - RPO = Perte de données acceptable

17. ⚖️ **Load Balancing** : Distribution du trafic entre serveurs


### Outils et interfaces

18. 🖥️ **Interfaces d'accès** :
    - **SSH** : Ligne de commande
    - **Console Web** : Interface graphique
    - **API/CLI** : Automatisation

19. 🏗️ **Infrastructure as Code (IaC)** :
    - Terraform, ARM Templates, CloudFormation
    - Reproductibilité et versionnement

20. 📊 **Stockage** :
    - **Block Storage** : Disques pour VM
    - **Object Storage** : Fichiers (S3, Blob Storage)

### Bonnes pratiques

21. ✅ **Toujours** :
    - Activer MFA
    - Faire des sauvegardes
    - Surveiller les coûts
    - Appliquer le principe du moindre privilège
    - Chiffrer les données sensibles

22. ❌ **Ne jamais** :
    - Laisser des ressources inutilisées actives
    - Stocker des données sensibles sans chiffrement
    - Donner des droits administrateurs par défaut
    - Ignorer les alertes de sécurité

23. 🔍 **Surveillance** :
    - Métriques (CPU, RAM, réseau)
    - Logs (applications, système)
    - Alertes automatiques

### Troubleshooting

24. 🔧 **Problèmes courants** :
    - Connexion impossible → Vérifier Security Groups et IP publique
    - Application lente → Vérifier métriques CPU/RAM
    - Email non envoyé → Vérifier DNS (MX, SPF, DKIM)

### Avenir du Cloud

25. 🚀 **Tendances** :
    - Serverless (Functions as a Service)
    - Edge Computing
    - AI/ML as a Service
    - Multi-cloud et cloud souverain

---
