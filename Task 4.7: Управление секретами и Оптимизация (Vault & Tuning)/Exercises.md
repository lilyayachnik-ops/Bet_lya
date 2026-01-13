# Checkpoints

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


**Задание** `7.2`: `SSL Challenge` (`Multiline Secrets`) Мы переводим `Nginx` на `HTTPS`. Это самая частая боль новичков — как передать многострочный приватный ключ через переменную, не сломав форматирование `YAML`.
Сгенерируй локально самоподписанный сертификат и ключ (`openssl req ...`).

Пропишем в `/etc/hosts` `phoenix.local` и `IP address` балансировщика, чтобы наш компьютер знал, куда обращаться:

```bash
sudo vim /etc/hosts
cat /etc/hosts
```

Покажем в терминале:

<img width="1452" height="361" alt="image" src="https://github.com/user-attachments/assets/3870b260-0815-4f73-9f52-be4c102fc78b" />


```bash
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/ssl/private/nginx-selfsigned.key -out /etc/ssl/certs/nginx-selfsigned.crt
```

Покажем в терминале:

<img width="1591" height="534" alt="image" src="https://github.com/user-attachments/assets/09a2ee07-1294-4b15-9f52-67a367199d9b" />

- Сложность: Ты не должен копировать файл ключа модулем copy с локального диска (потому что сам файл ключа нельзя хранить в репозитории незашифрованным)
- Зашифруй содержимое приватного ключа (`.key`) в переменную `nginx_ssl_key` (используй `encrypt_string`, обрати внимание на `|` для сохранения переносов строк в `YAML`):

```bash
sudo cat /etc/ssl/private/nginx-selfsigned.key

ansible-vault encrypt_string '-----BEGIN PRIVATE KEY-----
MIIEvgIBADANBgkqhkiG9w0BAQEFAASCBKgwggSkAgEAAoIBAQDh9ViBAErR5PCY
JkE91sbyzDgjny2bxqOEM1PledKR76MXoOxGPZqGoGxfECFoJKSRyATJPm2sft5x
TDt+SKu9NdxKi3EVRpXKjeOSsN86QSXzn8pCARf39zFF3dVEso7SWR97Rp25+ne5
y54Rd6qKVOpGpmYQ9JCJGLhjCQGi33whEe0Cx3+V/WqTO6zAkqp5xEYUiwcqAJhf
i6lAOjOtoBEPdwAlwprdfF1ksJvBmrUf2A7IDDrIYPh4XyZcRCcebblWbJkjRcQa
IngkaCaP1P8mR1i7WqVNOZ2xGL9+bbY71JAj3bN03vGYhNej8DZUDw0KGVVAzP0y
/qrxEPDnAgMBAAECggEALFgHYe2qQU3iL6HWTOoBCjYs0ETVYQxXG8Ms9Ex3mB9E
zQbOR6ygTkpeajuSqssCJv/vfIUNZfxR8D8rL8nYNl5DGLgL3caH8APvhOLEND0D
0y9pNJHdX9L5rwHtcDlwkPRfmsRNgOmYJHTC1R+8bsBiZ7IRNsOgN9+LlBf447sE
zLxzGwBOp/Z86u2jtR+Mv7LZVp8BJn2Y7h3D1uTSjrZNdpUqILo0ufspfG0+rAo4
Stsn4h/2AMW4zF6H/q22KbLd9DXlI8VvPG9Tu8RAA+etSDnA+iqao2DjM/rKtO6S
xS255N6qS00jwOY9+jhFDxgyOyoTn9h3ntM+iHt9BQKBgQDx6RbBkZi7C5b/mLSr
+jVSv53iNMrSDGRZuHjHJbwqagmvPOIqsArRPPm6VZZPUNvsYfKRE7brhSSAoYN1
vX7s4zd5vGQa11wh8uNmyDkw6oNXNvcjXk7Y5fdf9SzU2Eb+HmSo26VmgMM/8YSs
6nOJnO7GR4P1rmxuFgk7/mlqgwKBgQDvHmi04fuCx/s3ISF32PXK8tXtDjXyRFbc
FPtWVPVLTlasuZlO4O0XqIiyz8CjLY9qQ6KJtgxRWROnHsvCWc7BPpCw2yInLohF
ZECB7WfwrTKAfwrCEaqouaHo7JkH9rVmTZNuaTNFgAHgfI3/dpLVJ1yZnWLUlMYa
EdMuSLXizQKBgQCdG9emoWuC1vUbTM7R/m8RLddZKRYsMtYdmvv9wpkhJrrhb3Yn
aSutVp+Nj7ZODuK186r448fh0Bez3rGlDwvWx36o9lxBPmrctYVQpGrYRQZF5zG8
a6xjm378eBvEpC3/LjgLnpMeLjQgXDfhoWamEiDU729ZaQ2MtY0LxKAsBwKBgQDs
cNIyQd4zxFDYg28XlbX9dr8nx0x6gS6uMiUQibN/QjgcVO2N/IUURrbcsOcXsz5D
Yht/Nj7Z1U3Ei4QJakox9Q6Xgic0PSPMEcRjdP+5EVmFi5l0qoloix3ZNtJe+IkI
Xp7wpx9mkpvIgQDb4UBxeplq4CBQecelEYkTFByd5QKBgCQbMJ+tAXEp5xrJbj01
CSZT98SdLAvo7Od3rSDFsET+gGZWPf5QXlkXICUbXmsCc/b87sPx+joqsu7Q6F+U
yRq3dGSNjj25yFqvWQeIyxx7vr5q2iVkXmXEPvcBH8Z5fV6hKG6npNKe6UTqE8kp
34s9C917Ks1eClFt3ERQVq4r
-----END PRIVATE KEY-----' --name 'nginx_ssl_key'
```

