Pour récupérer facilement le contenu de la fonction phpinfo(), il est possible de lance la commande suivante :

```bash
php -i >> ./php_ini.txt
```

Le contenu sera enregistré dans le fichier php_ini.txt dans le répertoire courant.

## Activation de xdebug dans wampserver

Pour bénéficier de xdebug dans Wampserver, il faut activer l'option dans le manager tray.
Après un clic sur celui-ci > PHP > Configuration PHP > Extension Zend xdebug > xdebug.mode=debug

Il faut également que le fichier php.ini de la version de php utilisée dans Wampserver soit correctement configuré.
>NOTE : il vaut mieux naviguer dans l'arborescence que suivre le raccourci mis à disposition dans le menu Wampserver

Voici le contenu de mon php.ini pour lequel le pas à pas est disponible dans phpstorm :

```ini

[Xdebug]
;zend_extension="chemin\absolu\vers\l\extension";
zend_extension="C:/wamp64/bin/php/php7.4.26/ext/php_xdebug.dll";
xdebug.mode=develop,debug,profile
xdebug.start_with_request=yes 
xdebug.client_host="127.0.0.1"
xdebug.client_port=9003
xdebug.remote_handler=dbgp
xdebug.idekey=PHPSTORM
xdebug.remote_autostart= 1
xdebug.output_dir ="c:/wamp64/tmp"
xdebug.show_local_vars=0
xdebug.log="c:/wamp64/logs/xdebug.log"
xdebug.log_level=7
xdebug.cli_color=2
;xdebug.mode=coverage
```
