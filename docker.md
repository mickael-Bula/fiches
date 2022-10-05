# utilisation de docker pour utiliser une base de donnée postgresql

Il faut d'abord que docker soit lancé.
Ensuite, on lance un symfony/create-project où l'on accepte l'intégration de docker.

Si cela n'est pas fait, il est possible de créer le fichier docker-compose.yaml avec la commande :

```bash
$ symfony console make:docker:database
```

## ajout de l'extension pgsql

Si postgresql est accessible via docker, il faut encore  ajouter le driver PDO_pgsql pour que php puisse faire les migrations.

Cette extension est à décommenter dans le fichier php.ini de la version utilisée.
Pour voir les modules :
$ symfony php -m
Pour déterminer le fichier de la version courante :
$ symfony php --ini
Le fichier à modifier est celui qui n'est ni dev ni prod
