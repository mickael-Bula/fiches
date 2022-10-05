# Connexion à MySQL sur le pcp portable

Il existe plusieurs instances MySQL disponibles sur le pc portable. L'une a été installée en dur, les autres ont été ajoutées lors de l'installation de Wampserver.

## accéder à l'instance original de MySQL

Pour accéder à mysql (en version 8) sur le laptop, il faut se connecter avec l'utilisateur 'root' et le mdp généré avec le nouveau pattern.
Dans un terminal, il faut lancer MySQL avec la commande :

```
$ mysql -u root -p
```

Au prompt, il faut fournir le mdp composé.

Cette version de mysql est la première version installée et utilise le port par défaut - original - 3306.

## accéder aux instances MySQL dze Wampserver

Pour se connecter aux bases mysql enregistrées dans wampserver, il faut d'abord lancer celui-ci puis se connecter sur le port 3308.
Il faut donc veiller à bien modifier la ligne configurant la BDD dans les fichers .env* en renseignant `port=3308`.
