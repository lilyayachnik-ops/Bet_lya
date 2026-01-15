# Checkpoints

**Задание** `8.1`: Масштабирование (`Scale Out`) `Rolling Update` на одном сервере не имеет смысла (у тебя все равно будет простой):

- **Действие**: Клонируй свою виртуальную машину `appserver` (или создай новую). Назови её `vm-app-2`. Покажем в терминале:

<img width="1533" height="74" alt="image" src="https://github.com/user-attachments/assets/0c6851a6-8c4d-4b14-96c8-c72004531bc7" />

- Добавь её `IP` в инвентарь в группу `appservers`:

```bash
    appservers:
      hosts:
        appserver:
          ansible_host: 34.30.111.203
        appserver1:
          ansible_host: 34.68.78.218
```

Покажем в терминале:

<img width="1005" height="194" alt="image" src="https://github.com/user-attachments/assets/96a31c75-255d-43a6-ad29-fbc65f978a47" />

- Прогони свой старый `site.yml`. Благодаря идемпотентности, первый сервер не изменится, а второй настроится с нуля (пользователи, `Nginx`, ключи):

<img width="1574" height="692" alt="image" src="https://github.com/user-attachments/assets/71d030c3-21fd-4efd-9265-7f956e945ba5" />
  
- Убедись, что балансировщик (`Task 5`) видит оба сервера и распределяет трафик (проверь через `curl` на балансер — должен меняться ответ, если ты менял `index.html`):

```bash
curl -k https://phoenix.local
```

Покажем в терминале:

<img width="777" height="176" alt="image" src="https://github.com/user-attachments/assets/8b4c9dce-420c-4d92-857c-86ec11d38d3d" />

**Задание** `8.2`: Подготовка "Ломающегося" Артефакта Нам нужно что-то деплоить:

- Создай локально две версии "приложения" (просто `HTML` файлы или `Python` скрипты):
 - `v1.0`: `Hello from Version 1`
 - `v2.0`: `Hello from Version 2`
 - `v_broken`: Содержит ошибку (если это `HTML` — допустим, просто текст `ERROR`, если скрипт — `exit 1`).
- Эти файлы будут нашей "полезной нагрузкой", которую мы копируем на сервера.

```bash
cat ./app/v1.0.html
cat ./app/v2.0.html
cat ./app/v_broken.html
```

Покажем в терминале:

<img width="744" height="136" alt="image" src="https://github.com/user-attachments/assets/0b677c37-47b6-461d-9810-b247f4f4c698" />

**Задание** `8.3`: Плейбук `Rolling Update` (`deploy.yml`) Создай отдельный плейбук `deploy.yml`. Это не конфигурация, это именно процедура обновления.

Структура плейбука (`Play`):
- `hosts: appservers`
- `serial: 1` (Строго по одному!)
- `any_errors_fatal: true` (Если упал один — остановить весь деплой, не трогать остальные).

```bash
---
- name: Rolling update for Phoenix Application
  hosts: appservers
  serial: 1
  any_errors_fatal: true

  vars_files:
     - "./roles/phoenix_nginx/defaults/main.yml"
     - "./group_vars/balancers.yml"
```

Покажем в терминале:

<img width="761" height="288" alt="image" src="https://github.com/user-attachments/assets/725494a4-e9b1-42b9-add9-22f03b922367" />

**Логика внутри** (`tasks`):
- `Pre-task` (**Вывод из балансировки**):

 - Используй модуль `file` или `copy` для изменения `custom fact` (`/etc/ansible/facts.d/phoenix.fact`) на текущем сервере. Установи статус `drain` или `maintenance`:

```bash
 tasks: 
    
    - name: Drain from load balancer 
      become: true
      block: 
         
         - name: Set status to drain in custom facts
           copy:
              content: { "status": "drain"}
              dest: /etc/ansible/facts.d/phoenix.fact
           
         - name: Reload custom facts
           setup:
              fact_path: /etc/ansible/facts.d
              filter: "ansible_local"
           changed_when: false
```

Покажем в терминале:

<img width="808" height="377" alt="image" src="https://github.com/user-attachments/assets/807477bd-56dc-4c8d-8fee-f746c31f495a" />

 - **Важно**: В `Task 5` ты делал шаблон `Nginx`. Теперь тебе нужно заставить `Nginx` на балансере перечитать конфиг:

