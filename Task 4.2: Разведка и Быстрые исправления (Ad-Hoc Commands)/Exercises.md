# Checkpoints

Задача `2.1`: Управление пользователями (Модули `group` и `user`)

Создадим на всех серверах группу `phoenix_admins` с `GID 2024`. (Используем модуль `group`):

```bash
# -b/--become — повышение привилегий (становление суперпользователем) 
$ ansible production -m ansible.builtin.group -a "gid=2024 name=phoenix_admins state=present" -b
```

Покажем в терминале:

<img width="1594" height="904" alt="image" src="https://github.com/user-attachments/assets/5b794e75-7282-4ae1-bc29-4f24b6454e82" />
<img width="1603" height="52" alt="image" src="https://github.com/user-attachments/assets/713c9281-8734-4a73-b516-12d5d36c678e" />


Создадим пользователя `phoenix` со следующими параметрами (используем модуль `user`):

- `UID`: `2024`
- `Primary Group`: `phoenix_admins`
- `Shell`: `/bin/bash`
- `Comment`: `Phoenix Application Service Account`
- `Home`: `/opt/phoenix_home` (Нестандартная домашняя папка! Проверим, создал ли ее `Ansible` сам или нужно шаманить).

**Важно**: Пользователь не должен иметь возможность `sudo` (пока что).

Используем следующую `Ad-Hoc` команду:

```bash
ansible production -m ansible.builtin.user -a "name=phoenix uid=2024 group=phoenix_admins shell=/bin/bash comment='Phoenix Application Service Account' home=/opt/phoenix_home create_home=true state=present" -b
```

Покажем в терминале:

<img width="1592" height="816" alt="image" src="https://github.com/user-attachments/assets/bef910f9-df58-47ae-9984-833697ddc42a" />
<img width="1584" height="491" alt="image" src="https://github.com/user-attachments/assets/7b1e20c2-a9b9-4e0a-b2e7-e5bd59bee93c" />

**Чекпоинт**: Проверим на сервере через `id phoenix`, что `UID` и `GID` совпадают с требованиями.

Покажем в терминале:

<img width="770" height="40" alt="image" src="https://github.com/user-attachments/assets/d8fec3a5-cc6a-4c6a-80ab-b15a746b798b" />
<img width="903" height="41" alt="image" src="https://github.com/user-attachments/assets/1e73f848-d8c1-4261-82e3-13c773b34e17" />
<img width="935" height="44" alt="image" src="https://github.com/user-attachments/assets/d6d48fec-e708-45c8-9413-8b9abc870a73" />

Задача `2.2`: Внедрение ключей "на лету" (Модуль `authorized_key`) 

Мы работаем с `Control Node` под своим пользователем. Но разработчики просят доступ к пользователю `phoenix` напрямую.
Сгенерируем у себя локально отдельную пару ключей `SSH` специально для этого проекта (назовем файл `phoenix_dev_key`):

```bash
ssh-keygen -t rsa -b 4096 -f ~/.ssh/phoenix_dev_key
ls -la ~/.ssh/
```

Покажем в терминале:

<img width="1275" height="709" alt="image" src="https://github.com/user-attachments/assets/ddcf93a7-a142-4c58-8ca1-1a2053d46752" />

Используем `Ad-Hoc` команду и модуль `authorized_key`, зальем публичную часть этого ключа (`phoenix_dev_key.pub`) в `authorized_keys` пользователя `phoenix` на всех серверах:

```bash
ansible-galaxy collection install ansible.posix

# 1-й способ
ansible production -m ansible.posix.authorized_key -a "user=phoenix key='$(cat ~/.ssh/phoenix_dev_key.pub)' path=/opt/phoenix_home/.ssh/authorized_keys manage_dir=true state=present" -b

# 2-й способ

PUBLIC_KEY=$(cat ~/.ssh/phoenix_dev_key.pub)
ansible production -m ansible.posix.authorized_key -a "user=phoenix key='$PUBLIC_KEY' path=/opt/phoenix_home/.ssh/authorized_keys manage_dir=true state=present" -b 

# 3-й способ

ansible production -m ansible.posix.authorized_key -a "user=phoenix key=\"{{ lookup('file', '/home/ilya/.ssh/phoenix_dev_key.pub') }}\" path=/opt/phoenix_home/.ssh/authorized_keys manage_dir=true state=present" -b
```

Покажем в терминале:

<img width="1592" height="181" alt="image" src="https://github.com/user-attachments/assets/833a171e-aa74-40c6-bf23-5871c9ebfd9e" />
<img width="1590" height="818" alt="image" src="https://github.com/user-attachments/assets/bb56be98-4945-4496-9399-a3fa43b5179e" />
<img width="1599" height="819" alt="image" src="https://github.com/user-attachments/assets/e92a4153-b6e0-4e39-97c9-a97b26f22d05" />
<img width="1601" height="484" alt="image" src="https://github.com/user-attachments/assets/61c65dde-3a6b-44a6-97af-44fc92774b33" />

**Сложность**: Нам нужно прочитать содержимое файла-ключа на своем ноутбуке и передать его как строку в аргумент модуля. Подумаем, как это сделать, не копируя строку вручную.
**Внимание**: Мы подключаемся как `deployer` (или наш админский юзер), а ключ кладем пользователю `phoenix`. Не забудем про `--become` и правильного целевого юзера.

Задача `2.3`: Работа с правами и "грязный" `Shell` (Модуль `file vs shell`) 

Разработчики попросили создать структуру папок для логов. Создай папку `/var/log/phoenix_app`:

