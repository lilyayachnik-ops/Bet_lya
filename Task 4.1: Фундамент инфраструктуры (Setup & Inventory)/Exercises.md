# Checkpoints

**Цель**: 
Создать рабочую среду управления (`Control Node`) и подготовить `3` "боевых" сервера к управлению, объединив их в логическую структуру.

**Дано** (Наша локальная инфраструктура): 
Нам нужно поднять `3` виртуальные машины (используем `Google Cloud Console`). 
`OS`: `Ubuntu Server 20.04/22.04` (или `Rocky Linux`, на твой выбор, но придерживайся одного семейства).
**Сеть**: Статические `IP` или фиксированные `DHCP` (ты должен знать их `IP`).

Покажем в терминале:

<img width="1530" height="188" alt="image" src="https://github.com/user-attachments/assets/8201b87e-16e9-4594-a764-1996b921fe37" />

`Internal IP of managed-node-1`: `10.128.0.8`,
`Internal IP of managed-node-2`: `10.128.0.9`,
`Internal IP of managed-node-3`: `10.128.0.10`.

Задача `1.1`: Подготовка доступа

На своей рабочей машине (`Control Node`) сгенерируем `SSH`-ключ (если еще нет):

```bash
ssh-keygen -t rsa -b 4096 -f ~/.ssh/ansible_key
ls -la
```

Покажем в терминале:

<img width="1374" height="663" alt="image" src="https://github.com/user-attachments/assets/c1d4a7a4-6713-4878-b512-2b65ddf46e0b" />

Разольем этот ключ на все `3` ВМ. Для этого в идем в `Google Cloud Console`, `Compute Engine`, `Metadata`, `SSH Keys`. Нажмем `Add SSH key`. Покажем в терминале:

<img width="1285" height="267" alt="image" src="https://github.com/user-attachments/assets/79610894-c4fb-4689-93ea-9b52f6547d3a" />

Усложнение: Создадим на каждой `ВМ` пользователя `deployer` с правами `sudo` без пароля. Настроим `Ansible` так, чтобы он ходил под пользователем `deployer`, а не `root` или твоим локальным юзером:

1) Зайдем по `SSH` на первую `VM`, названная как `managed-node-1`:

```bash
ssh -i ~/.ssh/ansible_key deployer@34.71.138.167 
```

Покажем в терминале:

<img width="1586" height="711" alt="image" src="https://github.com/user-attachments/assets/bf183c68-8f8c-470f-9244-b43d5e0a5121" />

Далее выполним команду:

```bash
sudo vim /etc/sudoers
```

Покажем в терминале:

<img width="1460" height="27" alt="image" src="https://github.com/user-attachments/assets/a8acaec6-5f9a-421f-a699-d1e268c4f42d" />

Далее в файле `/etc/sudoers` напишем следующее и нажмем `Ctrl->Shift+:`, чтобы перейти в командный режим, `wq!`, чтобы сохранить изменения и выйти:

```bash
deployers ALL=(ALL) NOPASSWD: ALL
```

Покажем в терминале

<img width="663" height="131" alt="image" src="https://github.com/user-attachments/assets/ae41bdf9-3234-48fd-bef1-8d9b04f9df8d" />

Выполним те же шаги на двух других `VM`, названные `managed-node-2` и `managed-node-3`.

Установим `Ansible` на `Control Node`:

1) `sudo apt update && sudo apt install -y pipx`:

<img width="637" height="305" alt="image" src="https://github.com/user-attachments/assets/92e3b90c-abcc-4fc0-b599-d1be7f7a9841" />

2) `pipx ensurepath`:

<img width="633" height="159" alt="image" src="https://github.com/user-attachments/assets/aa78f0de-15a2-47a3-81a2-41f4c596a415" />

3) Перезагрузим оболочку `source ~/.bashrc`:

<img width="636" height="31" alt="image" src="https://github.com/user-attachments/assets/d661fe2a-ddae-437f-b857-0fa1159807d7" />

4) `pipx --version`:

<img width="628" height="46" alt="image" src="https://github.com/user-attachments/assets/0880bda1-727b-41dc-ac1a-f490f3c39f06" />

5) `pipx install ansible-core`:

<img width="638" height="351" alt="image" src="https://github.com/user-attachments/assets/9d1e0b76-5f8c-4964-a964-927fed3cb4b8" />

6) `ansible --version`:

