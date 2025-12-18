# Checkpoints

Скачаем конфигурационный файл `nginx`. Создадим `volume` с именем `inno-dkr-04-volume`. Запустим контейнер со следующими параметрами: должен работать в фоне,
слушает на хосте `127.0.0.1:8891`, имеет имя `inno-dkr-04`, должен пробрасывать скачанный нами конфигурационный файл внутрь контейнера как основной конфигурационный файл,
образ — `nginx:stable`, в директорию с логами `nginx` должен быть подключен созданный нами `Volume` (Монтирование должно осуществляться в `logs/external`, иначе под упадёт при старте):

```bash
cat nginx.conf
# docker volume create inno-dkr-04-volume — создает именнованный том inno-dkr-04-volume

$ docker volume create inno-dkr-04-volume

docker run -d -p 127.0.0.1:8891:80 --name inno-dkr-04 --mount type=bind,source="$(pwd)/nginx.conf",target=/etc/nginx/nginx.conf -v inno-dkr-04-volume:/logs/external nginx:stable 
```

Покажем в терминале:

<img width="1548" height="428" alt="image" src="https://github.com/user-attachments/assets/dc5af908-89e7-47dd-92a3-e7742f9288a8" />
<img width="1631" height="113" alt="image" src="https://github.com/user-attachments/assets/8b874e93-0c15-4e72-a96a-ac04369e35e1" />

Проверим работу, обратившись к `127.0.0.1:8891`, — в ответ должно возвращать строку: `Welcome to the training program Innowise: Docker!`. Выведим список запущенных контейнеров — контейнер должен быть запущен. Выведим список существующих `docker volumes`. Выведим содержимое `volume` на хостовой системе, воспользовавшись командой `ls -la`:

```bash
curl http://127.0.0.1:8891
curl http://127.0.0.1:8891
docker ps
docker volume ls
```

Покажем в терминале:

<img width="1222" height="92" alt="image" src="https://github.com/user-attachments/assets/2faeaa45-714c-42c6-97a8-974773b2d5bd" />
<img width="1611" height="205" alt="image" src="https://github.com/user-attachments/assets/0f4f9941-0610-448f-a92f-16d13559ab18" />


Дёрнем нашу сраницу ещё раз, глянем содержимое логов внутри контейнера, использовав команду `docker exec inno-dkr-04 cat /logs/exter
nal/access.log`, удалим наш контейнер:

<img width="1098" height="153" alt="image" src="https://github.com/user-attachments/assets/7bd503c2-4cf8-4c1f-ba53-e403f1ca8611" />

Создадим такой же контейнер, как и раньше, проверим содержимое логов:

<img width="1636" height="163" alt="image" src="https://github.com/user-attachments/assets/1bb9e03c-7adc-4a74-ad8b-4b7badbd7cfc" />

Как видим, в томе хранятся данные, которые были созданы во время работы предыдущего контейнера.

