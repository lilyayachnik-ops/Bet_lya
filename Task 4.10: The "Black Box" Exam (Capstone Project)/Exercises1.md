# Checkpoints

**Вводные данные**:

1. `Infrastructure Reset`: Твои `3` виртуальные машины должны быть чистыми (или считай их чистыми). Удали с них весь софт или пересоздай.
Ресурсы:

- `vm-1`: `Frontend (Public entry point)`.
- `vm-2`: `Backend (Application logic)`.
- `vm-3`: `Storage (Redis Database)`.

**Техническое задание** (`Spec Sheet`):

Тебе необходимо написать полный набор `Ansible`-кода (инвентарь, плейбук `deploy_chimera.yml`, роли) для реализации следующей архитектуры.

1. Уровень безопасности (`Security Layer`):

- `Users`: На всех серверах должен быть создан пользователь `chimera_admin` с доступом по `SSH`-ключу. Вход для `root` по `SSH` должен быть запрещен.

Сгенерируем `SSH`-ключ `black_box_key` для нашего проекта:

```bash
ls -la ~/.ssh/
```

Покажем в терминале:

<img width="601" height="316" alt="image" src="https://github.com/user-attachments/assets/1611e00d-3451-46d1-9604-8143ac4308e1" />

Вставим в `Metadata` публичную часть ключа для пользователя `chimera_admin`, который будет создан автоматически. Покажем в терминале:

<img width="1436" height="272" alt="image" src="https://github.com/user-attachments/assets/1405e3b8-fa94-4d4d-a5db-0b08e4201c62" />

- `Firewall` (**Новое**!): На серверах должен быть настроен фаервол (`UFW` для `Ubuntu` или `Firewalld` для `CentOS`).
  - `vm-1`: Открыты порты `22` (`SSH`), `80` (`HTTP`), `443` (`HTTPS`).
  - `vm-2` и `vm-3`: Открыт только `22` порт и порты приложений, но доступ к портам приложений разрешен только с внутренних `IP`-адресов соседних ВМ. (Никто извне не должен попасть напрямую в базу или бэкенд).

Создадим роль `security_layer`, используя команду:

```bash
ansible-galaxy init roles/security_layer
```

Покажем в терминале:

<img width="232" height="416" alt="image" src="https://github.com/user-attachments/assets/a5a69bd5-695e-4470-87a3-6da66c7583f9" />

Содержимое в `roles/security_layer/tasks/main.yml`:

```bash
---
# tasks file for roles/security_layer
- name: Reset UFW to clean state
  community.general.ufw:
     state: "reset"

- name: Set default deny policy for all incoming connections
  community.general.ufw:
     direction: "{{ security_layer_direction }}"
     policy: "{{ security_layer_ufw_default_policy }}"

- name: Allow SSH for all
  community.general.ufw:
     rule: "{{ security_layer_ssh.policy }}"
     port: "{{ security_layer_ssh.port }}"
     proto: "{{ security_layer_ssh.protocol }}"

- name: Configure public web ports on frontend server
  community.general.ufw:
     rule: "{{ security_layer_frontend.policy }}"
     port: "{{ item }}"
     proto: "{{ security_layer_frontend.protocol }}"
  loop: "{{ security_layer_frontend.allowed_ports }}"
  when: "security_layer_groups.frontend in group_names"

- name: Configure restricted application port on backend server
  community.general.ufw:
    rule: "{{ security_layer_backend.policy }}"
    port: "{{ item }}"
    src: "{{ hostvars['frontend']['ansible_default_ipv4']['address'] }}/32"
    proto: "{{ security_layer_backend.protocol }}"
  loop: "{{ security_layer_backend.allowed_ports }}"
  when: "security_layer_groups.backend in group_names"

- name: Configure restricted database port on storage server
  community.general.ufw:
    rule: "{{ security_layer_storage.policy }}"
    port: "{{ item }}"
    src: "{{ hostvars['backend']['ansible_default_ipv4']['address'] }}/32"
    proto: "{{ security_layer_storage.protocol }}"
  loop: "{{ security_layer_storage.allowed_ports }}"
  when: "security_layer_groups.storage in group_names"

- name: Enable UFW firewall
  community.general.ufw:
    state: enabled
```

