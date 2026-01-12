# Checkpoints

**Задание** `6.1`: Использование `Community Roles` (`Galaxy`) Мы не будем писать `hardening` с нуля:

- Создай файл `requirements.yml` в корне проекта:

```bash
touch requirements.yml
```

Покажем в терминале:

<img width="677" height="29" alt="image" src="https://github.com/user-attachments/assets/eab0dcab-d741-486f-a86c-1e184bd68174" />

- Опиши в нем зависимость от роли `geerlingguy.security` (это "золотой стандарт" в мире `Ansible`, пусть привыкают к качественному коду):

```bash
---
roles:
   - name: geerlingguy.security
```

Покажем в терминале:

<img width="940" height="76" alt="image" src="https://github.com/user-attachments/assets/d38df295-a932-4654-a066-0e9c2c334b35" />

- Установи роль локально в папку `roles/` командой: ansible-galaxy install -r requirements.yml -p ./roles

```bash
# ansible-galaxy install — основная команда для установки ролей из Ansible Galaxy (репозиторий ролей)
# -r requirements.yml — указывает файл требований, который содержит список ролей для установки
# -p ./roles — параметр -p указывает целевую директорию ./roles для установки ролей
$ ansible-galaxy install -r requirements.yml -p ./roles
```

Покажем в терминале:

<img width="1312" height="89" alt="image" src="https://github.com/user-attachments/assets/d358d5c3-60ff-4d6c-92ce-cb5141d3d4d0" />

- **Сложность**: Эта роль по умолчанию может отключить вход по `SSH` по паролю или сменить порт.
 - Изучи `defaults/main.yml` этой роли (на `GitHub` или в скачанной папке).
 - В своем `group_vars/all.yml` переопредели переменные этой роли так, чтобы она не сломала тебе доступ к ВМ (например, оставь `security_ssh_password_authentication: true`, если ты еще не настроил ключи везде идеально).
- Добавь вызов этой роли в `site.yml` для группы `all`.

```bash
---
- name: Base Setup
  hosts: all
  gather_facts: true
  roles:
    - geerlingguy.security
```

Покажем в терминале:

<img width="577" height="144" alt="image" src="https://github.com/user-attachments/assets/89946c3c-f9af-40d4-b191-8a1bc14afc01" />

**Задание** `6.2`: Рефакторинг `Nginx` (Создание своей роли) Преврати свой код настройки балансировщика в роль `phoenix_nginx`:

- Используй команду `ansible-galaxy init roles/phoenix_nginx` для создания скелета. Удали пустые папки, которые не используешь.
- **Перенос логики**:
 - Таски установки, настройки и шаблонизации перенеси в `tasks/main.yml`.
 - Хендлер (рестарт `Nginx`) перенеси в `handlers/main.yml`. Обрати внимание: в тасках теперь нужно вызывать хендлер просто по имени, без указания пути.
 - Шаблон `nginx.conf.j2` перенеси в `templates/`.
- **Гибкость** (`Hard`):
 - В `defaults/main.yml` роли вынеси порт, на котором слушает `Nginx` (по умолчанию `80`).
 - В `tasks/main.yml` замени хардкод порта `80` на переменную `{{ nginx_listen_port }}`.
 - Сделай так, чтобы имя файла конфига (`/etc/nginx/nginx.conf или /etc/nginx/conf.d/default.conf`) тоже было переменной.

```bash
# ansible-galaxy — утилита для управления ролями Ansible
# init — нужна для инициализации новой роли
# roles/phoenix_nginx — путь и имя создаваемой роли

$ ansible-galaxy init roles/phoenix_nginx
```

Cодержимое `tasks/main.yml`:

```bash
---
- name: add nginx repository GPG key
  become: true
  apt_key:
     url: https://nginx.org/keys/nginx_signing.key
     state: present

- name: add nginx apt repository
  become: true
  apt_repository:
     repo: 'deb https://nginx.org/packages/mainline/ubuntu/ jammy nginx'
     state: present
     filename: nginx


- name: install nginx
  become: true
  apt:
     name: "nginx={{ nginx.version }}"
     state: present
     update_cache: yes
         

- name: create the drop-in directory
  become: true
  file:
     owner: root
     group: root 
     path:  /etc/systemd/system/nginx.service.d 
     mode: 0755
     state: directory   
     
- name: add the configuration
  become: true
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
  become: true
  copy:
     content: |
        {{ project_name }} Load Balancer: {{ ansible_facts['nodename'] }}
     dest: /usr/share/nginx/html/index.html      
     owner: root
     group: root
     mode: 0644

- name: create nginx.conf from template
  become: true
  template:
     src: nginx.conf.j2
     dest: "{{ nginx_config_path }}"
     owner: root
     group: root
     mode: '0644'
     validate: nginx -t -c %s
  notify: reload nginx

```

Покажем в терминале:

