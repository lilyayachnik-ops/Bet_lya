# Chekpoints

Необходимо написать `Systemd unit`, который будет раз в 15 секунд писать в файл вывод команды `uptime`. При убийстве процесса он должен перезапускаться.
Для начала напишем скрипт `uptime_script.sh`, где на вход передаем файл, куда будет складываться результат вывода команды:

```bash
#!/bin/bash

#Задаем пути к файлам логов

UPTIME_LOG="/home/ilya/uptime.log"
touch "$UPTIME_LOG"

while true; do
        uptime >> $UPTIME_LOG
        sleep 15
done
```

Не забудем выполнить команду `sudo chmod 755 ~/uptime_script.sh`.

Дальше займемся нашим `Unit` файлам `uptime_logging.service`, который находиться в `/etc/systemd/system/`:

```bash
[Unit]
Description=Uptime logging service
Documentation=man:uptime(1)
After=multi-user.target

[Service]
Type=simple
User=ilya
Group=ilya
WorkingDirectory=/home/ilya
ExecStart=/home/ilya/uptime_script.sh
Restart=always
RestartSec=5
KillMode=control-group

[Install]
WantedBy=multi-user.target
```

**Systemd** — универсальный менеджер. Он управляет практически всей работающей системой. Отвечает за работу запущенных служб и может рассказать об их состоянии. Также управляет монтированием файловой системы, аппаратной частью, процессами.
Использует скомпилированные бинарные файлы. Они открыты. Их можно менять через командную строку. Отметим парочку функций:

-`Systemd` как `PID 1` осуществляет запуск стольких служб в параллельном режиме, сколько ему нужно. Благодаря этому загрузка ускоряется.

-`Systemd` создает журналы для хранения системных логов и дает инструменты для управления записями.

-`Systemctl` дает пользовательский интерфейс для управления службами.
 
- Обеспечивается обратная совместимость благодаря поддержке SystemV и LSB.

- Управление службами и логи дают информацию о состоянии служб.

- Можно управлять сокетами.

- Таймеры предоставляют расширенные возможности для планирования, включая запуск скриптов по времени от старта системы.

- Можно монтировать и размонтировать файловые системы с иерархическим уведомлением для безопасного каскадирования.

- Можно создавать временные файлы и управлять ими, в том числе удалять.

- Можно запускать скрипты при подключении или отключении устройств.

- Можно через анализ последовательности загрузки найти службы, запуск которых отнимает больше всего ресурсов или вызывает сбои.

**Важно**: все сервисы, которые можно запускать или останавливать, описываются в специальных юнитах `service`. В них описывается вся информация, которая поможет запустить сервис(службу). 
То есть `Unit` — это описание сервиса в текстовом виде. В нем указаны операции, которые выполняются до и после запуска службы. По сути, это описание параметров инициализации.

**Все `Unit` разложены по трем каталогам**:

- `/usr/lib/systemd/system/` — юниты из установленных пакетов.

- `/run/systemd/system/` — юниты, созданные в рантайме.

- `/etc/systemd/system/` — юниты, созданные системными администраторами.

```bash
# Посмотреть список всех запущенных юнитов можно командой
systemctl
```

В терминале отобразится также статус каждой службы. Основные параметры:

- `UNIT` — название юнита.
  
- `LOAD` — информация об успешной загрузке конфигурации.

- `ACTIVE` — сообщение о статусе. Может быть также `INACTIVE`, `LOADED`, `LISTENING`, `WAITING`, `EXITED`, `RUNNING`, `DEAD`, `MOUNTED`, `NOT-FOUNG` и `PLUGGED`.

- `SUB` — детальная системная информация об юните.

- `DESCRIPTION` — краткое описание юнита.

`Unit` бывают разных типов.

<img width="809" height="750" alt="image" src="https://github.com/user-attachments/assets/78517f50-27e2-49ba-b033-16e793d22d0c" />
<img width="785" height="191" alt="image" src="https://github.com/user-attachments/assets/1b4164b4-b1f5-4c16-a02c-d2ea19aca8ce" />

