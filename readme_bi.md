# Built-in documentation

## CD :

### **<ins>ms_bi_cd :</ins>**

1.	Verification de **`data`** et **`data->env`**.
	- Si il n'y a rien, retourne **`1`**.
		
2.	Verification du nombre d'arguments.
	- Si il y a plus d'un argument, affiche l'erreur **"`cd: too many arguments`"** et retourne **`1`**.

3.	**`oldpwd`** devient le chemin courant.
	- Si ca echoue, affiche une erreur avec **"`perror("getcwd")`"** et retourne **`1`**.
		
4.	**`path`** devient le chemin cible a faisant appel a **`ms_cd_get_path`**.
	- Si ca echoue, retourne **`1`**.

5.	Changement de repertoire avec **`chdir(path)`**.
	- Si ca echoue, free **`oldpwd`** (malloc avec la fonction **`getpwd`**), affiche une erreur avec **"`perror("cd")`"** et retourne **`1`**.
		
6.	Mise a jour de **`OLDPWD`** en faisant appel a **`ms_update_oldpwd`**.

7.	Mise a jour de **`PWD`** en faisant appel a **`ms_update_pwd`**.

8.	Free **`oldpwd`** (malloc avec la fonction **`getpwd`**)

9.	Retourne **`0`**.

### **<ins>ms_cd_get_path :</ins>**

1.	Si il y a un argument :
	- Si cet argument est **`-`**, **`path`** devient **`OLDPWD`** avec la commande **`getenv("OLDPWD")`**.
	  - Si ca echoue, free **`oldpwd`** (malloc avec **`getpwd`**), affiche l'erreur "**`cd: OLDPWD not set`**" et retourne **`NULL`**.
		- Sinon on affiche la valeur de **`OLDPWD`** et on retourne cette valeur comme le chemin cible.
	  - Si l'argument est autre chose que **`-`**, retourne l'argument en tant que chemin cible.

2.	Si il n'y a pas d'argument, **`path`** devient **`HOME`** avec **`getenv("HOME")`**.
	- Si **`HOME`** n'existe pas, free **`oldpwd`** (malloc avec **`getpwd`**), affiche l'erreur "**`cd: HOME not set`**" et retourne **`NULL`**.
	- Sinon retourne la valeur de **`HOME`** comme chemin cible.

### **<ins>ms_update_oldpwd :</ins>**

1.	Verification de **`data`**, **`data->env`** et **`oldpwd`**.
	- Si il n'y a rien, retourne **`1`**.

2.	**`new_oldpwd`** fait appel a **`ft_strjoin`** pour construire la chaine **`OLDPWD=`** avec **`oldpwd`**.
	- Si ca echoue, retourne **`1`**.

3.	Chercher **`OLDPWD=`** dans **`data->env[i]`**.
	- Si **`PWD=`** est trouve :
	  - Free l'ancienne valeur de **`data->env[i]`**.
	  - **`data->env[i]`** devient **`new_oldpwd`** (La nouvelle ancienne cible).
	  - 3.3	Retourne **`0`** ("SUCCES").

4.	Si **`OLDPWD`** n'existe pas dans **`data->env`** :
	- **`new_env`** fait appel a **`ft_realloc_env`** pour copier **`data->env`** et ajouter **`OLDPWD`** dans **`new_env`**.
	  - Si ca echoue, free **`mew_env`** (malloc dans **`ft_realloc_env`**) et retourne **`1`**.

5.	**`data->env`** devient **`new_env`** (la copie avec **`OLDPWD`**) et retourne **`0`** pour dire que tout s'est bien passe.

### **<ins>ft_realloc_env :</ins>**

1.	Compte le nombre d'arguments dans **`env`** avec pour index **`i`**.

2.	**`new_env`** fait appel a **`malloc`** pour allouer la taille de **`env`** + **`2`** (une place pour **`new_var`** et l'autre pour **`NULL`**).
	- Si ca echoue, retourne **`NULL`**.
3.	Copie **`env`** dans **`new_env`**.

4.	Ajoute **`new_var`** a **`new_env[i]`** et **`NULL`** a **`new_env[i + 1]`**.

5.	Free l'ancien tableau (**`env`**) et retourne **`new_env`**.

### **<ins>ms_uptade_pwd :</ins>**

1.	Verification de **`data`** et **`data->env`**.
	- Si il n'y a rien, retourne **`1`**.
		
2.	**`cwd`** devient le chemin courant avec **`getcwd`**.
	- Si ca echoue, affiche une erreur avec "**`perror("getcwd")`**" et retourne **`1`**.
		
3.	Remplacement du chemin de **`PWD`** par le chemin de **`cwd`** avec la fonction **`ms_replace_pwd_in_env`**.

4.	Free **`cwd`** (malloc avec **`getcwd`**) et retourne **`res`** (0) si tout a bien fonctionne.

### **<ins>ms_replace_pwd_in_env :</ins>**

1.	Chercher **PWD=** dans **data->env[i]**.
	- Si **`PWD=`** est trouve :
	  - **`new_pwd`** devient la nouvelle chaine construite avec **`ft_strjoin`**.
		- Si ca echoue, retorune **`1`**.
		- Free l'amcienne valeur de **`data->env[i]`**.
		- **`data->env[i]`** devient la nouvelle chaine (**`new_pwd`**).

2.	Retourne **`0`** pour dire que tout s'est bien passe.


## ECHO :

### **<ins>ms_bi_echo :</ins>**

1.	Initialisation de **`i`** a **`1`** pour ignorer le nom de la commande, et **`new_line`** a **`1`** pour ajouter par defaut un retour a la ligne.

2.	Cherche si il y a un (ou plusieurs) **`-n`** avec la fonction **`check_n_args`**.
	- Si il y a au moin un **`-n`**, met **`new_line`** a **`0`** pour retirer le retour a la ligne.
		
3.	Parcours **`args[i]`** pour afficher les arguments restants.
	- Si il y a un argument qui suit le precedent, ajoute un espace pour les separer.
		
4.	Ajout du retour a la ligne si **`new_line`** = **`1`**.

5.	Retourne **`0`**.

### **<ins>check_n_args :</ins>**

1.	Verification de **`str`** et que **`str[0]`** est **`-`**.
	- Si il n'y a rien ou que **`str[0]`** n'est pas **`-`**, retourne **`0`**.

2.	Parcours **`str`** tant que **`str[i]`** = **`n`**.
	- Si **`str[i]`** n'est pas **`n`**, retourne **`0`**.

3.	Retourne **`1`** si la chaine correspond a **`-n`** ou **`-nnnnnnnn...`**.


## ENV :

### **<ins>ms_bi_env :</ins>**

1.	Verification de **`data`** et **`data->env`**.
	- Si il n'y a rien, retourne **`1`**.
	
2.	Verification du nombre d'arguments.
	- Si il y a plus que **`env`**, affiche l'erreur "**`env: too many arguments`**" et retourne **`1`**.

3.	Parcours le tableau **`env`**.
	- Affiche les variables contenant **`=`**.

4. Retourne **`0`** pour dire que tout s'est bien passe.


## PWD :

### **<ins>ms_bi_pwd :</ins>**

1.	Initialisation du tableau **`cwd[PATH_MAX]`** (Taille maximale d'un chemin).

2.	Cherche le chemin absolu du repertoire courant avec **`getcwd`**.
	- Si **`getcwd`** fonctionne, l'affiche sur la sortie standard avec **`ft_putendl_fd`** et retourne **`0`** pour dire "SUCCES".

3.	En cas d'echec, retourne **`1`**.


## EXIT :

### **<ins>ms_bi_exit :</ins>**

1.	Affiche **`exit`** comme le vrai bash.

2.	Si il n'y a pas d'arguments, termine avec **`exit(0)`**.

3.	Si un **`+`** ou un **`-`** est trouve, incremente **`i`**.

4.	Verifie que les arguments sont bien des chiffres.
	- Si un caractere non numerique est trouve, affiche l'erreur "**`exit: <args>: numeric arguments required`**" et termine avec **`exit(255)`**.

5.	Verifie qu'il n'y a pas plus d'un argument apres **`exit`**.
	- Si il y a plus d'un argument, affiche l'erreur "**`exit: too many arguments`**" et retourne **`1`** sans quitter immediatement.
	
6.	Verifie que ma chaine ne depasse pas d'un **`long long`** en utilisant **`ms_atoll`**.
	- Si ca echoue ou que ca deborde, affiche l'erreur "**`exit: %s: numeric argument required`**" et termine avec **`exit(255)`**.

7.	Quitte avec **`exit((unsigned char)exit_code)`** caste en **`unsigned char`** pour etre dans la plage **`0 - 255`**.

### **<ins>ms_atoll :</ins>**

1.	Verification que la chaine entre dans un **`long long`** avec **`ms_check_count`**.
	- Si ca depasse, retourne **`0`**.

2.	Verification si **`*str`** est un **`-`** ou un **`+`**.
	- Si **`*str`** est un **`-`**, on modifie **`sign`** pour appliquer le signe negatif.
	- Avance le pointeur pour ignorer le signe.

3.	Conversion des caracteres.
	- Si un caractere non numerique est trouve, retourne **`0`**.
	- Si **`res`** depasse **`LLONG_MAX`**, retourne **`0`**.
	- **`res`** devient **`res * 10 + (*str - '0')`**
	- On recommence l'operation jusqu'a ce au'on ne puisse plus.

4.	**`*result`** devient **`res * sign`**.

5.	Retourne **`1`** pour dire que c'est ok.

### **<ins>ms_check_count :</ins>**

1.	Ignore les signes **`-`** et **`+`**.

2.	Compte le nombre de caracteres numeriques.

3.	Si le nombre de caracteres numeriques depasse **`20`**, retourne **`0`** pour dire que ca depasse.
	- Sinon retourne **`1`** pour dire que c'est ok.


## EXPORT :

### **<ins>ms_bi_export :</ins>**