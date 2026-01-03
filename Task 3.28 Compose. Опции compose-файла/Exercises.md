# Checkpoints

В репозитории из предыдущего задания создадим новую ветку с именем `dkr-29-compose-opt`:

```bash
git checkout -b dkr-29-compose-opt master
```

Покажем в терминале:

<img width="1356" height="43" alt="image" src="https://github.com/user-attachments/assets/c881fc61-f39f-4089-92a8-667f4939d219" />

Изменим `/home/user/dkr-14-gocalc/docker-compose.yml` файл, добавив следующее:

- установим политику перезапуска `unless-stopped` у `gocalc`;
- у `nginx` сократим `capabilities` до минимального уровня, как мы делали ранее;
- добавим `depends_on` у сервисов по их зависимости друг от друга;
- создадим `2` сети - в одной будет `gocalc` и `nginx`, а в другой - `gocalc` и `postgres`.

```bash
services:
  postgres:
    image: postgres:10
    environment:
      POSTGRES_PASSWORD: DatabasePassword
    networks:
      - backend
  gocalc:
    build: .
    depends_on:
      - postgres
    environment:
      POSTGRES_URI: "postgres://postgres:DatabasePassword@postgres:5432/postgres?sslmode=disable"
      LISTEN_ADDRESS: ":7000"
    networks:
      - backend
      - frontend
    restart: unless-stopped
  nginx:
    image: nginx:alpine
    depends_on:
      - gocalc
    ports:
      - "80:80"
    cap_drop:
      - ALL
    cap_add: 
      - CAP_NET_BIND_SERVICE
      - CAP_CHOWN
      - CAP_SETGID
      - CAP_SETUID 
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
    networks:
      - frontend

  networks:
    backend:
      driver: bridge
    frontend:
      driver: bridge
```

Покажем в терминале:

<img width="1533" height="842" alt="image" src="https://github.com/user-attachments/assets/5cf553b5-80d0-490e-a8b3-7e5309908694" />
<img width="1417" height="216" alt="image" src="https://github.com/user-attachments/assets/e17f5e0f-be3a-495b-bf0c-74cb0af4bc76" />

Запустим сервис `nginx` - должны подняться все `3` контейнера с именем проекта `inno29`:

```bash
docker-compose -f docker-compose.yml -p inno29 up -d nginx
```

Покажем в терминале:

<img width="1475" height="134" alt="image" src="https://github.com/user-attachments/assets/0c53fa28-f9ef-4a81-894f-f67a34872228" />

Подключимся к контейнеру сервиса `nginx` и попытаемся обратиться к сервису `postgres` командой `ping`:

```bash
docker exec -it inno29-nginx sh
/ # ping postgres
/ # ping gocalc
```

Покажем в терминале:

<img width="997" height="266" alt="image" src="https://github.com/user-attachments/assets/06a3c3f2-6f99-4b87-853c-38456ecb6f7f" />

Загрузим новую ветку с изменениями в репозиторий:

```bash
git add docker-compose.yml nginx.conf
git commit -m "Add docker-compose and nginx config"
git push -u inno28 dkr-29-compose-opt
```

Покажем в терминале:

<img width="1136" height="133" alt="image" src="https://github.com/user-attachments/assets/e17bca58-0b8e-4510-b46e-65b723ee02ae" />
<img width="1426" height="313" alt="image" src="https://github.com/user-attachments/assets/fd75f61d-e271-4aa5-ac10-82f829cac042" />



