# mise en place de grumphp

## Installation

```bash
$ composer require --dev phpro/grumphp friendsofphp/php-cs-fixer symfony/yaml phpunit/phpunit
$ vendor\bin\grumphp configure
$ vendor\bin\grumphp git:init
```

## Exécution de grumphp

```bash
$ vendor\bin\grumphp run  # lancer grumphp
$ vendor\bin\grumphp run --files path/to/your/file.php  # lancer un fichier particulier
$ vendor\bin\grumphp list # affiche la liste des tâches disponibles
```

## Exemple de configuration personnalisée

```yaml
parameters:
    git_dir: .
    bin_dir: vendor/bin
    tasks:
        phpcsfixer:
            config: .php_cs.dist
        phpunit:
            config_file: phpunit.xml.dist
        yamllint:
            parse_php: false
            syntax:
                - symfony
            directories:
                - src
                - config
        phpstan:
            level: max
            autoload_file: vendor/autoload.php
        jsonlint:
            ignore_patterns:
                - "vendor/*"
```

>NOTE : j'ai dû modifier la version de php (et y ajouter xdebug) pour corriger l'erreur suivante :
> 
>`Error: Your version of PHP is affected by serious garbage collector bugs related to fibers. Please upgrade to a newer version of PHP, i.e. >= 8.1.17 or => 8.2.4`

Après avoir téléchargé et configuré la nouvelle version de php, j'ai pu lancer `grumphp` sans erreurs.

### Troubleshooting avec grumphp

J'ai rencontré une difficulté dans la validation des commits faits depuis phpstorm.
En effet, malgré une exécution réussie de la commande `vendor\bin\grumphp run`,
le commit des modifications depuis les outils phpstorm n'a pas fonctionné.
L'erreur signalée reprenait l'erreur suivante :

```bash
`Error: Your version of PHP is affected by serious garbage collector bugs related to fibers. Please upgrade to a newer version of PHP, i.e. >= 8.1.17 or => 8.2.4`
```

Or, la version de php utilisée est bien >= à 8.1.17.

J'ai pu passer outre le problème en exécutant le commit depuis la console.
Il semblerait donc qu'il y ait une différence entre les versions utilisées pour commiter entre console et phpstorm...

#### Correction des variables d'environnement pour commiter depuis phpstorm

Avec l'ajout de **grumphp** qui nécessite l'utilisation d'une version de php >= 8.1.17,
je me suis aperçu que l'ajout d'un commit depuis phpstorm était mal configurée.
En effet, l'outil git de phpstorm récupère la version de php dans le PATH pour lancer les étapes de pré-commit.
Or, la version déclarée dans le PATH était 8.1.10.

Après modification de la version de php déclarée dans le PATH system et utilisé par git,
j'ai pu faire mes commits avec la bonne version de php et sans rencontrer d'erreur.

#### Exclure certains répertoires de l'analyse

La vérification du code devant se concentrer sur le code produit à l'exclusion du code tiers,
la manière de procéder est la suivante :

```yml
    excludePaths:
        analyse:
            - tests/Support
```

Ceci permet de ne pas couvrir la partie des tests qui n'a pas été écrite,
mais qui doit être utilisée pour la découverte des symboles.

## Exemple de configuration complète

```yaml
// grumphp
grumphp:
    git_hook_variables:
        EXEC_GRUMPHP_COMMAND: 'vendor\bin\grumphp'
    stop_on_failure: false
    process_timeout: 60
    tasks:
        phpstan:
            level: 5
            autoload_file: vendor/autoload.php
        phpunit:
            config_file: phpunit.xml.dist
        phpcsfixer:
            config: .php-cs-fixer.dist.php
            allow_risky: true
            diff: true
            verbose: true
        yamllint: ~
        phplint:
            triggered_by: [ 'php' ]
            exclude: [ 'vendor' ]
        phpmd: # PHP Messe Detector
            ruleset: [ 'phpmd.xml' ]
            exclude: [ 'tests' ]
            triggered_by: [ 'php' ]
        phpmnd: #PHP Magic Number Detector
            exclude_path: ['vendor', 'var']
```

## Exemple de configuration de phpmd.xml

```xml
// phpmd.xml
<?xml version="1.0"?>
<ruleset name="Règles personnalisées">
    <description>Mes règles personnalisées</description>

    <rule ref="rulesets/cleancode.xml">
        <exclude name="StaticAccess" />
        <exclude name="MissingImport" />    <!-- Cette règle entre en contradiction avec les vérifications de php-cs-fixer -->
        <exclude name="BooleanArgumentFlag" />
        <exclude name="IfStatementAssignment" />
    </rule>
    <!-- J'autorise l'accès aux super globales pour permettre l'exécution du code depuis le PHAR -->
    <rule ref="rulesets/controversial.xml">
        <exclude name="Superglobals" />
    </rule>
    <rule ref="rulesets/design.xml">
        <exclude name="CouplingBetweenObjects" />
    </rule>
    <rule ref="rulesets/naming.xml">
        <exclude name="LongVariable" />
        <exclude name="ShortVariable" />
    </rule>
    <rule ref="rulesets/unusedcode.xml">
    <exclude name="UnusedFormalParameter" />
    </rule>
    <!-- Lève la limite de numberOfChildren pour accepter tous les enfants de la classe AbstractMigrationCommand -->
    <rule ref="rulesets/design.xml/NumberOfChildren">
        <properties>
            <property name="minimum" value="30"/>
        </properties>
    </rule>
</ruleset>
```
