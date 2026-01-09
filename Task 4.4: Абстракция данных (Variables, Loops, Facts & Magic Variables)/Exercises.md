# Checkpoints

Подготовка: В корне проекта создай структуру папок:

```bash
group_vars/
    all.yml
    dbservers.yml
    balancers.yml
```

Покажем в терминале:

<img width="243" height="275" alt="image" src="https://github.com/user-attachments/assets/59e9c109-0174-460d-b2a8-a9502fcfdc60" />

**Задание** `4.1`: Рефакторинг (`Dry Run`) Вынеси "захардкоженные" данные из `site.yml` (`Task 3`) в переменные:

- В `group_vars/balancers.yml` перенеси версию `Nginx` и лимит открытых файлов.
- В `group_vars/dbservers.yml` перенеси имя базы данных, пользователя и версию `PostgreSQL`.
- В `group_vars/all.yml` вынеси название проекта (строку `Phoenix`, которая используется в `MOTD` и `index.html`).
- Обнови `site.yml`, заменив конкретные значения на `{{ variable_name }}`.
- Убедись, что плейбук продолжает работать идемпотентно.

Содержимое файла `group_vars/balancers.yml`:

```bash
---
nginx:
   version: "1.25.0-1~jammy"
   limit_nofile: 65536
```

Покажем в терминале:

<img width="597" height="137" alt="image" src="https://github.com/user-attachments/assets/8efd6c27-adf7-4a10-8119-244b2031f34c" />

Содержимое файла `group_vars/dbservers.yml`:

```bash
---
postgresql:
   db_name: phoenix_db
   user_name: phoenix_user
   version: 15
   login_user_name: postgres
```

Покажем в терминале:

<img width="659" height="172" alt="image" src="https://github.com/user-attachments/assets/26cf6aa2-c9c5-4435-8f35-296fd8243d14" />

Содержимое файла `group_vars/all.yml`:

```bash
---
project_name: Phoenix
```

Покажем в терминале:

<img width="563" height="77" alt="image" src="https://github.com/user-attachments/assets/257df210-9321-4e29-ace7-7f7673e80bfd" />

Содержимое сайта `site.yml` после обновления:

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
         name: "nginx={{ nginx.version }}"
         state: present
         update_cache: yes
         

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
             LimitNOFILE={{ nginx.limit_nofile }}
          dest: /etc/systemd/system/nginx.service.d/override.conf
          owner: root
          group: root
          mode: 0644
       notify: reload the systemd daemon and restart the Nginx service

     - name: customization index.html
       copy:
          content: |
             {{ project_name }} Load Balancer: {{ ansible_facts['nodename'] }}
          dest: /usr/share/nginx/html/index.html      
          owner: root
          group: root
          mode: 0644

  handlers:
     - name: reload the systemd daemon and restart the Nginx service
       systemd_service:
          daemon_reload: true
          enabled: true
          name: nginx
          state: restarted  


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
          repo: 'deb https://apt.postgresql.org/pub/repos/apt jammy-pgdg main {{ postgresql.version }}'
          filename: postgresql
          state: present
    
     - name: install postgresql
       apt:
         name: "postgresql-{{ postgresql.version }}"
         state: present
         update_cache: yes

     - name: install python postgresql adapter
       apt: 
          name: python3-psycopg2
          state: present 
     
     - name: network configuration
       lineinfile:
          path: /etc/postgresql/{{ postgresql.version }}/main/postgresql.conf
          regexp: '^#?\s*listen_addresses\s*=\s*.*localhost.*'
          line: "listen_addresses = '*'"
          state: present
       notify: restart postgresql service

     - name: create a new database with name "phoenix"
       community.postgresql.postgresql_db:
          name: "{{ postgresql.db_name }}"
          login_user: "{{ postgresql.login_user_name }}"
          login_password: "postgres"

     - name: create a new database ansible_user "phoenix_user"
       community.postgresql.postgresql_user:
          name: "{{ postgresql.user_name }}"
          password: secure_pass
          state: present
          login_user: "{{ postgresql.login_user_name }}"
          login_password: "postgres"

     - name: grant all privileges to phoenix_user on database
       community.postgresql.postgresql_privs:
          database: "{{ postgresql.db_name }}"
          roles: "{{ postgresql.user_name }}"
          privs: ALL
          type: database
          state: present
          login_user: "{{ postgresql.login_user_name }}"
          login_password: "postgres"

  handlers:
     - name: restart postgresql service
       systemd_service:
          enabled: true
          name: postgresql
          state: restarted  
