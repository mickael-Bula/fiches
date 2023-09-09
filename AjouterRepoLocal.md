# Commandes pour pousser sur Github un projet local

Il faut au ^réalable que le projet soit suivi par git

```bash
$ git init
$ git add .
$ git commit -m "initial commit"
```

Il faut ensuite créer un nouveau projet sur Github pour accueillir le projet local. Cette étape est à réaliser sur le site de Github.
Une fois le repo créer, on récupère son chemin (ssh ou https). Puis, de retour dans le projet local :

```bash
$ git remote add origin <lien ssh>
$ git push origin main
```
