# Installation de Vue dans une application Symfony

Les commandes et instructions suivantes permettent d'intégrer Vue en version 3 à une application Symfony.

## Installation d'un projet Symfony

```bash
$ symfony new symfony_vue --webapp
$ cd symfony_vue
$ composer require doctrine/annotations # pour corriger une erreur de paquet manquant
$ composer require symfony/webpack-encore-bundle
```

## Ajout de Vue 3

```bash
$ yarn add vue@next vue-loader@next @vue/compiler-sfc --dev # installe la dernière version de Vue
```

### Ajout du composant racine Vue

```js
// assets/js/app.js
import '../styles/app.css'; // ajoute le chemin vers le css (déplacé depuis l'ancien fichier assets/app.css)

import { createApp } from 'vue';
import Example from './Example.vue';    // importe le composant créé plus bas

const app = createApp({
    components: {
        Example
    }
});

app.mount("#app");

// assets/js/Example.vue
<template>
    <h1>Un exemple de composant Vue !</h1>
  </template>

  <script>
  export default {
    name: "Example",
  }
  </script>
```

## Modification du fichier webpack.config.js

```js
.addEntry('app', './assets/js/app.js')  // on modifie le chemin vers le fichier js principal
.enableVueLoader()  // on ajoute Vue à Webpack
```

On peut supprimer le fichier `assets/app.js` devenu inutile :

```bash
$ rm assets/app.js
```

## Ajout d'un controller et d'un template dans Symfony

```bash
$ php bin/console make:controller Home
```

```php
class HomeController extends AbstractController
{
    /**
     * @Route("/", name="app_home") // Ici je déclare la route comme étant à la racine de l'application ('/')
     */
    public function index(): Response
    {
        return $this->render('home/index.html.twig');
    }
}
```

```html
<!-- template/home/index.html.twig -->
<body>
  <div id="app">
    <Example />
  </div>
</body>
```

```bash
$ yarn encore dev   # lancement du build
$ symfony serve -d  # lancement du serveur de developpment
```

## NOTE

Lancer un serveur avec la commande `yarn dev-server` ne fonctionne pas. Celui-ci doit être lancé par Symfony.
En outre, pour vois les modifications apportées au code, il faut relancer le serveur et suivre le lien généré.
