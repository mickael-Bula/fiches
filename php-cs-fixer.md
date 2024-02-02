## Configuration de php-cs-fixer

Pour bénéficier de la correction automatique des `Coding Standards` de PHP dans Symfony, je le télécharge :

```bash
$ composer require friendsofphp/php-cs-fixer
```

Depuis les Settings (CTR+ALT+S) > PHP | Quality Tools | PHP CS Fixer : j'active le module et je choisis :
Configuration > System PHP > (...) > définition du path vers le .bat : `C:\laragon\www\codeception-tdd\vendor\bin\php-cs-fixer.bat`

> NOTE : il faut ajouter la version de php dans le path des variables d'environnement System,
> sinon php ne sera pas disponible pour php-cs-fixer

Pour personnaliser les règles de php-cs-fixer, il faut les ajouter dans le fichier `.php-cs-fixer.php` à la racine :

```php
return (new Config())
    ->setRules(
        [
            '@Symfony' => true,
            'strict_param' => true,
            'array_syntax' => ['syntax' => 'short'],
        ]
    )
    ->setFinder($finder)
;
```
