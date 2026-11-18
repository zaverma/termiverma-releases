# Локальный Ansible для Termiverma

Инструкция для **Termiverma 0.6.4**, обновлена **17 ноября 2026 года**.
Основной пример — Cisco IOS через Factory и `network_cli`.

Установка выполняется на компьютере, где запускается Termiverma и есть доступ
к устройствам. Приложение использует локальный `ansible-playbook`; установка
Ansible на другом ноутбуке сама по себе не подключает его к Factory.
Для выполнения заданий нужна Pro-лицензия с доступом **Factory**.

- [Совместимость и существующий Ansible](#compatibility)
- [Установка на macOS](#macos)
- [Установка на Linux](#linux)
- [Проверка окружения](#verify)
- [Настройка Termiverma](#termiverma)
- [Первый playbook](#playbook)
- [Обычный SSH к Linux-серверам](#builtin-ssh)
- [Ошибки и их причины](#troubleshooting)

<a id="compatibility"></a>
## Совместимость и существующий Ansible

| Компонент | Требование текущего Executor |
| --- | --- |
| Python контроллера | 3.11–3.13; для нового окружения на macOS ниже выбран 3.13 |
| `ansible-core` | Ветки 2.18 или 2.19 |
| Network CLI | `ansible.netcommon.network_cli` через libssh |
| Python-модуль для Network CLI | `ansible-pylibssh` 1.4.x; в рецепте зафиксирован 1.4.0 |
| Cisco IOS | Коллекции `ansible.netcommon`, `cisco.ios`; network OS — `cisco.ios.ios` |
| Пароль из Vault через Network CLI | Поддержан в core 2.18 и 2.19 |
| Пароль из Vault через `ansible.builtin.ssh` | Требуется core 2.19 |

Ограничения версий относятся к адаптеру **Termiverma 0.6.3**. Они не означают,
что стандартный Ansible не умеет работать с другими версиями или транспортами.
Установка самой новой версии Ansible без проверки может выйти за эти границы.

Если подходящий Ansible уже работает, переустанавливать его не требуется:
проверь версии и укажи его абсолютный путь в приложении. Отдельное окружение
удобно, когда зависимости установленного Ansible не подходят Factory.
Оно сохраняет существующую установку Homebrew/системного Ansible.

Полный пакет **`ansible==11.5.0`** включает коллекции и выбирает совместимый
`ansible-core` ветки 2.18. Его номер отличается от номера core: вывод
`ansible-playbook [core 2.18.…]` после установки Ansible 11.5.0 нормален.
Patch-версия core может изменяться при новой установке; рецепт не фиксирует
её на 2.18.5. Пакет `ansible-core` сам по себе содержит только ядро и builtin
plugins, поэтому для сетевых модулей потребуются отдельные коллекции.

Разработчиком проверены core **2.18.5 / 2.19.3**, netcommon **7.2.0**,
cisco.ios **9.2.0** и pylibssh **1.4.0** на локальных SSH-стендах. Владелец
также подтвердил успешное выполнение Cisco playbook после описанной подготовки
на другом Mac. Это отдельный успешный сценарий, а не проверка всех устройств,
методов аутентификации и платформ.

<a id="macos"></a>
## Установка на macOS

Команды ниже рассчитаны на Homebrew. Рабочее окружение создаётся в
`~/.local/share/termiverma/ansible-factory`. Если там уже есть нужная рабочая
установка, переходи к проверке; для другого окружения выбери другое имя каталога.

### 1. Python, libssh и компилятор

```bash
brew install python@3.13 libssh
xcode-select -p
```

Если последняя команда сообщает, что Command Line Tools отсутствуют, выполни
`xcode-select --install`, заверши системную установку и продолжи после неё.
Если путь уже показан, повторная установка инструментов не требуется.

Для `ansible-pylibssh 1.4.0` опубликованы macOS wheels для x86-64, но нет
нативного ARM64 wheel. Поэтому **на Apple Silicon пакет собирается из исходников**:
ему нужны заголовки и библиотека **libssh**, а также C-компилятор.
`libssh2` — другая библиотека и не заменяет `libssh` в этой сборке.

### 2. Отдельное Python-окружение и пакеты

```bash
TV_ANSIBLE_ENV="$HOME/.local/share/termiverma/ansible-factory"
TV_ANSIBLE_PYTHON="$(brew --prefix python@3.13)/bin/python3.13"
TV_LIBSSH_PREFIX="$(brew --prefix libssh)"

"$TV_ANSIBLE_PYTHON" -m venv "$TV_ANSIBLE_ENV"
ls "$TV_LIBSSH_PREFIX/include/libssh/libssh.h"

CFLAGS="-I${TV_LIBSSH_PREFIX}/include ${CFLAGS:-}" \
LDFLAGS="-L${TV_LIBSSH_PREFIX}/lib ${LDFLAGS:-}" \
"$TV_ANSIBLE_ENV/bin/python" -m pip install \
  "ansible==11.5.0" \
  "ansible-pylibssh==1.4.0"
```

Команда `ls` должна показать существующий файл. Флаги `CFLAGS`/`LDFLAGS`
действуют для этой установки: они сообщают компилятору и линкеру расположение
Homebrew libssh. Наличие установленного Ansible не добавляет эти пути автоматически.
Используется `python -m pip` именно из нового окружения; `sudo pip` не нужен.

Активация через `source .../activate` не обязательна: далее команды используют
полные пути. Не выбирай в приложении старый Homebrew executable, если пакеты
поставлены в новое окружение.

<a id="linux"></a>
## Установка на Linux

Нужен Python **3.11–3.13** с `venv`. Если пакет приходится собирать, нужны
компилятор, заголовки выбранного Python и development-пакет libssh.
Готовый Linux wheel, когда он подходит системе и Python, pip выберет сам.

Ниже примеры для двух дистрибутивов; выбери один подходящий вариант.

### Ubuntu 24.04: Python 3.12

```bash
sudo apt update
sudo apt install python3.12 python3.12-venv python3.12-dev build-essential libssh-dev
TV_ANSIBLE_PYTHON="$(command -v python3.12)"
```

### Debian 13: системный Python 3.13

```bash
sudo apt update
sudo apt install python3 python3-venv python3-dev build-essential libssh-dev
TV_ANSIBLE_PYTHON="$(command -v python3)"
```

На Ubuntu 22.04 системный Python обычно 3.10: этого недостаточно для данного
рецепта. Нужен отдельно установленный Python 3.11–3.13 с соответствующими
`venv` и заголовками. Не подменяй системный `/usr/bin/python3`; задай
`TV_ANSIBLE_PYTHON` путём к подходящему интерпретатору.
Для других дистрибутивов названия пакетов и способ установки Python отличаются.

### Создание окружения после выбора Python

```bash
"$TV_ANSIBLE_PYTHON" -c 'import sys; assert (3, 11) <= sys.version_info[:2] <= (3, 13), sys.version; print(sys.version)'

TV_ANSIBLE_ENV="$HOME/.local/share/termiverma/ansible-factory"
"$TV_ANSIBLE_PYTHON" -m venv "$TV_ANSIBLE_ENV"
"$TV_ANSIBLE_ENV/bin/python" -m pip install \
  "ansible==11.5.0" \
  "ansible-pylibssh==1.4.0"
```

Linux-команды здесь — инструкция подготовки, а не утверждение о пройденной
проверке каждой ОС/архитектуры и устройств. Сам запуск AppImage и окружение
Ansible — разные части установки.

<a id="verify"></a>
## Проверка окружения

Если открыт новый терминал, сначала снова задай путь:

```bash
TV_ANSIBLE_ENV="$HOME/.local/share/termiverma/ansible-factory"

"$TV_ANSIBLE_ENV/bin/python" -c 'import sys; from importlib.metadata import version; print("Python:", sys.executable); print("ansible:", version("ansible")); print("core:", version("ansible-core")); print("pylibssh:", version("ansible-pylibssh"))'
"$TV_ANSIBLE_ENV/bin/python" -c 'from pylibsshext import session; print("pylibsshext import: OK")'
"$TV_ANSIBLE_ENV/bin/ansible-playbook" --version
"$TV_ANSIBLE_ENV/bin/ansible-galaxy" collection list ansible.netcommon
"$TV_ANSIBLE_ENV/bin/ansible-galaxy" collection list cisco.ios
"$TV_ANSIBLE_ENV/bin/python" -m pip check
printf '%s\n' "$TV_ANSIBLE_ENV/bin/ansible-playbook"
```

Для основного рецепта ожидаются `ansible` **11.5.0**, core **2.18.x**,
pylibssh **1.4.0**, обе коллекции и отсутствие конфликтов в `pip check`.
Импорт должен вывести `pylibsshext import: OK`: он дополнительно проверяет
загрузку C-extension и libssh, которую не проверяет список версий пакетов.
Последняя команда печатает абсолютный путь для Settings.

Проверяй Python из того же окружения, которое запускает выбранный executable.
Версия пакета у случайного `python3` или `pip` из PATH может быть другой.

Полный пакет Ansible обычно уже содержит нужные Cisco-коллекции. Список может
показывать и другие копии из пользовательских каталогов: `venv` изолирует
Python-пакеты, но не все источники Ansible Collections. Каталог
`~/.ansible/collections` может использоваться несколькими установками;
коллекции рядом с плейбуком также могут влиять на загрузку.

Factory задаёт собственный `ansible.cfg` и удаляет унаследованные `ANSIBLE_*`
переменные. Поэтому произвольный `ansible-galaxy ... -p /some/path` вместе с
`ANSIBLE_COLLECTIONS_PATH` в shell не гарантирует, что Factory увидит коллекции.
Для начала используй полный пакет из рецепта. Дополнительные коллекции должны
быть доступны выбранному Ansible через его штатный поиск; проверяй путь и версию.

<a id="termiverma"></a>
## Настройка Termiverma

1. Открой **Settings → Factory → Ansible**.
2. В поле **ansible-playbook path** укажи абсолютный путь, напечатанный выше.
   Используй **Choose executable…**, если удобнее выбрать файл. Не вставляй
   `~` или `$HOME` буквально: поле ожидает абсолютный путь.
3. Нажми **Test Ansible**. Ручной путь имеет приоритет над автопоиском;
   PATH графического приложения может отличаться от PATH терминала.
4. В **Platform mapping** добавь соответствие для точного значения платформы
   выбранного устройства из Inventory:

   | Поле | Пример Cisco IOS |
   | --- | --- |
   | Inventory platform | `cisco_ios`, только если именно так она записана у устройства |
   | Connection plugin | `ansible.netcommon.network_cli` |
   | Ansible network OS | `cisco.ios.ios` |
   | Inventory groups | Можно оставить пустым |

5. Нажми **Save execution settings**. Для других значений Inventory platform
   нужны свои явные строки mapping: драйвер не угадывается по роли/названию.
6. Назначь устройству нужный account в Inventory, разблокируй Vault при работе
   с сохранённым паролем. Сначала подключись к устройству обычной вкладкой
   Termiverma и проверь/подтверди SSH host key.

**Available — это результат проверки `ansible-playbook --version`.** Кнопка
не запускает playbook и не проверяет все коллекции, `pylibssh`, сетевой доступ,
права учётной записи или совместимость каждого устройства. Поддержанная ветка
core и остальные условия дополнительно проверяются при запуске задания.

В Settings указывается официальный plugin `ansible.netcommon.network_cli`.
Собственный `termiverma.factory.network_cli` создаётся приложением на время
задания; искать и устанавливать его из Galaxy не требуется.

Текущий Network CLI допускает сохранённый пароль либо один доступный
незашифрованный private-key file. Agent-only, passphrase/MFA/GSSAPI,
`enable`/`become` и Ansible через bastion не поддерживаются. Нужен аккаунт
с необходимыми правами; пароль входа не переиспользуется для enable.
Поддержка bastion в обычном терминале или Snippet Factory не меняет это
ограничение Ansible Factory. Системный `known_hosts` не заменяет HostTrust
Termiverma, а проверку SSH-ключей отключать не нужно.

<a id="playbook"></a>
## Первый playbook

Сохрани файл, например `ios-readonly.yml`, с обычными пробелами для отступов:

```yaml
---
- name: Проверка Cisco IOS
  hosts: termiverma_selected
  gather_facts: false

  tasks:
    - name: Получить версию устройства
      cisco.ios.ios_command:
        commands:
          - show version
```

`hosts` обязателен для каждого play. Группа **`termiverma_selected`** формируется
из устройств, выбранных в Factory; внутренние aliases не обязаны совпадать
с hostname устройства. Адрес, account, connection plugin и network OS поступают
из Inventory/mapping — пароли и реальные адреса в YAML добавлять не требуется.
`gather_facts: false` отключает автоматический сбор фактов для этого примера.

### Только проверка синтаксиса, без подключения

Для проверки создай отдельный технический Inventory без адресов и секретов:

```bash
TV_ANSIBLE_ENV="$HOME/.local/share/termiverma/ansible-factory"
TV_CHECK_DIR="$(mktemp -d)"
cat > "$TV_CHECK_DIR/inventory.yml" <<'YAML'
all:
  children:
    termiverma_selected:
      hosts:
        syntax_only:
          ansible_connection: local
YAML

"$TV_ANSIBLE_ENV/bin/ansible-playbook" --syntax-check \
  --inventory "$TV_CHECK_DIR/inventory.yml" \
  /полный/путь/ios-readonly.yml
```

Замени последний путь своим файлом. `--syntax-check` не выполняет задачи
и не подтверждает аутентификацию или доступ к устройству.
Временный Inventory из этого примера нужен только для такой проверки;
для реального задания Termiverma создаст свой Inventory автоматически.

### Выполнение из приложения

В **Factory → Ansible** добавь локальный playbook, выбери устройства, проверь
mapping/accounts и нажми **Run**. Рассмотри ресурс, список целей и параметры
в подтверждении, затем нажми **Queue job**. Начни с одного разрешённого устройства
и примера `show version`, прежде чем использовать playbook изменения конфигурации.

Не переопределяй в YAML credentials, transport или host-key policy, которыми
управляет Factory. Локальный playbook исполняется с правами пользователя:
выбор устройств и `--limit` не являются песочницей для `delegate_to`, локальных
задач или другого кода playbook. Результаты текущего Executor временные
и очищаются при выходе из приложения.

<a id="builtin-ssh"></a>
## Обычный SSH к Linux-серверам

Для серверов с Python используется mapping **`ansible.builtin.ssh`** (или `ssh`);
поле network OS можно оставить пустым. При key/agent-аутентификации доступны
проверенные ветки core 2.18/2.19. Для сохранённого пароля из Vault нужен
**core 2.19** с его `ssh_askpass` механизмом.

Основной рецепт с `ansible==11.5.0` выбирает core 2.18. Для builtin SSH с
паролем можно создать отдельное окружение только с проверенным ядром:

```bash
TV_SSH_ENV="$HOME/.local/share/termiverma/ansible-ssh"
"$TV_ANSIBLE_PYTHON" -m venv "$TV_SSH_ENV"
"$TV_SSH_ENV/bin/python" -m pip install "ansible-core==2.19.3"
printf '%s\n' "$TV_SSH_ENV/bin/ansible-playbook"
```

`TV_ANSIBLE_PYTHON` — подходящий интерпретатор из раздела своей ОС. Это вариант
для builtin-модулей, а не готовая установка Cisco-коллекций/Network CLI.
Для задания выбирай соответствующий executable в Settings. Не устанавливай
core 2.19 поверх `ansible==11.5.0`: у полного пакета другая ветка зависимости.

<a id="troubleshooting"></a>
## Ошибки и их причины

| Сообщение или симптом | Что проверить |
| --- | --- |
| `libssh/libssh.h file not found` при pip install | Заголовки libssh; на macOS — `brew install libssh`, существование файла через `ls` и `CFLAGS`/`LDFLAGS` из рецепта. Строки `Cythonizing` сами по себе нормальны; причина находится дальше в выводе компилятора. |
| `Factory network_cli requires the verified ansible-pylibssh 1.4.x adapter` | Версия `ansible-pylibssh` именно у Python выбранного Ansible. Для 0.6.3 используется 1.4.x, в инструкции — 1.4.0. Затем проверь, что Settings указывает новый executable. |
| `Unsupported Ansible version` | Ветка core должна быть 2.18 или 2.19. Успешный Test Ansible сам по себе этого не гарантирует. |
| `couldn't resolve module/action` / Missing Collection | Коллекция и нужная версия доступны выбранному окружению. Полный `ansible` и `ansible-core` имеют разный состав; проверь `ansible-galaxy collection list` и пути. |
| `No selected device has a supported Ansible platform...` | Ошибка каждого устройства: точная Inventory platform, mapping/network OS, доверенный SSH-ключ и доступный account. Для Ansible цель должна использовать SSH без bastion. |
| `the field 'hosts' is required` | Добавь `hosts: termiverma_selected` в каждый play. Проверь YAML-отступы: обычные пробелы. |
| `Ansible event observation is incomplete` / нет final statistics | Сначала найди первичную ошибку Ansible в общем выводе: например, playbook мог завершиться до выполнения задач. Один код завершения не подтверждает успешную конфигурацию. |
| Ошибка на этапе `Gathering Facts` | Смотри конкретное `msg`. Для примера выше отключён сбор фактов, но это не исправляет несовместимую библиотеку, host key или credentials. |
| Host key unknown/changed | Проверь ключ обычным интерактивным подключением Termiverma; при изменении рассмотри fingerprints. Не отключай host-key checking ради запуска. |
| `become` / MFA / bastion unsupported | Это ограничения текущего адаптера; настройка только playbook не добавляет отсутствующий метод. |
| Playbook изменился после постановки в очередь | Создай новое задание с повторным review; старый snapshot не подменяется изменённым файлом. |

Для просмотра первичной ошибки выбери **Show all devices** и
**Device status → All statuses**. В 0.6.3 выбор конкретного устройства или
статуса скрывает общие stdout/stderr контроллера, поэтому нужная строка может
быть невидима при включённом фильтре.

Если сборка pip снова падает, повтори её с `-v` и сохрани последние строки
с `fatal error`, `clang`, `ld` или traceback. Сообщение о новой версии pip
не является объяснением отсутствующего `libssh.h`.

## Источники

- [Матрица Python и веток ansible-core](https://docs.ansible.com/projects/ansible/latest/reference_appendices/release_and_maintenance.html#ansible-core-support-matrix).
- [Пакет Ansible 11.5.0](https://pypi.org/project/ansible/11.5.0/).
- [Установка и сборка ansible-pylibssh](https://ansible-pylibssh.readthedocs.io/en/stable/installation_guide/), [файлы версии 1.4.0](https://pypi.org/project/ansible-pylibssh/1.4.0/#files).
- [Python и окружения Homebrew](https://docs.brew.sh/Language-Runtimes-and-Packages#python), [Python venv](https://docs.python.org/3/library/venv.html).
- [Поиск Collections](https://docs.ansible.com/projects/ansible-core/2.18/reference_appendices/config.html#collections-scan-sys-path), [коллекции рядом с playbook](https://docs.ansible.com/projects/ansible/latest/collections_guide/collections_installing.html#installing-collections-adjacent-to-playbooks).
- [Синтаксис playbook](https://docs.ansible.com/projects/ansible/latest/playbook_guide/playbooks_intro.html).
