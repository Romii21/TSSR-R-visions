# Fiche 07 — PowerShell AD & Commandes Essentielles

> **Formation TSSR — CP2**
> Révision : Commandes PowerShell AD, diagnostic, GPO, réplication

---

## 1. Prérequis — Module Active Directory

Avant d'utiliser les commandes AD en PowerShell, vérifier que le module est disponible :

```powershell
# Importer le module AD (si non chargé automatiquement)
Import-Module ActiveDirectory

# Vérifier que le module est disponible
Get-Module -ListAvailable | Where-Object {$_.Name -eq "ActiveDirectory"}
```

> ⚠️ Sur un serveur où AD DS est installé, le module est disponible automatiquement. Sur un poste client, il faut installer les **RSAT (Remote Server Administration Tools)**.

---

## 2. Gestion des Utilisateurs

### Créer un utilisateur

```powershell
New-ADUser `
  -Name "Jean Dupont" `
  -GivenName "Jean" `
  -Surname "Dupont" `
  -SamAccountName "jdupont" `
  -UserPrincipalName "jdupont@entreprise.local" `
  -Path "OU=Informatique,OU=Utilisateurs,DC=entreprise,DC=local" `
  -AccountPassword (ConvertTo-SecureString "P@ssw0rd!" -AsPlainText -Force) `
  -Enabled $true
```

**Paramètres essentiels :**
| Paramètre | Rôle |
|---|---|
| `-Name` | Nom affiché dans AD |
| `-SamAccountName` | Login (identifiant de connexion) |
| `-UserPrincipalName` | UPN — format email `login@domaine` |
| `-Path` | OU de destination (chemin LDAP) |
| `-AccountPassword` | Mot de passe (doit être un SecureString) |
| `-Enabled $true` | Active le compte dès la création |

---

### Lister les utilisateurs

```powershell
# Tous les utilisateurs du domaine
Get-ADUser -Filter *

# Utilisateurs d'une OU spécifique + nom et statut
Get-ADUser -Filter * `
  -SearchBase "OU=Informatique,OU=Utilisateurs,DC=entreprise,DC=local" `
  -Properties Enabled | Select-Object Name, Enabled

# Chercher un utilisateur par nom
Get-ADUser -Filter {Name -like "*dupont*"} -Properties *
```

---

### Modifier un utilisateur

```powershell
# Activer un compte
Enable-ADAccount -Identity "jdupont"

# Désactiver un compte
Disable-ADAccount -Identity "jdupont"

# Réinitialiser le mot de passe
Set-ADAccountPassword -Identity "jdupont" `
  -NewPassword (ConvertTo-SecureString "NouveauP@ss!" -AsPlainText -Force) `
  -Reset

# Déverrouiller un compte
Unlock-ADAccount -Identity "jdupont"

# Modifier un attribut (ex: département)
Set-ADUser -Identity "jdupont" -Department "Informatique"
```

---

### Supprimer un utilisateur

```powershell
Remove-ADUser -Identity "jdupont" -Confirm:$false
```

> ⚠️ Toujours **désactiver avant de supprimer**. La suppression est définitive et le SID ne peut pas être récupéré.

---

## 3. Gestion des Groupes

```powershell
# Créer un groupe
New-ADGroup -Name "GG_Informatique" `
  -GroupScope Global `
  -GroupCategory Security `
  -Path "OU=Groupes,DC=entreprise,DC=local"

# Ajouter un utilisateur à un groupe
Add-ADGroupMember -Identity "GG_Informatique" -Members "jdupont"

# Lister les membres d'un groupe
Get-ADGroupMember -Identity "GG_Informatique" | Select-Object Name, SamAccountName

# Lister les groupes d'un utilisateur
Get-ADUser "jdupont" -Properties MemberOf | Select-Object -ExpandProperty MemberOf
```

---

## 4. Gestion des OU

```powershell
# Créer une OU
New-ADOrganizationalUnit -Name "Informatique" `
  -Path "OU=Utilisateurs,DC=entreprise,DC=local"

# Lister les OU
Get-ADOrganizationalUnit -Filter * | Select-Object Name, DistinguishedName

# Déplacer un objet dans une autre OU
Move-ADObject -Identity "CN=jdupont,OU=Ancienne,DC=entreprise,DC=local" `
  -TargetPath "OU=Nouvelle,DC=entreprise,DC=local"
```

---

## 5. Gestion des Ordinateurs

```powershell
# Lister les ordinateurs du domaine
Get-ADComputer -Filter * | Select-Object Name, Enabled

