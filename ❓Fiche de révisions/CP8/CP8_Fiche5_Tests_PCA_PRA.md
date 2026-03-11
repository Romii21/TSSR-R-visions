# 📋 FICHE 5 – Tests, PCA/PRA et Politique de Sauvegarde
### CP8 – Mettre en place, assurer et tester les sauvegardes et restaurations

---

## 1. Pourquoi tester les restaurations

### La règle d'or

> **Une sauvegarde non testée est une sauvegarde dont on ne sait pas si elle fonctionne.**

Les causes de défaillance d'une sauvegarde sont nombreuses et silencieuses :

| Cause | Description |
|-------|-------------|
| **Corruption silencieuse** | Le support de stockage a développé des erreurs sans alarme visible |
| **Procédure obsolète** | L'infrastructure a changé (migration serveur, changement d'OS) mais la procédure de restauration n'a pas été mise à jour |
| **Espace insuffisant** | La destination de restauration n'a pas la capacité nécessaire |
| **Droits insuffisants** | Le compte de service de sauvegarde n'a plus les droits nécessaires |
| **Dépendances manquantes** | Pilotes, logiciels, licences absents sur la machine de restauration |

> 💡 **La seule preuve qu'une sauvegarde est valide, c'est de l'avoir restaurée avec succès.**

### Procédure de test de restauration – Serveur de fichiers Windows

```
1. PRÉPARATION
   - Identifier la sauvegarde à tester (date, version)
   - Préparer un environnement isolé (VM de test, réseau de quarantaine)
   - Ne jamais tester en production directement

2. EXÉCUTION
   - Restaurer la sauvegarde dans l'environnement isolé
   - Commande : wbadmin start recovery -version:... -recoveryTarget:D:\test\

3. VÉRIFICATION
   - Vérifier la présence et l'intégrité des fichiers (arborescence complète)
   - Vérifier les permissions NTFS sur les dossiers clés
   - Tester l'accès depuis un poste client de test (partage réseau simulé)
   - Vérifier que les applications utilisant ces fichiers fonctionnent

4. VALIDATION ET DOCUMENTATION
   - Mesurer la durée totale de restauration → comparer au RTO défini
   - Documenter le résultat (succès / échec, durée, observations)
   - Archiver le rapport de test
```

### Critères de validation d'une restauration réussie

- ✅ Les données restaurées sont **complètes** (même arborescence, aucun fichier manquant)
- ✅ Les **droits et permissions** sont corrects (héritage NTFS respecté)
- ✅ La **durée de restauration** est compatible avec le RTO défini
- ✅ Les **applications dépendantes** fonctionnent correctement avec les données restaurées
- ✅ Aucun **message d'erreur** pendant le processus de restauration

### Fréquence recommandée des tests

| Type de test | Fréquence | Périmètre |
|---|---|---|
| Test de restauration de fichiers | Mensuelle | Quelques fichiers/dossiers aléatoires |
| Test de restauration complète | Trimestrielle | Un serveur entier en environnement isolé |
| Test de reprise d'activité (PRA complet) | Annuelle | Simulation de sinistre majeur |

---

## 2. PCA et PRA

### Définitions

#### PCA – Plan de Continuité d'Activité

> L'objectif du PCA est de **maintenir l'activité sans interruption perceptible**, même en cas d'incident grave.

