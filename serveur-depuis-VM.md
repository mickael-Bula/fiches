# CONFIGURATION D'UN SERVEUR APACHE AU SEIN D'UNE VM VIRTUALBOX

Je m'exerce avec le tutoriel fourni ici : https://www.tutorialworks.com/linux-vm-vagrant/

Dans ce tutoriel, il s'agit de créer une machine virtuelle avec Virtualbox et Vagrant.

Le résultat du travail réalisé surla base de ce tutoriel se trouve disponible sur mon pc sous `C:\Users\bulam\Vagrant-ubuntu18`.
Après ouverture d'un Terminal, il faut lancer la commande `$ vagrant up` pour monter la VM. Une fois celle-ci provisionnée avec les fichiers récupérés depuis l'hôte, il faut lancer un navigateur - toujours de puis l'hôte - puis visiter l'url http://192.168.33.10/

```bash
$ cd C:\Users\bulam\                # je crée un répertoire vide pour accueillir mon projet
$ mkdir Vagrant-ubuntu18            
$ cd Vagrant-ubuntu18
$ vagrant init hashicorp/bionic64   # j'initialise un Vagrantfile pour monter une distribution ubuntu
$ ls -al                            # je m'assure de sa présence
$ code Vagrantfile                  # je lance VsCode pour configurer le réseau
```

Pour rendre le serveur accessible depuis l'extérieur de la VM, j'assigne une adresse ip spécifique en décommentant la ligne :

```ini
config.vm.network "private_network", ip: "192.168.33.10"    # j'ouvre sur l'extérieur
```

Ensuite, je lance ma VM :

```bash
$ vagrant up        # je lance la VM
$ vagrant ssh       # je m'y connecte
```

Une fois celle-ci lancée, je peux l'interroger depuis l'hôte et un navigateur en utilisant l'url `http://192.168.33.10` configurée dans le Vagrantfile.

Pour personnaliser la page présentée, je peux me rendre dans le Document Root pour modifier le contenu du fichier index.html :

```bash
vagrant@vagrant:~$ cd /var/www/html
vagrant@vagrant:~$ sudo nano index.html
```

Mais je peux aussi faire mes modifications depuis l'hôte, puis copier celles-ci dans la VM en faisant usage de la synchronisation entre le répertoire courant de l'app et le répertoire /vagrant de la VM.

Pour cela, après avoir créé ou récupéré le contenu (des pages html, des images...), je dois les déplacer au sein de la VM, du répertoire synchronisé vers le Document Root d'Apache :

```bash
$ cd C:\Users\bulam\Vagrant-ubuntu18
mkdir webContent
```

J'y ajoute les fichiers nécessaires à la création de ma page web. Ensuite, je retourne dans la VM pour servir les fichiers depuis Apache :

```bash
vagrant@vagrant:~$ sudo cp /vagrant/webcontent/* /var/www/html/
```

## automatisation des dernières tâches

Parce que le principe de la VM est de pouvoir être détruite et reconstruite à volonté, sa configuration permet de récupérer automatiquement les fichiers quin ont été transférés depuis l'hôte. Pour cela, il faut décommenter et modifier le Vagrantfile : 

```yaml
  config.vm.provision "shell", inline: <<-SHELL
    apt-get update
    apt-get install -y apache2
    cp -r /vagrant/webContent/* /var/www/html/
    echo "Machine virtuelle provisionnéele $(date)! Bienvenue!"
  SHELL
end
```

Pour vérifier que le Vagrantfile est conforme, depuis l'hôte et placé à la racine du projet contenant le Vagrantfile :

```bash
$ vagrantfile validate
```

Si tout est correct, la réponse est `Vagrantfile validated successfully.`

Il est alors possible d'arrêter, détruire et reconstruire la VM tout en récupérant de manière automatique les pages web depuis l'hôte !

```bash
$ vagrant destroy
$ vagrant up
```
