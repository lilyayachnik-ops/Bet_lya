# Checkpoints

**Задание** `5.1`: Подготовка механизма статусов (`Custom Facts`) Мы не хотим удалять хост из инвентаря, чтобы выключить на него трафик. Мы будем помечать его специальным фактом:

- Создай новый `Play` для группы `appservers`.
- **Задача**: Создать на серверах папку `/etc/ansible/facts.d`.
- **Задача**: Загрузить туда файл `phoenix.fact` (формат `JSON` или `INI`).
  - Содержимое по умолчанию: `{"status": "active"}`.
  - Сделай так, чтобы ты мог переопределить это состояние через внешнюю переменную при запуске плейбука (например, `-e "target_state=drain"`), но если переменная не передана — оставалось текущее состояние или `active`.
- **Задача**: Перечитать факты (`setup` модуль), чтобы `Ansible` увидел новые данные в `ansible_local`.

```bash
- name: Configure custom facts for App servers
  hosts: appservers
  become: true
  gather_facts: false
  tags: facts
  vars: 
     phoenix_status: "{{ target_state | default('active') }}"

  tasks:

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
```

Покажем в терминале:

<img width="1301" height="758" alt="image" src="https://github.com/user-attachments/assets/332975a5-961e-42f9-8402-b410f0381a3a" />

**Задание** `5.2`: Шаблон-монстр (`Jinja2 Logic`) Переходим к `balancers`. Нам нужно заменить статичный конфиг `Nginx` на шаблон `nginx.conf.j2`:

- Найди секцию `upstream backend { ... }` в конфиге `Nginx`.
- Напиши цикл на `Jinja2`, который:
  - Перебирает всех хостов из группы `appservers`.
  - Условие (`Hard`): В конфиг попадают только те серверы, у которых в кастомных фактах (`ansible_local.phoenix.status`) значение равно `active`.
- Если статус `drain` или факт отсутствует — сервер в конфиг не пишется.
- Формат строки внутри блока: `server <IP_ADDRESS>:80`;
- **Нюанс**: `IP`-адрес нужно брать из `hostvars[item]['ansible_host']`.

