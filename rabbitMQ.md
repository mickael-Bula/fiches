# Ajout de l'extension RabbitMQ

Pour suivre le libre **En Route pour Symfony 5**, j'ai dû installé RabbitMQ.

L'extension correspondant à ma version de Windows est la suivante : amqp 1.11.0 for Windows, en version **7.4 Thread Safe (TS) x64** (cette dernière information est disponible dans la sorite de la fonction phpinfo() à la ligne :

```
Thread Safety	enabled
```

Pour procéder à l'installation, j'ai téléchargé le répertoire, puis copié les fichiers **php_amqp.dll** sous C:\wamp64\bin\php\php7.4.26\ext
et **rabbitmq.4.dll** sous C:\wamp64\bin\php\php7.4.26

>NOTE : copier les 2 dll sous ext ne permet pas d'activer l'extension.
