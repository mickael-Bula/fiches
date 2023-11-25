# TEST CONNEXION SFTP ENTRE DEUX POSTES D'UN MEME RESEAU

J'utilise docker pour créer deux machines linux communiquant sur le même réseau.

## Les commandes pour vréer le réseau, les machines et configurer la connexion SFTP

Les sources m'ayant permis de réaliser la connexion sont https://chat.openai.com/c/2330f975-5d2a-4b83-8b21-1075827683d6

```bash
# je lance les conteneurs avec une tâche de fond sans fin pour les garder avec le statut 'running'
$ docker run -d --name conteneur_sftp_server --network mon_reseau ubuntu tail -f /dev/null
$ docker run -d --name conteneur_sftp_client --network mon_reseau ubuntu tail -f /dev/null
$ docker exec -it conteneur_sftp_server bash
docker $ apt-get update
docker $ apt-get install -y openssh-server
docker $ apt-get install -y nano
docker $ nano /etc/ssh/sshd_config

# dans nano, j'ajoute les instructions suivantes à la fin du fichier ouvert :
Subsystem sftp internal-sftp
Match group sftp
ChrootDirectory /home/%u    # %u est un espace de nom réservé qui est remplacé par le nom de l'utilisateur lors de la connexion.
X11Forwarding no
AllowTcpForwarding no
ForceCommand internal-sftp
# j'enregistre et je quitte nano

docker $ useradd -m user1
docker $ passwd user1
docker $ usermod -aG sftp user1 # usermod = usermodification | -a = append | -G = group

# redémarre le service ssh pour prise en compte de la nouvelle configuration
docker $ service ssh restart

# je quitte le premier conteneur
docker $ exit

# connexion et mise à jour du second conteneur
$ docker exec -it conteneur_sftp_client bash
docker $ apt-get update
docker $ apt-get install -y openssh-client

# j'établis la connexion avec le second conteneur
docker $ sftp user1@conteneur_sftp_server
```

Après ces commandes, j'obtiens un prompt pour la connexion SFTP.

## Utilisation d'une paire de clés ssh

Pour copier la clé publique d'un conteneur à l'autre, je dois passer par une copie sur le poste local :

```bash
# génération de la paire de clés ssh
$ docker exec -it conteneur_sftp_client ssh-keygen -t rsa
# affiche la clé publique pour copie dans un fichier local
$ docker exec -it conteneur_sftp_client cat ~/.ssh/id_rsa.pub
# crée le répertoire pour recevoir la clé copiée après affichage
$ docker exec -it conteneur_sftp_server mkdir -p /home/user1/.ssh
# crée le fichier authorized_keys dans le répertoire précédent et y copie la clé publique
$ docker exec -it conteneur_sftp_server bash 
docker $ nano /home/user1/.ssh/authorized_keys   # je colle dans le fichier ouvert la clé publique copiée
# ajuste les permissions
docker $ chmod 600 /home/user1/.ssh/authorized_keys
docker $ chown user1:user1 /home/user1/.ssh/authorized_keys
docker $ exit
# copie de la clé depuis le conteneur 1 dans un fichier local
$ docker cp conteneur_sftp_client:/root/.ssh/id_rsa.pub "C:\Users\bulam\OneDrive\Documents\testConnexionSftpAvecDocker\id_rsa.pub"
# création du répertoire devant accueillir la clé publique
$ docker exec -it conteneur_sftp_server mkdir -p /root/.ssh
# copie dans le conteneur 2 la clé qui a été enregistrée localement
$ docker cp "C:\Users\bulam\OneDrive\Documents\testConnexionSftpAvecDocker\id_rsa.pub" conteneur_sftp_server:/root/.ssh/id_rsa.pub

# connexion SFTP
$ docker exec -it conteneur_sftp_client sftp user1@conteneur_sftp_server
```

>NOTE : pour copier directement d'un conteneur à l'autre :

````bash
$ docker exec -i conteneur_sftp_client cat /root/.ssh/id_rsa.pub | docker exec -i conteneur_sftp_server sh -c 'cat > /root/.ssh/id_rsa.pub'
``