```bash
        - name: Update nginx config on balancer
           template:
              src: "./roles/phoenix_nginx/templates/nginx.conf.j2"
              dest: "{{ nginx_config_path }}"
           delegate_to: loadbalancer
```

Покажем в терминале:

<img width="822" height="119" alt="image" src="https://github.com/user-attachments/assets/08e686e9-1be9-41f3-8a4a-c7cddf468f10" />

- **Сложность**: Ты меняешь факт на appserver, но `reload` нужно сделать на `balancer`. Используй `delegate_to` или механизм хендлеров. (Подсказка: проще всего запустить таск `Reload Nginx` с `delegate_to: <имя_балансера>`):

```bash
         - name: Reload nginx on balancer
           systemd_service:
              name: nginx
              state: reloaded
           delegate_to: loadbalancer
```

Покажем в терминале:

<img width="800" height="121" alt="image" src="https://github.com/user-attachments/assets/3a312cfa-7970-4418-98f3-e3f8aa4390c3" />

- Добавь `wait_for` (паузу `5-10 сек`), имитируя слив соединений:

```bash
         - name: Wait for connections to drain  
           wait_for: 
              timeout: 10
```

Покажем в терминале:

<img width="896" height="73" alt="image" src="https://github.com/user-attachments/assets/4a698bf7-95c2-47af-9eb9-d749733b1942" />

- `Update` (**Блок обновления с защитой**):
 - Используй конструкцию `block`:
  - **Таск**: Скопировать новую версию приложения (`v2.0`) на сервер:

```bash
   - name: Deploy application
      become: true
      block:
         
         - name: Copy application version
           copy:
              src: "./app/{{ deploy_version }}.html"
              dest: "/usr/share/nginx/html/current_version.html"
              owner: root
              group: root 
              mode: 0644
```

Покажем в терминале:

<img width="869" height="260" alt="image" src="https://github.com/user-attachments/assets/496489ad-e405-4bd3-aefc-c0d40929941d" />

  - **Таск**: Перезапустить сервис приложения (если есть)
  - `Smoke Test`: Используй модуль `uri` (или `command curl`), чтобы проверить, что приложение на `localhost` (на самом апп-сервере) отвечает кодом `200` и содержит нужный текст:

```bash
         - name: Smoke test 
           uri:
              url: "http://localhost:80/current_version.html"
              return_content: yes
           register: smoke_test_result
           failed_when: > 
             smoke_test_result.status != 200 or
             'Hello from' not in smoke_test_result.content
```

Покажем в терминале:

<img width="879" height="188" alt="image" src="https://github.com/user-attachments/assets/4e199e5c-460f-4eb0-9d56-b3891002cc5b" />

- Используй конструкцию `rescue`:
 - Этот блок сработает, ТОЛЬКО если `Smoke Test` или `Copy` упали.
 - **Таск**: Вернуть старую версию приложения (v1.0):

```bash
         - name: Rollback to v1.0
           copy:
               src: "./app/v1.0.html"
               dest: "/usr/share/nginx/html/current_version.html"
               owner: root
               group: root
               mode: 0644
```

Покажем в терминале:

<img width="857" height="217" alt="image" src="https://github.com/user-attachments/assets/1057423b-e154-46ed-8b3c-95cf1600e4f7" />

  - **Таск**: `Fail` (принудительно завершить работу с ошибкой, чтобы `Ansible` не пошел на следующий сервер):

```bash
         - name: Set to maintenance mode
           copy:
              content: {"status": "maintenance"}
              dest: /etc/ansible/facts.d/phoenix.fact
           when: deploy_version == 'v_broken'

         - name: Update nginx config after rollback
           template:
              src: "./roles/phoenix_nginx/templates/nginx.conf.j2"
              dest: "{{ nginx_config_path }}"
           delegate_to: loadbalancer
        
         - name: Reload nginx after rollback
           systemd_service:
              name: nginx
              state: reloaded
           delegate_to: loadbalancer

         - name: Stop deployment
           fail:
              msg: "Deployment failed on {{ inventory_hostname }}. App2 will not be touched."
```

