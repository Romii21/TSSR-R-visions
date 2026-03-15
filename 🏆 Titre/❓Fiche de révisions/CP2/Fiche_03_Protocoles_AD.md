# Fiche 03 — Protocoles associés à Active Directory

> **Formation TSSR — CP2**
> Révision : DNS, NTP/SNTP, Kerberos, LDAP/LDAPS, NTLM

---

## Vue d'ensemble des protocoles AD

| Protocole | Rôle | Criticité |
|---|---|---|
| **DNS** | Résolution de noms et localisation des services AD | ⚠️ Obligatoire |
| **SNTP/NTP** | Synchronisation des horloges | ⚠️ Critique (Kerberos) |
| **Kerberos** | Authentification des utilisateurs | ⚠️ Central |
| **LDAP / LDAPS** | Accès et interrogation de l'annuaire | ⚠️ Fondamental |
| **NTLM** | Authentification legacy (compatibilité) | À éviter en production |

---

## 1. DNS — Domain Name System

### Rôle dans AD

Le DNS est **indispensable** au fonctionnement d'Active Directory. Sans DNS, AD ne peut pas fonctionner.

**Ce qu'il permet :**
- Résolution des noms d'hôtes (nom → IP)
- Localisation des services AD via des **enregistrements SRV** (Service Records)
- Permettre aux machines de **trouver les DC** du domaine

> ⚠️ **Point critique :** Le DNS est le **premier élément à vérifier** lors de tout problème AD. Si le DNS est défaillant : aucune authentification, aucune GPO, aucune jonction au domaine n'est possible.

**Conséquences d'un DNS mal configuré ou indisponible :**
- Les machines ne peuvent pas localiser les contrôleurs de domaine
- L'authentification Kerberos échoue
- Les GPO ne s'appliquent plus
- La jonction au domaine est impossible

**Commande de vérification :**
```cmd
nslookup entreprise.local
```

---

## 2. NTP / SNTP — Synchronisation du temps

### Rôle

Le protocole **SNTP (Simple Network Time Protocol)** synchronise les horloges de toutes les machines du domaine sur la même référence temporelle (UTC).

### Pourquoi c'est critique dans AD

**Kerberos exige que toutes les machines soient synchronisées à moins de 5 minutes d'écart.** Au-delà, Kerberos refuse l'authentification.

**Conséquences d'une désynchronisation :**
- Échec d'authentification Kerberos → personne ne peut se connecter
- Problèmes de réplication entre DC
- Blocage des accès aux ressources
- Invalidation des certificats et tickets

### La hiérarchie NTP dans AD

```
Internet (pool.ntp.org)
      ↓
PDC Emulator (source de temps du domaine)
      ↓
Autres DC
      ↓
Postes clients
```

> ⚠️ C'est le rôle **FSMO PDC Emulator** qui est la source de temps de référence du domaine.

**Commande de vérification :**
```cmd
w32tm /query /status
```

---

## 3. Kerberos — Le protocole d'authentification

### Présentation

Protocole d'authentification par défaut depuis Windows 2000. Basé sur un système de **tickets** à durée de vie limitée.

**Caractéristiques clés :**
- Authentification **mutuelle** (client ↔ serveur)
- Utilisation de **tickets** (pas de mot de passe envoyé sur le réseau)
- Durée de vie du TGT : **10 heures** par défaut
- Adresse du serveur Kerberos extraite du **DNS**

---

### Les 3 étapes de Kerberos

```
[Étape 1] AS-REQ / AS-REP — Obtenir le TGT
──────────────────────────────────────────
Utilisateur ──── envoie son identifiant ────→ KDC (sur le DC)
Utilisateur ←─── reçoit le TGT chiffré ───── KDC

[Étape 2] TGS-REQ / TGS-REP — Obtenir un ticket de service
────────────────────────────────────────────────────────────
Utilisateur ──── présente son TGT + demande accès à ressource ────→ KDC
Utilisateur ←─── reçoit un Ticket de Service (ST) pour la ressource ─ KDC

[Étape 3] AP-REQ — Accéder à la ressource
──────────────────────────────────────────
Utilisateur ──── présente le ticket de service ────→ Serveur cible
Serveur valide le ticket → Accès autorisé ✅
```

