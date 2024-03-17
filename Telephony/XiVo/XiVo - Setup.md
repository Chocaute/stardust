---
created: 2024-01-30T09:29
updated: 2024-01-31T12:27
tags:
  - LM
  - Linux
  - Telephony
---
> [!NOTE] La version actuelle de Xivo est en Debian 11
> Mis-à-jour de XiVo en avril, passage en Debian 12 ?

#### Pré-requis
- Proxmox

### Step-By-Step
#### A. Installation de XiVO
##### 1. Téléchargement de l'ISO 
   
Copier l'URL de téléchargement de l'ISO depuis http://mirror.xivo.solutions/iso/xivo-current/ :
![[Capture d’écran du 2024-01-30 11-46-28.png]]

![[Capture d’écran du 2024-01-30 11-46-53.png]]

Depuis Proxmox, dans un répertoire ISO :
![[Capture d’écran du 2024-01-30 11-45-54.png]]

Query l'URL de téléchargement de l'ISO :
![[Capture d’écran du 2024-01-30 11-47-25.png]]

Télécharger :
![[Capture d’écran du 2024-01-30 11-47-56.png]]

Résultat :
![[Capture d’écran du 2024-01-30 11-49-34.png]]

##### 2. Création de la VM

Créez une VM.
![[Capture d’écran du 2024-01-30 11-49-53.png]]

![[Capture d’écran du 2024-01-30 11-52-20.png]]

![[Capture d’écran du 2024-01-30 11-52-36.png]]

N'oublier pas l'agent Qemu.
![[Capture d’écran du 2024-01-30 11-52-46.png]]

Cocher "Discard" si votre espace de stockage le supporte (LVM-thin, BTRFS, entres autres). 
Configurer "Cache" sur Write back pour de meilleurs performances, ou ne changer pour privilégier la stabilité.
![[Capture d’écran du 2024-01-30 11-52-59.png]]

Ajuster le nombre de Cœurs selon vos besoins.
![[Capture d’écran du 2024-01-30 11-54-40.png]]

Ajuster la mémoire selon vos besoins. 8 Go fait généralement l'affaire.
![[Capture d’écran du 2024-01-30 11-55-21.png]]

![[Capture d’écran du 2024-01-30 11-55-34.png]]

![[Capture d’écran du 2024-01-30 11-55-42.png]]

Démarrer la VM. Procéder à l'installation comme si c'était un Debian classique.
![[Capture d’écran du 2024-01-30 11-56-35.png]]

Après l'installation de Debian, la VM redémarrera directement sur l'installation de XiVo, sans demander aucun input de la part de l'utilisateur.
L'installation de XiVo peut alors prendre de 20 jusqu'à 30 minutes.

Après l'installation de XiVo, vous devez passer par l'assistant de configuration. Ouvrez votre navigateur et entrez l'adresse IP de votre serveur dans la barre de navigation (par exemple : http://192.168.1.10).

##### 3. L'assistant de configuration

Sélectionner la langue que vous souhaitez utiliser pour l'assistant.
![[Capture d’écran du 2024-01-30 13-38-55.png]]

Accepter la license.
![[Capture d’écran du 2024-01-30 13-39-05.png]]

Configurer selon vos besoins.
![[Capture d’écran du 2024-01-30 13-39-58.png]]

- Le contexte des Appels internes gère les numéros de poste qui peuvent être joints en interne.
- Le contexte des Appels entrants gère les appels provenant de l'extérieur de votre système.
- Le contexte des Appels sortants gère les appels allant de votre système vers l'extérieur.

Les intervalles de numéros définissent les numéros de téléphone des utilisateurs pour votre système - vous pouvez les modifier par la suite.
![[Capture d’écran du 2024-01-30 13-42-13.png]]

![[Capture d’écran du 2024-01-30 13-41-48.png]]

Après la validation de la configuration, vous serez momentanément redirigé vers la page de connexion de XiVO. 

> [!NOTE] PATCH 2023.10.00 > 2023.10.04
> Après la configuration de XiVO, n'oublier pas de lancer "xivo-upgrade" dans la console de votre serveur pour bénéficier des derniers patchs.
> 


 
#### B. Configuration de l'environnement 
##### 1. Sécuriser l'accès SSH

##### 2. Connecter le Trunk SIP

##### 3. Tester l'application