Покажем в термиале:

<img width="1487" height="500" alt="image" src="https://github.com/user-attachments/assets/8d5d9647-c5d7-474c-bdac-2ed15db6787e" />

- **Post-task** (Возврат в балансировку):
 - Если `block` прошел успешно:
 - Меняем факт обратно на `active`.
 - Делаем `Reload Nginx` на балансере (`delegate_to`).

```bash
- name: Return to load balancer
      become: true
      block: 
         
         - name: Set status to active in custom facts
           copy:
              content: { "status": "active"}
              dest: /etc/ansible/facts.d/phoenix.fact
 
           
         - name: Reload custom facts
           setup:
              fact_path: /etc/ansible/facts.d
              filter: "ansible_local"
           changed_when: false

         - name: Update nginx confing on balancer
           template:
              src: "./roles/phoenix_nginx/templates/nginx.conf.j2"
              dest: "{{ nginx_config_path }}"
           delegate_to: loadbalancer
         
         - name: reload nginx
           systemd_service:
              name: nginx
              state: reloaded
           delegate_to: loadbalancer
```

Покажем в терминале:

<img width="989" height="639" alt="image" src="https://github.com/user-attachments/assets/5d2c7f37-c82a-4c67-918f-6eace7283953" />

**Задание** `8.4`: **Испытание огнем**

1. Запусти деплой версии `v2.0`.
 
 - Наблюдай: `Ansible` должен обновить `App1` -> Перезагрузить `Nginx` -> Обновить `App2` -> Перезагрузить `Nginx`.
 - Во время деплоя делай `curl` на балансер. Ты должен всегда получать ответ (`Zero Downtime`).

```bash
ansible-playbook deploy.yml -e "deploy_version=v2.0"
```

Покажем в терминале:

<img width="1549" height="642" alt="image" src="https://github.com/user-attachments/assets/285b6a74-4196-4cf5-b81d-23b7c64e45b6" />
<img width="1513" height="539" alt="image" src="https://github.com/user-attachments/assets/1975792b-665f-4086-8652-91af97282c34" />
<img width="1527" height="608" alt="image" src="https://github.com/user-attachments/assets/85657448-bfb0-49a1-b8e8-a1fea826e606" />
<img width="1518" height="542" alt="image" src="https://github.com/user-attachments/assets/67c6558c-02c0-47ba-952e-24c49be02522" />
<img width="1506" height="546" alt="image" src="https://github.com/user-attachments/assets/76edebd9-f801-49d7-9671-3375a71f8d15" />

2. `Rollback Test`:

 - Попробуй задеплоить `v_broken`.
 - `Ansible` должен:
  - Выключить `App1`.
  - Залить "битую" версию.
  - Провалить `Smoke Test` (`uri` вернет ошибку).
  - Зайти в `rescue`.
  - Восстановить версию `v1.0`.
  - Вернуть `App1` в балансировку (опционально, или оставить в maintenance для разбора полетов).
  - ОСТАНОВИТЬСЯ и не трогать `App2`.

```bash
ansible-playbook deploy.yml -e "deploy_version=v_broken"
while true; do   curl -k https://phoenix.local/current_version.html;   echo " - $(date +%H:%M:%S)";   sleep 1; done
```

Покажем в терминале:

<img width="1376" height="735" alt="image" src="https://github.com/user-attachments/assets/5b447946-d6b5-4fa0-a0b1-bb73701dbf8e" />

**Критерии приемки** (`Definition of Done`)

1. У тебя есть минимум `2` App-сервера.
2. В `deploy.yml` используется `serial: 1`.
3. Ты реализовал взаимодействие между узлами: изменение файла на App вызывает перезагрузку сервиса на `LB`.
4. Попытка деплоя сломанной версии приводит к ошибке в `Ansible`, но сервис `Phoenix` остается доступным для пользователей (так как `App2` не был затронут, а `App1` был откачен или изолирован).
5. Ты понимаешь разницу между `failed_when` (критерий сбоя) и `ignore_errors` (игнорирование сбоя). В `Smoke Test` мы не игнорируем ошибки, мы их ловим через `rescue`:

`failed_when` — даем своё определение ошибок. 

`ignore_errors` говорит "забей" → `playbook` продолжается 
