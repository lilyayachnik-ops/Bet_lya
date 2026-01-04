# Checkpoints

Склонируем репозиторий `https://gitlab.com/andrew1test-group/dkr30` в свой аккаунт на `gitlab` с именем `dkr-30-voting`. Перейдем по ссылке:

<img width="923" height="279" alt="image" src="https://github.com/user-attachments/assets/a95c466e-8951-4c4c-84f2-347c5dbd888f" />

Нажмём `Fork`:

<img width="925" height="803" alt="image" src="https://github.com/user-attachments/assets/25dcd8bd-a974-4b6b-98ad-a2c95fc33ba6" />
<img width="924" height="479" alt="image" src="https://github.com/user-attachments/assets/f71c671f-f092-4357-a639-fcfb8e9231f1" />

Нажмём `Fork project`:

<img width="927" height="842" alt="image" src="https://github.com/user-attachments/assets/ca9e6a01-0e7a-4b78-9534-a32078e1c2fc" />

Склонируем репозиторий себе на машинку:

```bash
git clone https://gitlab.com/lilyayachnik/dkr-30-voting.git
```

Напишим `docker-compose.yml` файл, который бы собирал приложение, запускал его и все требуемые зависимости:

- `nginx` - проксирует запросы на `voting` и доступен на хосте на порту `20000`. Пример конфигурационного файла находится в папке `nginx`, смонтируем его в `/etc/nginx/conf.d/default.conf`. Используем `alpine`-версию образа.
- `voting` - собирается из репозитория;
- `mysql` - база данных, к которой подключается `voting`;
- `redis` - `In-memory` хранилище для кэша;

```bash
version: "3.8"
services:
  nginx:
    image: nginx:alpine
    depends_on:
      - voting
    ports:
      - "20000:80"
    volumes:
      - ./nginx/default.conf:/etc/nginx/conf.d/default.conf
    networks:
      - frontend
  voting:
    build:
      context: .
      dockerfile: Dockerfile
    depends_on:
      - mysql
      - redis
    environment:
      REDIS_HOST: redis
      REDIS_PORT: 6379
      REDIS_PASSWORD: null

      APP_ENV: "local"
      APP_DEBUG: "true"
  
      DB_HOST: mysql
      DB_DATABASE: voting_db
      DB_USERNAME: mysql
      DB_PASSWORD: password
    networks:
      - frontend
      - backend
    restart: unless-stopped

  
  mysql:
    image: mysql:5.7
    environment:
      MYSQL_ROOT_PASSWORD: rootpassword
      MYSQL_DATABASE: voting_db
      MYSQL_USER: mysql
      MYSQL_PASSWORD: password
    networks:
      - backend

  redis:
    image: redis:alpine
    networks:
      - backend    

networks:
  backend:
     driver: bridge
  frontend:
     driver: bridge   
```

Покажем в терминале:

<img width="1176" height="690" alt="image" src="https://github.com/user-attachments/assets/325d4485-6583-4b71-a02a-a5c3e2bd5ac9" />
<img width="1276" height="758" alt="image" src="https://github.com/user-attachments/assets/86b475c7-b8cb-43ac-9783-08cdfa7d3175" />

Запустим сервис с именем проекта `inno30`:

```bash
docker-compose -f docker-compose.yml -p inno30 up -d
```

Покажем в терминале:

<img width="1633" height="204" alt="image" src="https://github.com/user-attachments/assets/4c514ff6-71f0-41fa-9ed9-6099551327aa" />

Содержимое `README`:

## Platform
* `php >= 7.2` (served best with `FPM`)
* `MySQL` database backend
* `Redis` database backend

## Run configuration
Application entrypoint is `public/index.php`, all the requests must be marshalled here.
When running application rely on environment variables. Initial subset of variables is expected in `.env` located
in project root. `.env.dist` can be used as a blueprint. 

## Database
Application uses MySQL and Redis as storage backends.

#### Migration
```shell script
$ php artisan migrate --force
```
Rollback failed migration:
```
$ php artisan migrate:rollback --force
```

#### Seeding command
*Usually takes place upon first deployment to populate DB with initial values*
```shell script
$ php artisan db:seed --force
```

### API

* RPC `[GET] /ping` - health check endpoint must return 200 OK if service is configured
* REST resource `/polls` - polls CRUD
* RPC `[POST] /polls/{id}/vote` - to vote in defined poll
* RPC `[GET] /polls/{id}/results` - poll results
* RPC `[GET] /summary` - polls summary

### CLI

```
# Periodic collect to aggregate polls statustics which will be used in /summary API endpoint
$ php artisan polls:collect:status

# Running unit tests
$ php vendor/bin/phpunit -c phpunit.xml 
```

Сконфигурируем приложение, выполнив команды из раздела `Migration` и `Seeding` в `README` репозитория:

```bash
docker exec inno30-voting-1 php artisan migrate --force
docker exec inno30-voting-1 php artisan db:seed --force
```

Покажем в терминале:

<img width="1328" height="229" alt="image" src="https://github.com/user-attachments/assets/257bc167-13df-4ef5-9977-fdf613ca0f57" />

Обратимся к сервису по `localhost:20000/polls` (в ответе мы должны увидеть `json`-объект):

```bash
curl http://localhost:20000/polls
```

Покажем в терминале:

<img width="955" height="690" alt="image" src="https://github.com/user-attachments/assets/00cbfde2-4e3c-421f-bdf8-bd9f3267deb3" />

Загрузим новые файлы в репозиторий:

```bash
git remote add inno30 https://gitlab.com/lilyayachnik/dkr-30-voting
git add Dockerfile docker-compose.yml
git push -u inno30 --all
```

Покажем в терминале:

<img width="1120" height="23" alt="image" src="https://github.com/user-attachments/assets/79d9c4b6-2fba-4b2d-b6d6-a7133c0b7c1f" />
<img width="1350" height="356" alt="image" src="https://github.com/user-attachments/assets/b3b4daa0-3e44-4e36-8fbe-849f0b2b0ba6" />

