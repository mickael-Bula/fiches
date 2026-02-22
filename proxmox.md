# COMMANDES ET SNIPETS PROXMOX

## Création d'un LXC

Depuis l'interface de Proxmox : Create CT
Ajouter mot de passe pour la connexion root

NOTE : la connexion root en ssh est désactivée par défaut sur les Debian. La bonne pratique consiste à se connecter depuis Proxmox pour créer un nouvel utilsateur :

```bash
# Créer l'utilisateur
$ adduser monuser

# Ajouter l'utilisateur au groupe sudo pour pouvoir effectuer des tâches d'administration
$ usermod -aG sudo monuser

su - monuser
mkdir .ssh
chmod 700 .ssh
nano .ssh/authorized_keys
```

Créer une clé SSH, puis la coller dans le fichier :

```bash
$ chmod 600 .ssh/authorized_keys
```

## Installer la commande sudo

Se connecter avec l'utilisateur root, puis :

```bash
# Mettre à jour la liste des paquets
$ apt update

# Installer sudo
$ apt install -y sudo

# Ajouter ton utilisateur au groupe sudo
$ usermod -aG sudo monuser 
```

## Sécuriser SSH pour l'utilisateur

Maintenant, on verrouille le serveur pour qu'il n'accepte que cet utilisateur et sa clé :

Éditer le fichier SSH :

```Bash
nano /etc/ssh/sshd_config
```

S'assurer que ces options sont configurées :

PermitRootLogin no (On interdit formellement le root).

PasswordAuthentication no (On interdit les mots de passe).

PubkeyAuthentication yes (On autorise les clés).

Ajouter cette ligne tout en bas pour ne restreindre l'accès qu'à cet utilisateur :
AllowUsers monuser

Redémarree le service SSH :

```Bash
systemctl restart ssh
```



