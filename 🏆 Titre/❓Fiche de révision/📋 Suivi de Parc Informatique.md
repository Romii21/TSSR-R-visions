## 1. Définition

> Un **parc informatique** = l'ensemble des ressources **matérielles et logicielles** d'un système informatique.

|Catégorie|Exemples|
|---|---|
|Ordinateurs|PC fixes, portables, écrans, claviers, souris|
|Réseau|Switch, bornes Wi-Fi, firewall, câblages|
|Mobiles|Smartphones, tablettes, scanners code-barres|
|Périphériques|Imprimantes, copieurs, disques durs externes|
|Logiciels|OS, applications + licences associées|

---

## 2. Les 3 axes de gestion

### 🔧 Entretenir

- **Inventaire** : découverte logicielle (FusionInventory, SCCM, NextThink), recensement humain, bons de commande
- **CMDB** : base de données de configuration (GLPI, Ivanti LanDesk)
- **Maintenance préventive** : mensuelle/annuelle, matérielle ou fonctionnelle
- **Dépannage** classé par criticité : standard → bloquant → urgent (via Help-Desk)

### 📈 Développer

- **Cycle de renouvellement** matériel selon un cycle de vie prédéfini
- **Turn-over** à l'achat : tous les 4–6 ans (⅓ ou ¼ du parc)
- **Leasing** : durée standard 3 ans
- **Budget** : planification sept–nov, prévision sur N+3 ans
- Charte informatique obligatoire

### ⚡ Optimiser

- Outils de sécurité + veille
- Formation utilisateurs (RGPD, bonnes pratiques)
- Conformité matérielle et logicielle (méthode 5M)
- Gestion des prestataires (limiter les accès techniques)

---

## 3. Cycles de vie matériel

|Équipement|Durée recommandée|
|---|---|
|PC fixe|5 ans|
|PC portable|3 ans|
|Serveur|5 ans|
|Écran/Périphérique|3–5 ans|

---

## 4. Méthodes & Outils

### 🗄️ Logiciels de gestion de parc (CMDB)

|Logiciel|Type|
|---|---|
|**GLPI**|Open source — inventaire + helpdesk|
|**Microsoft SCCM**|Sous licence — déploiement + inventaire|
|**Ivanti LanDesk**|Sous licence — gestion avancée|

### 📱 MDM — Mobile Device Management

- Gestion des appareils mobiles (smartphones, tablettes, IoT, industriels)
- **Nature active** : intervention en temps réel
- Fonctions : MAJ système, politiques de sécurité, déploiement d'apps, géolocalisation
- Outils : **IBM MaaS360**, **MobileIron**

### 📊 Différence GLPI vs MDM

||GLPI|MDM|
|---|---|---|
|Nature|Statique (passive)|Dynamique (active)|
|Intervention|Différée|Temps réel|
|Usage|Inventaire + helpdesk|Gestion mobile|

---

## 5. Méthode 5M (Ishikawa)

> Diagramme en arête de poisson — analyse causes/effets d'un problème.

|M|Contenu|
|---|---|
|**Main d'œuvre**|Personnel, savoir-faire|
|**Matériel**|Outils, installations|
|**Méthode**|Procédures, référentiels|
|**Matière**|Composants, énergie|
|**Milieu**|Locaux, ambiance, hiérarchie|

---

## 6. Données conservées en inventaire

**Pour les ordinateurs :**

- Identification : nom, code-barre, S/N
- Réseau : @IP, @MAC
- Composants : CPU, RAM, disque dur
- Logiciels : OS, pilotes
- Domaine : OU, domaine lié
- Statut : production / stock / réparation
- Budget : prix, date achat/livraison

**Pour les périphériques :** nom, marque, modèle  
**Pour les logiciels :** nom, licence, cible utilisateurs

---

## 7. Points clés à retenir

- ✅ La gestion de parc = **entretenir + développer + optimiser**
- ✅ Toujours stocker les données dans une **CMDB**
- ✅ Prévoir un **cycle de vie** et un **budget** par appareil
- ✅ **GLPI** = gestion passive | **MDM** = gestion active des mobiles
- ✅ La méthode **5M** permet d'analyser les causes d'un dysfonctionnement