Покажем в терминале:

<img width="914" height="787" alt="image" src="https://github.com/user-attachments/assets/26a3b180-19bd-462c-9c35-b48b91a5849a" />
<img width="1298" height="306" alt="image" src="https://github.com/user-attachments/assets/10bae9fa-2e93-428b-bd03-3958bd17e0dd" />

Содержимое в `roles/security_layer/defaults/main.yml`:

```bash
---
# defaults file for roles/security_layer
security_layer_ufw_default_policy: "deny"
security_layer_direction: "incoming"

security_layer_ssh:
   port: "22"
   policy: "allow"
   protocol: "tcp"

security_layer_frontend:
   allowed_ports:
      - "80"
      - "443"
   policy: "allow"
   protocol: "tcp"

security_layer_backend:
   allowed_ports:
      - "8000"
   policy: "allow"
   protocol: "tcp"

security_layer_storage:
   allowed_ports:
      - "6379"
   policy: "allow"
   protocol: "tcp"

security_layer_groups:
   frontend: "frontends"
   backend: "backends"
   storage: "storages"
```

Покажем в терминале:

<img width="989" height="779" alt="image" src="https://github.com/user-attachments/assets/b1619b64-d733-47e9-913a-53d7455eef5b" />

- `Secrets`: Все пароли и токены должны быть в `Vault`.

2. **Уровень Приложения** (`Application Layer`) — `vm-2`

- `Language`: `Python 3`.

- `App Logic`: Тебе нужно написать (или найти) простейшее приложение на `Flask`, которое:
 
  - Принимает запрос.
  - Инкрементирует счетчик посещений в Redis.
  - Возвращает строку: `Chimera Node [Hostname]: Visits [Count]`.

- `Execution`: Приложение должно запускаться через `Gunicorn` (`WSGI server`) и управляться через `Systemd`:

  - Не запускай `python app.py` вручную в `screen` или `bg`. Это должен быть полноценный сервис: `systemctl start chimera-app`.

- `Config`: Приложение должно брать хост `Redis` из переменной окружения `REDIS_HOST`, которую `Systemd` получает из файла конфигурации.

Содержимое `roles/backend/tasks/main.yml`:

```bash
---
- name: Install Python
  ansible.builtin.apt:
    name: "{{ item }}"
    state: present
  loop: "{{ backend_.software }}"
- name: Create app directory
  ansible.builtin.file:
    path: /opt/{{ app_name }}
    state: directory
    owner: "{{ app_user }}"
    group: "{{ app_user }}"
    mode: "0750"

- name: Install Python packages
  ansible.builtin.pip:
    virtualenv: /opt/{{ app_name }}/venv
    virtualenv_python: python3
    name:
      - gunicorn
      - flask
      - redis

- name: Create Flask app
  ansible.builtin.template:
    src: app.py.j2
    dest: /opt/{{ app_name }}/app.py
    owner: "{{ app_user }}"
    group: "{{ app_user }}"
    mode: "0640"

- name: Create systemd service
  ansible.builtin.template:
    src: chimera-app.service.j2
    dest: /etc/systemd/system/chimera-app.service
    mode: "0644"
  notify: Restart app

- name: Start application
  ansible.builtin.service:
    name: chimera-app
    state: started
    enabled: true

```

Покажем в терминале:

<img width="758" height="872" alt="image" src="https://github.com/user-attachments/assets/f8e754ac-a568-422d-809a-b98e87a96be4" />
<img width="984" height="238" alt="image" src="https://github.com/user-attachments/assets/2b760b50-ea46-46e9-aadb-64fa124b784d" />

Содержимое `roles/backend/handlers/main.yml`:

```bash
---
# handlers file for roles/backend
- name: Restart app
  ansible.builtin.service:
    daemon_reload: yes
    name: chimera-app
    enabled: yes
    state: restarted
```

Покажем в терминале:

<img width="583" height="219" alt="image" src="https://github.com/user-attachments/assets/5d3504f5-8d20-4642-8b53-5fcd5be9f59e" />

Содержимое в `roles/backend/defaults/main.yml`:

