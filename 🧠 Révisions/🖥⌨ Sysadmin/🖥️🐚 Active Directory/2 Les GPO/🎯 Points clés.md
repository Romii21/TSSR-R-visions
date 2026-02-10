### Architecture

|Composant|Emplacement|Rôle|
|---|---|---|
|**Entrée LDAP**|`CN=Policies,CN=System,DC=xx,DC=xx`|Métadonnées (nom, GUID, droits)|
|**Contenu**|`SYSVOL\Policies\{GUID}`|Instructions de la GPO|
|**Attribut gPLink**|Sur OU/Site/Domaine|Liaison et ordre de traitement|

### Priorité LSDOU

```
Local < Site < Domain < OU
    ↑                     ↑
  Faible              Forte
  priorité           priorité
```

> **Exception** : GPO Enforced = priorité absolue (ignore tout, même le blocage d'héritage)

### Règles d'or

1. **Une action = Une GPO** (principe de séparation des responsabilités)
2. **Ne jamais modifier** la Default Domain Policy
3. **Ne jamais modifier** une GPO active en production (créer une nouvelle version)
4. **Ne pas bloquer l'héritage** sauf cas exceptionnel
5. **LIFO** : La dernière GPO liée est prioritaire au sein d'une OU

### Filtrage de sécurité

Pour qu'une GPO s'applique, il faut :

- ✅ Lien actif (Link Enabled)
- ✅ GPO active (Enabled)
- ✅ Droits **Read** + **Apply Group Policy** sur l'objet cible

### Timing d'application

|GPO Ordinateurs|GPO Utilisateurs|
|---|---|
|Démarrage → Boot|Début login → Fin login|

### Commandes essentielles

```bash
gpupdate /force          # Forcer la mise à jour des GPO
gpresult /r              # Voir les GPO appliquées
gpmc.msc                 # Console de gestion (serveur)
gpedit.msc               # Éditeur local (client)
```

---
