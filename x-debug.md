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
;zend_extension="chemin\absolu\vers\l\extension"
zend_extension="C:/wamp64/bin/php/php7.4.26/ext/php_xdebug.dll"
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

```ìni
;zend_extension="c:/wamp64/bin/php/php7.4.26/zend_ext/php_xdebug-3.1.1-7.4-vc15-x86_64.dll"  # xdebug configuré par Wampserver
zend_extension="C:/wamp64/bin/php/php7.4.26/ext/php_xdebug.dll"     # xdebug en version 3.1.6
```

Pour utiliser l'une ou l'autre version, il suffit de la décommenter et de commenter l'autre. A noter que commenter les deux désactive xdebug.

## Note concernant mysql

Pour accéder au mysql de Wampserver en ligne de commande, il faut d'abord se déplacer jusqu'au mysql.exe qui se trouve dans le répertoire bin de la version mysql utilisée. Par exemple, pour un mysql utilisé en version 8.0.27 dans Wampserver :

```bash
$ cd C:\wamp64\bin\mysql\mysql8.0.27\bin
$ mysql.exe -u webtraderUser -p
```

## INSTALLATION DE XDEBUG DANS LARAGON

Récupérer la version courante de php dans Laragon, disponible en suivant le lien `phpinfo()` depuis l'onglet `www`.
Aller sur la page de xdebug : https://xdebug.org/wizard
Passer le contenu du phpinfo() et suivre les instructions pour le téléchargement et l'enregistrement de la dll.
Activer l'extension xdebug dans Laragon en faisant un clic droit sur son icone dans la barre de tâche.
Renseigner les lignes suivantes dans le php.ini de la version courante :

```ini
[xdebug]
;zend_extension="chemin\absolu\vers\l\extension";
zend_extension="C:/laragon/bin/php/php-8.1.10-Win32-vs16-x64/ext/php_xdebug.dll"  ;renseigner le chemin vers la dll téléchargée précédemment
xdebug.mode=develop,debug,profile
xdebug.start_with_request=yes
xdebug.client_host="127.0.0.1"
xdebug.client_port=9003
xdebug.remote_handler=dbgp
xdebug.idekey=PHPSTORM
xdebug.remote_autostart=1
xdebug.profiler_enable=1
xdebug.output_dir="c:/laragon/www/xdebug"  ;ce répertoire doir être créé dans Laragon afin de recevoir les logs en provenance de xdebug
xdebug.show_local_vars=0
xdebug.log="c:/wamp64/logs/xdebug.log"
xdebug.log_level=7
xdebug.cli_color=2
;xdebug.mode=coverage
```

Puis, sélectionner PHP > Extensions > xdebug.
Relancer le serveur, puis naviguer à nouveau vers phpinfo() afin de vérifier que xdebug est bien installé.

## Lancer une session de debug Javascript dans phpstorm

Pour lancer une séance de debug Javascript depuis phpstorm :

- Run > Edit configurations : sélectionner Javascript debug
- Renseigner un nom (Javascript debug par exemple)
- url : http://localhost:8000 (l'url lancée avec le serveur symfony et le port utilisé)
- choisir le navigateur chrome
- Apply
- Mettre un point d'arrêt dans le code JS
- Lancer le serveur avec: $ symfony serve -d

Sélectionner la session de debug Javascript dans la fenêtre à gauche de la flèche verte : Javascript debug (le nom défini précédemment)

