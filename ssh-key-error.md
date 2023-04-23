# Gestion de l'erreur WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!

Cette erreur apparaît lorsqu'une clé ssh a été modifiée après avoir été partagée.
J'ai notamment rencontré ce cas lors de l'installation de Vagrant : plutôt que de créer une nouvelle clé ssh, j'ai partagé celle de Github, ce qui a entraîné des alertes de mes connexions à la plateforme.

## Procédure à suivre pour résoudre le problème

Il faut se rendre dans le répertoire contenant le fichier .ssh :

```bash
$ cd C:\Users\bulam\.ssh
code known_hosts
```

Une fois le fichier ouvert dans VsCode, il faut y supprimer la référence à Github. Celle-ci sera à nouveau insérée dans le fichier lorsque l'authentification auprès de Github aura été réalisée :

```bash
$ git clone git@github.com:<projet>.git
Cloning into '4087056-testez-unitairement-votre-application-symfony-php'...
The authenticity of host 'github.com (140.82.121.3)' can't be established.
ECDSA key fingerprint is SHA256:p2QAMXNIC1TJYWeIOttrVc98/R1BUFWu3/LiyKgUfQM.
Are you sure you want to continue connecting (yes/no/[fingerprint])? y
Please type 'yes', 'no' or the fingerprint: yes
Warning: Permanently added 'github.com' (ECDSA) to the list of known hosts.
Warning: the ECDSA host key for 'github.com' differs from the key for the IP address '140.82.121.3'
Offending key for IP in /c/Users/bulam/.ssh/known_hosts:1
Are you sure you want to continue connecting (yes/no)? yes
Enter passphrase for key '/c/Users/bulam/.ssh/id_ed25519':
```

Une fois la passphrase fournie, l'accès à Github est rétabli.