Содержимое файла `nginx.conf.j2`:

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
        {% for host in groups['appservers'] %}
            {% if hostvars[host]['ansible_local'] is defined 
                  and hostvars[host]['ansible_local']['phoenix'] is defined
                  and hostvars[host]['ansible_local']['phoenix']['status'] is defined
                  and hostvars[host]['ansible_local']['phoenix']['status'] == 'active' %}
        
        server {{ hostvars[host]['ansible_host'] }}:80;
        
            {% endif %}
        {% endfor %}
    }

    server {
        listen 80;
        location / {
            proxy_pass http://backend;
        }
    }
    include /etc/nginx/conf.d/*.conf;
}
```

Покажем в терминале:

<img width="1673" height="845" alt="image" src="https://github.com/user-attachments/assets/44cf00f5-6cef-4bd9-99d0-ec5fdfeff373" />
<img width="1249" height="191" alt="image" src="https://github.com/user-attachments/assets/d91b105e-23e6-40f9-a611-3d5291d15166" />


**Задание** `5.3`: Защита от дурака (`Validate & Handlers`) Если ты ошибся в синтаксисе шаблона, `Nginx` упадет при рестарте. Это недопустимо:

- В таске `template` используй параметр `validate`.
  - Команда валидации: `nginx -t -c %s` (где `%s` — временный файл, который создает `Ansible`).
  - Если валидация не проходит, `Ansible` должен прервать выполнение и не перезаписывать реальный файл конфига.
- Используй `notify: Reload Nginx` для применения настроек. Рестарт (`stop/start`) не нужен, только `reload`.

```bash
     - name: create nginx.conf from template
       template:
          src: nginx.conf.j2
          dest: /etc/nginx/nginx.conf
          owner: root
          group: root
          mode: '0644'
          validate: nginx -t -c %s
       notify: reload nginx

handlers:
     - name: reload the systemd daemon and restart the Nginx service
       systemd_service:
          daemon_reload: true
          enabled: true
          name: nginx
          state: restarted

     - name: reload nginx
       systemd_service:
          daemon_reload: true
          enabled: true
          name: nginx
          state: reloaded  
```

Покажем в терминале:

<img width="1001" height="571" alt="image" src="https://github.com/user-attachments/assets/ccb46cba-8065-421a-9468-817834633474" />

**Задание** `5.4`: Проверка боем (Оркестрация) Теперь самое сложное — собрать всё в один сценарий обновления. Твой `site.yml` должен работать так:

- Сначала собирает факты с `appservers` (чтобы знать их статусы).
- Потом настраивает `balancers`, используя данные из пункта выше.

Добавим в начала `site.yml` сборк факто на серверах:

```bash
- name: Gather facts from all servers
  hosts: all
  gather_facts: true
```

Покажем в терминале:

<img width="974" height="98" alt="image" src="https://github.com/user-attachments/assets/0f1e4539-6af6-4a1c-a845-966c3287fd6f" />

**Сценарий проверки** (`Action`):

1. Запусти плейбук. Убедись, что в `nginx.conf` на балансере прописан `IP` апп-сервера:

```bash
ansible-playbook site.yml
ansible balancers -m shell -a "cat /etc/nginx/nginx.conf" 
```

Покажем в терминале:

<img width="1839" height="875" alt="image" src="https://github.com/user-attachments/assets/66d26f2d-d482-4022-a137-86178ff207e2" />
<img width="1495" height="865" alt="image" src="https://github.com/user-attachments/assets/1b5a1202-6d5b-4bf8-b9ca-72d2c6c38324" />

2. Измени статус апп-сервера на `drain`:
  - Либо через `Ad-Hoc`: `ansible appservers -m copy -a "content='{\"status\":\"drain\"}' dest=/etc/ansible/facts.d/phoenix.fact"`
  - Либо написав отдельный мини-плейбук для переключения режима.

```bash
ansible appservers -m copy -a "content='{\"status\":\"drain\"}' dest=/etc/ansible/facts.d/phoenix.fact"
```

Покажем в терминале:

<img width="1836" height="491" alt="image" src="https://github.com/user-attachments/assets/98649211-3ef1-4670-99a5-519b39a6c51f" />

3. Снова запусти основной `site.yml`:

`ansible-playbook site.yml -e "target_state=drain"`

Покажем в терминале:

<img width="1598" height="887" alt="image" src="https://github.com/user-attachments/assets/2e2e1e22-2b5e-40e0-afd4-14b15e85f177" />

4. **Результат**: `Ansible` должен увидеть изменение факта на апп-сервере, перегенерировать конфиг на балансере (убрав оттуда `IP`) и сделать `Reload Nginx`:

`ansible balancers -m shell -a "cat /etc/nginx/nginx.conf"`

Покажем в терминале:

<img width="1572" height="798" alt="image" src="https://github.com/user-attachments/assets/cb87c95f-ca62-4b4c-87e8-e740280ff218" />

**Критерии приемки** (`Definition of Done`)

1. В шаблоне `nginx.conf.j2` используется цикл `{% for host in groups['appservers'] %}`.
2. Внутри цикла есть проверка `if hostvars[host]['ansible_local']['phoenix']['status'] == 'active'`.
3. Ты реализовал `Defensive Coding`: если `ansible_local` еще не создан на каком-то хосте, шаблон не падает с ошибкой `undefined variable`, а просто пропускает этот хост (используй `is defined` или `default`):

```bash
  upstream backend {
        {% for host in groups['appservers'] %}
            {% if hostvars[host]['ansible_local'] is defined 
                  and hostvars[host]['ansible_local']['phoenix'] is defined
                  and hostvars[host]['ansible_local']['phoenix']['status'] is defined
                  and hostvars[host]['ansible_local']['phoenix']['status'] == 'active' %}
        
        server {{ hostvars[host]['ansible_host'] }}:80;
        
            {% endif %}
        {% endfor %}
    }
```

Покажем в терминале:

<img width="1201" height="279" alt="image" src="https://github.com/user-attachments/assets/de43238e-147b-4b6e-971a-533ff06580fb" />
