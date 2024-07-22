# Tagger une version

## Mettre à jour la branche master

faire le merge de la dernière MR dans develop
merge de develop dans master avec une MR
Fusionner la demande

## Déplacer le tag

suppression du tag 3.1.3
	$ git tag -d <SHA>
	$ git push origin :refs/tags/3.1.3
création d'un nouveau tag après récupération du SHA du commit de référence avec git log
	$ git tag 3.1.3 <new_commit_SHA>
	$ git push tag 3.1.3

Mettre à jour le changelog
