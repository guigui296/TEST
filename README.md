## Tableau des test built-in

---

### Emoji

✅ || ❌

### Tests de la commande `cd`

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

### Tests de la commande `exit`

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