**СТРУКТУРА `UNIT` ФАЙЛОВ**: представляет собой текстовый файл. Внутри него — обязательные секции и переменные.

- секция `[UNIT]`: описываются метаданные службы и правила взаимодействия с другими службами. В ней доступны следующие переменные:
<img width="800" height="293" alt="image" src="https://github.com/user-attachments/assets/14fcc677-75a0-4b4b-957c-db5fdc9ca04b" />

- секция `[SERVICE]`: указывается, какими командами и под каким пользователем запускать сервис.
  <img width="796" height="908" alt="image" src="https://github.com/user-attachments/assets/899886da-9c49-44a7-ab27-ebcdd78e7560" />
  <img width="815" height="253" alt="image" src="https://github.com/user-attachments/assets/2690c1cb-a714-4218-ad97-17ea8a325d48" />

- секция `[INSTALL]`: в ней описывается на каком уровне запуска стартует настраиваемый сервис.
  <img width="793" height="122" alt="image" src="https://github.com/user-attachments/assets/8c786c14-787a-4866-b9cb-e06be8cdf991" />

Теперь перейдем к написанию нашего юнит файла `uptime_logging.service`:

```bash
[Unit]
# Описание службы
Description=Uptime logging service
# Ссылка на man-страницу команды uptime
Documentation=man:uptime(1)
# Служба запустится после достижения multi-user.target
After=multi-user.target

[Service]
# Тип службы
Type=simple
# Служба запускается от имени пользователя и группы ilya
User=ilya
Group=ilya
# Рабочая директория, скрипт будет выполняться в рабочей директории пользователя
WorkingDirectory=/home/ilya
# Исполняемый скрипт
ExecStart=/home/ilya/uptime_script.sh
# Cлужба перезапускается при любом завершении 
Restart=always
# Нужно подождать 5 секунд перед повторным запуском после падения
RestartSec=5
# При остановке службы убиваются основной процесс и дочерние
KillMode=control-group

[Install]
# Служба будет автоматически запускаться при загрузке системы до уровня 3.
WantedBy=multi-user.target
```

Запустим наш сервис с помощью команды `sudo systemctl start uptime_logging.service` и проверим его статус `sudo systemctl status uptime_logging.service`:

<img width="835" height="564" alt="image" src="https://github.com/user-attachments/assets/a577675e-a507-461f-9fc0-a49909b63206" />

Давайте усложним задачу. Если значение **Load Average** за минуту больше 1, то вывод команды `uptime` будет записываться в файл `overload`:

```bash
#!/bin/bash

#Задаем пути к файлам логов

UPTIME_LOG="/home/ilya/uptime.log"
OVERLOAD_LOG="/home/ilya/overload.log"
#Создаем файлы
touch "$UPTIME_LOG" "$OVERLOAD_LOG"

while true
do
        current_uptime=$(uptime)
        echo "$current_uptime" >> "$UPTIME_LOG"

        load_one_min=$(echo "$current_uptime" | awk -F'load average:' '{print $2}' | awk -F, '{print $1}' | sed 's/ //g')
        if [ $(echo "$load_one_min > 1.0" | bc -l) -eq 1 ]
        then
                echo "HIGH LOAD: $current_uptime" >> "$OVERLOAD_LOG"
        fi
        sleep 15
done
```

Перезапустим службу, нагрузим нашу систему, чтобы проверить работу:

```bash
sudo systemctl restart uptime_logging.service
sudo apt update
sudo apt install stress
stress --cpu 4 --timeout 50s, x — количество процессоров у нашей системы
```

Проверим в терминале:

<img width="839" height="680" alt="image" src="https://github.com/user-attachments/assets/2820abbf-0cd9-4cb9-8413-76c813673564" />
<img width="831" height="143" alt="image" src="https://github.com/user-attachments/assets/c1a9f224-6d89-4707-af5d-0227c2d6dce7" />