<img width="1167" height="736" alt="image" src="https://github.com/user-attachments/assets/ffb082c4-4ee6-4515-8f51-5a7c683e1ad1" />
<img width="1278" height="780" alt="image" src="https://github.com/user-attachments/assets/6caac6c0-58b4-4360-9853-65c42ab4301b" />

Содержимое `handlers/main.yml`:

```bash
---
- name: reload the systemd daemon and restart the Nginx service
  become: true
  systemd_service:
     daemon_reload: true
     enabled: true
     name: nginx
     state: restarted

- name: reload nginx
  become: true
  systemd_service:
      daemon_reload: true
      enabled: true
      name: nginx
      state: reloaded 
```

Покажем в терминале:

<img width="957" height="403" alt="image" src="https://github.com/user-attachments/assets/6ae98de9-a925-4954-b91b-a1495df89c4d" />

Перенесем содержимое `nginx.conf.j2` в `templates/`:

<img width="228" height="240" alt="image" src="https://github.com/user-attachments/assets/93081dc7-c8dd-4c9f-b766-197f0421220b" />

Содержимое `defaults/main.yml`:

```bash
---
nginx_listen_port: 80

nginx_config_path: "/etc/nginx/nginx.conf"
```

Покажем в терминале:

<img width="611" height="147" alt="image" src="https://github.com/user-attachments/assets/603048fa-0da3-47d3-8f1d-fd8b4d8e990a" />

Заменим порт `80` на `{{ nginx_listen_port }}` в файле `nginx.conf.j2`:

```bash
listen {{ nginx_listen_port }};
```

Покажем в терминале:

<img width="1580" height="697" alt="image" src="https://github.com/user-attachments/assets/99287620-0f2e-4810-b7da-24bc87d54288" />
<img width="1292" height="215" alt="image" src="https://github.com/user-attachments/assets/4c753ad7-cdab-4e33-9b54-211da400a8dd" />

**Задание** `6.3`: Роль Базы Данных и Зависимости (`Dependencies`) Выдели настройку `PostgreSQL` в роль `phoenix_db`:

- Все таски по установке и созданию БД/юзера — в роль.
- **Мета-зависимости**: Допустим, для работы БД нам критически важен `NTP` (синхронизация времени).
 - Найди или напиши простую роль `timesync` (или используй `geerlingguy.ntp`).
 - В файле `roles/phoenix_db/meta/main.yml` пропиши в секцию `dependencies роль NTP`.
 - **Эффект**: Теперь, когда ты применяешь роль БД, `Ansible` сам сначала запустит роль `NTP`, даже если ты не указал её в `site.yml`.

```bash
ansible-galaxy init roles/phoenix_db
```

Содержимое `tasks/main.yml`:

```bash
---
- name: add postgreSQL repository GPG key
  become: true
  apt_key: 
     url: https://www.postgresql.org/media/keys/ACCC4CF8.asc
     state: present

- name: add postgreSQL apt repository
  become: true
  apt_repository:
     repo: 'deb https://apt.postgresql.org/pub/repos/apt jammy-pgdg main {{ postgresql.version }}'
     filename: postgresql
     state: present
    
- name: install postgresql
  become: true
  apt:
     name: "postgresql-{{ postgresql.version }}"
     state: present
     update_cache: yes

- name: install python postgresql adapter
  become: true
  apt: 
     name: python3-psycopg2
     state: present 
     
- name: network configuration
  become: true
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
```

Покажем в терминале:

<img width="1233" height="852" alt="image" src="https://github.com/user-attachments/assets/4dd22215-d108-46c9-8a6a-e7a4487e8f94" />
<img width="921" height="568" alt="image" src="https://github.com/user-attachments/assets/6d6fd025-1e48-46b3-8d60-74fbe2b6955e" />

Содержимое `handlers/main.yml`:

```bash
---
- name: restart postgresql service
  systemd_service:
     enabled: true
     name: postgresql
     state: restarted
```

Покажем в терминале:

<img width="688" height="165" alt="image" src="https://github.com/user-attachments/assets/0cb49a3b-e8ba-44bb-b627-2ad0e2332486" />

Содержимое файла `defaults/main.yml`:

```bash
---
postgresql:
  db_name: phoenix_db
  user_name: phoenix_user
  version: 15
  login_user_name: postgres
```

Покажем в терминале:

<img width="525" height="172" alt="image" src="https://github.com/user-attachments/assets/149d7cd5-2cd6-4bab-b22b-b7a99bd2cb4e" />

Пропишем в `requirements.yml` роль `geerlingguy.ntp`. Покажем в терминале:

<img width="533" height="105" alt="image" src="https://github.com/user-attachments/assets/e18ea9f5-2997-4e56-a4c5-2721ad77c097" />

Далее в файле `roles/phoenix_db/meta/main.yml` пропишем в секцию `dependencies` роль `NTP`:

