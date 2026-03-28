## 🔑 Concepts fondamentaux

|Terme|Définition|
|---|---|
|**Incident**|Événement imprévu perturbant un SI → diminution de la QoS|
|**Problème**|Cause inconnue d'un ou plusieurs incidents|
|**ITIL**|Cadre méthodologique pour l'ITSM (IT Service Management)|
|**Helpdesk / Support SI**|Centre de service d'informaticiens gérant incidents et demandes|
|**SLA**|Contrat définissant les niveaux de service attendus|

> ⚠️ **Incident ≠ Problème** : l'incident est l'_effet_, le problème est la _cause_

---

## 🏗️ Organisation du support SI

**Niveaux de support :**

- **N0** — Enregistrement, catégorisation
- **N1** — N0 + priorisation, résolution par procédures
- **N2** — Analyse approfondie, résolution, suivi
- **N3** — Expertise, résolution complexe, suivi

---

## 🔄 Procédure de gestion des incidents (8 étapes)

```
1. Identification / Détection
2. Notification
3. Enregistrement (ticket → BDD)
4. Catégorisation & Priorisation
5. Diagnostic & Investigation
6. Suivi (ou escalade)
7. Résolution + Documentation
8. Clôture
```

---

## 🎯 Catégorisation & Priorisation

### Niveaux de gravité

|Niveau|Description|
|---|---|
|**Faible**|Utilisateur peut continuer à travailler|
|**Normal**|Gênant mais contournable (solution de contournement)|
|**Urgent**|Utilisateur bloqué|
|**Critique**|Utilisateur ne peut plus travailler|

### Niveaux d'impact

`Utilisateur` → `Service` → `Site` → `Entreprise`

### Priorités

|Priorité|Condition|
|---|---|
|🟢 **Mineure**|Faible impact, résolution non urgente|
|🟠 **Majeure**|Gênant, intervention rapide requise|
|🔴 **Critique**|Bloquant → procédure de gestion de crise|

---

## 🔍 Démarche de diagnostic

1. **Recueil d'informations** — logs, interviews, supervision (SolarWinds, PRTG, Wireshark)
2. **Analyse** — chaîne de valeur (modèle OSI), diagramme d'Ishikawa
3. **Test des hypothèses** — tests de régression, de charge, de sécurité
4. **Détermination de la cause** ⚠️ précision essentielle (une mauvaise cause = coûts inutiles)
5. **Planification de la résolution** — selon priorité, ressources, impact utilisateur

---

## 📡 Outils & Protocoles associés

|Outil / Protocole|Usage|
|---|---|
|**Wireshark**|Analyse réseau, capture de paquets|
|**PRTG / SolarWinds**|Supervision réseau et systèmes|
|**SNMP (UDP 161/162)**|Supervision des équipements réseau|
|**Syslog (UDP 514)**|Collecte des journaux d'événements|
|**ICMP**|Diagnostic réseau (ping, traceroute)|
|**SSH (TCP 22)**|Accès distant sécurisé pour intervention|
|**RDP (TCP 3389)**|Prise en main à distance (Windows)|
|**HTTP/S (80/443)**|Accès aux portails helpdesk web|
|**SMTP (25/587)**|Notifications d'incidents par mail|
|**LDAP (389) / LDAPS (636)**|Authentification utilisateurs (AD)|

---

## 📌 À retenir absolument

- Un **problème peut être latent** (pas encore déclenché d'incident) → importance de la prévention
- La **documentation** post-résolution est cruciale (wiki, FAQ, CMDB)
- La **communication utilisateur** valide la résolution au-delà des métriques internes
- Le **SLA** définit temps de réponse, disponibilité et qualité attendus