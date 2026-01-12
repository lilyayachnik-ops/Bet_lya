<img width="1354" height="361" alt="image" src="https://github.com/user-attachments/assets/d89efd35-e884-4e51-b0ad-ec44359680e8" /><img width="1354" height="361" alt="image" src="https://github.com/user-attachments/assets/a8f834e6-b7c1-4220-a23d-7b6230626ad6" /># Checkpoints

**Задание** `7.1`: Работа с `Vault` (`No Interactive Mode`) Сейчас у тебя пароль от БД лежит в `group_vars/dbservers.yml` или `roles/.../defaults`:

- Создай файл `.vault_pass` в корне проекта (добавь его в `.gitignore!`). Запиши туда произвольный пароль:

```bash
touch .vault_pass
touch .gitignore
```

Покажем в терминале:

<img width="607" height="45" alt="image" src="https://github.com/user-attachments/assets/45981a11-7440-478c-8856-e8bd77ef45ec" />

Содержимое `.gitignore`:

```bash
.vault_pass
```

Покажем в терминале:

<img width="285" height="64" alt="image" src="https://github.com/user-attachments/assets/7163400f-5cc6-41b9-ba06-ea2083a3dc40" />

- Настрой `ansible.cfg`, чтобы он автоматически подхватывал этот файл (параметр `vault_password_file`):

```bash
[default]
vault_password_file=./.vault_pass

[vault]
vault_identity_list=default@./.vault_pass
```

Покажем в терминале:

<img width="551" height="243" alt="image" src="https://github.com/user-attachments/assets/3d4f41a5-925f-4f23-99f4-dd0ebd209e2b" />

- Задача: Не шифруй файл `dbservers.yml` целиком (это неудобно для `Code Review`).
 - Используй команду `ansible-vault encrypt_string` чтобы зашифровать пароль `secure_pass`:

```bash
ansible-vault encrypt_string 'secure_pass' --name 'db_password'
```

Покажем в терминале:

<img width="1119" height="202" alt="image" src="https://github.com/user-attachments/assets/f215c757-9885-4315-8509-14a9e00071a5" />

 - Вставь полученный зашифрованный блок в `YAML-файл` переменной `db_password`:

```bash
db_password: !vault |
          $ANSIBLE_VAULT;1.1;AES256
          33656165306561336430383665376132366337383334663639316636656436333365313565633963
          6332303564353734653763366237306234616365326266340a633731633931633836353832383163
          62316237656164663739353432313037393138363665343830396130396234633431653531623935
          3534383831396466650a336262633131616335636335623465373337643361336632643862393833
          3665
```

Покажем в терминале:

<img width="1354" height="361" alt="image" src="https://github.com/user-attachments/assets/6657d4f7-93ed-472f-9a64-17060a421db5" />

 - Убедись, что плейбук отрабатывает, и приложение коннектится к БД:

```bash
ansible-playbook site.yml
```

Покажем в терминале:

<img width="1334" height="78" alt="image" src="https://github.com/user-attachments/assets/ebec8653-88d1-445e-80d1-2b11ccf56105" />
<img width="1108" height="120" alt="image" src="https://github.com/user-attachments/assets/ade3a479-5186-407b-8b96-f883cd4af0b4" />

Подключимся с нашего хоста и с приложения. Покажем в терминале:

<img width="1103" height="242" alt="image" src="https://github.com/user-attachments/assets/1398d8f4-9de6-4a95-b712-183efe7db0fb" />


Задание 7.2: SSL Challenge (Multiline Secrets) Мы переводим Nginx на HTTPS. Это самая частая боль новичков — как передать многострочный приватный ключ через переменную, не сломав форматирование YAML.
Сгенерируй локально самоподписанный сертификат и ключ (openssl req ...).
Сложность: Ты не должен копировать файл ключа модулем copy с локального диска (потому что сам файл ключа нельзя хранить в репозитории незашифрованным).
Зашифруй содержимое приватного ключа (.key) в переменную nginx_ssl_key (используй encrypt_string, обрати внимание на | для сохранения переносов строк в YAML).
Зашифруй содержимое сертификата (.crt) в переменную nginx_ssl_cert (можно без vault, но через переменную).
В роли phoenix_nginx используй модуль copy с параметром content, чтобы создать эти файлы на сервере в папке /etc/nginx/ssl/.
Обнови шаблон Nginx, чтобы он слушал 443 порт и использовал эти пути.
Задание 7.3: SSH Tuning & Pipelining (Оптимизация) Засеки время выполнения текущего site.yml (команда time ansible-playbook ...). Теперь ускоряем. Внеси изменения в ansible.cfg:
Включи pipelining = True.
Ловушка: Если ты используешь sudo, это может сломать выполнение с ошибкой requiretty.
Action: Если получишь ошибку — не выключай pipelining. Иди на сервера и исправь конфигурацию /etc/sudoers (отключи requiretty для пользователя deployer или глобально). Это нужно сделать через Ansible (отдельным предварительным плейбуком или через Ad-Hoc, если плейбук уже не работает).
Настрой ssh_args для использования ControlPersist. Сделай так, чтобы SSH-сокет жил 15 минут. Это мгновенно ускорит повторные таски.
Увеличь количество forks до 10 (по умолчанию 5).
Сравни время выполнения после оптимизации.
Задание 7.4: Callback Plugins (Профилирование) Ты должен знать, какой таск тормозит.
В ansible.cfg включи плагин profile_tasks.
Запусти прогон. Найди в выводе "Top 3" самых медленных задач.
Сделай вывод: почему установка пакетов занимает столько времени и можно ли это кэшировать (нет, но понимать надо)?
