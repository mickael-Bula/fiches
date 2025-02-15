# Configuration de phpUnit dans phpstorm

Si lors de tests focntionnels le Kernel de Symfony n'est pas trouvé,
il est possible de le déclarer dans le fichier `phpunit.xml` comme ceci :

```xml
    <php>
        <ini name="display_errors" value="1" />
        <ini name="error_reporting" value="-1" />
        <server name="APP_ENV" value="test" force="true" />
        <server name="APP_DEBUG" value="1" />
        <server name="KERNEL_CLASS" value="App\Kernel" />  <!-- chemin vers le Kernel -->
        <server name="SHELL_VERBOSITY" value="-1" />
        <server name="SYMFONY_PHPUNIT_REMOVE" value="" />
        <server name="SYMFONY_PHPUNIT_VERSION" value="9.6" />
    </php>
```

Il peut également être utilse de déclarer le chemin vers le fichier phpunit.xml dans la configuration de phpstorm :

Settings > php > Test frameworks : dans testRunner, resnseigner dans Default **configuration file** le chemin absolu vers phpunit.xml ou phpunit.xml.dist,
comme par exemple `C:\laragon\www\test_auth\phpunit.xml.dist`.

## Base de données de test

Si la base de données de test a été déclarée dans un fichier `.env.test` et que la variable **APP_ENV** l'est dans le fichier phpunit.xml,
alors il n'est pas utile de déclarer la base dans le xml. En effet, le fichier `.env.test` est prioritaire: 
Symfony charge automatiquement le fichier  `.env.test` lorsqu'il détecte que l'environnement est configuré sur test.
Les variables d'environnement définies dans ce fichier écrasent celles du fichier `.env` principal.
