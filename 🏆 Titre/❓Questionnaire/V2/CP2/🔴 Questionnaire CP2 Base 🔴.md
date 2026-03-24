

**Niveau** : TSSR  
**Nombre de questions** : 15  
**Format** : Questions ouvertes — répondez sans document, les réponses seront corrigées ensemble.

---

## Partie 1 – Fondamentaux d'Active Directory

**Question 1**  
Qu'est-ce qu'Active Directory et sur quel protocole repose-t-il ? Quelle était la solution utilisée avant AD pour la gestion des comptes, et pourquoi a-t-elle été abandonnée dans les environnements de taille moyenne et grande ?

---

**Question 2**  
Expliquez la différence entre un **Workgroup** et un **domaine Active Directory**. À partir de combien de machines est-il généralement recommandé de basculer vers un domaine, et pourquoi ?

---

**Question 3**  
Décrivez la hiérarchie logique d'Active Directory en partant du plus petit élément jusqu'à la plus grande structure. Donnez une définition succincte de chaque niveau.

---

**Question 4**  
Qu'est-ce qu'une **OU (Organizational Unit)** ? À quoi sert-elle concrètement, et quelle est la bonne pratique d'organisation recommandée pour les OU dans un environnement d'entreprise ?

---

## Partie 2 – Infrastructure et rôles

**Question 5**  
Citez et expliquez les **5 rôles FSMO**. Pour chacun, précisez s'il est de portée forêt ou domaine, et indiquez lequel est considéré comme le plus critique et pourquoi.

---

**Question 6**  
Qu'est-ce que la **réplication Active Directory** ? Quel composant en est responsable, comment fonctionne-t-il, et quelle est l'intervalle de réplication recommandée selon vos cours ?

---

**Question 7**  
Qu'est-ce qu'un **RODC (Read-Only Domain Controller)** ? Dans quel contexte est-il utilisé, et quelles sont ses limitations par rapport à un DC classique ?

---

**Question 8**  
Qu'est-ce que le **niveau fonctionnel** d'un domaine ou d'une forêt Active Directory ? Quelle règle s'applique lorsque des contrôleurs de domaine de versions différentes coexistent, et peut-on revenir en arrière après une montée de niveau ?

---

## Partie 3 – GPO (Group Policy Objects)

**Question 9**  
Qu'est-ce qu'une **GPO** ? Donnez au moins 4 exemples concrets de ce qu'il est possible de configurer via une GPO dans un environnement Windows.

---

**Question 10**  
Expliquez l'ordre de priorité **LSDOU** dans l'application des GPO. Que se passe-t-il lorsqu'une GPO est configurée avec l'option **Enforced (Appliqué)** ?

---

**Question 11**  
Quelles sont les deux conditions nécessaires pour qu'une GPO s'applique à un objet Active Directory ? Expliquez le mécanisme de **filtrage de sécurité** et comment restreindre une GPO à un groupe spécifique.

---

**Question 12**  
Quelles sont les commandes permettant de forcer la mise à jour des GPO sur un poste client, et d'afficher les GPO actuellement appliquées ? Précisez le rôle de chacune.

---

## Partie 4 – Sécurité et bonnes pratiques

**Question 13**  
Qu'est-ce que la méthode **AGDLP** ? Décrivez chaque lettre et expliquez pourquoi cette méthode est recommandée pour la gestion des permissions dans un domaine Active Directory.

---

**Question 14**  
Qu'est-ce que **LAPS (Local Administrator Password Solution)** ? Quel problème résout-il, et pourquoi est-il recommandé de le déployer sur l'ensemble des postes du domaine ?

---

**Question 15**  
Citez au moins **5 bonnes pratiques de sécurité** à appliquer dans un environnement Active Directory, en les justifiant brièvement. Vous pouvez aborder les thèmes : comptes d'administration, mots de passe, infrastructure DC, VLAN, et surveillance.

---

_Rendez vos réponses au format texte libre — les corrections seront faites question par question avec explication des points clés._