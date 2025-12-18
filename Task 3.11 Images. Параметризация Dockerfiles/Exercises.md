# Checkpoints

Создадим файл `/home/user/Dockerfile`, который: 

- собирает из образа `nginx`, версия которого параметризуется через аргумент `ARG` с именем `NG_VERSION`;
- устанавливает переменную окружения `ENV` с таким же именем `NG_VERSION` значение которой берется из `ARG`;
- создает файл `/opt/$ARG_FILE`, где значение переменной передается через `ARG`, но не сохраняется в `ENV`.

Содержимое `Dockerfile`:

```bash
ARG NG_VERSION=latest
FROM nginx:${NG_VERSION}
ARG NG_VERSION
ENV NG_VERSION=${NG_VERSION}
ARG ARG_FILE
RUN touch /opt/${ARG_FILE}
```

Покажем в терминале:

<img width="825" height="157" alt="image" src="https://github.com/user-attachments/assets/93a10e28-55a1-47a5-b94c-d0e23dd2e133" />

Соберём образ из данного Dockerfile, передав необходимые аргументы и указав образу имя nginx:inno-dkr-10:

```bash
# --build-arg NG_VERSION=stable — передаёт аргумент сбоорки NG_VERSION со значением stable
# --build-arg ARG_FILE=test.txt — передаёт аргумент сборки ARG_FILE со значением test.txt

$ docker build --build-arg NG_VERSION=stable --build-arg ARG_FILE=test.txt -t nginx:inno-dkr-10 .
```

Покажем в терминале:

<img width="1366" height="404" alt="image" src="https://github.com/user-attachments/assets/8720c3c0-523f-43d7-ae4a-7cbf12809d6a" />

Выведем список образов на нашем хосте. Запустим контейнер из этого образа с именем `inno-dkr-10` в фоновом режиме и с передачей переменной окружения `INNO=DKR10`:

```bash
docker image ls

# --env INNO=DKR10 — устанавливает переменную окружения INNO, а DKR=10 — значение переменной INNO

$ docker run -d --name inno-dkr-10 --env INNO=DKR10 nginx:inno-dkr-10
```

Покажем в терминале:

<img width="1362" height="334" alt="image" src="https://github.com/user-attachments/assets/62fdddd8-f932-4a56-bbc3-3d779e15a012" />

Выведим список запущенных контейнеров. Выведим в контейнере список переменных окружения — должна присутствовать установленная нами переменная окружения. Выведим в контейнере список файлов в директории `/opt/`:

```bash
docker ps

# docker exec — команда для выполнения команды внутри запущенного контейнера
# inno-dkr-10 — имя контейнера
# bash — запускает оболочку Bash
# -c — флаг, указывающий, что следующая строка содержит команду для выполнения
# /usr/bin/env — исполняемый файл, который выводит все переменные окружения

$ docker exec inno-dkr-10 bash -c /usr/bin/env
```

Покажем в терминале:

<img width="1384" height="423" alt="image" src="https://github.com/user-attachments/assets/5e8221ce-20f4-464c-9200-c072cfbe630c" />

Выведем в контейнере список файлов в директории `/opt/`:

```bash
docker exec inno-dkr-10 ls -la /opt
```

Покажем в терминале:

<img width="1001" height="115" alt="image" src="https://github.com/user-attachments/assets/53eddd08-2987-45ae-9b03-6f44c8f138ae" />


