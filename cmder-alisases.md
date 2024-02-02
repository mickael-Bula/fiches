# Ajout d'alias dans Cmder

Tous se passe dans le fichier `Cmder/config/user_aliases.cmd`. que l'on peut modifier dans un éditeur de code (comme VsCode).
Pour créer un alias pour phpStorm par exemple, à la fin du fichier, ajouter :

```cmd
rem mes aliases personnalisés
phs=phpstorm64.exe .
```

Ouvrir ensuite un **nouveau** terminal et saisir `phs` pour ouvrir le répertoire **courant** dans phpStorm.