Покажем в терминале:

<img width="1048" height="642" alt="image" src="https://github.com/user-attachments/assets/dc307c2a-1cf4-4ccb-96f7-957b184ebe3b" />
<img width="1240" height="623" alt="image" src="https://github.com/user-attachments/assets/8c863032-8f33-45ee-b710-cb2f14c8c4d5" />
<img width="1120" height="733" alt="image" src="https://github.com/user-attachments/assets/feff8fb2-28ef-403f-b702-dc94f22b52ad" />
<img width="1217" height="792" alt="image" src="https://github.com/user-attachments/assets/c7670a42-dace-471a-afa8-458eedb54bea" />
<img width="1149" height="504" alt="image" src="https://github.com/user-attachments/assets/159da7ee-a94d-4b29-9641-1b4e05d440bb" />
<img width="1423" height="902" alt="image" src="https://github.com/user-attachments/assets/42444688-c9ea-4f48-a6fc-f5fe0c42bc5a" />

- Зашифруй содержимое сертификата (`.crt`) в переменную `nginx_ssl_cert` (можно без `vault`, но через переменную):

```bash
ansible-vault encrypt_string "$(sudo cat /etc/ssl/certs/nginx-selfsigned.crt)" --name nginx_ssl_cert
```

Покажем в терминале:

<img width="1487" height="507" alt="image" src="https://github.com/user-attachments/assets/745366c3-d74d-45e1-996c-eb0aadccde17" />
<img width="1315" height="511" alt="image" src="https://github.com/user-attachments/assets/7d575f14-7e98-4a4f-8131-25fc75943380" />

Далее вставим `nginx_ssl_cert` в `group_vars/balancers.yml`. Покажем в терминале:

<img width="1333" height="571" alt="image" src="https://github.com/user-attachments/assets/fa68ac63-a379-4097-9ae5-2a0b4fa46c6c" />
<img width="1285" height="448" alt="image" src="https://github.com/user-attachments/assets/3ae03da3-9c03-4a61-a929-ca46547b788b" />

- В роли `phoenix_nginx` используй модуль `copy` с параметром `content`, чтобы создать эти файлы на сервере в папке `/etc/nginx/ssl/`:

```bash
- name: adding a key and certificate
  block:

     - name: adding a key
       copy:
          content: "{{ nginx.nginx_ssl_key }}"
          dest: /etc/ssl/private/nginx-selfsigned.key
          owner: root
          group: root
          mode: 0600

     - name: adding a certificate
       copy:
          content: "{{ nginx.nginx_ssl_cert }}"
          dest: /etc/ssl/certs/nginx-selfsigned.crt
          owner: root
          group: root
          mode: 0644
```