Activer Debug (icone du 'bug' s'active avec un point vert)

Recharger le navigateur : le point d'arrêt est atteint

## Lancer simultanément les sessions de debug PHP et JS

Pour ce faire, il faut que les configurations déclarées soient communes.
Par exemple, déclarer http://webtrader pour les configurations :

- Run > Debug configurations > Php Web Page => server = wampserver, start url = / (http://webtrader)
- Run > Debug configurations > Javascript Debug > URL = http://webtrader

Ensuite, dans le menu déroulant à gauche de la flèche verte (run) :
- choisir javascript debug (configuration js) et cliquer sur le bug vert (il s'allume) : une session du navigateur est lancée
- cliquer sur l'icone Start listening for PHP debug connection pour activer le debug php : le debug php écoute les connexions depuis la session ouverte

Recharger le navigateur : l'exécution du code est arrêtée lorsque les points d'arrêt js ET php sont touchés 😀

>NOTE : il est possible de voir le contenu de la console des DevTools dans le terminal de phpstorm sous le bin nommé onglet `console`.

## AJOUTER XDEBUG A UNE NOUVELLE VERSION DE PHP INSTALLÉE DANS LARAGON

Il faut télécharger les sources de la version de php.
Le fichier doit ensuite être décompressé dans le répertoire laragon/bin/php.

Pour ajouter xdebug à la version de php, il faut passer le contenu de la commande php -i sur la page de xdebug.
Cette version doit ensuite être téléchargée, puis placée dans le répertoire ext de la version de la nouvelle version de php.
Dans le fichier php.ini de cette version, il faut également renseigner le chemin vers le php_xdebug.dll.

Pour bénéficier du pas à pas dans phpstorm, il faut déclarer la version de php à utiliser en spécifiant :

- le chemin vers l'exécutable php (php.exe)
- le chemin vers le fichier de configuration (php.ini)
- le chemin vers le module xdebug (ext/x_debug.php)

>NOTE : Il faut veiller à activer les drivers des BDD dans le menu php > Extensions dans Laragon

## Configurer xdebug sur un serveur distant

Exemple de configuration de Xdebug dans WSL (distribution Ubuntu 20.04)

Pour me connecter au serveur Apache de WSL, j'utilise l'IP. Le port apache configuré est 8080 :
http://172.30.161.248:8080/info.php

Pour récupérer l'IP :

```bash
$ ip addr show eth0
```

Pour récupérer le port sur lequel écoute apache :

```bash
$ sudo nano /etc/apache2/ports.conf
```

Il faut configurer le service ssh correctement, en autorisant :

```bash
PermitRootLogin yes
PasswordAuthentication yes
```

Il faut penser à relancer le service ssh :

```bash
$ sudo service ssh restart
```

Il faut que xdebug soit installé pour la version de php utilisée par apache.

Il faut configurer le serveur WSL dans phpstorm : CTRL + ALT + S > CLI Interpreter > ... > + > Ajouter l'IP de WSL, ainsi que user et mdp

On peut laisser l'interpréteur php par défaut : usr/bin/php : phpstorm devrait résoudre l'exécutable et le fichier de configuration (php.ini) sur la base de ces infos.

Le test effectué avec un point d'arrêt dans le code de test (sans même que ce code soit présent localement) est concluant.

## Configuration de xdebug vers un serveur distant

>NOTE : Pour tester ce type de connexion, j'utilise une VM de mon homelab Proxmox.

Pour activer les connexions xdebug depuis un serveur distant vers mon phpstorm local et sur le réseau local,
il faut veiller à utiliser la découverte automatique du client pour s'affranchir des adresses IP dynamiques allouées par le router, surtout sur réseau WIFI :

- aller dans le fichier de configuration de xdebug sous `/etc/php/8.2/apache2/conf.d/20-xdebug.ini` et indiquer `xdebug.discover_client_host=1`
- configurer le PHP CLI pour utiliser celui du serveur distant : CTRL + ALT + S > PHP > CLI Interpreter > renseigner les informations en cliquant sur les ... : le debugger xdebug doit afficher sa version s'il est trouvé


- si après avoir mis un point d'arrêt, si celui-ci n'est pas atteint, s'assurer que le réseau est considéré comme sûr par le firewall de l'anti-virus.

## Xdebug dans WSL

À garder en tête

Sur Ubuntu, le début du `~/.bashrc` contient une garde `case $- in *i*) ;; *) return;; esac`. 
Un export placé après cette ligne n’est pas chargé par les shells non interactifs, ce qui est le cas quand PhpStorm lance un interpréteur via `wsl.exe`. 
Il faut placer l’export avant cette garde, ou dans `~/.profile`.

Pour un nouveau projet, il faut recréer le serveur « WSL » avec ses mappings dans ce projet. 
Si le message d’erreur réapparaît, ajouter `PHP_IDE_CONFIG=serverName=WSL` dans le template PHPUnit de ce projet.
Le template ne s’applique qu’aux configurations créées après sa modification. 
Si une configuration existante continue d’échouer, la supprimer ou ajouter la variable directement dessus.
Conserver le `PHP_IDE_CONFIG=serverName=WSL` du `.bashrc` pour les sessions lancées depuis le terminal.
