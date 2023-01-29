# Résolution de conflit depuis phpStorm

Lorsqu'un conflit apparaît suite à une commande ```git```, que celle-ci ait été saisie depuis le menu de l'IDE ou en console, il est possible d'accéder à l'outil de résolution de conflit de phpStorm.
Pour cela, il suffit d'aller dans le menu Git > Resolve conflicts pour voir la fenêtre dédiée apparaître.
Dans celle-ci, il est possible d'accepter les modifications de l'une ou l'autre source, ou encore d'ouvrir une nouvelle fenêtre présentant trois versions du fichier concerné.
Pour ouvrir celle-ci, il suffit de cliquer sur le bouton **merge**. Cette fenêtre contient :

- à gauche, le code local
- à droite, le code distant
- au centre, le fichier en écriture où les versions différentes doivent être corrigées.


Une fois les conflits résolus, il faut faire un ```commit``` pour conserver les modifications, voire un ```push``` si la copie distante doit être mise à jour.
