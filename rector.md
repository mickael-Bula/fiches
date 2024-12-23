# Montées de versions PHP et Symfony avec Rector

## Montée de version de Symfony

1 - Il faut commencer par régler les problèmes de code dépréciés signalés. 

Par exemple, traiter l'avertissement suivant :
  `User Deprecated: Since symfony/framework-bundle 5.1: The "session.flash_bag" service is deprecated, use "$session->getFlashBag()" instead.`
  Pour ce faire, il faut remplacer l'injection de RequestStack en lieu et place de FlashBagInterface et utiliser la méthode :
  `$this->requestStack->getCurrentRequest()?->getSession()`

2 - Modifier :
```
  "extra": {
  "symfony": {
  "allow-contrib": false,
  "require": "6.0.*"  # préciser ici la nouvelle version
  }
  ```

3 - Dans `composer.json`, remplacer toutes les occurrences de la version actuelle par le nouveau numéro de version :
```
  "symfony/asset": "6.0.*",
  "symfony/console": "6.0.*",
```

4 - Exécuter la commande `composer update`

5 - Vérifier les dépréciations restantes pour correction : `php bin/console debug:container --deprecations`

## Rector

Pour l'installer :

```bash
$ composer require rector/rector --dev
```

Générer le fichier de configuration de Rector :

```bash
$ php vendor/bin/rector
```

## Configuration de Rector

Il faut compléter le fichier généré en y ajoutant les règles disponibles [ici](https://getrector.com/documentation)

Exemple :

```php
<?php

declare(strict_types=1);

use Rector\Config\RectorConfig;
use Rector\Symfony\Set\SymfonySetList;
use Rector\Set\ValueObject\LevelSetList;

return RectorConfig::configure()
    ->withPaths([
                    __DIR__ . '/config',
                    __DIR__ . '/public',
                    __DIR__ . '/src',
                    __DIR__ . '/tests',
                ])
    // Remplace les annotations par des attributs PHP
    ->withAttributesSets(
        symfony:    true,
        doctrine:   true,
        sensiolabs: true,
    )
    // Règles supplémentaires pour améliorer la qualité du code
    ->withPreparedSets(
        deadCode:           true,
        codeQuality:        true,
        codingStyle:        true,
        typeDeclarations:   true,
        earlyReturn:        true,
        symfonyCodeQuality: true,
    )
     // Les méthodes suivantes ne sont à utiliser qu'une fois par montée de version
    ->withSets([LevelSetList::UP_TO_PHP_81,])
    ->withSets([SymfonySetList::SYMFONY_60,])
    ;
```

## Exécuter Rector

Pour vérifier les modifications :

```bash
$ php vendor/bin/rector process --dry-run
```

Pour appliquer les modifications :

```bash
$ vendor/bin/rector process
```
