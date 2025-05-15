### Tests de la commande `exit`

| Commande                        | Sortie attendue                                                                                    | Code de sortie | Test |
|---------------------------------|----------------------------------------------------------------------------------------------------|----------------|------|
| `exit`                          | `exit`                                                                                             | 0              |  ✅  |
| `exit exit`                     | `bash: exit: exit: numeric argument required`                                                      | 2              |  ✅  |
| `exit hola`                     | `bash: exit: hola: numeric argument required`                                                      | 2              |  ✅  |
| `exit hola que tal`             | `bash: exit: hola: numeric argument required`                                                      | 2              |  ❌  |
| `exit 42`                       | `exit`                                                                                             | 42             |  ✅  |
| `exit 000042`                   | `exit`                                                                                             | 42             |  ❌  |
| `exit 666`                      | `exit`                                                                                             | 154            |  ❌  |
| `exit 666 666`                  | `bash: exit: too many arguments`                                                                   | 1              |  ❌  |
| `exit -666 666`                 | `bash: exit: too many arguments`                                                                   | 1              |  ❌  |
| `exit hola 666`                 | `bash: exit: hola: numeric argument required`                                                      | 2              |  ❌  |
| `exit 259`                      | `exit`                                                                                             | 3              |  ❌  |
| `exit -4`                       | `exit`                                                                                             | 252            |  ❌  |
| `exit -42`                      | `exit`                                                                                             | 214            |  ❌  |
| `exit -0000042`                 | `exit`                                                                                             | 214            |  ❌  |
| `exit -259`                     | `exit`                                                                                             | 253            |  ❌  |
| `exit -666`                     | `exit`                                                                                             | 102            |  ❌  |
| `exit +666`                     | `exit`                                                                                             | 154            |  ❌  |
| `exit 0`                        | `exit`                                                                                             | 0              |  ❌  |
| `exit +0`                       | `exit`                                                                                             | 0              |  ❌  |
| `exit -0`                       | `exit`                                                                                             | 0              |  ❌  |
| `exit +42`                      | `exit`                                                                                             | 42             |  ❌  |
| `exit -69 -96`                  | `bash: exit: too many arguments`                                                                   | 1              |  ❌  |
| `exit --666`                    | `bash: exit: --666: numeric argument required`                                                     | 2              |  ❌  |
| `exit ++++666`                  | `bash: exit: ++++666: numeric argument required`                                                   | 2              |  ❌  |
| `exit ++++++0`                  | `bash: exit: ++++++0: numeric argument required`                                                   | 2              |  ❌  |
| `exit ------0`                  | `bash: exit: ------0: numeric argument required`                                                   | 2              |  ❌  |
| `exit "666"`                    | `exit`                                                                                             | 154            |  ❌  |
| `exit '666'`                    | `exit`                                                                                             | 154            |  ❌  |
| `exit '-666'`                   | `exit`                                                                                             | 102            |  ❌  |
| `exit '+666'`                   | `exit`                                                                                             | 154            |  ❌  |
| `exit '----666'`                | `bash: exit: ----666: numeric argument required`                                                   | 2              |  ❌  |
| `exit '++++666'`                | `bash: exit: ++++666: numeric argument required`                                                   | 2              |  ❌  |
| `exit '6'66`                    | `exit`                                                                                             | 154            |  ❌  |
| `exit '2'66'32'`                | `exit`                                                                                             | 8              |  ❌  |
| `exit "'666'"`                  | `bash: exit: '666': numeric argument required`                                                     | 2              |  ❌  |
| `exit '"666"'`                  | `bash: exit: "666": numeric argument required`                                                     | 2              |  ❌  |
| `exit '666'"666"666`            | `exit`                                                                                             | 170            |  ❌  |
| `exit +'666'"666"666`           | `exit`                                                                                             | 170            |  ❌  |
| `exit -'666'"666"666`           | `exit`                                                                                             | 86             |  ❌  |
| `exit 9223372036854775807`      | `exit`                                                                                             | 255            |  ❌  |
| `exit 9223372036854775808`      | `bash: exit: 9223372036854775808: numeric argument required`                                       | 2              |  ❌  |
| `exit -9223372036854775808`     | `exit`                                                                                             | 0              |  ❌  |
| `exit -9223372036854775809`     | `bash: exit: -9223372036854775809: numeric argument required`                                      | 2              |  ❌  |