Покажем в терминале:

<img width="816" height="427" alt="image" src="https://github.com/user-attachments/assets/2ac28f24-db7d-480f-89e0-c745e8057b63" />

- Обнови шаблон `Nginx`, чтобы он слушал `443` порт и использовал эти пути:

```bash
user  nginx;
worker_processes  auto;

error_log  /var/log/nginx/error.log notice;
pid        /var/run/nginx.pid;

events {
    worker_connections  1024;
}

http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile        on;
    keepalive_timeout  65;


    upstream backend {
        {%- for host in groups['appservers'] -%}
            {%- if hostvars[host]['ansible_local'] is defined 
                  and hostvars[host]['ansible_local']['phoenix'] is defined
                  and hostvars[host]['ansible_local']['phoenix']['status'] is defined
                  and hostvars[host]['ansible_local']['phoenix']['status'] == 'active' -%}
        
        server {{ hostvars[host]['ansible_host'] }}:80;
        
            {%- endif -%}
        {%- endfor -%}
    }

    server {
        listen {{ nginx_listen_port }};
        server_name phoenix.local;
        return 301 https://$host$request_uri;
    }

    server {
        listen 443 ssl;
        server_name phoenix.local;
        ssl_certificate /etc/ssl/certs/nginx-selfsigned.crt;
        ssl_certificate_key /etc/ssl/private/nginx-selfsigned.key;
        
        location / {
            proxy_pass http://backend;
        }
    }
    
    include /etc/nginx/conf.d/*.conf;
}
```

Покажем в терминале:

<img width="1484" height="758" alt="image" src="https://github.com/user-attachments/assets/d7141c5a-525b-48d5-b471-7915da705c7f" />
<img width="1812" height="569" alt="image" src="https://github.com/user-attachments/assets/eea33a92-46e8-48a3-9199-95396ec558a2" />

**Задание** `7.3`: `SSH Tuning & Pipelining` (Оптимизация) Засеки время выполнения текущего `site.yml` (команда `time ansible-playbook ...`). Теперь ускоряем. Внеси изменения в `ansible.cfg`:

```bash
time ansible-playbook site.yml
```

Покажем в терминале:

<img width="1359" height="426" alt="image" src="https://github.com/user-attachments/assets/02b4a356-1787-47d0-92aa-54501ed29a55" />

- Включи `pipelining = True`.
 - **Ловушка**: Если ты используешь `sudo`, это может сломать выполнение с ошибкой `requiretty`.
 - `Action`: Если получишь ошибку — не выключай `pipelining`. Иди на сервера и исправь конфигурацию `/etc/sudoers` (отключи `requiretty` для пользователя `deployer` или глобально). Это нужно сделать через `Ansible` (отдельным предварительным плейбуком или через `Ad-Hoc`, если плейбук уже не работает).

Вставим в `ansible.cfg` следующую строку:

 ```bash
# Каждая команда — это новое SSH соединение
# С pipelining все команды используют одное соединение SSH
[ssh_connection]
pipelining=true
```

Покажем в терминале:

<img width="1051" height="51" alt="image" src="https://github.com/user-attachments/assets/fafb2abd-3707-4452-8820-0dc9ec7152a8" />

- Настрой `ssh_args` для использования `ControlPersist`. Сделай так, чтобы `SSH-сокет` жил `15` минут. Это мгновенно ускорит повторные таски.

Вставим в `ansible.cfg` следующую строку:

```bash
# -C — сжимает данные перед отправкой по сети
# -o ControlMster=auto — автоматически создает главное SSH соединение
# -o ControlPersist=15m — главное SSH соединение живет 15 минут

[ssh_connection]
ssh_args = -C -o ControlMaster=auto -o ControlPersist=15m
```

Покажем в терминале:

<img width="797" height="60" alt="image" src="https://github.com/user-attachments/assets/e9393766-04a3-4f29-86a2-11a2fa6a47b2" />

- Увеличь количество `forks` до `10` (по умолчанию `5`).

 Вставим в `ansible.cfg` следующую строку:

 ```bash
# forks — это максимальное количество хостов, которые Ansible может обрабатывать параллельно

[defaults]
forks=10
```