Давайте сделаем следующее, если размер файла `overload` достигает размера в 5Кб, то он должен очищаться. В файл `cleanup` должны складываться логи об успешных очистках с временем самой очистки:

```bash
#!/bin/bash

#Задаем пути к файлам логов

UPTIME_LOG="/home/ilya/uptime.log"
OVERLOAD_LOG="/home/ilya/overload.log"
CLEANUP_LOG="/home/ilya/cleanup.log"

#Создаем файлы
touch "$UPTIME_LOG" "$OVERLOAD_LOG" #"$CLEANUP_LOG"

while true
do
        current_uptime=$(uptime)
        echo "$current_uptime" >> "$UPTIME_LOG"

        load_one_min=$(echo "$current_uptime" | awk -F'load average:' '{print $2}' | awk -F, '{print $1}' | sed 's/ //g')
        if [ $(echo "$load_one_min > 1.0" | bc -l) -eq 1 ]
        then
                echo "HIGH LOAD: $current_uptime" >> "$OVERLOAD_LOG"
                file_size=$(ls -l "$OVERLOAD_LOG" | awk '{print $5}')
                if [ "$file_size" -gt 5120 ]
                then
                        echo "$(date) OVERLOAD LOG CLEAREAD" >> "$CLEANUP_LOG"
                        > "$OVERLOAD_LOG"
                fi

        fi
        sleep 15
done
```

Проверим в терминале:

<img width="838" height="579" alt="image" src="https://github.com/user-attachments/assets/f2d8b650-aa34-48b8-84f6-615b4738c292" />

Напишем `cron job`, проверяющий каждые 10 минут статус `uptime_logging.service`. Для этого напишем простой скрипт `uptime_status.sh`:

```bash
#!/bin/bash

SERVICE_NAME="uptime_logging.service"
LOG_FILE="/home/ilya/service_status.log"

touch "$LOG_FILE"

status=$(sudo systemctl is-active "$SERVICE_NAME")

if [ "$status" = "active"  ]
then
        echo "$(data '+%Y-%m-%d %H:%M:%S') - Service $SERVICE_NAME is ACTIVE" >> "$LOG_FILE"

else
        echo "$(data '+%Y-%m-%d %H:%M:%S') - Service $SERVICE_NAME is UNACTIVE" >> "$LOG_FILE"
fi
```

Дальше напишем команду `crontab -e`:

```bash
*/10 * * * * /home/ilya/uptime_status.sh
```

Проверим в терминале:

<img width="1081" height="401" alt="image" src="https://github.com/user-attachments/assets/ae5b9356-8c26-4d3e-826d-327e00526ad6" />

**Важно**: `Cron` —  это сервис, как и большинство других сервисов Linux, он запускается при старте системы и работает в фоновом режиме. Его основная задача выполнять нужные процессы в нужное время. Существует несколько конфигурационных файлов, из которых он берет информацию о том что и когда нужно выполнять. Сервис открывает файл /etc/crontab, в котором указаны все нужные данные.

```bash
минута час день месяц день_недели /путь/к/исполняемому/файлу
# посмотреть задачи cron для суперпользователя
crontab -l
# удалить все существующие задачи
crontab -r
# раз в 10 минут
*/10 * * * * /usr/local/bin/serve
Кроме того, для некоторых часто используемых наборов были придуманы переменные, вот они:

@reboot - при загрузке, только один раз;
@yearly, @annually - раз год;
@monthly - раз в месяц;
@weekly - раз в неделю;
@daily, @midnight - каждый день;
@hourly - каждый час.

#Откладка
grep CRON /var/log/cron or /var/log/syslog
```

Давайте запустим утилиту ping, выведем его в background, проверим, что процесс находится в фоновом режиме. Затем вернём процесс в foregroung. Остановим процесс, после чего удалим его:

```bash
ping google.com
# Перевести в процесс в background (&)
ping google.com & 
# Проверим фоновые задачи
jobs -l
# Вовзвращаем в foreground
fg %1
```

Проверим в терминале:

