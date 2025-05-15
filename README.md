# Built-in documentation

## **Sommaire**
|      Built-in      |    Tableau des tests    |
|:------------------:|:-----------------------:|
| [CD](#cd)          | [CD](#test-cd)          |
| [ECHO](#echo)      | [ECHO](#test-echo)      |
| [ENV](#env)        | [ENV](#tes-env)         |
| [PWD](#pwd)        | [PWD](#test-pwd)        |
| [EXIT](#exit)      | [EXIT](#test-exit)      |
| [UNSET](#unset)    | [UNSET](#test-unset)    |
| [EXPORT](#export)  | [EXPORT](#test-export)  |

---

## CD

## **Resume :**

**`cd`** permet de changer le repertoire courant du shell grace a **`chdir()`** qui permet de faire le changement de dossier.\
Met egalement a jour automatiquement les variables d'environnement **`$OLDPWD`** (ancien repertoire) et **`$PWD`** (nouveau repertoire) en utilisant **`getcwd()`**.
- **`cd`** sans arguments : va dans le dossier contenu dans **`$HOME`**.
- **`cd <path>`** va vers le chemin specifie.
- **`cd -`** revient au dernier repertoire contenu dans **`$OLDPWD`** et l'affiche.

---

### **ms_bi_cd :**

```c
int	ms_bi_cd(char **args, t_data *data)
```

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

---

### **ms_cd_get_path :**

```c
static char	*ms_cd_get_path(char **args, char *oldpwd)
```

1.	Si il y a un argument :
    - Si cet argument est **`-`**, **`path`** devient **`OLDPWD`** avec la commande **`getenv("OLDPWD")`**.
      - Si ca echoue, free **`oldpwd`** (malloc avec **`getpwd`**), affiche l'erreur "**`cd: OLDPWD not set`**" et retourne **`NULL`**.
        - Sinon on affiche la valeur de **`OLDPWD`** et on retourne cette valeur comme le chemin cible.
      - Si l'argument est autre chose que **`-`**, retourne l'argument en tant que chemin cible.

2.	Si il n'y a pas d'argument, **`path`** devient **`HOME`** avec **`getenv("HOME")`**.
    - Si **`HOME`** n'existe pas, free **`oldpwd`** (malloc avec **`getpwd`**), affiche l'erreur "**`cd: HOME not set`**" et retourne **`NULL`**.
    - Sinon retourne la valeur de **`HOME`** comme chemin cible.

---

### **ms_update_oldpwd :**

```c
static int	ms_update_oldpwd(t_data *data, char *oldpwd)
```

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

---

### **ft_realloc_env :**

```c
char	**ms_realloc_env(char **env, char *new_var)
```

1.	Compte le nombre d'arguments dans **`env`** avec pour index **`i`**.

2.	**`new_env`** fait appel a **`malloc`** pour allouer la taille de **`env`** + **`2`** (une place pour **`new_var`** et l'autre pour **`NULL`**).
    - Si ca echoue, retourne **`NULL`**.
3.	Copie **`env`** dans **`new_env`**.

4.	Ajoute **`new_var`** a **`new_env[i]`** et **`NULL`** a **`new_env[i + 1]`**.

5.	Free l'ancien tableau (**`env`**) et retourne **`new_env`**.

---

### **ms_uptade_pwd :**

```c
static int	ms_update_pwd(t_data *data)
```

1.	Verification de **`data`** et **`data->env`**.
    - Si il n'y a rien, retourne **`1`**.
        
2.	**`cwd`** devient le chemin courant avec **`getcwd`**.
    - Si ca echoue, affiche une erreur avec "**`perror("getcwd")`**" et retourne **`1`**.
        
3.	Remplacement du chemin de **`PWD`** par le chemin de **`cwd`** avec la fonction **`ms_replace_pwd_in_env`**.

4.	Free **`cwd`** (malloc avec **`getcwd`**) et retourne **`res`** (0) si tout a bien fonctionne.

---

### **ms_replace_pwd_in_env :**

```c
static int	ms_replace_pwd_in_env(t_data *data, char *cwd)
```

1.	Chercher **PWD=** dans **data->env[i]**.
    - Si **`PWD=`** est trouve :
      - **`new_pwd`** devient la nouvelle chaine construite avec **`ft_strjoin`**.
        - Si ca echoue, retorune **`1`**.
        - Free l'amcienne valeur de **`data->env[i]`**.
        - **`data->env[i]`** devient la nouvelle chaine (**`new_pwd`**).

2.	Retourne **`0`** pour dire que tout s'est bien passe.

---
- [Retourner au sommaire](#sommaire)
---

## ECHO

## **Resume :**

En cours d'ecriture.

---

### **ms_bi_echo :**

```c
int	ms_bi_echo(char **args)
```

1.	Initialisation de **`i`** a **`1`** pour ignorer le nom de la commande, et **`new_line`** a **`1`** pour ajouter par defaut un retour a la ligne.

2.	Cherche si il y a un (ou plusieurs) **`-n`** avec la fonction **`check_n_args`**.
    - Si il y a au moin un **`-n`**, met **`new_line`** a **`0`** pour retirer le retour a la ligne.
        
3.	Parcours **`args[i]`** pour afficher les arguments restants.
    - Si il y a un argument qui suit le precedent, ajoute un espace pour les separer.
        
4.	Ajout du retour a la ligne si **`new_line`** = **`1`**.

5.	Retourne **`0`**.

---

### **check_n_args :**

```c
static int	check_n_args(char *str)
```

1.	Verification de **`str`** et que **`str[0]`** est **`-`**.
    - Si il n'y a rien ou que **`str[0]`** n'est pas **`-`**, retourne **`0`**.

2.	Parcours **`str`** tant que **`str[i]`** = **`n`**.
    - Si **`str[i]`** n'est pas **`n`**, retourne **`0`**.

3.	Retourne **`1`** si la chaine correspond a **`-n`** ou **`-nnnnnnnn...`**.

---
 - [Retourner au sommaire](#sommaire)
---

## ENV

## **Resume :**

En cours d'ecriture.

---

### **ms_bi_env :**

```c
int	ms_bi_env(char **args, t_data *data)
```

1.	Verification de **`data`** et **`data->env`**.
    - Si il n'y a rien, retourne **`1`**.
    
2.	Verification du nombre d'arguments.
    - Si il y a plus que **`env`**, affiche l'erreur "**`env: too many arguments`**" et retourne **`1`**.

3.	Parcours le tableau **`env`**.
    - Affiche les variables contenant **`=`**.

4. Retourne **`0`** pour dire que tout s'est bien passe.

---
- [Retourner au sommaire](#sommaire)
---

## PWD

## **Resume :**

En cours d'ecriture.

---

### **ms_bi_pwd :**

```c
int	ms_bi_pwd(void)
```

1.	Initialisation du tableau **`cwd[PATH_MAX]`** (Taille maximale d'un chemin).

2.	Cherche le chemin absolu du repertoire courant avec **`getcwd`**.
    - Si **`getcwd`** fonctionne, l'affiche sur la sortie standard avec **`ft_putendl_fd`** et retourne **`0`** pour dire "SUCCES".

3.	En cas d'echec, retourne **`1`**.

---
- [Retourner au sommaire](#sommaire)
---

## EXIT

## **Resume :**

En cours d'ecriture.

---

### **ms_bi_exit :**

```c
int	ms_bi_exit(char **args, t_data *data)
```

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

---

### **ms_atoll :**

```c
static int	ms_atoll(const char *str, long long *result)
```

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

---

### **ms_check_count :**

```c
static int	ms_check_count(const char *str)
```

1.	Ignore les signes **`-`** et **`+`**.

2.	Compte le nombre de caracteres numeriques.

3.	Si le nombre de caracteres numeriques depasse **`20`**, retourne **`0`**.
    - Sinon retourne **`1`** pour dire que c'est ok.

---
- [Retourner au sommaire](#sommaire)
---

## UNSET

## **Resume :**

En cours d'ecriture.

---

### **ms_bi_unset :**

```c
int	ms_bi_unset(char **args, t_data *data)
```

Explication en cours d'ecriture.

---

### **is_valid_key**

```c
int	is_valid_key(const char *str)
```

Explication en cours d'ecriture.

---

### **remove_env_var**

```c
static void	remove_env_var(char ***env, const char *key)
```

Explication en cours d'ecriture.

---

### **env_len**

```c
static int	env_len(char **env)
```

Explication en cours d'ecriture.

---

### **key_matches**

```c
static int	key_matches(const char *env_entry, const char *key)
```

Explication en cours d'ecriture.

---
- [Retourner au sommaire](#sommaire)
---

## EXPORT

## **Resume :**


---

### **ms_bi_export :**



---
- [Retourner au sommaire](#sommaire)
---

## Tableau des test built-in

### Emoji

✅ || ❌

---

### Test CD

| Commande                           | Géré      | Sortie attendue                                                                                      | Code de sortie | Test |
|------------------------------------|:---------:|------------------------------------------------------------------------------------------------------|:--------------:|:----:|
| `cd`                               | Oui       | Se rendre dans `$HOME`                                                                               | 0              |  ❌  |
| `cd .`                             | Oui       | Reste dans le même répertoire                                                                        | 0              |  ❌  |
| `cd ./`                            | Oui       | Idem                                                                                                 | 0              |  ❌  |
| `cd ./././.`                       | Oui       | Idem                                                                                                 | 0              |  ❌  |
| `cd ..`                            | Oui       | Aller dans le dossier parent                                                                         | 0              |  ❌  |
| `cd ../`                           | Oui       | Idem                                                                                                 | 0              |  ❌  |
| `cd ../..`                         | Oui       | Monter deux niveaux                                                                                  | 0              |  ❌  |
| `cd .././././.`                    | Oui       | Idem                                                                                                 | 0              |  ❌  |
| `cd srcs`                          | Oui       | Aller dans le dossier `srcs`                                                                         | 0              |  ❌  |
| `cd srcs objs`                     | Oui       | Trop d’arguments                                                                                     | 1              |  ❌  |
| `cd 'srcs'`                        | Oui       | Aller dans `srcs` avec simple quote                                                                  | 0              |  ❌  |
| `cd "srcs"`                        | Oui       | Aller dans `srcs` avec double quote                                                                  | 0              |  ❌  |
| `cd '/etc'`                        | Oui       | Aller dans `/etc`                                                                                    | 0              |  ❌  |
| `cd /e'tc'`                        | Oui       | Aller dans `/etc` avec mix quotes                                                                    | 0              |  ❌  |
| `cd /e"tc"`                        | Oui       | Idem                                                                                                 | 0              |  ❌  |
| `cd sr`                            | Oui       | Dossier inexistant                                                                                   | 1              |  ❌  |
| `cd Makefile`                      | Oui       | N’est pas un dossier                                                                                 | 1              |  ❌  |
| `cd ../../../../../../..`          | Oui       | Atteint `/`                                                                                          | 0              |  ❌  |
| `cd ../minishell`                  | Oui       | Revient dans le dossier initial                                                                      | 0              |  ❌  |
| `cd _`                             | Oui       | Dossier inexistant                                                                                   | 1              |  ❌  |
| `cd -`                             | Oui       | Retour à `$OLDPWD`                                                                                   | 0              |  ❌  |
| `cd --`                            | NON GERE  | Devrait aller dans `$HOME`                                                                           | 0              |  ❌  |
| `cd ---`                           | Oui       | Option invalide                                                                                      | 2              |  ❌  |
| `cd $HOME`                         | Oui       | Aller dans le HOME                                                                                   | 0              |  ❌  |
| `cd $HOME $HOME`                   | Oui       | Trop d’arguments                                                                                     | 1              |  ❌  |
| `cd "$PWD/srcs"`                   | Oui       | Aller dans srcs à partir de PWD                                                                      | 0              |  ❌  |
| `cd '$PWD/srcs'`                   | Oui       | Chemin interprété littéralement                                                                      | 1              |  ❌  |
| `unset HOME` + `cd $HOME`          | Oui       | `$HOME` non défini                                                                                   | 1              |  ❌  |
| `unset HOME` + `export HOME` + `cd` | Oui      | `$HOME` non défini (variable exportée sans valeur)                                                   | 1              |  ❌  |
| `cd minishell Docs crashtest.c`    | Oui       | Trop d’arguments                                                                                     | 1              |  ❌  |
| `cd /`                             | Oui       | Aller dans la racine                                                                                 | 0              |  ❌  |
| `cd ~/`                            | NON GERE  | Aller dans HOME                                                                                      | 0              |  ❌  |
| `cd *`                             | NON GERE  | Comportement dépend du nombre de résultats dans le dossier courant                                   | 0 ou 1         |  ❌  |
| `mkdir a && mkdir a/b && cd a/b && rm -r ../../a && cd ..` | Oui       | Échec du `cd ..` car le dossier parent a été supprimé                        | 1              |  ❌  |
| `cd /minishell`                    | Oui       | Dossier inexistant                                                                                   | 1              |  ❌  |
| `export CDPATH=/ && cd home/vietdu91` | NON GERE  | Utilisation de `CDPATH` pour trouver un chemin relatif                                            | 0              |  ❌  |
| `export CDPATH=./ && cd ..`        | NON GERE  | CDPATH dans le contexte local                                                                        | 0              |  ❌  |

---
- [Retourner au sommaire](#sommaire)
---

### Test ECHO

| Commande                       | Géré      | Sortie attendue                                              | Code de sortie |  Test  |
|--------------------------------|:---------:|--------------------------------------------------------------|:--------------:|:------:|
| `echo hola`                    | Oui       | `hola`                                                       | 0              |   ❌   |
| `echo -n hola`                 | Oui       | `hola` (sans retour à la ligne)                              | 0              |   ❌   |
| `echo -nnnnn hola`             | Oui       | `hola` (sans retour à la ligne)                              | 0              |   ❌   |
| `echo -n -n -n hola`           | Oui       | `hola` (sans retour à la ligne)                              | 0              |   ❌   |
| `echo`                         | Oui       | Ligne vide (juste un retour à la ligne)                      | 0              |   ❌   |
| `echo -n`                      | Oui       | Ligne vide (sans retour à la ligne)                          | 0              |   ❌   |
| `echo -n -n`                   | Oui       | Ligne vide (sans retour à la ligne)                          | 0              |   ❌   |
| `echo -n -n -n`                | Oui       | Ligne vide (sans retour à la ligne)                          | 0              |   ❌   |
| `echo -n -nn -n -n hola`       | Oui       | `-nn -n -n hola` (seul le premier `-n` est interprété)       | 0              |   ❌   |
| `echo -nnn -nnnn -nn hola`     | NON GERE  | `-nnn -nnnn -nn hola`                                        | 0              |   ❌   |
| `echo -nnnnnnn hola`           | NON GERE  | `-nnnnnnn hola`                                              | 0              |   ❌   |
| `echo hola`                    | Oui       | `hola`                                                       | 0              |   ❌   |
| `echo -n hola`                 | Oui       | `hola` (sans retour à la ligne)                              | 0              |   ❌   |
| `echo hola que tal`            | Oui       | `hola que tal`                                               | 0              |   ❌   |
| `echo hola ""`                 | Oui       | `hola`                                                       | 0              |   ❌   |
| `echo ""`                      | Oui       | Ligne vide                                                   | 0              |   ❌   |
| `echo ''`                      | Oui       | Ligne vide                                                   | 0              |   ❌   |
| `echo "" hola`                 | Oui       | ` hola`                                                      | 0              |   ❌   |
| `echo '' hola`                 | Oui       | ` hola`                                                      | 0              |   ❌   |
| `echo hola "" hola`            | Oui       | `hola  hola`                                                 | 0              |   ❌   |
| `echo hola '' hola`            | Oui       | `hola  hola`                                                 | 0              |   ❌   |
| `echo hola """" hola`          | Oui       | `hola  hola`                                                 | 0              |   ❌   |
| `echo hola ''"" hola`          | Oui       | `hola  hola`                                                 | 0              |   ❌   |
| `echo hola ""'' hola`          | Oui       | `hola  hola`                                                 | 0              |   ❌   |

---
- [Retourner au sommaire](#sommaire)
---

### Test ENV

| Commande                 | Géré      | Sortie attendue                                          | Code de sortie | Test |
|--------------------------|:---------:|----------------------------------------------------------|:--------------:|:----:|
| `env`                    | Oui       | Affiche toutes les variables d’environnement             | 0              | ❌   |
| `env hola`               | NON GERE  | `env: ‘hola’: No such file or directory`                 | 127            | ❌   |
| `env -i`                 | NON GERE  | Aucune variable affichée (environnement vide)            | 0              | ❌   |
| `env -i hola=bonjour`    | NON GERE  | Aucune sortie affichée                                   | 0              | ❌   |
| `env -i hola=bonjour env`| NON GERE  | Affiche `hola=bonjour`                                   | 0              | ❌   |
| `env hola=bonjour`       | NON GERE  | Lance `bonjour` avec une variable `hola` (bash behavior) | 127            | ❌   |
| `env -i env`             | NON GERE  | N’affiche rien (aucune variable)                         | 0              | ❌   |

---
- [Retourner au sommaire](#sommaire)
---

### Test EXIT

| Commande                       | Géré | Sortie attendue                                                                                    | Code de sortie | Test |
|--------------------------------|:----:|----------------------------------------------------------------------------------------------------|:--------------:|:----:|
| `exit`                         | Oui  | `exit`                                                                                             | 0              |  ✅  |
| `exit exit`                    | Oui  | `bash: exit: exit: numeric argument required`                                                      | 2              |  ❌  |
| `exit hola`                    | Oui  | `bash: exit: hola: numeric argument required`                                                      | 2              |  ❌  |
| `exit hola que tal`            | Oui  | `bash: exit: hola: numeric argument required`                                                      | 2              |  ❌  |
| `exit 42`                      | Oui  | `exit`                                                                                             | 42             |  ❌  |
| `exit 000042`                  | Oui  | `exit`                                                                                             | 42             |  ❌  |
| `exit 666`                     | Oui  | `exit`                                                                                             | 154            |  ❌  |
| `exit 666 666`                 | Oui  | `bash: exit: too many arguments`                                                                   | 1              |  ❌  |
| `exit -666 666`                | Oui  | `bash: exit: too many arguments`                                                                   | 1              |  ❌  |
| `exit hola 666`                | Oui  | `bash: exit: hola: numeric argument required`                                                      | 2              |  ❌  |
| `exit 259`                     | Oui  | `exit`                                                                                             | 3              |  ❌  |
| `exit -4`                      | Oui  | `exit`                                                                                             | 252            |  ❌  |
| `exit -42`                     | Oui  | `exit`                                                                                             | 214            |  ❌  |
| `exit -0000042`                | Oui  | `exit`                                                                                             | 214            |  ❌  |
| `exit -259`                    | Oui  | `exit`                                                                                             | 253            |  ❌  |
| `exit -666`                    | Oui  | `exit`                                                                                             | 102            |  ❌  |
| `exit +666`                    | Oui  | `exit`                                                                                             | 154            |  ❌  |
| `exit 0`                       | Oui  | `exit`                                                                                             | 0              |  ❌  |
| `exit +0`                      | Oui  | `exit`                                                                                             | 0              |  ❌  |
| `exit -0`                      | Oui  | `exit`                                                                                             | 0              |  ❌  |
| `exit +42`                     | Oui  | `exit`                                                                                             | 42             |  ❌  |
| `exit -69 -96`                 | Oui  | `bash: exit: too many arguments`                                                                   | 1              |  ❌  |
| `exit --666`                   | Oui  | `bash: exit: --666: numeric argument required`                                                     | 2              |  ❌  |
| `exit ++++666`                 | Oui  | `bash: exit: ++++666: numeric argument required`                                                   | 2              |  ❌  |
| `exit ++++++0`                 | Oui  | `bash: exit: ++++++0: numeric argument required`                                                   | 2              |  ❌  |
| `exit ------0`                 | Oui  | `bash: exit: ------0: numeric argument required`                                                   | 2              |  ❌  |
| `exit "666"`                   | Oui  | `exit`                                                                                             | 154            |  ❌  |
| `exit '666'`                   | Oui  | `exit`                                                                                             | 154            |  ❌  |
| `exit '-666'`                  | Oui  | `exit`                                                                                             | 102            |  ❌  |
| `exit '+666'`                  | Oui  | `exit`                                                                                             | 154            |  ❌  |
| `exit '----666'`               | Oui  | `bash: exit: ----666: numeric argument required`                                                   | 2              |  ❌  |
| `exit '++++666'`               | Oui  | `bash: exit: ++++666: numeric argument required`                                                   | 2              |  ❌  |
| `exit '6'66`                   | Oui  | `exit`                                                                                             | 154            |  ❌  |
| `exit '2'66'32'`               | Oui  | `exit`                                                                                             | 8              |  ❌  |
| `exit "'666'"`                 | Oui  | `bash: exit: '666': numeric argument required`                                                     | 2              |  ❌  |
| `exit '"666"'`                 | Oui  | `bash: exit: "666": numeric argument required`                                                     | 2              |  ❌  |
| `exit '666'"666"666`           | Oui  | `exit`                                                                                             | 170            |  ❌  |
| `exit +'666'"666"666`          | Oui  | `exit`                                                                                             | 170            |  ❌  |
| `exit -'666'"666"666`          | Oui  | `exit`                                                                                             | 86             |  ❌  |
| `exit 9223372036854775807`     | Oui  | `exit`                                                                                             | 255            |  ❌  |
| `exit 9223372036854775808`     | Oui  | `bash: exit: 9223372036854775808: numeric argument required`                                       | 2              |  ❌  |
| `exit -9223372036854775808`    | Oui  | `exit`                                                                                             | 0              |  ❌  |
| `exit -9223372036854775809`    | Oui  | `bash: exit: -9223372036854775809: numeric argument required`                                      | 2              |  ❌  |

---
- [Retourner au sommaire](#sommaire)
---

### Test EXPORT

| Commande                     | Géré      | Sortie attendue                                              | Code de sortie | Test |
|------------------------------|:---------:|--------------------------------------------------------------|:--------------:|:----:|
| `export`                     | Oui       | Affiche toutes les variables exportées                       | 0              | ❌   |
| `export hola`                | Oui       | Ajoute une variable vide `hola`                              | 0              | ❌   |
| `export hola=bonjour`        | Oui       | Ajoute une variable avec une valeur                          | 0              | ❌   |
| `export 1hola=bonjour`       | Oui       | `export: 1hola: not a valid identifier`                      | 1              | ❌   |
| `export =bonjour`            | Oui       | `export: =bonjour: not a valid identifier`                   | 1              | ❌   |
| `export ho=la=bonjour`       | Oui       | Définit `ho=la=bonjour`                                      | 0              | ❌   |
| `export hola=bonjour HOLA=BONJOUR` | Oui | Ajoute deux variables                                        | 0              | ❌   |
| `export hola+=bonjour`       | NON GERE  | Comportement non pris en charge                              | 0              | ❌   |
| `export _=bonjour`           | NON GERE  | Variable spéciale `_` non gérée                              | 0              | ❌   |
| `export ""`                  | NON GERE  | `export: '': not a valid identifier`                         | 1              | ❌   |
| `export =`                   | NON GERE  | `export: '=': not a valid identifier`                        | 1              | ❌   |
| `export %`                   | NON GERE  | `export: '%': not a valid identifier`                        | 1              | ❌   |
| `export $?`                  | NON GERE  | `export: '0': not a valid identifier`                        | 1              | ❌   |
| `export ?=2`                 | NON GERE  | `export: '?=2': not a valid identifier`                      | 1              | ❌   |
| `export 9HOLA=`              | NON GERE  | `export: '9HOLA=': not a valid identifier`                   | 1              | ❌   |
| `export HOL@=bonjour`        | NON GERE  | `export: 'HOL@=bonjour': not a valid identifier`             | 1              | ❌   |
| `export HOL~A=bonjour`       | NON GERE  | `export: 'HOL~A=bonjour': not a valid identifier`            | 1              | ❌   |
| `export -HOLA=bonjour`       | NON GERE  | `export: -H: invalid option`                                 | 2              | ❌   |
| `export --HOLA=bonjour`      | NON GERE  | `export: --: invalid option`                                 | 2              | ❌   |
| `export HOLA-=bonjour`       | NON GERE  | `export: 'HOLA-=bonjour': not a valid identifier`            | 1              | ❌   |
| `export HO-LA=bonjour`       | NON GERE  | `export: 'HO-LA=bonjour': not a valid identifier`            | 1              | ❌   |
| `export HOL.A=bonjour`       | NON GERE  | `export: 'HOL.A=bonjour': not a valid identifier`            | 1              | ❌   |
| `export HOL\$A=bonjour`      | NON GERE  | `export: 'HOL\$A=bonjour': not a valid identifier`           | 1              | ❌   |
| `export HO\\LA=bonjour`      | NON GERE  | `export: 'HO\\LA=bonjour': not a valid identifier`           | 1              | ❌   |
| `export HOL}A=bonjour`       | NON GERE  | `export: 'HOL}A=bonjour': not a valid identifier`            | 1              | ❌   |
| `export Hola`                | Oui       | Ajoute `Hola` sans valeur                                    | 0              | ❌   |
| `export Hola9hey`            | Oui       | Ajoute une variable valide                                   | 0              | ❌   |
| `export HOLA9=bonjour`       | Oui       | Ajoute `HOLA9`                                               | 0              | ❌   |
| `export _HOLA=bonjour`       | Oui       | Ajoute `_HOLA`                                               | 0              | ❌   |
| `export ___HOLA=bonjour`     | Oui       | Ajoute `___HOLA`                                             | 0              | ❌   |
| `export _HO_LA_=bonjour`     | Oui       | Ajoute `_HO_LA_`                                             | 0              | ❌   |
| `export $DONTEXIST`          | Oui       | Variable non importée                                        | 0              | ❌   |
| `export HOLA | echo hola`    | Oui       | `hola` affiché (commande exécutée après export)              | 0              | ❌   |
| `export | echo hola`         | Oui       | `hola` affiché                                               | 0              | ❌   |

---
- [Retourner au sommaire](#sommaire)
---

### Test PWD

| Commande                  | Géré      | Sortie attendue                                                       | Code de sortie |  Test  |
|---------------------------|:---------:|-----------------------------------------------------------------------|:--------------:|:------:|
| `pwd`                     | Oui       | Affiche le chemin courant (`$PWD`)                                    | 0              | ❌     |
| `pwd hola`                | Oui       | Affiche le chemin courant (arguments ignorés)                         | 0              | ❌     |
| `pwd ./hola`              | Oui       | Affiche le chemin courant                                             | 0              | ❌     |
| `pwd hola que tal`        | Oui       | Affiche le chemin courant                                             | 0              | ❌     |
| `pwd -p`                  | Oui       | `bash: pwd: -p: invalid option`                                       | 2              | ❌     |
| `pwd --p`                 | Oui       | `bash: pwd: --: invalid option`                                       | 2              | ❌     |
| `pwd ---p`                | Oui       | `bash: pwd: --: invalid option`                                       | 2              | ❌     |
| `pwd -- p`                | NON GERE  | Affiche le chemin courant                                             | 0              | ❌     |
| `pwd pwd pwd`             | Oui       | Affiche une seule fois le chemin courant                              | 0              | ❌     |
| `pwd ls`                  | Oui       | Affiche le chemin courant                                             | 0              | ❌     |
| `pwd ls env`              | Oui       | Affiche le chemin courant                                             | 0              | ❌     |
| `> pwd`                   | Oui       | Crée un fichier `pwd`                                                 | 0              | ❌     |
| `< pwd`                   | Oui       | `bash: pwd: No such file or directory`                                | 1              | ❌     |
| `cat <pwd`                | Oui       | `bash: pwd: No such file or directory`                                | 1              | ❌     |
| `cat <srcs/pwd`           | Oui       | `bash: srcs/pwd: No such file or directory`                           | 1              | ❌     |
| `mkdir a && mkdir a/b && cd a/b && rm -r ../../a && pwd` | Oui  | `pwd: error retrieving current directory: getcwd: cannot access...` | 1              | ❌     |
| `pwd && ls`               | NON GERE  | Affiche le chemin courant puis la liste de fichiers                  | 0              | ❌     |
| `pwd || ls`               | NON GERE  | Affiche uniquement le chemin courant                                 | 0              | ❌     |
| `pwd && ls && echo hola`  | NON GERE  | Affiche chemin courant, fichiers, puis `hola`                        | 0              | ❌     |
| `pwd || ls && echo hola`  | NON GERE  | Affiche chemin courant puis `hola`                                   | 0              | ❌     |
| `pwd && ls || echo hola`  | NON GERE  | Affiche chemin et fichiers                                           | 0              | ❌     |
| `pwd || ls || echo hola`  | NON GERE  | Affiche chemin courant                                               | 0              | ❌     |
| `(pwd | wc)`              | NON GERE  | Affiche le résultat de `wc` sur le chemin                            | 0              | ❌     |
| `(ls && pwd | wc)`        | NON GERE  | Affiche la sortie combinée et son `wc`                               | 0              | ❌     |
| `(ls && pwd | wc) > hola` | NON GERE  | Résultat écrit dans le fichier `hola`                                | 0              | ❌     |
| `(pwd | wc) < hola`       | NON GERE  | Donne la sortie `wc` même si l’entrée ne contient pas pwd            | 0              | ❌     |
| `(ls -z || pwd | wc) < hola` | NON GERE | Affiche erreur `ls`, puis chemin, puis wc                          | 0              | ❌     |
| `(ls > Docs/hey && pwd) > hola` | NON GERE | Chemin écrit dans hola                                          | 0              | ❌     |
| `ls > Docs/hey && pwd > hola` | NON GERE  | Idem                                                             | 0              | ❌     |
| `cd ../.. && pwd && pwd`  | NON GERE  | Affiche deux chemins                                                 | 0              | ❌     |
| `(cd ../.. && pwd) && pwd`| NON GERE  | Affiche chemin modifié puis chemin courant                           | 0              | ❌     |
| `ls -z || cd ../../.. && pwd` | NON GERE | Affiche chemin après erreur                                       | 0              | ❌     |
| `ls -z || (cd ../../.. && pwd)` | NON GERE | Idem avec sous-shell                                            | 0              | ❌     |

---
- [Retourner au sommaire](#sommaire)
---

### Test UNSET

| Commande                   | Géré      | Sortie attendue                                                | Code de sortie |  Test  |
|----------------------------|:---------:|----------------------------------------------------------------|:--------------:|:------:|
| `unset HOLA`               | Oui       | Supprime la variable HOLA                                      | 0              | ❌     |
| `unset HOLA HOLA2`         | Oui       | Supprime plusieurs variables                                   | 0              | ❌     |
| `unset HOLA HOLA HOLA`     | Oui       | Supporte les doublons                                          | 0              | ❌     |
| `unset`                    | Oui       | Ne fait rien                                                   | 0              | ❌     |
| `unset INEXISTANT`         | Oui       | Ne renvoie pas d'erreur                                        | 0              | ❌     |
| `unset 9HOLA`              | Oui       | `not a valid identifier`                                       | 1              | ❌     |
| `unset HOLA=`              | Oui       | `not a valid identifier`                                       | 1              | ❌     |
| `unset HOL@`               | Oui       | `not a valid identifier`                                       | 1              | ❌     |
| `unset HOL^A`              | Oui       | `not a valid identifier`                                       | 1              | ❌     |
| `unset HOL#A`              | Oui       | `not a valid identifier`                                       | 1              | ❌     |
| `unset HOL!A`              | Oui       | `event not found`                                              | 1              | ❌     |
| `unset HOL$?A`             | Oui       | Utilise la valeur de $? dans le nom                            | 0              | ❌     |
| `unset $HOLA`              | Oui       | Interprète la variable HOLA                                    | 0              | ❌     |
| `unset $PWD`               | Oui       | Interprète comme nom de répertoire                             | 1              | ❌     |
| `unset HOLA HOL?A`         | Oui       | Supprime HOLA et affiche erreur pour HOL?A                     | 1              | ❌     |
| `unset HOL.A`              | Oui       | `not a valid identifier`                                       | 1              | ❌     |
| `unset HOL-A`              | Oui       | `not a valid identifier`                                       | 1              | ❌     |
| `unset HOL+A`              | Oui       | `not a valid identifier`                                       | 1              | ❌     |
| `unset HOL=A`              | Oui       | `not a valid identifier`                                       | 1              | ❌     |
| `unset HOL{A`              | Oui       | `not a valid identifier`                                       | 1              | ❌     |
| `unset HOL}A`              | Oui       | `not a valid identifier`                                       | 1              | ❌     |
| `unset HOL\\A`             | NON GERE  | `not a valid identifier` (échappement mal géré)                | 1              | ❌     |
| `unset HOL;A`              | NON GERE  | `command not found`                                            | 127            | ❌     |
| `unset -HOLA`              | Oui       | `invalid option`                                               | 2              | ❌     |
| `unset _HOLA`              | Oui       | OK                                                             | 0              | ❌     |
| `unset HOL_A`              | Oui       | OK                                                             | 0              | ❌     |
| `unset HOLA_`              | Oui       | OK                                                             | 0              | ❌     |
| `unset _______`            | Oui       | OK                                                             | 0              | ❌     |
| `unset PATH`               | Oui       | Supprime PATH, empêche les binaires de s'exécuter              | 0              | ❌     |
| `unset PATH && ls`         | Oui       | `bash: ls: No such file or directory`                          | 127            | ❌     |
| `unset ""`                 | Oui       | `not a valid identifier`                                       | 1              | ❌     |
| `unset =`                  | Oui       | `not a valid identifier`                                       | 0              | ❌     |
| `unset ======`             | Oui       | `not a valid identifier`                                       | 0              | ❌     |
| `unset ++++++`             | Oui       | `not a valid identifier`                                       | 0              | ❌     |
| `unset "" HOLA`            | Oui       | HOLA est tout de même unset malgré une erreur sur ""           | 1              | ❌     |
| `unset HOLA=bonjour`       | Oui       | `not a valid identifier`                                       | 1              | ❌     |
| `unset export`             | Oui       | Supprime la variable `export` si définie                       | 0              | ❌     |
| `unset echo`               | Oui       | Supprime la variable `echo` si définie                         | 0              | ❌     |
| `unset pwd`                | Oui       | Supprime la variable `pwd` si définie                          | 0              | ❌     |
| `unset cd`                 | Oui       | Supprime la variable `cd` si définie                           | 0              | ❌     |
| `unset unset`              | Oui       | Supprime la variable `unset` si définie                        | 0              | ❌     |
| `unset sudo`               | Oui       | Supprime la variable `sudo` si définie                         | 0              | ❌     |

---
- [Retourner au sommaire](#sommaire)
---
