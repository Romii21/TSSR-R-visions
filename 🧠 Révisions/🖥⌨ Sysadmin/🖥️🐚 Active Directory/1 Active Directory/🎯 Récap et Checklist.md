## Récap

| Concept         | Description                         | Commande PowerShell                     |
| --------------- | ----------------------------------- | --------------------------------------- |
| **Objet AD**    | Élément de base de l'annuaire       | `Get-ADObject`                          |
| **OU**          | Conteneur pour organiser les objets | `Get-ADOrganizationalUnit`              |
| **Domaine**     | Unité administrative et de sécurité | `Get-ADDomain`                          |
| **Forêt**       | Collection d'arbres de domaines     | `Get-ADForest`                          |
| **DC**          | Serveur contrôleur de domaine       | `Get-ADDomainController`                |
| **FSMO**        | Rôles maîtres uniques               | (visible dans `Get-ADDomainController`) |
| **Groupe**      | Conteneur d'utilisateurs/objets     | `Get-ADGroup`                           |
| **Utilisateur** | Compte d'accès                      | `Get-ADUser`                            |

---
## Checklist des Bonnes Pratiques

### Structure

- [ ] Organiser les OU par fonction/métier (pas par géographie)
- [ ] Séparer clairement OU Utilisateurs et OU Ordinateurs
- [ ] Éviter une granularité excessive

### Sécurité de Base

- [ ] Appliquer le principe de moindre privilège
- [ ] Utiliser des comptes d'administration séparés
- [ ] Déployer LAPS sur tous les postes
- [ ] Politique de mots de passe conforme ANSSI

### Infrastructure

- [ ] Minimum 2 DC pour la redondance
- [ ] DC dans des VLANs sécurisés
- [ ] Pas de redémarrage automatique des DC
- [ ] Réplication configurée (10-20 min recommandé)
- [ ] Séparer les rôles FSMO sur différents DC

### Sécurité Avancée

- [ ] Implémenter le Tiering Model
- [ ] Mettre en place JIT et JEA
- [ ] Appliquer systématiquement AGDLP
- [ ] Utiliser LDAPS pour toutes les communications

### Surveillance

- [ ] Audits réguliers de l'AD
- [ ] Monitoring des logs
- [ ] Stratégie de sauvegarde 3-2-1
- [ ] Plan de restauration testé

---