Покажем в терминале:

<img width="1298" height="358" alt="image" src="https://github.com/user-attachments/assets/14907b02-3c89-4a52-82d8-9f8f4d98d1d0" />

- Сравни время выполнения после оптимизации:

```bash
time ansible-playbook site.yml
```

Покажем в терминале:

<img width="1590" height="257" alt="image" src="https://github.com/user-attachments/assets/0c57309e-83eb-421c-bf00-2c95333364ff" />
<img width="1600" height="268" alt="image" src="https://github.com/user-attachments/assets/86fafab5-2cc9-4ded-8f1b-478f4ec516d8" />

**Задание** `7.4`: `Callback Plugins` (Профилирование) Ты должен знать, какой таск тормозит:

- В `ansible.cfg` включи плагин `profile_tasks`:

```bash
# profile_tasks — добавляет информацию об выполнении tasks

[default]
callbacks_enabled=ansible.posix.profile_tasks
```

Покажем в терминале:

<img width="1033" height="385" alt="image" src="https://github.com/user-attachments/assets/6b0f944d-8ff5-465d-adbb-a55f34650de5" />

- Запусти прогон. Найди в выводе "Top `3`" самых медленных задач:

```bash
ansible-playbookr site.yml
```

Покажем в терминале:

<img width="1589" height="509" alt="image" src="https://github.com/user-attachments/assets/5ee77045-aa75-404f-9bbb-7d1256de2428" />


Сделай вывод: почему установка пакетов занимает столько времени и можно ли это кэшировать (нет, но понимать надо)?

**Установка пакетов занимает время из-за сетевых задержек, разрешения зависимостей и дисковых операций. Кэшировать сам процесс распаковки и настройки пакета (dpkg/rpm) нельзя**

**Критерии приемки** (`Definition of Done`):

1. В репозитории нет ни одного `plain-text секрета`. Поиск по слову `password` или содержимому ключа ничего не дает. Покажем в терминале:

<img width="236" height="918" alt="image" src="https://github.com/user-attachments/assets/8bbcd2f9-58dc-496d-af54-aa86c9db6478" />
<img width="235" height="421" alt="image" src="https://github.com/user-attachments/assets/db5ec813-4cec-448c-a5f1-03fbc378a926" />

2. При запуске `ansible-playbook` не запрашивается пароль от `Vault` (берется из файла). Покажем в терминале:

<img width="1599" height="416" alt="image" src="https://github.com/user-attachments/assets/bbcb122a-4856-4b3a-96ee-aaae46ba3e85" />

3. На сервере балансировщика в `/etc/nginx/ssl/` лежат валидные ключ и сертификат.
Проверка: `openssl x509 -in /etc/nginx/ssl/server.crt -text -noout` не выдает ошибок:\

```bash
ansible balancers -m shell -a "openssl x509 -in /etc/ssl/certs/nginx-selfsigned.crt -text -noout"
```

Покажем в терминале:

<img width="1607" height="525" alt="image" src="https://github.com/user-attachments/assets/13cdcdbb-9d23-41b5-bb06-cd573819f6d2" />
<img width="1473" height="568" alt="image" src="https://github.com/user-attachments/assets/c4cfe082-b8c6-41bb-af3d-d94c52bd6197" />
<img width="1518" height="224" alt="image" src="https://github.com/user-attachments/assets/b7faeab1-5efb-4f20-a789-d0d4f3598ec6" />

Nginx успешно рестартует:

```bash
ansible balancers -m shell -a "systemctl status nginx" -b
```

Покажем в терминале:

<img width="1544" height="484" alt="image" src="https://github.com/user-attachments/assets/60f728de-df14-4930-a76d-228169358cba" />

4. Включен `Pipelining`. В логах `/var/log/auth.log` (или `secure`) на сервере видно меньше сессий `sshd` на один прогон плейбука, чем раньше.
Ты понимаешь, что такое `ControlMaster` и где лежат сокеты `SSH`:

!!`ControlMaster` — механизм `OpenSSH` для переиспользования `SSH` соединений. Создаёт мастер-процесс, к которому подключаются последующие сессии вместо создания новых.!!

Сокеты лежат в !!`~/.ansible/cp/`!!
