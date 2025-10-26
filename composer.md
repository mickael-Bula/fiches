## Configuration de l'exécutable composer

Pour lancer composer depuis le bandeau supérieur à l'ouverture du composer.json :

CTRL+ALT+S > PHP > Composer > Execution : je spécifie le chemin vers le composer.phar de Laragon :

`C:\laragon\bin\composer\composer.phar`

## Commandes utiles

Vérifier les mises à jour disponibles :

```bash
$ composer outdated
```

Mettre à jour un paquet spécifique :

```bash
$ composer update symfony/symfony
```

Mettre à jour toutes les dépendances :

```bash
$ composer update
```

pour mettre à jour une dépendance, la solution la plus efficace est de mettre à jour vers une version corrigée,
en exécutant la commande suivante :

```bash
$ composer update symfony/http-client --with-all-dependencies  # pour mettre à jour avec ses dépendances
```

Voir pourquoi une dépendance est installée :

```bash
$ composer why symfony/http-client
```

La commande `composer why` donne des informations sur les dépendances et les contraintes de version liées à un paquet. Exemple :

```bash
$ composer why symfony/http-client
__root__                 dev-main requires  symfony/http-client (6.4.*) 
symfony/framework-bundle v6.4.10  conflicts symfony/http-client (<6.3)
symfony/http-kernel      v6.4.17  conflicts symfony/http-client (<5.4)
symfony/security-bundle  v6.4.13  conflicts symfony/http-client (<5.4)
```

Décryptage des lignes : `__root__ dev-main requires symfony/http-client (6.4.*)`
**__root__** : représente le projet principal.
**dev-main** : indique la branche actuelle du projet.
**requires symfony/http-client (6.4.*)** : le projet dépend explicitement de symfony/http-client dans la version 6.4.*, comme indiqué dans votre composer.json.
**conflicts symfony/http-client (<6.3)** : symfony/framework-bundle version 6.4.10 est incompatible avec toute version de symfony/http-client inférieure à 6.3.

Conflit géré automatiquement : Le paquet framework-bundle impose des contraintes pour éviter que symfony/http-client ne soit installé dans une version incompatible (<6.3). Composer gère ces conflits en installant une version compatible.

Pour comprendre pourquoi une version spécifique de symfony/http-client n'est pas installée, utiliser :

```bash
$ composer why-not <paquet> <version>
```

Pour vérifier les dépendances avec des vulnérabilités connues :

```bash
$ composer audit
```

Pour s'assurer que la version installée est exempte de vulnérabilités :

```bash
$ composer show symfony/http-client
```

## TROUBLESHOOTINGS

Dans le cas d'une erreur survenant au lancement s'une commande composer :
```bash
C:\laragon\www\webtrader-stimulus
λ composer self-update
The following exception indicates a possible issue with the Avast Firewall
Check https://getcomposer.org/local-issuer for details

In CurlDownloader.php line 394:
                                                                                                                                     
  curl error 60 while downloading https://getcomposer.org/versions: SSL certificate problem: unable to get local issuer certificate
```

La solution pourrait être de désactiver la vérification https de l'anti-virus.
Pour cela, aller dans Explore > Web Shield > Open Web Shield > Décocher l'option https scanning
