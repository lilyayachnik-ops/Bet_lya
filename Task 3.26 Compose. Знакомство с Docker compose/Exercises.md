# Checkpoints

Создадим `compose`-файл `docker-compose-unmounted.yml`, который будет запускать сервис со следующими параметрами:

- слушает на хосте `127.0.0.1:18888`,
- имеет имя `nginx`,
- образ — `nginx:stable`.

```bash
version: '3.8'
services:
  nginx:
    image: nginx:stable
    ports:
      - "127.0.0.1:18888:80"
```

Покажем в терминале:

<img width="703" height="204" alt="image" src="https://github.com/user-attachments/assets/e13a08d3-a17b-4e1d-82a4-3c2545a7a936" />

Запустим данный сервис работающий в фоне и имеющий имя проекта `inno27`:

```bash
docker-compose -f docker-compose-unmounted.yml up -d
```

Покажем в терминале:

<img width="1593" height="114" alt="image" src="https://github.com/user-attachments/assets/e0562d23-505c-44e7-86db-1f3f61ea40e0" />

Выведим список запущенных контейнеров:

```bash
docker ps
```

Покажем в терминале:

<img width="1327" height="73" alt="image" src="https://github.com/user-attachments/assets/e40c3e3b-9d92-4f23-981d-f5d122e13a97" />

Скачаём конфигурационный файл `nginx`:

```bash
cp ~/docker_3.2/nginx-new.conf ~/inno27/
ls -la
cat ./nginx-new.conf
```

Покажем в терминале:

<img width="1284" height="534" alt="image" src="https://github.com/user-attachments/assets/b08daa3c-6cfb-4ac2-bfc8-f06d1b9efc9c" />

Создадим второй `compose`-файл `docker-compose-mounted.yml`, который будет запускать тот же сервис с теми же параметрами, и еще подключать скачанный нами конфигурационный файл `nginx` внутрь контейнера:

```bash
version: '3.8'
services:
  nginx:
    image: nginx:stable
    ports:
      - "127.0.0.1:18888:80"
    volumes:
      -./nginx-new.conf:/etc/nginx/nginx.conf
```

Покажем в терминале:

<img width="1298" height="251" alt="image" src="https://github.com/user-attachments/assets/ae4d0451-af95-4119-92d2-e0410740ccbf" />

Перезапустим сервис с использованием нового файла, но со старым именем, чтобы старый контейнер был заменен новым:

```bash
docker-compose -f docker-compose-mounted.yml up -d
curl http://127.0.0.1:18888
```

Покажем в терминале:

<img width="1591" height="132" alt="image" src="https://github.com/user-attachments/assets/4127bf72-1098-41af-b33a-4cf4fdb1659a" />

Выведим список запущенных контейнеров:

```bash
docker ps
```

Покажем в терминале:

<img width="1570" height="68" alt="image" src="https://github.com/user-attachments/assets/386d1318-df14-4cf3-a1d2-10f5275664a9" />
