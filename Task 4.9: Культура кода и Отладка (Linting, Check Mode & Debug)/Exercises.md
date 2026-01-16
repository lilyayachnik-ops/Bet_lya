# Checkpoints

**Задание** `9.1`: **Чистка кода** (`Ansible Lint`):

- Установи `ansible-lint` (через `pip install ansible-lint` или пакетный менеджер).
- Запусти проверку на всем своем проекте: `ansible-lint site.yml`.
- Action: Ты увидишь кучу предупреждений (`warnings`) и ошибок.
 - Исправь их все!
 - Примеры частых ошибок:
  - Отсутствие name у таска.
  - Использование `command` вместо специализированного модуля (например `git` или `service`).
  - Использование `chmod 777`.
  - Отсутствие `changed_when` для `shell/command` (`Ansible` не знает, изменил ли скрипт что-то, и всегда пишет `changed`. Ты должен явно указать условие).
- Добейся пустого вывода линтера.

```bash
pip install ansible-lint
```

Покажем в термминале:

<img width="1453" height="645" alt="image" src="https://github.com/user-attachments/assets/0965bbdc-6503-462f-a816-54faf94e3155" />
<img width="1533" height="664" alt="image" src="https://github.com/user-attachments/assets/ec35bebb-fb40-42aa-a631-c60493dc510f" />

```bash
sudo ansible-lint site.yml
```

Покажем в терминале:

<img width="1495" height="18" alt="image" src="https://github.com/user-attachments/assets/dd5d410f-011a-4895-a74e-8d222a0d225b" />
<img width="1132" height="452" alt="image" src="https://github.com/user-attachments/assets/2e17b856-da1a-45fa-aa58-f31d37fb282f" />
<img width="1524" height="378" alt="image" src="https://github.com/user-attachments/assets/ba1b08e2-f559-44a0-ae0f-fedd975a45c8" />

**Задание** `9.2`: **Безопасный прогон** (`Check & Diff`) Внеси небольшое изменение в конфигурацию (например, измени порт `Nginx` в переменных или добавь нового пользователя):

- Запусти плейбук с флагами: `ansible-playbook site.yml --check --diff`.
- **Анализ**:
 - `Ansible` не должен реально менять файлы на сервере.
 - В выводе консоли ты должен увидеть "дифф" (разницу) между текущим файлом и тем, который был бы создан.
 - Если таск `command` или `shell` падает с ошибкой — добавь к нему `check_mode: false` (если он только читает данные) или `ignore_errors`, так как в `check`-режиме скрипт не выполняется, и регистрации переменной не происходит. Это важный урок: скрипты часто ломают `check-mode`.

Изменим версию `postgresql` и порт, на котором будет слушать балансировщик, на  `8081`. Покажем в терминале:

<img width="582" height="194" alt="image" src="https://github.com/user-attachments/assets/f48c1703-dc29-424b-b5c3-6af5de672460" />
<img width="546" height="170" alt="image" src="https://github.com/user-attachments/assets/83c20255-ec54-48fc-967d-2c0e1f74ed80" />

Выполним команду:

```bash
ansible-playbook site.yml --check --diff
```

Покажем в терминале:

<img width="1593" height="370" alt="image" src="https://github.com/user-attachments/assets/eb83cbf3-0e68-4d0a-80c1-3c36757ac0aa" />
<img width="1583" height="423" alt="image" src="https://github.com/user-attachments/assets/2a0d8409-2b1f-4a11-84d3-6d95b9a8e6c1" />
<img width="1573" height="618" alt="image" src="https://github.com/user-attachments/assets/a5b9fc01-06ae-432d-b0d1-918654c1d875" />

**Задание** `9.3`: **Интерактивный дебаг**. Спровоцируй ошибку. Например, укажи несуществующий путь к файлу в модуле `copy` или опечатку в имени пакета:

<img width="906" height="24" alt="image" src="https://github.com/user-attachments/assets/494e0464-b86c-45a8-baf7-1820702b624b" />

- Добавь в этот ломающийся таск (или на уровень плейбука) директиву: `debugger: on_failed`.

```bash
---
- name: Add nginx repository GPG key
  debugger: on_failed
  ansible.builtin.apt_key:
     url: "{{ phoenix_nginx_url }}"
     state: present
```

Покажем в терминале:

<img width="792" height="143" alt="image" src="https://github.com/user-attachments/assets/6c2ccd3b-f336-466d-829b-8cb80a72bcf7" />

- Запусти плейбук:

```bash
ansible-playbook site.yml
```

Покажем в терминале:

<img width="1590" height="378" alt="image" src="https://github.com/user-attachments/assets/664f4e7d-247d-4e06-90bb-60aae83d9d1b" />

- Когда он упадет, ты попадешь в интерактивную консоль `[ip] >`.
- **Действия в консоли**:
 - Введи `p task.args` (посмотреть аргументы таска):

```bash
p task.args
```

Покажем в терминале:

<img width="1521" height="553" alt="image" src="https://github.com/user-attachments/assets/1373af46-f6d8-447c-867a-70b357e14256" />

 - Введи `p task_vars['имя_переменной']` (проверить значение переменной в момент падения):

```bash
p task_vars['phoenix_nginx_url']
```

Покажем в терминале:

<img width="1354" height="50" alt="image" src="https://github.com/user-attachments/assets/8901d33b-f332-4de8-b991-f8b79094074c" />

 - Измени значение аргумента на лету: `task.args['src'] = 'правильный_путь'`:

```bash
task.args['url'] = 'https://nginx.org/keys/nginx_signing.key'
p task.args
```

Покажем в терминале:

<img width="1402" height="25" alt="image" src="https://github.com/user-attachments/assets/2be0b2ea-dfed-4403-bc18-a0cb091bfef1" />
<img width="1320" height="573" alt="image" src="https://github.com/user-attachments/assets/22628d89-479a-4a85-974e-a7b6556dc6ae" />


 - Перезапусти таск: `r (redo)`:

```bash
r
```

Покажем в терминале:

<img width="1575" height="314" alt="image" src="https://github.com/user-attachments/assets/bb38e33a-e004-41ad-9579-c78fe70e31e1" />

- Плейбук должен продолжиться. Это магия, которая спасает при долгих деплоях.

**Критерии приемки**
1. Проект проходит проверку `ansible-lint` без единого ворнинга.
2. В коде, где используются модули `shell/command`, обязательно прописан `changed_when` (или `failed_when`).
3. Ты продемонстрировал лог запуска с `--diff`, где видно конкретные строки изменений конфига.
4. Ты умеешь пользоваться дебаггером и спас "падающий" плейбук без перезапуска с нуля.
