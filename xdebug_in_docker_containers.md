# 🐞 Xdebug dans des containers Docker

## 🔧 Prérequis

Docker doit être installé sur le poste et activé.

Afin d'utiliser Xdebug sans se soucier de son installation, voici une configuration à base de containers Nginx et PHP-FPM.

Pour faire le debug d'un code, il faut que le présent repo soit installé à la racine du projet concerné.

## 📦 Installation des containers

```bash
$ git clone git@github.com:mickael-Bula/xdebug_in_containers.git .
$ docker-compose up -d --build
```

## 🌐 Accès à l'application

Nginx étant configuré pour écouter sur le port 8081, l'application est disponible à l'URL http://localhost:8081/

## 🐛 Exécution du debug

Aucune configuration particulière de PhpStorm n'est requise. Il suffit de mettre un point d'arrêt et d'activer l'écoute en cliquant sur le bouton **Start Listening for PHP Debug Connections**.

## ⚙️ Configuration de Xdebug dans le container

```ini
zend_extension=xdebug

[xdebug]
xdebug.mode=develop,debug
xdebug.client_host=host.docker.internal
xdebug.start_with_request=yes
```

**host.docker.internal** est un nom DNS spécial fourni par Docker Desktop et spécifique à l'environnement **Windows/macOS**.  
Il permet à un conteneur Docker d’atteindre l’hôte (la machine Windows/macOS), ce qui est utile pour Xdebug qui doit envoyer les connexions vers l’IDE de la machine hôte.

Sur **Linux**, `host.docker.internal` **n’existe pas par défaut**. À la place, il faut utiliser l’adresse IP réelle de l’hôte Linux (ex : `192.168.1.X`).

## 🧰 Installation de Xdebug dans l'image PHP-FPM

Pour une version de PHP spécifique, par exemple 7.4, il faut préciser la version de Xdebug compatible (ici `3.1.6`) :

```dockerfile
# ./docker/php/Dockerfile
FROM php:7.4-fpm

RUN apt-get update && apt-get install -y \
    libzip-dev unzip zip git curl \
    && docker-php-ext-install zip

# Spécifie la version compatible de Xdebug
RUN pecl install xdebug-3.1.6 \
    && docker-php-ext-enable xdebug
```

## 🧾 Configuration du container Nginx

Pour que Nginx puisse servir les fichiers PHP, il faut ajouter la configuration suivante dans le fichier **default.conf** :

```nginx
# ./docker/nginx/default.conf
server {
    listen 80;
    server_name localhost;

    root /var/www/html;
    index index.php index.html;

    location / {
        try_files $uri $uri/ =404;
    }

    location ~ \.php$ {
        include fastcgi_params;
        fastcgi_pass php:9000;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME /var/www/html$fastcgi_script_name;
    }
}
```

Ici, l'exposition du port 9000 sert à faire communiquer les containers Nginx et PHP-FPM.
Le port 9000 n’est donc pas là pour Xdebug : c’est en fait le port FastCGI de PHP-FPM.
Le process principal du conteneur php est php-fpm, qui écoute toujours sur 9000 (sauf configuration contraire).
Nginx, dans webserver, contient cette déclaration :

```ini
fastcgi_pass php:9000;
```

Il doit donc pouvoir atteindre ce port à l’intérieur du réseau Docker.
`expose: - 9000` rend ce port visible pour les autres services sans l’ouvrir vers l’extérieur, ce qui suffit.
