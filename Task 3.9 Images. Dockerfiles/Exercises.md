# Checkpoints

Скачаем конфигурационный файл `nginx`. Опишим Dockerfile, в котором: как базовый образ используется `nginx:stable`, внутрь контейнера как основной конфигурационный файл копируется скачанный тобой `nginx.conf`. Соберём этот образ с именем `nginx` и тегом `inno-dkr-08`. Выведим список образов на нашем хосте:

```bash
cp ~/docker_3.3/nginx.conf ~/docker_3.8/nginx.conf
cat nginx.conf
```

Покажем в терминале:

<img width="1568" height="26" alt="image" src="https://github.com/user-attachments/assets/f463ff21-c2ac-45dc-87c3-5694b78563e9" />
<img width="1595" height="353" alt="image" src="https://github.com/user-attachments/assets/94826d78-e507-4fab-bfd5-efb7bcf98c0c" />

Содержание `Dockerfile`:

```bash
FROM nginx:stable
COPY nginx.conf /etc/nginx/nginx.conf
```

Покажем в терминале:

<img width="1031" height="66" alt="image" src="https://github.com/user-attachments/assets/65d00993-1d23-4a1b-bc32-2597d0d63d12" />

Соберём образ, используя команду `docker build -t nginx:inno-dkr-08 .`. Покажем в терминале:

<img width="1178" height="451" alt="image" src="https://github.com/user-attachments/assets/0dc2f602-4f2d-4732-a28b-45d206620bf2" />

Выведем список образов на хосте, используя команду `docker image ls`. Покажем в терминале:

<img width="1088" height="246" alt="image" src="https://github.com/user-attachments/assets/a1107562-13af-42a7-be93-8ebf9d0de1c2" />

Запустим контейнер со следующими параметрами: образ — собранный нами образ, должен работать в фоне, слушает на хосте `127.0.0.1:8900`. Выведим список запущенных контейнеров — контейнер должен быть запущен. Проверим работу, обратившись к `127.0.0.1:8900`, — в ответ должно возвращать строку: `Welcome to the training program Innowise: Docker!`:

```bash
docker run -d -p 127.0.0.1:8900:80 --name inno-dkr-08 nginx:inno-dkr-08
docker ps
curl http://127.0.0.1:8900
```

Покажем в терминале:

<img width="1213" height="159" alt="image" src="https://github.com/user-attachments/assets/c80236b6-15bf-4e8b-a3bc-dd01e377b327" />
