# Built-in documentation

## CD :

#### Objectif :

		Changer le repertoire courant du shell vers un autre dossier.

#### Fonctionnement :

		**1. Verifier que data et  data->env existent.**

		**2. Verifier si il y a <ins>plus d'un argument</ins>, affiche erreur :** cd: too many arguments., retourne 1 et ne change pas de repertoire. 

		**3. Si <ins>aucun argument</ins> avec cd :** Cherche la variable $HOME, **si absente affiche erreur :** cd: HOME not set. et retourne 1.

