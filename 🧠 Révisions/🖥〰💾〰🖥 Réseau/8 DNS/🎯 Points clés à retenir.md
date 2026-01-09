### Le DNS est essentiel au fonctionnement d'Internet

- Traduit les noms en adresses IP
- Infrastructure invisible mais critique
- Base distribuée = robustesse

### Architecture hiérarchique

- Racine → TLD → Domaines → Sous-domaines
- Délégation de l'autorité à chaque niveau
- Réplication pour la fiabilité

### Deux types de serveurs principaux

- **Serveurs faisant autorité** : détiennent les données officielles
- **Résolveurs récursifs** : interrogent pour le compte des clients

### Le cache est fondamental

- Réduit la charge sur les serveurs
- Accélère les résolutions
- Contrôlé par le TTL

### Résolution en cascade

- Client → Stub resolver → Résolveur récursif → Racine → TLD → Autoritaire
- Chaque niveau connaît le suivant
- Cache à chaque étape

---
