# Built-in documentation

## **Sommaire**
|          Built-in            |     Tableau des tests     |
|:----------------------------:|:-------------------------:|
| [Built-in: cd](#cd)          | [Test: cd](#test-cd)      |
| [Built-in: echo](#echo)      | [Test: exit](#test-exit)  |
| [Built-in: env](#env)        |
| [Built-in: pwd](#pwd)        |
| [Built-in: exit](#exit)      |
| [Built-in: unset](#unset)    |
| [Built-in: export](#export)  |

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