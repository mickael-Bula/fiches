# INSTALLATION DE PLUSIEURS VERSIONS DE PHP POUR SYMFONY

Pour bénéficier de la possibilité de choisir notre version de php, voici une procédure :
- télécharger la version désirée ici : https://windows.php.net/download/
- ajouter le path dans les variables d'environnement : cmd modifier les variables > path > modifier > parcourir <path vers new php>
- vérifier la liste des versions disponible pour symfony : $ symfony local:php:list
- à la racine du projet, édition d'un fichier contenant la version à utiliser :
  

```bash
$ echo 7.3.13 > .php-version	# modifier la version en conséquence
```

Dès lors, on peut demander la version désirée à symfony :

```bash
$ symfony php -v	# 7.3.13
```

En éditant ce fichier `.php-version`, on détermine la version de php à utiliser.

Pour lancer un serveur de prod, on préfixe simplement avec 'symfony' :
  
```bash
$ symfony php -S localhost:8080
```

Pour utiliser cette version sélectionnée avec composer :
  
```bash
$ symfony composer <action>
```

sources : https://atranchant.developpez.com/tutoriels/php/version-windows/

##### NOTE : il faut veiller à ce que le fichier php.ini soit bien présent (en renommer une copie depuis php.ini-development si nécessaire). Dans ce fichier, il faut décommenter la ligne 'extension=openssl'
  
## INSTALLATION D'UNE NOUVELLE VERSION DE PHP SUR WAMPSERVER
  
Les sources se trouvent sur https://wampserver.aviatechno.net/
  
La procédure est des plus simples, puisqu'il suffit de sélectionner la version de php désirée pour ensuite lancer l'installateur.
  Une fois installée, la version de php se trouve disponible, sans qu'il soit nécessaire de mettre à jour les variables d'environnement.
  Pour preuve, il suffit de lancer la commande : 
  
  ```bash
  $ symfony local:php:list
  ```
  
  > NOTE : il faut veiller à relancer le terminal pour que le système soit mis à jour
  
