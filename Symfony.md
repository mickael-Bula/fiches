# Rassemble des informations sur le framework (fourre-tout)

### Pour connaître la version de Symfony utilisée :

```bash
$ php bin/console --version  # Symfony 6.4.12 (env: dev, debug: true)
```

### Pour vérifie si un composant est disponible dans un projet avant de l'installer :

```bash
$ composer show symfony/serializer # affiche des infos si le composant est déjà installé
```

### Pour vérifier la syntaxe des templates Twig

```bash
$ php bin/console lint:twig templates
```

Cette commande analyse tous les fichiers de templates (Twig) d'un projet.

**Ce qu'elle vérifie** : Elle s'assure que la syntaxe des fichiers `.html.twig` est correcte. Elle détectera, par exemple, une balise {% for %} jamais fermée, une parenthèse manquante, ou une structure de bloc mal placée.

**Pourquoi l'utiliser** : C'est indispensable dans un pipeline de CI/CD (Intégration Continue). Cela évite de déployer une page qui provoquerait une "Erreur 500" dès son affichage à cause d'une simple faute de frappe dans le template.

**Astuce** : Il est possible de cibler un dossier spécifique en ajoutant le chemin : `php bin/console lint:twig templates/emails`.

### Pour vérifier le container de service :

```bash
$ php bin/console ling:container
```

Cette commande vérifie la configuration du Service Container (le cœur de Symfony qui gère l'Injection de Dépendances).

Ce qu'elle vérifie : Elle compile le conteneur en interne pour vérifier que tous les services déclarés (via YAML, XML ou PHP) sont valides. Elle vérifie notamment si :

- Les classes injectées existent réellement.
- Les arguments passés aux constructeurs sont corrects.
- Il n'y a pas de références circulaires (le Service A a besoin du Service B, qui lui-même a besoin du Service A).