```bash
dependencies:
  - role: geerlingguy.ntp
    become: true
```

Покажем в терминале:

<img width="477" height="80" alt="image" src="https://github.com/user-attachments/assets/6d120e3c-14f3-41e4-9709-01a7218a3f4e" />

**Задание** `6.4`: Финальная сборка (`site.yml` - `Clean Code`) Твой `site.yml` должен превратиться в чистый манифест. Примерно так:

```bash
- name: Base Setup
  hosts: all
  roles:
    - geerlingguy.security
- name: Load Balancers
  hosts: balancers
  roles:
    - role: phoenix_nginx
      vars:
        nginx_listen_port: 8080
- name: Database Servers
  hosts: dbservers
  roles:
    - phoenix_db
```

Выполним команду `ansible-galaxy init roles/phoenix_app`. Содержимое файла `tasks/main.yml`:

```bash
---
- name: Configure custom facts for App servers
  block:
     - name: create a directory 
       file:
          owner: root
          group: root
          path: /etc/ansible/facts.d
          mode: 0755
          state: directory

     - name: add the file
       copy:
          content: |
             {"status": "{{ phoenix_status }}"}
          dest: /etc/ansible/facts.d/phoenix.fact
          owner: root 
          group: root
          mode: 0644
      
     - name: reload only custom facts
       setup:
          fact_path: /etc/ansible/facts.d
          filter: "ansible_local"
       changed_when: false


- name: create configuration directory
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

- name: create users
  user:
     name: "{{ item.name }}"
     uid: "{{ item.id }}"
     shell: "{{ item.shell }}"
     state: present
  loop: "{{ phoenix_services }}"  
       
- name: install net-tools
  become: true
  package:
     name: net-tools
     state: present 
  when: ansible_os_family in ["Debian", "RedHat"]
```

Покажем в терминале:

<img width="1144" height="828" alt="image" src="https://github.com/user-attachments/assets/37a5110f-c87e-47fa-a76c-d674bc97e6cb" />
<img width="1295" height="619" alt="image" src="https://github.com/user-attachments/assets/76029dfb-d39d-456d-867c-20d8be7ad2ed" />


**Критерии приемки** (`Definition of Done`)

1. В корне проекта чисто. Плейбук `site.yml` занимает не более `20-30` строк:

```bash
---
- name: Base Setup
  hosts: all
  gather_facts: true
  roles:
    - geerlingguy.security

- name: Load Balancers
  hosts: balancers
  gather_facts: false
  roles:
    - role: phoenix_nginx
      vars:
        nginx_listen_port: 8080

- name: Database Servers
  hosts: dbservers
  gather_facts: false
  roles:
    - phoenix_db

- name: Application Servers
  hosts: appservers
  gather_facts: false
  roles:
     - role: phoenix_app
       vars: 
         phoenix_status: "{{ target_state | default('active') }}"
```

Покажем в терминале:

<img width="1029" height="666" alt="image" src="https://github.com/user-attachments/assets/d934caeb-0575-47df-80c3-e19c9191a8e7" />

2. Команда `ansible-galaxy install` работает и скачивает внешние роли:

```bash
ansible-galaxy install -r ./requirements.yml -p ./roles
```

Покажем в терминале:

<img width="1091" height="85" alt="image" src="https://github.com/user-attachments/assets/4daa8322-6960-4d0b-aac1-7597e65f460f" />

3. На балансировщике `Nginx` теперь слушает порт `8080` (или тот, который ты задал в `site.yml`, перекрыв `defaults`):

```bash
ansible balancers -m shell -a "sudo cat /etc/nginx/nginx.conf"
```

Покажем в терминале:

<img width="1196" height="581" alt="image" src="https://github.com/user-attachments/assets/f10fabfe-34eb-49bd-8227-e587ba4d436c" />
<img width="887" height="186" alt="image" src="https://github.com/user-attachments/assets/7d4d64b3-1c40-41ad-924e-c66b9c6c017b" />

4. При запуске прогона на `dbservers` в логе видно, что сначала отрабатывают таски настройки времени (из зависимости в `meta`), а потом уже `Postgres`. Покажем в терминале:

<img width="1150" height="564" alt="image" src="https://github.com/user-attachments/assets/71eb946a-80b5-4ccf-b29f-1bc627c5789c" />
<img width="1312" height="713" alt="image" src="https://github.com/user-attachments/assets/c238e3a5-3047-4fb0-9dce-5970033f56fb" />
<img width="1095" height="124" alt="image" src="https://github.com/user-attachments/assets/d55f960c-318d-475c-abdc-d0caa9b1d359" />


5. Ты понимаешь, почему переменные в `roles/x/vars/main.yml` — это зло для переиспользуемых ролей (они слишком сложно переопределяются), и используешь `defaults/main.yml`.
