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

Workgroup est le domaine local de chaque machine, il ne bénéficie pas la centralisation qu'offre le domaine d'un serveur AD : DHCP, DNS, LDAP, ...
Sur Workgroup toutes les configurations sont indépendante et au delà de 10 machines, la gestion devient compliquée

---

**Question 3**  
Décrivez la hiérarchie logique d'Active Directory en partant du plus petit élément jusqu'à la plus grande structure. Donnez une définition succincte de chaque niveau.

La structure hiérarchique de l'AD permet de partir d'une base le domaine et de descendre d'OU en OU jusqu'à arriver à l'objet AD.
Le DN correspond au chemin jusqu'à l'objet "CN=Objet, OU=Lieux 2, OU=Lieux 1, DC=lab, DC=lan"

---

**Question 4**  
Qu'est-ce qu'une **OU (Organizational Unit)** ? À quoi sert-elle concrètement, et quelle est la bonne pratique d'organisation recommandée pour les OU dans un environnement d'entreprise ?

Une OU est un objet de l'AD qui sert de dossier ou de compartiment dans lequel peut se trouver, des utilisateurs, des ordinateurs, des groupes ou d'autres OUs. En entreprise il est important de ne pas séparer par lieux mais par services sauf cas spéciaux pour donner la même configuration à chaque utilisateurs. Les utilisateur et les ordinateurs doivent aussi être séparé pour ne pas appliquer les mêmes règles au deux types d'objets.

---

## Partie 2 – Infrastructure et rôles

**Question 5**  
Citez et expliquez les **5 rôles FSMO**. Pour chacun, précisez s'il est de portée forêt ou domaine, et indiquez lequel est considéré comme le plus critique et pourquoi.

Un rôles FSMO est un rôle qui permet le bon fonctionnement d'un domaine. Deux rôles concernent la forêt et trois le domaine.

Dans la forêt :
- Le schéma master
- 

Dans le domaine :
- Le RIP
- 

---

**Question 6**  
Qu'est-ce que la **réplication Active Directory** ? Quel composant en est responsable, comment fonctionne-t-il, et quelle est l'intervalle de réplication recommandée selon vos cours ?

La réplication de l'AD est le faite de synchroniser les DC du domaine, pour faire la redondance, si un des serveur AD tombe, l'autre peut prendre la relève car toutes les configuration ont été synchronisé dessus. L'intervalle de base est de 5min mais la bonne pratique doit être de 30min pour ne pas saturer les serveurs

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

Une GPO est une règle mise en place dans l'environnement AD elle peut s'appliqué aux utilisateurs, aux ordinateurs ou aux groupes.  Les GPO peuvent configurer le droit de connexion à une machine ou les horaires de connexions, les mots de passes, les fonds d'écrans, l'accès à certains logiciels ou services comme l'invite de commande, de mapper des dossiers partagés

---

**Question 10**  
Expliquez l'ordre de priorité **LSDOU** dans l'application des GPO. Que se passe-t-il lorsqu'une GPO est configurée avec l'option **Enforced (Appliqué)** ?

Il me semble que c'est le principe de Last In First Out, la dernière GPO mise en place est la première appliquée. Si elle est Enforced elle devient prioritaire sur toutes les autres.

---

**Question 11**  
Quelles sont les deux conditions nécessaires pour qu'une GPO s'applique à un objet Active Directory ? Expliquez le mécanisme de **filtrage de sécurité** et comment restreindre une GPO à un groupe spécifique.

Qu'elle soit relié à une OU et à un groupe, utilisateur ou ordinateur

---

**Question 12**  
Quelles sont les commandes permettant de forcer la mise à jour des GPO sur un poste client, et d'afficher les GPO actuellement appliquées ? Précisez le rôle de chacune.

gpudate /force permet d'appliquer les GPO

gpresult -r permet d'afficher les GPO appliqués 

---

## Partie 4 – Sécurité et bonnes pratiques

**Question 13**  
Qu'est-ce que la méthode **AGDLP** ? Décrivez chaque lettre et expliquez pourquoi cette méthode est recommandée pour la gestion des permissions dans un domaine Active Directory.



---

**Question 14**  
Qu'est-ce que **LAPS (Local Administrator Password Solution)** ? Quel problème résout-il, et pourquoi est-il recommandé de le déployer sur l'ensemble des postes du domaine ?

LAPS est un protocole qui permet de donner un mots de passe aléatoire sur les utilsateurs locaux des machines du réseau. Cela permet de na pas avoir de MDP commun sur les machines du domaine et ainsi augmenter la sécurité du domaine.

---

**Question 15**  
Citez au moins **5 bonnes pratiques de sécurité** à appliquer dans un environnement Active Directory, en les justifiant brièvement. Vous pouvez aborder les thèmes : comptes d'administration, mots de passe, infrastructure DC, VLAN, et surveillance.

- Désactiver l'accès au compte administrateur (Compte privilégié facile d'accès)
- Nomenclature non explicite de l'AD (Pour ne pas mettre une étiquette sur chaque lieux de l'entreprise)
- Redondance et synchronisation des DC (Pour conserver le domaine et sa configuration en cas de perte)

---

_Rendez vos réponses au format texte libre — les corrections seront faites question par question avec explication des points clés._