```bash
---
# defaults file for roles/backend
backend_:
   software:
      - python3
      - python3-pip
      - virtualenv
```

Покажем в терминале:

<img width="740" height="193" alt="image" src="https://github.com/user-attachments/assets/f2c39986-d65c-455f-a108-61c617c609f3" />

Содержимое в `roles/backend/templates/app.py.j2`:

```bash
import os
import redis
from flask import Flask

app = Flask(__name__)

redis_client = redis.Redis(
    host=os.getenv('REDIS_HOST'),
    password=os.getenv('REDIS_PASSWORD'),
    decode_responses=True
)

@app.route('/')
def index():
    visits = redis_client.incr('visits')
    hostname = os.getenv('HOSTNAME')
    return f"Chimera Node {hostname}: Visits {visits}"
```

Покажем в терминале:

<img width="994" height="430" alt="image" src="https://github.com/user-attachments/assets/21bb3adf-9ea3-4812-bb0c-f8658955077a" />

Содержимое в `roles/backend/templates/chimera-app.service.j2`:

```bash
[Unit]
Description=Chimera App
After=network.target

[Service]
User={{ app_user }}
WorkingDirectory=/opt/{{ app_name }}
Environment="REDIS_HOST={{ hostvars['storage']['ansible_default_ipv4']['address'] }}"
Environment="REDIS_PASSWORD={{ storage_redis_password }}"
Environment="HOSTNAME={{ ansible_hostname }}"
ExecStart=/opt/{{ app_name }}/venv/bin/gunicorn --bind 0.0.0.0:{{ app_port }} app:app
Restart=always

[Install]
WantedBy=multi-user.target
```

Покажем в терминале:

<img width="1446" height="394" alt="image" src="https://github.com/user-attachments/assets/4a599dbb-1cfa-4c2f-9f99-4898abab76f2" />

3. **Уровень Хранения** (`Storage Layer`) — `vm-3`
 
 - Установить `Redis`.
 - Настроить `bind`, чтобы он слушал не только `localhost`, но и внутренний интерфейс (для доступа с `vm-2`).
 - Настроить `requirepass` (пароль для подключения). Приложение должно знать этот пароль.

Содержимое в `roles/storage/tasks/main.yml`:

```bash
---
# tasks file for roles/storage
- name: Install Redis
  ansible.builtin.package:
    name: redis-server
    state: present
  when: ansible_os_family == 'Debian'

- name: Create Redis configuration directory
  ansible.builtin.file:
    path: /etc/redis
    state: directory
    mode: "2750"

- name: Configure Redis
  ansible.builtin.template:
    src: redis.conf.j2
    dest: /etc/redis/redis.conf
    owner: redis
    group: redis
    mode: "0640"
  notify: Restart redis

- name: Enable Redis
  ansible.builtin.service:
    name: redis-server
    state: started
    enabled: true
```

Покажем в терминале:

<img width="842" height="693" alt="image" src="https://github.com/user-attachments/assets/526b20a1-728c-4a54-9432-e77148ccb15c" />

Содержимое в `storage/handlers/main.yml`:

```bash
---
# handlers file for roles/storage
- name: Restart redis
  ansible.builtin.service:
     name: redis-server
     state: restarted
```

Покажем в терминале:

<img width="571" height="171" alt="image" src="https://github.com/user-attachments/assets/0a9eea61-420e-4608-8e83-87308117633c" />


Содержимое в `roles/storage/default/main.yml`:

```bash
---
# defaults file for roles/storage
storage_redis_port: 6379
storage_redis_password: !vault |
          $ANSIBLE_VAULT;1.1;AES256
          33613665303433613832623738343066663262663833373938306463323464613232373064633432
          3761666561356631663434626337396464643631306563610a356332646235653938663263376362
          63623938393332316237653663326231363434343064653232343566663762393263666565643061
          3737336131313365620a336163333133626534623262383336316533663265313637653435646139
          6237
```

Покажем в терминале:

<img width="1205" height="239" alt="image" src="https://github.com/user-attachments/assets/7fc8d433-d4b3-4d6d-b3d1-939c73601d8b" />

Содержимое в `roles/storage/templates/redis.conf.j2`:

