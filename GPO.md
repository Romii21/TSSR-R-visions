* GPO = Tout gérer dans le système
* MCO = maintient en condition opérationnel
* GPO = Windows
* GPO = Permet de poser des règles à chaque objets du ou des domaine.s
* GPO = Se trouve de manière général dans le dossier SYSVOL
* Etat forcé (enforced) donne une priorité a la GPO sur les autres du domaine
* ⚠️ Ne jamais modifier une GPO si elle est déjà active 
* ⚠️ Ne pas bloquer l'héritage des GPO des niveaux supérieurs 
* Ordre (LIFO = Last In First Out) la dernière GPO traité sera la prioritaire et écrasera les autres GPO
* GPO ordinateurs => entre le démarrage et le boot
* GPO utilisateurs => entre le début du login et la fin du login 
* ⚠️ Ne pas toucher à la GPO "default domain policy"
* Une action = Une GPO
* 