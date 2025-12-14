# Checkpoints

Установи `Docker` на свою систему, как указано в официальной документации. Запусти запись вывода терминала в файл. Запусти контейнер на порту `28080` из официального образа `nginx`: `docker run -d -p 127.0.0.1:28080:80 --name rbm-dkr-01 nginx:stable`. Выведи в консоли список запущенных контейнеров командой: `docker ps`. При помощи утилиты `curl` запроси адрес `http://127.0.0.1:28080` - должна появиться приветственная страница `nginx`. Если не появляется, то запуск контейнера неуспешен по какой-то причине.

Покажем, что у нас работает `Docker`, используя команду `docker version`:

<img width="1213" height="627" alt="image" src="https://github.com/user-attachments/assets/1f5fa505-9f0c-4fad-98e7-0b3062b120da" />


```bash
# утилита script применяется для записи сеанса терминала. Её синтаксис следующей: script [options] [file], если [file] не указан, то по умолчанию используется файл typescript.

Options:
-a — позволяет пользователю дополнять файл журнала новыми командами в конец журнала. Без этого параметра файл будет перезаписан
-c — записывает в файл всего одну команду. Запись автоматически прекратиться после того, как мы подтвердим введение этой команды. Пример: script -c 'hostname'  log-file.txt
-q — запускает утилиту в фоновом режиме

script --timing=<time_file> <script_file>, где --timing записывает временные метки в указанный файл <time_file>, <script_file> — файл, куда записывается сама текстовая сессия.

ВАЖНО: эти два документа будут автоматически синхронизированы друг с другом и в дальнейшем могут пригодиться нам для воспроизведения введенных утилит.

Команда scriptreplay — это инструмент в Linux, который позволяет воспроизвести ранее записанный сеанс с помощью команды script. Она считывает временную информацию и вывод из записанного файла сценария и воспроизводит его в терминале. Важно, что данная утилита воспроизводит только те команды, которые были записаны вместе с их хронометражем:

scriptreplay --timing=<time_file> <script_file> — воспроизведет записи терминала

$ script ~/docker_3.1/my_terminal_session.txt
# docker run — создает и запускает новый контейнер,

-d — контейнер запускается в фоновом режиме, управление терминала возвращается сразу после запуска,

-p 127.0.0.1:28080:80 — проброс портов, где 127.0.0.1 указывает, что порт будет будет доступен только с локальной машины (не снаружи), 28080 — порт на хосте, 80 — порт внутри контейнера,

--name rbm-dkr-01 — присваивает контейнеру имя rbm-dkr-01 вместо сгенерированного

$ nginx:stable — образ nginx с тегом stable

docker run -d -p 127.0.0.1:28080:80 --name rbm-dkr-01 nginx:stable

# Показывает список работающих контейнеров Docker

$ docker ps

# curl отправляе GET-запрос на адрес 127.0.0.1, порт 28080 — это порт, который проброщен в контейнер, запрос перенаправляется в контейнер rbm-dkr-01 на порт 80, nginx внутри контейнера обрабатывает запрос и возвращает ответ

$ curl http://127.0.0.1:28080
```

Покажем в терминале:

<img width="1485" height="858" alt="image" src="https://github.com/user-attachments/assets/9a722ad2-936b-426e-827c-a496fa598fe7" />
<img width="1583" height="115" alt="image" src="https://github.com/user-attachments/assets/82ac5ae0-179c-4fa2-b3c2-9ce1dade1310" />

Останови ранее запущенный контейнер командой: `docker stop rbm-dkr-01`. Выполни команду `docker ps`, тем самым проверь произошедшие изменения в списке запущенных контейнеров — список контейнеров должен быть пуст. При помощи утилиты `curl` запроси адрес `http://127.0.0.1:28080` — сейчас должна быть ошибка, поскольку мы уже удалили контейнер и ничего не слушается на этом порту:

```bash
# docker stop останавливает контейнер с именем rbm-drk-01
docker stop rbm-dkr-01
docker ps
curl http://127.0.0.1:28080
```

Покажем в терминале:

<img width="1581" height="139" alt="image" src="https://github.com/user-attachments/assets/8a1eb99c-c873-41f2-b6c6-74620c57d237" />

При помощи команды `docker ps -a` выведи список всех контейнеров в системе — в списке будет твой остановленный контейнер. Останови запись в файл и загрузите полученные логи в репозиторий на `gitlab`:

```bash
# docker ps -a показывает все контейнеры, включая запущенные, остановленные, завершившиеся
docker ps -a
# останавливает выполнение сессии терминала в файл
exit
git init
git remote add docker3.1 https://gitlab.com/lilyayachnik/docker3.1.git
git config --global user.name "ilya"
git config --global user.email "lilyayachnik@gmail.com"
git add my_terminal_session.txt
git commit -m "Added docker session for task3.1"
git push -u docker3.1 master
```

Покажем в терминале:

<img width="1457" height="202" alt="image" src="https://github.com/user-attachments/assets/724df7f5-523a-42e4-95f9-8fe01904ff49" />
<img width="1128" height="271" alt="image" src="https://github.com/user-attachments/assets/3c585ead-051f-4d97-adf9-d45b8ad01565" />
<img width="1109" height="32" alt="image" src="https://github.com/user-attachments/assets/a0c9eac7-501a-4bf2-a69d-dbad15deeec7" />
<img width="1078" height="45" alt="image" src="https://github.com/user-attachments/assets/e70daf81-d6fc-4a91-82e2-46f7b9987128" />
<img width="751" height="30" alt="image" src="https://github.com/user-attachments/assets/db9e15cf-f7f5-475e-bdd9-103edbe0fc19" />
<img width="918" height="97" alt="image" src="https://github.com/user-attachments/assets/b2475c09-9973-4cfb-a291-10e7525bc1ea" />
<img width="691" height="223" alt="image" src="https://github.com/user-attachments/assets/5390d9b5-4cbf-4259-9830-20c6be7582fb" />







