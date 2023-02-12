# exécution d'un script shell dans un container Docker

L'objet de ce programme est de pratiquer le *scripting shell* tout en approfondissant l'utilisation des containers Docker.

Pour ce faire, l'idée est de lancer une instance de linux (une image Ubuntu en l'occurence) et d'ajouter un partage de fichier afin que les scripts *shell* écrits sur l'hôte soient accessible depuis l'intérieur du container puis exécutés :

```docker
$ docker container run -v "//i/fichiers mika/docker/test1":/home/data/test -it --name ubuntu ubuntu
```

## explication des commandes

**run** : je lance le container, s'il n'existe pas localement, il est chargé depuis le hub

**-v** : pour --volume, permet de mapper un chemin sur l'hôte avec un autre à l'intérieur du container. A noter la transposition sous forme Linux du path Windows (//c/ pour C:\, les / pour les \, et les guillements utilisés pour que les espaces soient acceptés par Linux)

**-it** : ouverture d'un terminal interactif

**--name** : je nomme mon container pour qu'il soit plus facile à manipuler (ici ubuntu directement à la suite de l'option --name)

**ubuntu** : l'image à lancer (ou à puller si elle n'existe pas localement)

## accès au fichier mappé

Le fichier est créé préalablement sur l'hôte (son chemin est I:\fichiers mika\docker\test1\script.sh) :

```bash
#!/bin/sh
# This is a comment!
echo Hello World !       # This is a comment, too!
```

Dans le script, le shabang '#!' précise que ce programme doit être exécuté depuis le programme accessible depuis '/bin/sh'. C'est le path vers shell et/ou son alias bash.
Autrement dit, au lancement du script dans la console, c'est le fichier lui-même qui indique qu'il doit être exécuté dans le shell.

## lancement du script

Depuis le terminal ouvert dans le container :

```bash
$ cd /home/data/test1
$ ./script.sh
```

Dans la console, j'obtiens le résultat suivant :

```bash
$ Hello World !
```

Désormais, toute modification effectuée depuis l'hôte est reflétée dans le container !

Je peux donc ecrire mes scripts sur le pc, sous Windows, puis lancer leur exécution dans une instance de Linux hébergée dans un container Docker.

## Manipuler le container

Pour rappel, voici les commandes pour charger, lancer, arrêter et détruire le container :

```bash
$ docker ps                         # affiche les processus (containers) actifs
$ docker ps -a                      # affiche tous les containers, mêmes inactifs
$ docker start ubuntu               # permet de relancer le container préalablement créé
$ docker stop ubuntu                # arrêt du container sans le détruire
$ docker rm ubuntu                  # supprime le container
$ docker exec ubuntu -ti bin/bash   # exécute la commande bin/bash depuis un terminal interactif lancé dans le container
$ exit                              # permet de sortir du terminal lancé dans le container pour retourner dans le terminal hors Docker
```

## Remarque et bilan

Il faut ajouter la déclaration des volumes partagés dès la construction du container.
Autrement dit, on ne peut pas ajouter la déclaration des volumes à un container déjà créé.

Pour avoir une instance de linux avec un terminal ouvert lors d'un run, je dois ajouter l'option -ti. Inutile d'ajouter la commande 'bin/bash' :
$ docker run --name base-linux -ti ubuntu

Une fois créé, le container ouvre un terminal que je peux alors explorer.
Pour le quitter : $ exit

Si je veux retourner dans le container, il faut d'abord que je le relance :
$ docker start <container-name>

Ensuite, je dois explicitement déclaré l'ouverture d'un terminal dans le container, ainsi que la commande qui doit y être exécutée :
$ docker exec -ti <container-name> bin/bash

Il faut comprendre qu'un container n'est accessible qu'aussi longtemps que son processus principal est actif.
Autrement dit, on ne peut y lancer un terminal que s'il est actif, et il n'est actif que dans la mesure où un processus long s'y déroule.

> Bilan : il faut ajouter l'option -ti lors de la création du container pour que celui-ci ne soit pas arrêté juste après son lancement.
