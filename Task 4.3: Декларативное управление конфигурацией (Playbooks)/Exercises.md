# Checkpoints

Убедимся, что виртуальные машины `managed-node-1` (группа `balancers`) и `managed-node-2` (группа `dbservers`) запущены и доступны по `SSH`:

```bash
ansible production -m ping
```

Покажем в терминале:

<img width="1598" height="757" alt="image" src="https://github.com/user-attachments/assets/c7746d1b-1d21-4e6f-90b5-e821dc7e0793" />

Больше никаких ручных действий на серверах. Все изменения только через `Ansible`


Задание для `Ansible` (`site.yml`):

Создадим плейбук `site.yml`, содержащий два отдельных сценария (`Plays`) в одном файле.

**Сценарий** `1`: Настройка `Load Balancer` (Целевая группа: `balancers`):

1. Подключение внешнего репозитория:

- Стандартная версия `Nginx` в репозиториях `OS` устарела. Необходимо подключить официальный репозиторий `nginx.org`.
- Реализовать добавление `GPG` ключа репозитория.
- Добавить сам репозиторий в список источников (`sources.list.d` или аналог):

```bash
---
- name: Configuring the Load Balancer
  hosts: balancers
  become: true
  tags: nginx

  tasks:

     - name: add nginx repository GPG key
       apt_key:
         url: https://nginx.org/keys/nginx_signing.key
         state: present

     - name: add nginx apt repository
       apt_repository:
          repo: 'deb https://nginx.org/packages/mainline/ubuntu/ jammy nginx'
          state: present
          filename: nginx


     - name: install nginx
       apt:
         name: nginx=1.25.0-1~jammy
         state: present
         update_cache: yes
```

Покажем в терминале:

<img width="1371" height="595" alt="image" src="https://github.com/user-attachments/assets/923865ea-3f1b-44d6-b1d9-4159df0048ed" />

2. Установка и фиксация версии:

- Установить пакет `nginx`.
- Требование: Установить не последнюю версию, а конкретную (например, `1.24.0` или любую другую, отличную от `latest`). Изучить синтаксис указания версии в модуле пакетов:

```bash
- name: install nginx
       apt:
         name: nginx=1.25.0-1~jammy
         state: present
         update_cache: yes
```

Покажем в терминале:

<img width="1229" height="137" alt="image" src="https://github.com/user-attachments/assets/6e7af9c0-c70b-4c9b-8160-24294c3c479c" />

- Компания требует увеличить лимит открытых файлов для сервиса `Nginx`.
- Создать `directory drop-in` для `systemd`: `/etc/systemd/system/nginx.service.d/.`
- Внутри создать файл `override.conf` с содержимым:
`Ini`, `TOML`

```bash
[Service]
LimitNOFILE=65536
```

- Обеспечить, чтобы `systemd` перечитал конфигурацию (`daemon-reload`) и перезапустил `Nginx` при изменении этого файла. (Использовать `notify` и `handlers` или модуль `systemd`):

```bash
- name: create the drop-in directory
       file:
          owner: root
          group: root 
          path:  /etc/systemd/system/nginx.service.d 
          mode: 0755
          state: directory   
     
     - name: add the configuration
       copy: 
          content: |
             [Service]
             LimitNOFILE=65536
          dest: /etc/systemd/system/nginx.service.d/override.conf
          owner: root
          group: root
          mode: 0644
       notify: reload the systemd daemon and restart the Nginx service

handlers:
     - name: reload the systemd daemon and restart the Nginx service
       systemd_service:
          daemon_reload: true
          enabled: true
          name: nginx
          state: restarted  
```

Покажем в терминале:

<img width="1324" height="434" alt="image" src="https://github.com/user-attachments/assets/5406d1e1-40ca-4750-b619-0ccdbfa2ef34" />
<img width="1172" height="178" alt="image" src="https://github.com/user-attachments/assets/4f6e9f08-0671-4e2f-bc14-95883725659e" />

4. Кастомизация `index.html`:
   
- Заменить стандартный `/usr/share/nginx/html/index.html` (путь может отличаться в зависимости от `OS`) на файл, содержащий строку: `Phoenix Load Balancer: <HOSTNAME>`, где `<HOSTNAME>` — это реальное `hostname` виртуальной машины (использовать факты `Ansible`):

