# Ajout de l'extension RabbitMQ

Pour suivre le libre **En Route pour Symfony 5**, j'ai dû installé RabbitMQ.

L'extension correspondant à ma version de Windows est la suivante : amqp 1.11.0 for Windows, en version **7.4 Thread Safe (TS) x64** (cette dernière information est disponible dans la sorite de la fonction phpinfo() à la ligne :

```
Thread Safety	enabled
```

Pour procéder à l'installation, j'ai téléchargé le répertoire, puis copié les fichiers **php_amqp.dll** sous C:\wamp64\bin\php\php7.4.26\ext
et **rabbitmq.4.dll** sous C:\wamp64\bin\php\php7.4.26

>NOTE : copier les 2 dll sous ext ne permet pas d'activer l'extension.

Après installation des extensions, le résultat de la commande `symfony book:check-requirements` est la suivante :

```bash
[OK] Git installed
[OK] PHP installed version 7.4.26 (C:\wamp64\bin\php\php7.4.26\php.exe)
[OK] PHP extension "xsl" installed - required
[OK] PHP extension "openssl" installed - required
[OK] PHP extension "zip" installed - optional - needed only for chapter 17 (Panther)
[OK] PHP extension "gd" installed - optional - needed only for chapter 23 (Imagine)
[OK] PHP extension "amqp" installed - optional - needed only for chapter 32
[OK] PHP extension "json" installed - required
[OK] PHP extension "intl" installed - required
[OK] PHP extension "pdo_pgsql" installed - required
[OK] PHP extension "mbstring" installed - required
[OK] PHP extension "sodium" installed - required
[OK] PHP extension "curl" installed - optional - needed only for chapter 17 (Panther)
[OK] PHP extension "session" installed - required
[OK] PHP extension "ctype" installed - required
[OK] PHP extension "xml" installed - required
[OK] PHP extension "tokenizer" installed - required
[OK] PHP extension "redis" installed - optional - needed only for chapter 31
[OK] Composer installed
[OK] Docker installed
[OK] Docker Compose installed
[OK] Yarn installed
```