```bash
bind 127.0.0.1 {{ ansible_default_ipv4['address'] }}
port {{ storage_redis_port }}
requirepass {{ storage_redis_password }}
```

Покажем в терминале:

<img width="752" height="103" alt="image" src="https://github.com/user-attachments/assets/e5168107-9cb7-4b81-a518-d26c735f6516" />

4. **Уровень Маршрутизации** (`Frontend Layer`) — `vm-1`

- Установить `Nginx`.
- Настроить `Reverse Proxy`: Все запросы, приходящие на `vm-1`, должны проксироваться на `Gunicorn` (`vm-2`).
- `TLS/SSL`: Сайт должен работать по `HTTPS`.
  - Сгенерируй самоподписанный сертификат прямо в плейбуке (через модуль `openssl_...`).
  - Настрой редирект с `HTTP` на `HTTPS`.
 
Содержимое в `roles/frontend/tasks/main.yml`:

```bash
---
# tasks file for roles/frontend
- name: Install Nginx
  ansible.builtin.apt:
    name: nginx
    state: present

- name: Generate SSL certificate
  ansible.builtin.openssl_privatekey:
    path: /etc/ssl/private/chimera.key

- name: Generate self-signed certificate
  ansible.builtin.openssl_certificate:
    path: /etc/ssl/certs/chimera.crt
    privatekey_path: /etc/ssl/private/chimera.key
    provider: selfsigned

- name: Configure Nginx
  ansible.builtin.template:
    src: chimera-site.conf.j2
    dest: /etc/nginx/sites-available/chimera
    mode: "0644"
   # validate: "nginx -t"
  notify: Restart nginx

- name: Remove default Nginx site
  ansible.builtin.file:
    path: /etc/nginx/sites-enabled/default
    state: absent
  notify: Restart nginx

- name: Enable site
  ansible.builtin.file:
    src: /etc/nginx/sites-available/chimera
    dest: /etc/nginx/sites-enabled/chimera
    state: link
  notify: Restart nginx

- name: Start Nginx
  ansible.builtin.service:
    name: nginx
    state: started
    enabled: true
```

Покажем в терминале:

<img width="1467" height="901" alt="image" src="https://github.com/user-attachments/assets/5f47d5b4-5119-4114-9866-6d372f651e93" />
<img width="1206" height="313" alt="image" src="https://github.com/user-attachments/assets/e87ffde8-de95-427e-9b30-8543038179b6" />

Содержимое в `roles/frontend/defaults/main.yml`:

```bash
---
# defaults file for roles/frontend
frontend_nginx_listen_port: 80
frontend_nginx_config_path: "/etc/nginx/nginx.conf"
```

Покажем в терминале:

<img width="1198" height="123" alt="image" src="https://github.com/user-attachments/assets/1032edff-f5b1-4c3d-bbd3-318066b2625d" />

Содержимое в `roles/frontend/handlers/main.yml`:

```bash
---
# handlers file for roles/frontend
- name: Restart nginx
  ansible.builtin.service:
    name: nginx
    state: restarted
```

Покажем в терминале:

<img width="1000" height="169" alt="image" src="https://github.com/user-attachments/assets/340d426f-74dd-4ba0-b240-78ad5226afc8" />

Содержимое в `roles/frontend/templates/chimera-site.conf.j2`:

```bash
upstream backend {
    server {{ hostvars['backend']['ansible_default_ipv4']['address'] }}:8000;
}

server {
    listen 80;
    server_name blackbox.local;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl;
    server_name blackbox.local;
    
    ssl_certificate /etc/ssl/certs/chimera.crt;
    ssl_certificate_key /etc/ssl/private/chimera.key;
    
    location / {
        proxy_pass http://backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

Покажем в терминале:

<img width="1404" height="626" alt="image" src="https://github.com/user-attachments/assets/e57ad7e7-24c4-4e48-aab8-beba031cd083" />

Содержимое в `deploy_chimera.yml`:

```bash
---
- name: Base Setup
  become: true
  hosts: all
  gather_facts: true
  roles:
    - geerlingguy.security
    - security_layer

- name: Deploy Redis Storage
  hosts: storage
  gather_facts: false
  become: true
  roles:
    - storage

