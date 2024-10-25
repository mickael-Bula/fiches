# Rassemble des informations sur le framework (fourre-tout)

Pour connaître la version de Symfony utilisée :

```bash
$ php bin/console --version  # Symfony 6.4.12 (env: dev, debug: true)
```

Pour vérifie si un composant est disponible dans un projet avant de l'installer :

```bash
$ composer show symfony/serializer # affiche des infos si le composant est déjà installé
```