```

Проверим, что `playbook` продолжает работает идемпотентно:

```bash
ansible-playbook site.yml --tags "nginx, postgresql"
```

Покажем в терминале:

<img width="1469" height="782" alt="image" src="https://github.com/user-attachments/assets/d49b3ec1-e67c-4814-8566-c0615c457e7c" />
<img width="1371" height="528" alt="image" src="https://github.com/user-attachments/assets/c12c6487-a045-47fc-8910-97108bc6209d" />

**Задание** `4.2`: Динамическое обнаружение (`Magic Variables`) Это самая важная часть таска: 

- На сервере группы `appservers` (создай новый `Play` в `site.yml` для этой группы) нужно создать конфигурационный файл подключения к БД.
- Создай файл `/etc/phoenix/db_connection.conf`. Содержимое файла должно быть следующим:


```bash
DB_HOST=<Внутренний_IP_сервера_DB>
DB_PORT=5432
DB_NAME=<Имя_БД_из_переменной>
```

**Условие**: Ты НЕ имеешь права писать `IP`-адрес сервера БД руками ни в `site.yml`, ни в `group_vars`. `Ansible` должен "подсмотреть" `IP`-адрес сервера, находящегося в группе `dbservers`, используя магическую переменную `hostvars` и списки групп `groups['dbservers']`.

```bash
- name: Configuring the App Servers
  hosts: appservers
  become: true
  tags: app

  tasks:
     
    # - name: Display a variable for a specific host
    #   debug:
    #      msg: "Dbserve IP address  is {{ hostvars['dbserver'].ansible_default_ipv4.address }}"

     - name: create configure directory
       file:
          owner: root
          group: root
          path: /etc/phoenix
          mode: 0755
          state: directory
     
     - name: add the configuration
       copy:
          content: |
             DB_HOST={{ hostvars[groups['dbservers'][0]]['ansible_default_ipv4']['address'] }}
             DB_PORT=5432
             DB_NAME={{ hostvars[groups['dbservers'][0]]['postgresql']['db_name'] }}
          dest: /etc/phoenix/db_connection.conf
          owner: root
          group: root
          mode: 0644
```

Покажем в терминале:

<img width="1162" height="686" alt="image" src="https://github.com/user-attachments/assets/59c3ea0a-f22f-404d-8cf9-8b91a8568863" />

**Задание** `4.3`: Циклы и Сложные структуры данных 

Нам нужно создать системных пользователей для микросервисов на `appservers`:

- В `group_vars/appservers.yml` определи список словарей:

```bash
phoenix_services:
  - name: phoenix-auth
    id: 3001
    shell: /bin/false
  - name: phoenix-core
    id: 3002
    shell: /bin/bash
  - name: phoenix-worker
    id: 3003
    shell: /bin/sh
```

Покажем в терминале:

<img width="779" height="332" alt="image" src="https://github.com/user-attachments/assets/1e346c5c-ffae-477a-8262-62994c1015ab" />

- В плейбуке используй один таск с циклом (`loop`), чтобы создать всех этих пользователей.
- Параметры пользователя (`uid`, `shell`, `name`) должны браться из элементов списка.

```bash
- name: create users
       user:
          name: "{{ item.name }}"
          uid: "{{ item.id }}"
          shell: "{{ item.shell }}"
          state: present
       loop: "{{ phoenix_services }}"  
```

Покажем в терминале:

<img width="827" height="169" alt="image" src="https://github.com/user-attachments/assets/705aa20e-b2bd-4bf6-8667-20f0fdab1794" />

**Задание** `4.4`: Условная логика (`Conditionals`) Допустим, на проекте много операционных систем:

- Добавь таск установки пакета `net-tools` (выдуманный пакет, заменим на реальный).
- Реализуй логику:
 - Если `ОС` семейства `Debian` (`Ubuntu`) — установить пакет `net-tools`.
 - Если `ОС` семейства `RedHat` (`CentOS/Rocky`) — установить пакет `net-tools` (имя совпадает, но для тренировки используй условие `when` по семейству `ОС`).
 - Используй факт `ansible_os_family`.

```bash
    - name: install net-tools
       package:
          name: net-tools
          state: present 
       when: ansible_os_family in ["Debian", "RedHat"]
```

Покажем в терминале:

<img width="757" height="126" alt="image" src="https://github.com/user-attachments/assets/fed88c3a-6d4e-4e5a-9eb8-dfda369e6013" />

**Критерии приемки** (`Definition of Done`)

1. В файлах `site.yml` нет ни одного `IP`-адреса и ни одного номера версии софта.
2. На `appservers` создан файл `/etc/phoenix/db_connection.conf`, и в поле `DB_HOST` записан верный `IP`-адрес твоей виртуалки `managed-node-3`:

```bash
cat /etc/phoenix/db_connection.conf
```

Покажем в терминале:

<img width="964" height="79" alt="image" src="https://github.com/user-attachments/assets/e527524d-0fee-4b89-b93e-281a2fc43ad0" />
 
3. Если ты изменишь IP-адрес `vm-db` (на уровне инвентаря) и перезапустишь плейбук — конфиг на appservers должен обновиться автоматически. Перезагрузили `managed-node-3`, `IP` поменлся. Покажем в термнинале:

<img width="1288" height="71" alt="image" src="https://github.com/user-attachments/assets/735d15d5-3bcc-448a-b918-2ea109b6fc82" />

Внесём изменения в файл `./hosts`:

<img width="594" height="102" alt="image" src="https://github.com/user-attachments/assets/7786c13a-e64a-4591-a04c-fb9c4b54b051" />

Запустим `playbook`:

<img width="1150" height="98" alt="image" src="https://github.com/user-attachments/assets/802804bb-942a-424b-bc1a-3764fff7ca0b" />

Изменений не было, так как внутренние `IP`-адреса у `VM` в `Google Cloud` статические.

4. Команда `getent passwd | grep phoenix` на апп-сервере показывает `3` пользователей с правильными `UID` и `Shell`:

```bash
getent passwd | grep phoenix
```

Покажем в терминале:

<img width="1066" height="105" alt="image" src="https://github.com/user-attachments/assets/68ba4db7-5edc-4fa7-813a-b48e912a5b6b" />
