# Montées de versions PHP et Symfony avec Rector

## Montée de version de Symfony

1 - Il faut commencer par régler les problèmes de code dépréciés signalés. Elles sont visibles avec la commande suivante :

```bash
$ php bin/console debug:container --deprecations
```

Par exemple, traiter l'avertissement suivant :
  `User Deprecated: Since symfony/framework-bundle 5.1: The "session.flash_bag" service is deprecated, use "$session->getFlashBag()" instead.`
  Pour ce faire, il faut injecter RequestStack en lieu et place de FlashBagInterface et utiliser la méthode :
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


# Migration Symfony 5.4 → 6.4 & PHP 7.4 → 8.2

Ce guide décrit les étapes pour mettre à jour une application Symfony 5.4 fonctionnant sous PHP 7.4 vers Symfony 6.4 et PHP 8.2.

---

## 🧭 Étapes générales

1. **Sauvegarder le projet**  
   - Créer une nouvelle branche `upgrade/symfony6-php82`
   - Sauvegarder la base de données et les fichiers importants

2. **Mettre à jour la version de PHP (local & CI)**  
   - Passer à PHP **8.2** sur ton environnement
   - Mettre à jour `.php-version` (si utilisé), Dockerfile, GitHub Actions, etc.

3. **Mettre à jour PHPStan**
   - Ajouter la version cible dans `phpstan.neon` :
     ```neon
     parameters:
         phpVersion: 80200
     ```

4. **Préparer le projet à la migration avec Rector**

---

## 🧰 Configuration de Rector

### Installation

```bash
composer require --dev rector/rector
```

### Fichier `rector.php`

À créer à la racine du projet :

```php
<?php

declare(strict_types=1);

use Rector\Config\RectorConfig;
use Rector\Set\ValueObject\LevelSetList;
use Rector\Symfony\Set\SymfonySetList;
use Rector\Set\ValueObject\SetList;

return static function (RectorConfig $rectorConfig): void {
    $rectorConfig->paths([
        __DIR__ . '/src',
        __DIR__ . '/tests',
    ]);

    $rectorConfig->sets([
        LevelSetList::UP_TO_PHP_82,
        SetList::TYPE_DECLARATION,
        SetList::EARLY_RETURN,
        SetList::CODE_QUALITY,
        SymfonySetList::SYMFONY_52,
        SymfonySetList::SYMFONY_54,
        SymfonySetList::SYMFONY_60,
        SymfonySetList::SYMFONY_61,
        SymfonySetList::SYMFONY_62,
        SymfonySetList::SYMFONY_63,
        SymfonySetList::SYMFONY_64,
    ]);

    $rectorConfig->phpVersion(\Rector\ValueObject\PhpVersion::PHP_82);

    $rectorConfig->skip([
        __DIR__ . '/var',
        __DIR__ . '/vendor',
    ]);
};
```

### Exécution

- Mode test :
  ```bash
  vendor/bin/rector process --dry-run
  ```

- Appliquer les modifications :
  ```bash
  vendor/bin/rector process
  ```

---

## 🧱 Mise à jour Symfony

### Mettre à jour les dépendances :

```bash
composer remove symfony/symfony
composer require symfony/flex --no-plugins
composer require symfony/framework-bundle:^6.4 symfony/runtime
```

Ensuite, mettre à jour les autres composants Symfony :

```bash
composer require symfony/*:^6.4 --update-with-dependencies
```

**💡 Conseil** : procéder par lots si tu rencontres des conflits.

---

## 🧹 Nettoyage du code déprécié

### 1. Identifier les dépendances bloquant PHP 8.2

```bash
composer why-not php 8.2
```

- Vérifie si certaines bibliothèques empêchent la mise à jour. Si oui :
  - Regarde s'il existe une version plus récente compatible.
  - Sinon, envisage de les remplacer ou de contribuer à leur mise à jour.

### 2. Identifier le code déprécié

```bash
php bin/console debug:container --deprecated
```

- Affiche les services utilisant des fonctionnalités dépréciées.

### 3. Nettoyage dans le code

- Lance l'application en environnement de développement : Symfony affiche automatiquement les dépréciations.
- Utilise le profiler Symfony pour analyser les appels dépréciés.
- Supprime ou adapte les usages concernés.

- Tu peux aussi activer l'affichage des dépréciations dans PHPUnit avec :
  ```xml
  <php>
    <ini name="error_reporting" value="-1"/>
  </php>
  ```

---

## 🔍 Vérifications à faire après la mise à jour

- ✅ S'assurer que toutes les dépendances sont compatibles PHP 8.2 et Symfony 6.4.
- ✅ Exécuter tous les tests (`phpunit` ou autre).
- ✅ Repasser `rector`, `phpstan` et `php-cs-fixer`.
- ✅ Regarder les fichiers `config/` : certains formats YAML/XML peuvent avoir changé.
- ✅ Vérifier les dépréciations (`bin/console deprecation:find` via SymfonyInsight ou `php bin/console debug:container --deprecated`).

---

## ❌ Ce qu'il ne faut PAS faire

- ❌ Ne pas ajouter `Rector` à GrumPHP (outil ponctuel, pas pour chaque commit).
- ❌ Ne pas oublier de fixer `phpVersion` dans `phpstan.neon`.
- ❌ Ne pas négliger les tests de régression après migration.

---

## ✅ Outils recommandés pour la migration

- [`rector/rector`](https://github.com/rectorphp/rector) pour automatiser le refactoring
- [`phpstan/phpstan`](https://github.com/phpstan/phpstan) pour l’analyse statique
- [`friendsofphp/php-cs-fixer`](https://github.com/FriendsOfPHP/PHP-CS-Fixer) pour le style de code
- [`symfony/monolog-bundle`](https://github.com/symfony/monolog-bundle) pour suivre les erreurs dans les logs
- [Symfony Upgrade Docs](https://symfony.com/doc/current/setup/upgrade_major.html)

---

## 🏁 Une fois terminé

- Supprimer `rector.php` et la dépendance `rector/rector` si plus utile
- Fusionner la branche sur `main` ou `master`
- Déployer sur un environnement de staging/test
