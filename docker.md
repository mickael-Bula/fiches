# Prise en main de Docker

> NOTE : cette fiche est rédigée en utilisant un pc sous windows 10 et après avoir installé Docker Desktop

## le cycle de vie d'un container Docker

Pour accéder aux commandes Docker, il est d'abord nécessaire de lancer l'appli Docker Desktop.

Ensuite, depuis un terminal, on peut lancer un container, par exemple nginx : 

```bash
$ docker run -pd 8000:80 nginx
```

Cette commande lance un container s'il existe déjà, sinon le télécharge avant de le lancer.

Voici les commandes essentielles au cycle de vie d'un container :

- run :     lance un container et au besoin télécharge une image pour ensuite le lancer
- create :  crée un container à partir d'une image, mais sans le démarrer
- start :   démarre un container préalablement téléchargé
- stop :    arrête un container
- kill :    stoppe le container de manière plus rapide mais aussi plus brutale
- rm :      supprime un container

## identifier un container

Pour récupérer l'ID d'un container, la commande est :

```bash
docker ps
```

Cette commande ne liste que les processus actifs. Pour lister tous les processus, la commande est :

```bash
$ docker ps -a
```

## lancer un terminal dans le container

Il est possible de lancer un pseudo-terminal au sein même d'un container actif afin d'y lancer des commandes ou de le parcourir :

```bash
$ docker exec -ti 8e45 /bin/bash
```

> Il est possible de lancer la commande ```bash``` directement, c'est-à-dire sans le ```bin/```

Pour quitter ce pseudo-terminal, la commande est :

```bash
$ exit
```