- name: Deploy Backend Application
  hosts: backend
  gather_facts: false
  become: true
  vars_files:
    - "./roles/storage/defaults/main.yml"
  vars:
    storage_redis_password: "{{ hostvars['storage']['storage_redis_password'] }}"
    redis_host: "{{ hostvars['storage']['ansible_default_ipv4']['address'] }}"
  roles:
    - backend

- name: Deploy Frontend Proxy
  hosts: frontend
  gather_facts: false
  become: true
  roles:
    - role: frontend
```

Покажем в терминале:

<img width="1277" height="831" alt="image" src="https://github.com/user-attachments/assets/c4db803a-0aa4-4dda-9be6-7eb223ead796" />

Содержимое в `requirements.yml`:

```bash
---
collections:
   - name: community.general
   - name: community.crypto
roles:
   - name: geerlingguy.security
```

Покажем в терминале:

<img width="656" height="175" alt="image" src="https://github.com/user-attachments/assets/cd2838f5-8afc-4f3a-b0ae-a5be241fb9d6" />

Содержимое в `ansible.cfg`:

```bash
[defaults]
inventory=./inventory/hosts.yml
host_key_checking=false
log_path=./ansible.log
action_warnings=false
vault_password_file=./.vault_pass
forks=10
callbacks_enabled=ansible.posix.profile_tasks

[vault]
vault_identity_list=default@./.vault_pass

[ssh_connection]
pipelining=true
ssh_args= -C -o ControlMaster=auto -o ControlPersist=15m
```

Покажем в терминале:

<img width="961" height="389" alt="image" src="https://github.com/user-attachments/assets/f25eda6e-4184-4ae4-90a2-3e8192d3de50" />

Содержимое в `group_vars/all.yml`:

```bash
---
app_name: "chimera"
app_user: "chimera_admin"
app_port: 8000
```

Покажем в терминале:

<img width="842" height="124" alt="image" src="https://github.com/user-attachments/assets/224478be-48c4-4587-ba8a-144eaa555581" />

**Правила выполнения**

1. **Строгая модульность**: Никаких `tasks` в основном плейбуке. Только роли (`roles/firewall`, `roles/backend`, `roles/frontend`, `roles/common`).

2. `Linting`: Проект должен проходить `ansible-lint` без ошибок.

3. `One-Button Deploy`: Я должен клонировать твой репозиторий, добавить свой SSH-ключ, запустить `ansible-playbook deploy_chimera.yml` и через `5` минут получить рабочий `URL`.

4. **Самостоятельность**: В этом задании нет подсказок по модулям. Тебе нужно нагуглить, как настроить `UFW` через `Ansible` или как создать `systemd unit`-файл через шаблон.

**Критерии приемки** (`Final Definition of Done`)

**Ты сдал экзамен, если**:

1.1. `Functional Test`: Я выполняю команду: `curl -k https://<IP_VM_1>` И получаю ответ: `Chimera Node vm-2: Visits 1` Повторяю команду: Ответ: `Chimera Node vm-2: Visits 2`. Покажем в терминале:

<img width="959" height="96" alt="image" src="https://github.com/user-attachments/assets/df15acbf-bd3a-4ffd-87be-081b07fa4ce5" />

2. `Security Test`: Я пытаюсь подключиться к `Redis` напрямую с твоего ноутбука: `redis-cli -h <IP_VM_3>` — и получаю таймаут (`Connect failed`), потому что фаервол меня не пускает. Покажем в терминале:

<img width="834" height="46" alt="image" src="https://github.com/user-attachments/assets/436e80ac-96ac-4f62-8ba6-e37d1e51fd60" />

3. `Resilience Test`: Я перезагружаю все `3` сервера (`reboot`). После загрузки приложение само поднимается, и сайт снова доступен.

<img width="1310" height="795" alt="image" src="https://github.com/user-attachments/assets/531186e4-009d-4cac-a761-608fb251e3a7" />
<img width="1179" height="186" alt="image" src="https://github.com/user-attachments/assets/0149b1c0-4031-4fa5-90b4-5816ce0111e0" />

4. `Code Quality`: В репозитории нет открытых секретов. Структура папок соответствует `Best Practices`.
