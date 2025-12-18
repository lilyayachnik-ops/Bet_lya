# Checkpoints

Напиши `Dockerfile` следующей конфигурации: 
- собирается из образа `ubuntu:18.04`,
- в нем устанавливается пакет `nginx`,
- в него копируется конфигурационный файл `nginx`,
- в `ENTRYPOINT` используется `запуск nginx`,
- в `CMD` должны быть определены такие же параметры запуска nginx, как в образе `nginx:stable` (обычно строка запуска `nginx` выглядит так `nginx -g daemon off` )
- основная рабочая директория внутри контейнера — `/etc/nginx/`,
- должен быть определен `Volume` с путем `/var/lib/nginx`.

Содержимое `Dockerfile`:

```bash
  FROM ubuntu:18.04
RUN apt-get update && \
    apt-get install -y nginx
COPY nginx.conf /etc/nginx/nginx.conf
WORKDIR /etc/nginx
VOLUME [ "/var/lib/nginx" ]
ENTRYPOINT [ "nginx" ]
CMD ["-g", "daemon off;"]
```

Собери этот образ с именем `nginx:inno-dkr-09`. Выведи список образов на вашем хосте. Запусти контейнер со следующими параметрами:
образ — собранный нами образ, должно работать в фоне, слушает на хосте `127.0.0.1:8901`. Выведи список запущенных контейнеров - контейнер должен быть запущен.
Проверь работу, обратившись к `127.0.0.1:8901`, — в ответ должно возвращать строку: `Welcome to the training program Innowise: Docker!`:

```bash
docker build -t nginx:inno-dkr-09 .
docker image ls
docker run -d -p 127.0.0.1:8901:80 --name inno-dkr-09 nginx:inno-dkr-09
docker ps
curl http://127.0.0.1:8901
```

Покажем в терминале:

<img width="1327" height="560" alt="image" src="https://github.com/user-attachments/assets/c14d57c7-a521-4a2d-88e2-e0e4f028e40f" />
<img width="1392" height="201" alt="image" src="https://github.com/user-attachments/assets/a08c4cc8-9c50-48db-87c7-7799803aeae6" />



