# Built-in documentation

## CD :

#### ms_bi_cd :

1.	Verification de data et data->env.\
		Si il n'y a rien, retourne 1.
		
2.	Verification du nombre d'arguments.\
		Si il y a plus d'un argument, affiche l'erreur "cd: too many arguments" et retourne 1.

3.	"oldpwd" devient le chemin courant.\
		Si ca echoue, affiche une erreur avec "perror("getcwd")" et retourne 1.
		
4.	"path" devient le chemin cible a faisant appel a "ms_cd_get_path".\
		Si ca echoue, retourne 1.

5.	Changement de repertoire avec "chdir(path)".\
		Si ca echoue, free "oldpwd" (malloc avec la fonction "getpwd"), affiche une erreur avec "perror("cd")" et retourne 1.
		
6.	Mise a jour de "OLDPWD" en faisant appel a "ms_update_oldpwd".

7.	Mise a jour de "PWD" en faisant appel a "ms_update_pwd".

8.	Free "oldpwd" (malloc avec la fonction "getpwd")

9.	Retourne 0.

#### ms_cd_get_path :

1.	Si il y a un argument :\
		Si cet argument est "-", "path" devient "OLDPWD" avec la commande "getenv("OLDPWD")".\
			Si ca echoue, free "oldpwd" (malloc avec "getpwd"), affiche l'erreur "cd: OLDPWD not set" et retourne NULL.
			Sinon on affiche la valeur de "OLDPWD" et on retourne cette valeur comme le chemin cible.\
		Si l'argument est autre chose que "-", retourne l'argument en tant que chemin cible.

#### ms_update_oldpwd :

		1.	Verification de data, data->env et oldpwd.
				Si il n'y a rien, retourne 1.

		2.	"new_oldpwd" fait appel a "ft_strjoin" pour construire la chaine "OLDPWD=" avec "oldpwd".
				Si ca echoue, retourne 1.

		3.	Chercher "OLDPWD=" dans "data->env[i]".
				Si "PWD=" est trouve :
					3.1	Free l'ancienne valeur de "data->env[i]".
					3.2	"data->env[i]" devient "new_oldpwd" (La nouvelle ancienne cible).
					3.3	Retourne 0 ("SUCCES").

		4.	Si "OLDPWD" n'existe pas dans "data->env" :
				"new_env" fait appel a "ft_realloc_env" pour copier "data->env" et ajouter "OLDPWD" dans new_env.
					Si ca echoue, free "mew_env" (malloc dans "ft_realloc_env") et retourne 1.

		5.	"data->env" devient "new_env" (la copie avec "OLDPWD") et retourne 0 pour dire que tout s'est bien passe.

#### ft_realloc_env :

		1.	Compte le nombre d'arguments dans "env" avec pour index "i".

		2.	"new_env" fait appel a "malloc" pour allouer la taille de "env" + 2 (une place pour "new_var" et l'autre pour NULL).
				Si ca echoue, retourne NULL.
		3.	Copie "env" dans "new_env".

		4.	Ajoute "new_var" a "new_env[i]" et NULL a "new_env[i + 1]".

		5.	Free l'ancien tableau ("env") et retourne "new_env".

#### ms_uptade_pwd :

		1.	Verification de data et data->env.
				Si il n'y a rien, retourne 1.
		
		2.	"cwd" devient le chemin courant avec "getcwd".
				Si ca echoue, affiche une erreur avec "perror("getcwd")" et retourne 1.
		
		3.	Remplacement du chemin de "PWD" par le chemin de "cwd" avec la fonction "ms_replace_pwd_in_env".

		4.	Free "cwd" (malloc avec "getcwd") et retourne "res" (0) si tout a bien fonctionne.

#### ms_replace_pwd_in_env :

		1.	Chercher "PWD=" dans "data->env[i]".
				Si "PWD=" est trouve :
					1.1	"new_pwd" devient la nouvelle chaine construite avec "ft_strjoin".
						Si ca echoue, retorune 1.
					1.2	Free l'amcienne valeur de "data->env[i]".
					1.3	"data->env[i]" devient la nouvelle chaine ("new_pwd").

		2.	Retourne 0 pour dire que tout s'est bien passe.
