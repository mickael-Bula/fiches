# Création d'un Makefile pour les commandes Symfony

Permet de lancer les commandes symfony à l'aide de raccourcis déclarés dans le fichier Makefile :

```bash
$ make serve  # lance la commande symfony serve -d
```

## Installation de la commande make

Pour que les commandes déclarées dans le fichier Makefile soit reconnues par le terminal, il faut installer la commande make dans Windows.

L'installation est des plus simples en passant par scoop :

```cmd
> scoop install make
```

Pour vérifier que la commande est bien installée :

```cmd
> make --version
```

## Création d'un fichier Makefile

Dans un projet Symfony, ce fichier doit se trouver à la racine du projet et se nomme **Makefile** (avec une majuscule).

>NOTE : Dans le langage Make, l'indentation des commandes dans une règle doit se faire avec une tabulation (pas d'espaces)

Voici un exemple simple de fichier Makefile :

```make
# Symfony
serve:
	symfony serve -d

# Cleaning
clean:
	del /Q var\cache\* var\log\*
```

## Utilisation des commandes déclarées dans le fichier

Pour lancer les commandes avec make :

```bash
$ make serve
```

La sortie générée est alors :

```bash
symfony serve -d
```
