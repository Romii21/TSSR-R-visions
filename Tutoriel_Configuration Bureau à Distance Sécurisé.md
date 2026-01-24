**Objectif :** Contrôler le PC Fixe (Windows 11 Pro) depuis le PC Portable (Lubuntu) via un tunnel sécurisé (Tailscale), en utilisant le compte utilisateur principal.

## Prérequis

- **PC Fixe :** Windows 11 Pro.
    
- **PC Portable :** Lubuntu (ou tout Linux).
    
- **Compte :** Un compte Google/Microsoft commun pour Tailscale.
    

---

## Phase 1 : Le Réseau Sécurisé (Tailscale)

Cette étape crée un réseau privé virtuel (VPN) entre les deux machines.

1. **Sur le PC Fixe (Windows) :**
    
    - Télécharger et installer Tailscale depuis [tailscale.com](https://tailscale.com).
        
    - Se connecter avec votre compte (Google/Microsoft).
        
    - **IMPORTANT :** Noter l'adresse IP assignée au PC Fixe (ex: `100.x.y.z`).
        
2. **Sur le PC Portable (Linux) :**
    
    - Ouvrir un terminal et installer Tailscale :
        
        Bash
        
        ```
        curl -fsSL https://tailscale.com/install.sh | sh
        sudo tailscale up
        ```
        
    - Cliquez sur le lien généré pour vous connecter au même compte.
        
3. **Vérification :**
    
    - Sur Linux, tapez `ping 100.x.y.z` (l'IP du fixe). Si ça défile, le tunnel est actif.
        

---

## Phase 2 : Préparer Windows 11 (Le Serveur)

Il faut activer le service et autoriser la connexion par mot de passe (et pas seulement par PIN).

### 1. Activer le Bureau à Distance

- Aller dans **Paramètres > Système > Bureau à distance**.
    
- Passer l'interrupteur sur **Activé**.
    
- Dérouler la flèche et **DÉCOCHER** la case _"Exiger que les appareils utilisent l'authentification au niveau du réseau (NLA)"_. (Cela évite bien des soucis de compatibilité avec Linux).
    

### 2. Autoriser le "Vrai" Mot de Passe (Crucial pour le compte principal)

Windows 11 bloque par défaut les connexions sans Windows Hello (PIN/Visage). Il faut lever ce blocage.

- Aller dans **Paramètres > Comptes > Options de connexion**.
    
- Section "Paramètres supplémentaires".
    
- **DÉSACTIVER** l'option : _"Pour plus de sécurité, autoriser uniquement la connexion à Windows Hello pour les comptes Microsoft..."_.
    

### 3. Récupérer le nom d'utilisateur système

- Ouvrir un terminal (PowerShell ou cmd).
    
- Tapez `whoami`.
    
- Notez le nom après l'antislash (ex: pour `desktop\romai`, le nom est **`romai`**).
    

---

## Phase 3 : Configurer le Client (Lubuntu)

Nous utilisons **Remmina**.

1. Lancer Remmina.
    
2. Créer un nouveau profil (Bouton `+`) :
    
    - **Protocole :** RDP (Remote Desktop Protocol).
        
    - **Serveur :** L'adresse IP Tailscale (`100.x.y.z`).
        
    - **Nom d'utilisateur :** Le nom trouvé via `whoami` (ex: `romai`) ou votre email complet.
        
    - **Mot de passe :** Votre mot de passe de compte Microsoft (le long, pas le PIN).
        
    - **Domaine :** Laisser vide (ou mettre `MicrosoftAccount` si échec).
        
3. **Réglages recommandés (Onglet "Basique" ou "Qualité") :**
    
    - Couleurs : "GFX RFX" ou "H264" (Meilleure fluidité).
        
    - Résolution : "Utiliser la résolution du client".
        
4. Cliquer sur **Enregistrer et Connecter**.
    

---

## ANNEXE : Résolution des Bugs (Troubleshooting)

Si ça ne marche pas du premier coup, référez-vous à cette liste basée sur nos tests.

### Bug 1 : Le service Windows est planté ("Sourd")

- **Symptôme :** Remmina charge pendant 15-20 secondes puis affiche "Impossible de se connecter". La commande `netstat -an | find "3389"` sur Windows ne renvoie rien.
    
- **Solution :** Redémarrer complètement le PC Windows (Menu Démarrer > Redémarrer). Ne pas juste éteindre/rallumer.
    

### Bug 2 : Erreur d'Authentification ("Login Failed")

- **Symptôme :** La connexion s'établit mais rejette le mot de passe.
    
- **Cause :** Vous utilisez le code PIN ou l'option "Hello uniquement" est encore active.
    
- **Solution :** Vérifiez la **Phase 2, étape 2**. Assurez-vous d'utiliser le mot de passe du compte Microsoft/Google associé à Windows.
    

### Bug 3 : Le Pare-feu bloque Tailscale

- **Symptôme :** Le `ping` ne passe pas, ou le ping passe mais pas le RDP (timeout).
    
- **Solution :** Ouvrir un PowerShell en mode **Administrateur** sur Windows et lancer cette commande magique :
    
    PowerShell
    
    ```
    New-NetFirewallRule -DisplayName "Autoriser RDP Tailscale" -Direction Inbound -LocalPort 3389 -Protocol TCP -Action Allow -Profile Any
    ```
    

### Astuces clavier Remmina

- **Ctrl Droite + F :** Basculer en Plein Écran (Immersion totale).
    
- **Ctrl Droite :** Libérer la souris si elle est bloquée.