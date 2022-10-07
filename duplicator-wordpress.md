# clonage de projet Wordpress

## sur le pc source

Pour transférer un projet Wordpress développé localement sur le pc vers le laptop, j'ai procédé comme ceci :

- sur le site Wordpress à migrer, installation de l'extension Duplicator > Créer un paquet > indiquer le site source > Scan 
- récupération des fichiers Installer.php et Archive depuis le fichier Téléchargements

Il faut ensuite copier ces fichiers (ne pas décompresser Archive) sur un support (HDD externe, carte SD...) ou via FTP.

## sur le pc de destination

- OPTIONNEL : créer une BDD pour accueillir les tables de Wordpress, avec un utilisateur dédié. Conserver les identifiants nécessaire pour la suite (cf NOTE 2)
- Créer un répertoire pour accueillir les fichiers (par exemple dans Wampserver/www).
- avec un serveur lancé (ou dans Wampserver), naviguer vers le localhost/<répertoire>
- cliquer sur installer.php pour lancer l'installation.
- fournir les identifiants de la BDD créée précédemment, ou fournir les identifants root locaux dans le cas d'une installation locale de développement.

## connexion au site cloné

En suivant le lien vers l'administration Wordpress fourni à la fin de l'installation, saisir les identifiants de connexion du site original.
Le site est cloné !

NOTE : pour un clonage sur le laptop, une version antérieure de mysql occupant le port 3306, l'utilisation du moteur fourni par Wampserver n'est pas accessible.
Il faut fournir les identifiants de connexion du moteur mysql 8.0 précédemment installé.
Une configuration avec un Wordpress sur le localhost de Wampserver et une BDD hors de cette appli est tout à fait fonctionnelle.

NOTE 2 : Si la BDD n'est pas créée antérieurement à la décompression des fichiers, l'installateur s'en chargera (il suffit de choisir l'option CREATE DATABASE).