# Chercher un ordinateur spécifique
Get-ADComputer -Identity "PC-BUREAU-01" -Properties *

# Lire le mot de passe LAPS d'un poste
Get-ADComputer -Identity "PC-BUREAU-01" -Properties ms-Mcs-AdmPwd | `
  Select-Object Name, ms-Mcs-AdmPwd
```

---

## 6. Informations sur le Domaine et la Forêt

```powershell
# Infos sur le domaine
Get-ADDomain | Select-Object DomainMode, Name, DNSRoot

# Infos sur la forêt
Get-ADForest | Select-Object Name, ForestMode, Domains

# Lister les contrôleurs de domaine
Get-ADDomainController -Filter * | Select-Object Name, IPv4Address, Site

# Voir les rôles FSMO
Get-ADDomainController -Filter * | Select-Object Name, OperationMasterRoles
netdom query fsmo   # (via cmd)
```

---

## 7. Commandes GPO

```cmd
REM Forcer la mise à jour immédiate des GPO
gpupdate /force

REM Voir les GPO appliquées sur la machine locale
gpresult /r

REM Générer un rapport HTML complet des GPO appliquées
gpresult /h C:\rapport_gpo.html

REM Ouvrir la console GPO (serveur AD)
gpmc.msc

REM Ouvrir l'éditeur de stratégie locale (poste client)
gpedit.msc
```

---

## 8. Commandes de Diagnostic AD

### Diagnostic général

```cmd
REM Tester la connectivité réseau
ping nomDuDC

REM Vérifier la résolution DNS du domaine
nslookup entreprise.local

REM Vérifier la synchronisation de l'heure
w32tm /query /status

REM Diagnostic complet du DC
dcdiag

REM Diagnostic des réplications uniquement
dcdiag /test:replications
```

---

### Diagnostic de la réplication

```cmd
REM État détaillé de la réplication (erreurs, horodatages)
repadmin /showrepl

REM Résumé rapide de l'état de la réplication
repadmin /replsummary

REM Forcer une réplication immédiate depuis un DC spécifique
repadmin /syncall /AdeP
```

---

### Diagnostic d'un problème de connexion utilisateur

```cmd
REM 1. Tester la connectivité réseau
ping nomDuDC

REM 2. Vérifier la résolution DNS
nslookup entreprise.local

REM 3. Vérifier la synchronisation de l'heure (Kerberos = 5 min de tolérance)
w32tm /query /status

REM 4. Vérifier si la machine est bien dans le domaine (GUI)
Système → Paramètres avancés → Nom de l'ordinateur

REM 5. Ouvrir la console AD pour vérifier le compte (GUI)
dsa.msc
```

---

## 9. Récapitulatif — Commandes indispensables à connaître

| Action | Commande |
|---|---|
| Créer un utilisateur | `New-ADUser` |
| Lister les utilisateurs | `Get-ADUser -Filter *` |
| Activer / Désactiver un compte | `Enable-ADAccount` / `Disable-ADAccount` |
| Déverrouiller un compte | `Unlock-ADAccount` |
| Réinitialiser un mot de passe | `Set-ADAccountPassword` |
| Créer un groupe | `New-ADGroup` |
| Ajouter un membre à un groupe | `Add-ADGroupMember` |
| Créer une OU | `New-ADOrganizationalUnit` |
| Forcer les GPO | `gpupdate /force` |
| Voir les GPO appliquées | `gpresult /r` |
| Infos domaine | `Get-ADDomain` |
| Rôles FSMO | `netdom query fsmo` |
| État réplication | `repadmin /showrepl` |
| Diagnostic DC | `dcdiag` |
| Synchronisation temps | `w32tm /query /status` |
| Résolution DNS | `nslookup entreprise.local` |

---

## Points clés à retenir

- `-Filter *` = tous les objets / `-Filter {Name -like "..."}` = filtrer par nom
- Le mot de passe doit toujours être converti en **SecureString** : `ConvertTo-SecureString "..." -AsPlainText -Force`
- Le chemin d'une OU s'écrit toujours en **notation LDAP** : `OU=...,DC=...,DC=...`
- `gpupdate /force` = forcer les GPO | `gpresult /r` = voir ce qui est appliqué
- `repadmin /showrepl` = diagnostic réplication | `dcdiag` = diagnostic global du DC
- `w32tm /query /status` = vérifier l'heure (critique pour Kerberos)