<img width="643" height="309" alt="image" src="https://github.com/user-attachments/assets/5fdbfe17-f7b8-4255-9503-cd4d6dac2b14" />

Задача `1.2`: Конфигурация проекта 

Создадим папку `ansible-project`. В ней создадим файл `ansible.cfg`. Добьёмся того, чтобы при запуске из этой папки:

- Инвентарь брался по умолчанию из файла `./hosts` (чтобы не писать `-i`).
- Отключим проверку `host_key_checking` (чтобы не писать `yes` при первом подключении к новому серверу — в закрытых контурах это стандарт).
- Лог выполнения команд писался в файл `ansible.log` в этой же папке (полезно для дебага).

```bash
mkdir ansible-project
cd ./ansible-project/
touch ansible.cfg
```

Покажем в терминале:

<img width="642" height="73" alt="image" src="https://github.com/user-attachments/assets/3717cd49-59cd-405b-9401-1214f5fadd01" />

Содержимое `ansible.cfg`:

```bash
[defaults] # Секция в ansible.cfg — это основной раздел, где задаются глобальные настройки поведения Ansible.

inventory=./hosts.yml
host_key_checking=false
log_path=./ansible.log
action_warnings=false
```

Покажем в терминале:

<img width="625" height="150" alt="image" src="https://github.com/user-attachments/assets/8a0dfa27-49aa-4b45-9a38-ee17a02f27cf" />

**ТЕОРИЯ!!!**

<img width="516" height="322" alt="image" src="https://github.com/user-attachments/assets/2ad6ec38-32fc-4366-b15a-cfbbe3c83dc2" />

<img width="1466" height="318" alt="image" src="https://github.com/user-attachments/assets/4c998f93-c60b-4805-8cdb-8bf20f9d1b1b" />

<img width="1007" height="281" alt="image" src="https://github.com/user-attachments/assets/c423b7a6-8b33-4905-9d79-ca291db291df" />

<img width="538" height="422" alt="image" src="https://github.com/user-attachments/assets/fd473c08-4eb2-4466-9c2e-e98f55edfade" />

Задача `1.3`: Структурированный инвентарь 

Создадим файл `hosts` (формат `INI` или `YAML`). Распределим свои `3` ВМ по ролям (придумаем, какая ВМ чем будет, это важно для будущих тасков):
- `Load Balancer`: Группа `[balancers]` (`1` ВМ).
- `App Server`: Группа `[appservers]` (`1` ВМ).
- `Database`: Группа `[dbservers]` (`1` ВМ)
- Создадим общую группу `[production:children]`, которая включает в себя все три группы выше.

Содержимое `hosts.yml`:

```bash
all:
  children:
    balancers:
      hosts:
        loadbalancer:
          ansible_host: 34.71.138.167
    
    appservers:
      hosts:
        appserver:
          ansible_host: 34.67.95.48
    
    dbservers:
      hosts:
        dbserver:
          ansible_host: 34.60.24.1
    
    production:
      children:
        balancers:
        appservers:
        dbservers:
  
  vars:
    ansible_user: deployer
    ansible_ssh_private_key_file: "~/.ssh/ansible_key"
```

Покажем в терминале:

<img width="1256" height="616" alt="image" src="https://github.com/user-attachments/assets/0175a858-2277-40c1-9a19-d0a015169d32" />

**ТЕОРИЯ!!!**

<img width="469" height="246" alt="image" src="https://github.com/user-attachments/assets/ab6ddafa-e5a0-4069-bc6d-b1d55011bcf6" />

<img width="489" height="95" alt="image" src="https://github.com/user-attachments/assets/68916fe9-df26-4640-816f-f994907fb37f" />

<img width="482" height="127" alt="image" src="https://github.com/user-attachments/assets/3ce6313c-5107-4042-84b6-26ecb097c957" />

**Критерий успеха** (`Definition of Done`): 

Мы запускаем из папки проекта одну команду: `ansible production -m ping`. В ответ мы должны получить три зеленых `pong` от всех серверов. Покажем в терминале:

<img width="1607" height="752" alt="image" src="https://github.com/user-attachments/assets/8e0ae04c-a083-4309-b04d-4fa4ffcc1696" />

В файле `ansible.log` должны появиться записи об этом запуске. Выполним `ls -la; cat ./ansible.log`. Покажем в терминале:

<img width="1307" height="377" alt="image" src="https://github.com/user-attachments/assets/0011ca25-b5f4-4578-8222-9b44b628792e" />






