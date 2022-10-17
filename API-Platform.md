# installation d'API Platform en utilisant php 7.4

J'ai procédé comme suit pour lier pai platform et Symfony :

```bash
$ symfony new apiPlatform
$ cd apiPlatform\
$ code .
$ symfony serve -d
$ symfony composer require api
$ symfony composer require --dev symfony/maker-bundle
$ symfony console make:entity
```

Lors de l'étape de création de l'entité, j'ai ajouté cette annotation :

```php

use ApiPlatform\Core\Annotation\ApiResource;

/**
 * @ApiResource()
 */
```

Pour que les ressources soient affichées dans l'interface, j'ai également ajouté cette instruction :

```yaml
# config/package/api_platform.yaml

api_platform:
    metadata_backward_compatibility_layer: false
```

Après ces commandes, les entités déclarées dans Doctrine se trouvent disponibles sur l'interface api platform sur la route `/api`.
Ce n'est que dans un second temps que la connexion à la BDD devient nécessaire :

```bash
$ cp .env .env.local                        # veiller à mettre le portr 3308 si on se trouve sur le laptop
$ symfony console doctrine:database:create
$ symfony console doctrine:schema:create    # cette ligne est optionnelle puisqu'aucune table n'a encore été créée
$ symfony console make:migration
$ symfony console d:m:m
```

## installation du profiler Symfony

Pouie débugger plus aisèment notre api, et contre toute attente, il est possible d'utiliser la debug-toolbar.

```bash
$ symfony composer require profiler --dev
```

## création api resource

Les entités sont bien reconnues d'une certaine manière par api platform puisque je peux les manipuler depuis l'interface.
Mais même dans la dernière version d'api platform et en utilisant symfony 6, aucune entity n'est reconnue comme une `api resource`.
L'utilisation du flag `make:entity --api-resource` reste lui aussi inefficace.
Lorsque l'on clic sur api platform dans la debug toolbar, les ressources ne sont pas reconnues...