- Mise en œuvre **proactive** (avant l'incident)
- Nécessite une infrastructure **redondante** (coût élevé)
- RTO ≈ quelques secondes à quelques minutes
- RPO ≈ quasi-zéro (réplication synchrone)

#### PRA – Plan de Reprise d'Activité

> L'objectif du PRA est de **reprendre l'activité après une interruption**, dans un délai acceptable.

- Mise en œuvre **réactive** (après l'incident)
- Moins coûteux que le PCA
- RTO = quelques heures
- RPO = quelques heures à 24h

### Tableau comparatif PCA / PRA

| Critère | PCA | PRA |
|---------|-----|-----|
| **Objectif** | Continuité sans interruption | Reprise après interruption |
| **Interruption** | Nulle ou imperceptible | Interruption acceptée |
| **RTO typique** | < 1 minute | 2h à 8h |
| **RPO typique** | Quasi-zéro | 1h à 24h |
| **Coût** | Très élevé | Modéré |
| **Infrastructure** | Double site actif/actif | Site de secours passif ou cloud |
| **Déclenchement** | Automatique (failover) | Manuel ou semi-automatique |

### Exemples concrets

**Exemple PCA :**
Un site e-commerce avec deux datacenters en **actif/actif** géographiquement distincts. Les données sont répliquées de façon synchrone entre les deux sites. Si le DC principal tombe, le load balancer redirige automatiquement le trafic vers le second site en quelques secondes. RTO ≈ 30s, RPO ≈ 0.

**Exemple PRA :**
Une PME avec un serveur de fichiers sauvegardé toutes les nuits sur un NAS, avec une copie hebdomadaire sur le cloud. En cas de sinistre (incendie), l'entreprise restaure les données sur un nouveau serveur depuis la sauvegarde cloud. RTO = 4h, RPO = 24h (perte d'une journée de données).

### Schéma de positionnement

```
                         Temps de reprise (RTO)
Aucune ←────────────────────────────────────→ Long

Haute disponibilité    PCA                  PRA
(Cluster, RAID, HA) → (Failover auto) →  (Restauration manuelle)

    RTO ≈ 0s          RTO < 1 min        RTO = quelques heures
    RPO ≈ 0           RPO ≈ 0            RPO = dernière sauvegarde
    Coût ★★★★★        Coût ★★★★          Coût ★★
```

---

## 3. Politique de sauvegarde

### Définition

Une **politique de sauvegarde** est un document formel qui définit les règles, procédures et responsabilités liées à la protection des données d'une organisation.

### Les 10 éléments essentiels

#### 1. Périmètre et classification des données

> Quelles données sont sauvegardées ? Avec quelle priorité ?

- Inventaire des serveurs, bases de données, partages, postes utilisateurs
- Classification par criticité (critique / important / standard)

#### 2. Types et fréquences de sauvegarde

> Quel type de sauvegarde, à quelle fréquence, pour chaque ressource ?

| Ressource | Type | Fréquence |
|-----------|------|-----------|
| Serveur AD | System State | Quotidien |
| Serveur de fichiers | Complète + Incrémentale | Hebdo + Quotidien |
| Base de données | Dump + Log | Quotidien + Toutes les heures |

#### 3. Durée de rétention

> Combien de temps les sauvegardes sont-elles conservées ?

- Exemple : 30 jours pour les incrémentales, 1 an pour les complètes mensuelles
- Contraintes légales à prendre en compte (RGPD, obligations comptables : 10 ans)

#### 4. Supports et destinations (Règle 3-2-1)

> Où et sur quoi sont stockées les sauvegardes ?

- NAS local, cloud, bandes, disque hors ligne
- Application obligatoire de la règle **3-2-1**

#### 5. RPO et RTO par niveau de criticité

| Niveau | RPO | RTO |
|--------|-----|-----|
| Critique (AD, ERP) | < 1h | < 2h |
| Important (messagerie) | < 4h | < 4h |
| Standard (fichiers bureautiques) | 24h | 8h |

#### 6. Responsabilités

> Qui fait quoi ?

- Responsable de la configuration et du suivi des jobs de sauvegarde
- Responsable des tests de restauration
- Responsable de la validation et de la signature de la politique

#### 7. Procédure de test de restauration

> Comment et quand tester ?

- Fréquence des tests (mensuelle / trimestrielle / annuelle)
- Méthode et environnement de test
- Critères de validation
- Rapport de test obligatoire

#### 8. Sécurité des sauvegardes

> Comment protéger les sauvegardes elles-mêmes ?

- **Chiffrement** des sauvegardes (AES-256 minimum)
- **Accès restreint** aux supports de sauvegarde (comptes dédiés, pas les mêmes que la production)
- Sauvegardes **hors ligne ou immuables** pour se protéger des ransomwares

#### 9. Surveillance et alertes

> Comment savoir si une sauvegarde a échoué ?

- Supervision des jobs de sauvegarde (notification par email en cas d'échec)
- Tableaux de bord (Veeam, Zabbix, etc.)
- Consultation quotidienne des rapports de sauvegarde

#### 10. Procédure de restauration documentée

> Comment restaurer, étape par étape, pour chaque type de serveur ?

- Procédures rédigées et à jour
- Accessibles même si le SI est en panne (version papier ou hors ligne)
- Testées et validées régulièrement

---

## 4. Ordre de restauration avec sauvegardes mixtes

### Cas type : Complète + Incrémentales

```
Lundi 23h    → [COMPLÈTE]         (base)
Mardi 6h     → [INCRÉMENTALE 1]   (modifs depuis lundi 23h)
Mardi 12h    → [INCRÉMENTALE 2]   (modifs depuis mardi 6h)
Mardi 14h    → INCIDENT

Ordre de restauration :
  1. Restaurer la COMPLÈTE du lundi 23h
  2. Appliquer l'INCRÉMENTALE 1 (mardi 6h)
  3. Appliquer l'INCRÉMENTALE 2 (mardi 12h)

Données récupérées : état à mardi 12h
Perte de données (RPO effectif) : 2h (de 12h à 14h)
```

> ⚠️ **Les incrémentales doivent être appliquées dans l'ordre chronologique strict.** Sauter une incrémentale intermédiaire rend la restauration impossible ou incohérente.

### Cas type : Complète + Différentielle

```
Lundi 23h    → [COMPLÈTE]
Mardi 23h    → [DIFFÉRENTIELLE 1] (modifs depuis lundi 23h)
Mercredi 23h → [DIFFÉRENTIELLE 2] (modifs depuis lundi 23h, cumulée)
Jeudi 14h    → INCIDENT

Ordre de restauration :
  1. Restaurer la COMPLÈTE du lundi 23h
  2. Appliquer UNIQUEMENT la DIFFÉRENTIELLE 2 (mercredi 23h)

Données récupérées : état à mercredi 23h
```

---

## 5. Cas pratique : Attaque ransomware

### Diagnostic de la situation

```
Symptômes :
- Fichiers chiffrés sur le serveur de production
- Extension inconnue ajoutée aux fichiers (.locked, .encrypted, etc.)
- Note de rançon sur le bureau ou dans les dossiers
- Sauvegardes NAS également chiffrées
```

### Analyse de la politique de sauvegarde défaillante

| Problème identifié | Cause |
|--------------------|-------|
| NAS chiffré | Connecté en permanence au réseau → accessible au ransomware |
| Aucune copie hors ligne | Absence de sauvegarde sur support déconnecté |
| Pas de sauvegarde immuable | Les fichiers de sauvegarde étaient modifiables |
| Rétention insuffisante | Le ransomware dormait peut-être depuis plusieurs jours |

### Mesures correctives

1. **Immédiatement :** Isoler du réseau tous les systèmes infectés
2. **Identifier la souche** : consulter https://www.nomoreransom.org (clé de déchiffrement potentielle gratuite)
3. **Ne pas payer** la rançon (aucune garantie de récupération, finance les criminels)
4. **Rechercher une copie non chiffrée** : cloud avec versioning, bande hors ligne, copie chez un prestataire
5. **Reconstruire** les serveurs depuis une image propre
6. **Restaurer les données** depuis la sauvegarde la plus ancienne non affectée
7. **Déposer plainte** (obligation légale, et permet de bénéficier d'une assistance de l'ANSSI)

### Mesures préventives à mettre en place

| Mesure | Description |
|--------|-------------|
| **Règle 3-2-1** | Avec obligatoirement 1 copie hors ligne ou immuable |
| **Sauvegardes immuables** | Option "Object Lock / WORM" sur le stockage cloud ou NAS |
| **NAS isolé** | VLAN dédié, accès restreint, déconnecté hors des fenêtres de sauvegarde |
| **Rétention suffisante** | Minimum 30 jours pour détecter le ransomware avant que toutes les versions soient atteintes |
| **EDR / Antivirus** | Détection comportementale (chiffrement massif de fichiers = alerte) |
| **Principe de moindre privilège** | Le compte de sauvegarde n'a accès qu'à ce dont il a besoin |

---

## 🎯 Récapitulatif à mémoriser

| Concept | Définition courte |
|---------|-------------------|
| **Test de restauration** | Indispensable régulièrement, seule preuve de validité d'une sauvegarde |
| **PCA** | Continuité sans interruption, coûteux, RTO ≈ 0 |
| **PRA** | Reprise après interruption, moins coûteux, RTO = quelques heures |
| **RPO** | Perte de données max acceptable (en temps) |
| **RTO** | Durée d'indisponibilité max acceptable |
| **Politique de sauvegarde** | Document formel : périmètre, fréquences, rétention, responsables, tests |
| **Ransomware** | NAS connecté = vulnérable → besoin de copie hors ligne ou immuable |
| **Ordre de restauration** | Complète → Incrémentales dans l'ordre chronologique |
