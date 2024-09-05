# Fiche Cmder

## Configuration du terminal Laragon dans phpstorm

Le terminal configuré avec Laragon est Cmder.
Or, Cmder est déjà installé sur mon poste et configuré dans phpstorm.
Pour bénéficier du scope de Laragon (qui est une machine virtuelle), je dois donc reconfigurer mon terminal.

Pour ce faire, je commence par créer un fichier C:\laragon\bin\cmder\phpstorm.bat avec le contenu suivant :

```bat
@echo off
SET CurrentWorkingDirectory=%CD%
SET CMDER_ROOT=C:\laragon\bin\cmder
CALL "%CMDER_ROOT%\vendor\init.bat"
CD /D %CurrentWorkingDirectory%
```

Je commence par libérer le nom de la variable d'environnement système que j'avais déclarée pour Cmder : CMDER_ROOT.
En effet, cette variable est utilisée par le fichier init.bat de Cmder.

Ce fichier est ensuite appelé au lancement du terminal dans phpstorm, grâce à la configuration suivante :

Dans le menu PHPStorm Settings > Tools > Terminal je déclare : "cmd.exe" /k "C:\laragon\bin\cmder\phpstorm.bat"

## Ajout d'alias dans Cmder

Tous se passe dans le fichier `Cmder/config/user_aliases.cmd`. que l'on peut modifier dans un éditeur de code (comme VsCode).
Pour créer un alias pour phpStorm par exemple, à la fin du fichier, ajouter :

```cmd
rem mes aliases personnalisés
phs=phpstorm64.exe .
```

Ouvrir ensuite un **nouveau** terminal et saisir `phs` pour ouvrir le répertoire **courant** dans phpStorm.

Un autre alias utile pour afficher la liste du contenu d'un répertoire en y ajoutant les droits et la coloration :

```cmd
ll=ls -alh --show-control-chars -F --color $*
```

>NOTE : Ajouter $* à la fin de l'alias permet de transmettre tous les arguments supplémentaires à la commande ls,
>ce qui signifie que l'on peut toujours utiliser les options habituelles de ls avec cet alias.

## Ajouter le Cmder de Laragon dans le menu contextuel de Windows

Si le raccourci de Cmder fournit par Laragon ne fonctionne pas depuis le menu contextuel (clic droit dans un répertoire),
il faut lancer l'exécutable du Cmder avec les droits administrateur puis saisir dans le terminal :

```bash
# l'exécutable Cmder se trouve dans le répertoire de laragon\bin\cmder
cmder /REGISTER ALL
```