- Владельцем должен быть `phoenix`, группой `phoenix_admins`.
- Права доступа: `0770` (полный доступ владельцу и группе, остальным — ничего).
- Также установим `SetGID` бит (чтобы новые файлы наследовали группу). Изучим, как задать это в `mode`.

```bash
# 1770, где 1 — это бит Sticky Bit (для директорий: разрешает удалять/переименовывать файлы только их владельцам), а 2 — это SetGID (для директорий: новые файлы наследуют группу директории, для файлов: программа выполняется с правами группы файла)
ansible production -m ansible.builtin.file -a "owner=phoenix group=phoenix_admins path=/var/log/phoenix_app mode=2770 state=directory" -b
```

Покажем в терминале:

<img width="1596" height="820" alt="image" src="https://github.com/user-attachments/assets/a5629415-e85b-40dd-aedb-41ba9502d169" />
<img width="1594" height="427" alt="image" src="https://github.com/user-attachments/assets/5f85ce42-8a87-4a3e-9e13-180447b3f172" />

 
`Action`: Теперь проверочное задание на понимание прав. Попробуем от имени пользователя `phoenix` создать файл внутри этой папки через модуль `shell`: `ansible production -m shell -a "sudo -u phoenix touch /var/log/phoenix_app/test.log" --become`. Если права в пункте `3` настроены неверно — мы получим ошибку: `Permission denied`.

```bash
ansible production -m shell -a "sudo -u phoenix touch /var/log/phoenix_app/test.log" --become
```

Покажем в терминале:
  
<img width="1687" height="152" alt="image" src="https://github.com/user-attachments/assets/e4244ee1-4f9f-47d3-8d8f-25414265c36d" />

Задача `2.4`: Массовое удаление и очистка (Сложные фильтры) 

Представим, что на серверах есть мусор.
Создадим на одном из серверов (вручную, через `ssh`) файл `/tmp/dangerous_script.sh`:

```bash
ssh -i ~/.ssh/phoenix_dev_key phoenix@34.60.24.1
cd /tmp
vim dangerous_script.sh
cat dangerous_script.sh
chmod +x dangerous_script.sh
ls -la
```

Покажем в терминале:

<img width="1190" height="753" alt="image" src="https://github.com/user-attachments/assets/8022b2db-fc22-4801-9c69-ca2a648171d4" />
<img width="1506" height="334" alt="image" src="https://github.com/user-attachments/assets/fceefcaf-8254-4f84-b904-671629156068" />


**Наша задача**: Найти и удалить все файлы в папке `/tmp`, которые заканчиваются на `.sh`, НО только те, которые старше `1` часа (модуль `find` + удаление):
**Подсказка**: В `Ansible` нет модуля "найти и удалить". Нам нужно использовать модуль `find`, зарегистрировать найденные файлы (но в `Ad-Hoc` это сложно) или... использовать параметры модуля `find` (там есть скрытые возможности или флаги `file_type` и `age`).
**Альтернатива для хардкора**: Если мы не справимся с модулем `find` в одну строку `Ad-Hoc` (это реально сложно), сделаем это через `shell`, но команда должна быть безопасной (идемпотентной).

```bash
# /tmp —	искать в директории /tmp
# -maxdepth 1 —	искать только в /tmp
# -name '*.sh' —	ищем файлы, заканчивающиеся на .sh
# -type f —	указывает тип файл 
# -mmin +60 —	старше 60 минут (1 часа)
# -delete — удалить найденные файлы

$ ansible production -m ansible.builtin.shell -a "find /tmp -maxdepth 1 -name '*.sh' -type f -mmin +60 -delete" -b
```

Покажем в терминале:

<img width="1593" height="143" alt="image" src="https://github.com/user-attachments/assets/6a0b847f-dbb1-4749-a43c-5e63accdd3a5" />

**Критерий успеха**:

Мы можем зайти по `SSH`: `ssh -i phoenix_dev_key phoenix@<IP_VM>` и попасть сразу в `/opt/phoenix_home`:

1) Проверим подключение и выведем текущую директорию, используя команду `pwd`:

```bash
ssh -i ~/.ssh/phoenix_dev_key phoenix@34.60.24.1
ssh -i ~/.ssh/phoenix_dev_key phoenix@34.67.95.48
ssh -i ~/.ssh/phoenix_dev_key phoenix@34.71.138.167
```

2) Покажем в терминале:

<img width="1384" height="753" alt="image" src="https://github.com/user-attachments/assets/78315673-621a-4eed-b59c-baf440717ad1" />
<img width="1636" height="751" alt="image" src="https://github.com/user-attachments/assets/820539f8-5223-4896-bbb5-cdeeffdf528e" />
<img width="1563" height="753" alt="image" src="https://github.com/user-attachments/assets/f43a3f4d-2aa7-4c98-8bcf-e3ceb050b13a" />

Папка `/var/log/phoenix_app` имеет права `drwxrws---` (обратим внимание на `s` в группе):

1) Подключимся к каждому серверу и выполним следующие команды:

```bash
ls -la /var/log | grep phoenix_app
ls -la /var/log/phoenix_app
```

2) Покажем в терминале:

<img width="1507" height="861" alt="image" src="https://github.com/user-attachments/assets/8669f501-7723-4a5e-a48f-56d30418398d" />
<img width="840" height="858" alt="image" src="https://github.com/user-attachments/assets/5df32c47-e54f-439b-a425-4f9cf1a9b840" />
<img width="870" height="859" alt="image" src="https://github.com/user-attachments/assets/bb9aff58-a6a0-4787-80ff-47dcb482582f" />


Был недоволен, пока экранировал кавычки при передаче `SSH`-ключа в `Ad-Hoc` команде.