---

### TGT — Ticket Granting Ticket

Le TGT est le **"laissez-passer" initial** délivré après la première authentification réussie.

- Durée de vie : **10 heures** (par défaut)
- Chiffré avec la clé du KDC → ne peut pas être falsifié
- Évite de retaper le mot de passe à chaque accès à une ressource
- C'est le principe du **SSO (Single Sign-On)**

---

### Authentification mutuelle Kerberos vs NTLM

| Critère | NTLM | Kerberos |
|---|---|---|
| Authentification mutuelle | ❌ Non — seul le client est vérifié | ✅ Oui — client ET serveur se vérifient |
| Résistance aux attaques | ❌ Vulnérable au pass-the-hash | ✅ Plus résistant |
| Performance | Moins bonne | Meilleure (ticket réutilisable) |
| Usage recommandé | Compatibilité legacy uniquement | ✅ Standard en production |

**Ce qu'apporte l'authentification mutuelle :** Le client peut vérifier que le serveur auquel il parle est bien le vrai serveur (protection contre l'usurpation d'identité serveur / man-in-the-middle).

---

## 4. LDAP / LDAPS — Accès à l'annuaire

### Présentation

**LDAP (Lightweight Directory Access Protocol)** est le protocole standard utilisé par AD pour accéder, interroger et modifier l'annuaire.

| Protocole | Port | Chiffrement | Usage |
|---|---|---|---|
| **LDAP** | **389** | ❌ Aucun — données en clair | À éviter en production |
| **LDAPS** | **636** | ✅ TLS/SSL | ✅ Recommandé en production |

**Ce que permet LDAP :**
- Accéder aux données de l'annuaire
- Effectuer des recherches structurées
- Modifier les objets AD
- Interopérabilité avec Linux, OpenLDAP, applications SSO

**Pourquoi LDAPS en production :** LDAP transmet les identifiants et les données de l'annuaire **en clair** sur le réseau → vulnérable aux attaques de type "man-in-the-middle". LDAPS chiffre tout le trafic via TLS.

---

### Fichiers LDIF

**LDIF (LDAP Data Interchange Format)** = fichiers permettant d'importer/exporter des données AD en masse.

> ⚠️ Méthode puissante mais **risquée** — toute modification est directement appliquée à l'annuaire. À utiliser avec précaution et toujours après une sauvegarde.

---

## Synthèse des ports à connaître

| Service | Protocole | Port |
|---|---|---|
| LDAP | TCP/UDP | 389 |
| LDAPS | TCP | 636 |
| Kerberos | TCP/UDP | 88 |
| DNS | TCP/UDP | 53 |
| NTP/SNTP | UDP | 123 |
| WS-Management (WEF) | HTTP | 5985 |
| WS-Management (WEF) | HTTPS | 5986 |

---

## Diagnostic rapide — Que vérifier en cas de problème AD

```
Problème de connexion au domaine ?
    ↓
1. Le câble / Wi-Fi est-il connecté ?         ping 8.8.8.8
2. Le DC est-il joignable ?                   ping nomduDC
3. Le DNS fonctionne-t-il ?                   nslookup entreprise.local
4. L'heure est-elle synchronisée ?            w32tm /query /status
5. Le compte est-il activé dans AD ?          dsa.msc
6. La machine est-elle jointe au domaine ?    Système > Paramètres avancés
```

---

## Points clés à retenir

- **DNS** = socle d'AD — premier élément à vérifier en cas de problème
- **SNTP** = synchronisation du temps — tolérance Kerberos de **5 minutes max**
- **Kerberos** = 3 étapes (TGT → Ticket Service → Accès), authentification mutuelle
- **TGT** = laissez-passer initial, durée de vie 10h, base du SSO
- **LDAP port 389** (non chiffré) / **LDAPS port 636** (chiffré TLS) → toujours LDAPS en production
- **NTLM** = protocole legacy, vulnérable, à remplacer par Kerberos