```bash
     - name: customization index.html
       copy:
          content: |
             Phoenix Load Balancer: {{ ansible_facts['nodename'] }}
          dest: /usr/share/nginx/html/index.html      
          owner: root
          group: root
          mode: 0644
```

Покажем в терминале:

<img width="1531" height="191" alt="image" src="https://github.com/user-attachments/assets/88d56e48-f77c-4c53-8d96-e226fa93d623" />

**Сценарий** `2`: Настройка `Database Server` (Целевая группа: `dbservers`)

1. Установка `PostgreSQL`:

- Установить `PostgreSQL` версии `14` (или `15`). Если в стандартных репозиториях нет этой версии — подключить официальный репозиторий `PGDG` (`PostgreSQL Global Development Group`):

```bash
- name: Configuring the Database Server
  hosts: dbservers
  become: true
  tags: postgresql

  tasks:

     - name: add postgreSQL repository GPG key
       apt_key: 
          url: https://www.postgresql.org/media/keys/ACCC4CF8.asc
          state: present

     - name: add postgreSQL apt repository
       apt_repository:
          repo: 'deb https://apt.postgresql.org/pub/repos/apt jammy-pgdg main 15'
          filename: postgresql
          state: present
    
     - name: install postgresql
       apt:
         name: postgresql-15
         state: present
         update_cache: yes
```

Покажем в терминале:

<img width="1448" height="545" alt="image" src="https://github.com/user-attachments/assets/bbb6e5b6-7daf-4255-87f6-73b834c1b577" />

2. Установка `Python`-библиотек:

- Установить пакет `python3-psycopg2` (или соответствующий адаптер для вашей `OS`), необходимый `Ansible` для управления базой данных (модули `postgresql_db`, `postgresql_user`):

```bash
     - name: install python postgresql adapter
       apt: 
          name: python3-psycopg2
          state: present 
```

Покажем в терминале:

<img width="1229" height="102" alt="image" src="https://github.com/user-attachments/assets/b9fc7edf-dc83-430d-b877-cdb9126ad092" />

3. Конфигурация сети (`Listen Addresses`):

- По умолчанию `Postgres` слушает только `localhost`.
- Найти конфигурационный файл `postgresql.conf`.
- Заменить строку `listen_addresses = 'localhost'` на `listen_addresses = '*'`. Использовать модуль `lineinfile` с регулярным выражением (`regexp`), чтобы замена была идемпотентной (не дублировалась при повторных запусках):

```bash
     - name: network configuration
       lineinfile:
          path: /etc/postgresql/15/main/postgresql.conf
          regexp: '^#?\s*listen_addresses\s*=\s*.*localhost.*'
          line: "listen_addresses = '*'"
          state: present
       notify: restart postgresql service
  handlers:
     - name: restart postgresql service
       systemd_service:
          enabled: true
          name: postgresql
          state: restarted  
```

Покажем в терминале:

<img width="1453" height="171" alt="image" src="https://github.com/user-attachments/assets/eee23b63-b489-46e1-9643-ffcc5ce197ce" />
<img width="1497" height="152" alt="image" src="https://github.com/user-attachments/assets/603a483b-b2ce-4b4a-a1a6-e46bde53b38a" />


4.Управление состоянием базы:

- Создать базу данных с именем `phoenix_db`.
- Создать пользователя БД `phoenix_user` с паролем `secure_pass` (пароль передавать текстом, шифрование `vault` пока не используем).
- Выдать пользователю полные права на базу.
**Условие**: Сервис `postgresql` должен быть запущен и добавлен в автозагрузку:

  ```bash
     - name: create a new database with name "phoenix"
       community.postgresql.postgresql_db:
          name: phoenix_db
          login_user: postgres
          login_password: "postgres"

     - name: create a new database ansible_user "phoenix_user"
       community.postgresql.postgresql_user:
          name: phoenix_user
          password: secure_pass
          state: present
          login_user: postgres
          login_password: "postgres"

     - name: grant all privileges to phoenix_user on database
       community.postgresql.postgresql_privs:
          database: phoenix_db
          roles: phoenix_user
          privs: ALL
          type: database
          state: present
          login_user: postgres
          login_password: "postgres"
  ```

  Покажем в терминале:

  <img width="1203" height="545" alt="image" src="https://github.com/user-attachments/assets/d229e99d-2653-4ccc-ad03-7ed3378535bf" />

