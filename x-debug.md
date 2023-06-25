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

## Note concernant le php.ini

Wampserver configure la version de php utilisée par le webserver apache. Ce php.ini est affichée lors du lancement de la fonction phpinfo() : Loaded Configuration File	C:\wamp64\bin\apache\apache2.4.51\bin\php.ini

Danc ce fichier, xdebug est automatiquement configuré avec une version qui se trouve dans le répertoire `zend_ext`.
Il est possible d'y ajouter une autre version de xdebug téléchargée dans le répertoire ext du php utilisé :

``ìniù
;zend_extension="c:/wamp64/bin/php/php7.4.26/zend_ext/php_xdebug-3.1.1-7.4-vc15-x86_64.dll"  # xdebug configuré par Wampserver
zend_extension="C:/wamp64/bin/php/php7.4.26/ext/php_xdebug.dll"     # xdebug en version 3.1.6
```

Pour utiliser l'une ou l'autre version, il suffit de la décommenter et de commenter l'autre. A noter que commentre les deux désactive xdebug.

## Note concernant mysql

Pour accéder au mysql de Wampserver en ligne de commande, il faut d'abord se déplacer jusqu'au mysql.exe qui se trouve dans le répertoire bin de la version mysql utilisée. Par exemple, pour un mysql utilisé en version 8.0.27 dans Wampserver :

```bash
$ cd C:\wamp64\bin\mysql\mysql8.0.27\bin
$ mysql.exe -u webtraderUser -p