На сервере `dbserver` внесём изменения в файл `/etc/postgresql/15/main/pg_hba.conf`:

```bash
# pg_hba.conf управляет аутентификацией: кто, откуда и как может подключиться к PostgreSQL
# host — подключение по TCP/IP
# phoenix_db — к какой бд подключение осуществляется
# phoenix_user — какому пользователю разрешено подключение к данной бд
# 0.0.0.0/0 — с любого IP-адреса
# md5 — метод аутентификаци: пароль в зашифрованном виде

host    phoenix_db      phoenix_user    0.0.0.0/0               md5
```

Покажем в терминале:

<img width="949" height="27" alt="image" src="https://github.com/user-attachments/assets/ac94e887-fc62-4f58-9728-e494e6d51974" />

А файл `/etc/postgresql/15/main/postgresql.conf` определяет параметры работы сервера `PostgreSQL`: сетевые настройки, производительность, безопасность и логирование. 

**Критерии приемки** (`Definition of Done`)

1. Команда `ansible-playbook site.yml` проходит без ошибок:

```bash
ansible-playbook site.yml
```

Покажем в терминале:

<img width="1812" height="708" alt="image" src="https://github.com/user-attachments/assets/98d80de1-2ac4-4271-92c6-90fd601988c3" />
<img width="1664" height="552" alt="image" src="https://github.com/user-attachments/assets/1ecbf70e-e689-4282-97e0-e1cbb3e1979d" />

2. Повторный запуск этой же команды возвращает `changed=0` для всех тасков (кроме, возможно, `apt update`, если кэш устарел). Это строгое требование идемпотентности. Покажем в терминале:

<img width="1694" height="851" alt="image" src="https://github.com/user-attachments/assets/6759385e-394f-4e09-9587-3663c0701e91" />
<img width="1560" height="426" alt="image" src="https://github.com/user-attachments/assets/611cb9fa-82ed-4b86-b917-c163c2999e47" />


3. На `vm-lb`:

- `nginx -v` показывает требуемую зафиксированную версию:

```bash
ansible balancers -m shell -a "nginx -v"
```

Покажем в терминале:

<img width="1296" height="72" alt="image" src="https://github.com/user-attachments/assets/21fc6e5e-8274-4629-af90-7a375488b302" />

- `cat /proc/$(pgrep -f "nginx: master")/limits` показывает` Max open files` равным `65536`:

```bash
ansible balancers -m shell -a 'cat /proc/$(pgrep -f "nginx: master" | head -n1)/limits | grep "Max open files"'
```

Покажем в терминале:

<img width="1693" height="69" alt="image" src="https://github.com/user-attachments/assets/5184fa57-bbfa-492b-ad69-4635ec9b1771" />

- `curl localhost` возвращает строку с `hostname`:

```bash
ansible balancers -m shell -a "curl localhost"
```

Покажем в терминале:

<img width="1825" height="116" alt="image" src="https://github.com/user-attachments/assets/faedd1dd-e1d0-4537-a9ff-b1eda1e390c6" />

4. На `vm-db`:

- Порт `5432` слушает интерфейс `0.0.0.0` (проверка `ss -tulpn | grep 5432`):

```bash
ansible dbservers -m shell -a "ss -tulpn | grep 5432"
```

Покажем в терминале:

<img width="1698" height="94" alt="image" src="https://github.com/user-attachments/assets/5b30147e-60e7-473e-bf15-6206eef1095f" />

- Можно подключиться к базе: `psql -h <IP_VM_DB> -U phoenix_user -d phoenix_db -W` (с другой машины):

```bash
psql -h 104.154.193.121 -U phoenix_user -d phoenix_db -W 
```

Покажем в терминале:

<img width="1744" height="201" alt="image" src="https://github.com/user-attachments/assets/5aba6823-865d-4b3a-89d9-11ea45d415be" />
  
**Важно**: так как мои `instances` находятся в `Google Cloud`, то мне пришлось настроить `Firewall`, чтобы я мог подключаться к сети `dbservers`.
