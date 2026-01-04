# Checkpoints

# ELK Стек — ElasticSearch (хранилище, индексирование, масштабирование) + Logstash (сбор логов, парсинг и трансформация, маршрутизация логов в разные хранилища) + Kibana (визуализация)
# EFK Стек — ElasticSearch (хранилище, индексирование, масштабирование) + Fluentd (сбор логов, парсинг и трансформация, маршрутизация логов в разные хранилища) + Kibana (визуализация)
#

В репозитории из предыдущего задания создадим новую ветку с именем `dkr-31-voting-efk`:

```bash
git checkout -b dkr-31-voting-efk main
git branch
```

Покажем в терминале:

<img width="979" height="136" alt="image" src="https://github.com/user-attachments/assets/932d6eb6-aa57-4c1e-b231-ff75a222fc00" />

Изменим `docker-compose.yml` файл, добавив следующее:
- добавим `EFK` стек;
- `Fluentd` должен быть доступен с хоста;
- `Kibana` должна быть доступна с хоста;
- остальные сервисы должны отправлять логи в `fluentd` через `fluentd log driver`.

```bash
version: "3.8"
services:
  nginx:
    image: nginx:alpine
    depends_on:
      - voting
      - fluentd
    ports:
      - "20000:80"
    volumes:
      - ./nginx/default.conf:/etc/nginx/conf.d/default.conf
    networks:
      - frontend
    logging:
      driver: "fluentd"
      options: 
        fluentd-address: localhost:24224
        tag: "nginx"
  voting:
    build:
      context: .
      dockerfile: Dockerfile
    depends_on:
      - mysql
      - redis
      - fluentd
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
    logging:
      driver: "fluentd"
      options: 
        fluentd-address: localhost:24224
        tag: voting

  
  mysql:
    image: mysql:5.7
    environment:
      MYSQL_ROOT_PASSWORD: rootpassword
      MYSQL_DATABASE: voting_db
      MYSQL_USER: mysql
      MYSQL_PASSWORD: password
    networks:
      - backend
    logging:
      driver: "fluentd"
      options: 
        fluentd-address: localhost:24224
        tag: mysql

  redis:
    image: redis:alpine
    networks:
      - backend    
    logging:
      driver: "fluentd"
      options: 
        fluentd-address: localhost:24224
        tag: redis

  fluentd:
    build: ./fluentd
    volumes:
      - ./fluentd/conf:/fluentd/etc
    depends_on:
      # Launch fluentd after that elasticsearch is ready to connect
      elasticsearch:
        condition: service_healthy
    ports:
      - "24224:24224"
      - "24224:24224/udp"
  
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:8.17.1
    container_name: elasticsearch
    hostname: elasticsearch
    environment:
      - discovery.type=single-node
      - xpack.security.enabled=false # Disable security for testing
    healthcheck:
      # Check whether service is ready
      test: ["CMD", "curl", "-f", "http://localhost:9200/_cluster/health"]
      interval: 10s
      retries: 5
      timeout: 5s
    ports:
      - 9200:9200

  kibana:
    image: docker.elastic.co/kibana/kibana:8.17.1
    depends_on:
      # Launch fluentd after that elasticsearch is ready to connect
      elasticsearch:
        condition: service_healthy
    ports:
      - "5601:5601"

networks:
  backend:
     driver: bridge
  frontend:
     driver: bridge
```

Покажем в терминале:

<img width="1489" height="887" alt="image" src="https://github.com/user-attachments/assets/edeafb25-e2a3-47aa-b5bb-143d81c17796" />
<img width="1542" height="784" alt="image" src="https://github.com/user-attachments/assets/0a5c5a4c-8e1f-44f1-a46d-57428fd21ac3" />
<img width="1469" height="727" alt="image" src="https://github.com/user-attachments/assets/7ab31446-8fc2-4aac-9594-edaab8e95739" />
<img width="1475" height="574" alt="image" src="https://github.com/user-attachments/assets/719827c7-b06a-440d-9560-5e9a5abf67f7" />

Создадим папку `./fluentd`, в которой будет лежать `Dockerfile` для сборки образа `fluentd`, а также конфигурационный файл `./fluentd/conf/fluent.conf`. Содержимое `Dockerfile`:

```bash
# fluentd/Dockerfile

FROM fluent/fluentd:edge-debian
USER root

# To connect to docker.elastic.co/elasticsearch/elasticsearch:8.x, it requires elasticsearch v8 gem
# Ref. https://github.com/elastic/elasticsearch-ruby/blob/main/README.md#compatibility
RUN ["gem", "install", "elasticsearch", "--no-document", "--version", "8.19.0"]

RUN ["gem", "install", "fluent-plugin-elasticsearch", "--no-document", "--version", "5.4.3"]
USER fluent
```

Покажем в терминале:

<img width="1519" height="262" alt="image" src="https://github.com/user-attachments/assets/2f353b36-0f0a-413a-a0a0-0334dd31067a" />

Содержимое `./fluentd/conf/fluent.conf`:

```bash
# fluentd/conf/fluent.conf

# Секция SOURCE (Источник) - определяет откуда получать логи
<source>
  # Используем плагин forward для получения логов через протокол forward
  # Этот плагин используется для приема логов от Docker, других Fluentd агентов и т.д.
  @type forward
  
  # Порт, на котором Fluentd будет слушать входящие соединения
  # Docker использует порт 24224 для отправки логов через fluentd драйвер
  port 24224
  
  # Привязка к IP-адресу: 0.0.0.0 означает "все сетевые интерфейсы"
  # Fluentd будет принимать соединения с любого IP-адреса
  bind 0.0.0.0
</source>

# Секция MATCH (Совпадение) - определяет что делать с полученными логами
# Паттерн *.** означает "все теги и все их подтеги"
# Пример: web.access, app.error, db.query и т.д.
<match *.**>
  # Используем плагин copy для копирования логов в несколько мест назначения
  # Каждый <store> внутри будет получать копию одних и тех же логов
  @type copy

  # Первое хранилище — отправляем логи в Elasticsearch
  <store>
    # Используем плагин elasticsearch для отправки в Elasticsearch
    @type elasticsearch
    
    # Хост Elasticsearch сервера
    # 'elasticsearch' — это имя сервиса в Docker Compose, будет разрешено через DNS
    host elasticsearch
    
    # Порт Elasticsearch (по умолчанию 9200 для REST API)
    port 9200
    
    # Использовать формат Logstash для именования индексов
    # Создает индексы в формате: logstash-YYYY.MM.DD
    logstash_format true
    
    # Префикс для индексов (будет использоваться вместо 'logstash')
    # Результат: fluentd-YYYY.MM.DD
    logstash_prefix fluentd
    
    # Формат даты в имени индекса
    # %Y%m%d = ГодМесяцДень (например: 20240115)
    logstash_dateformat %Y%m%d
    
    # Включать тег в каждую запись Elasticsearch
    include_tag_key true
    
    # Тип документа в Elasticsearch (в версиях до 7.x)
    # В Elasticsearch 7.x+ это поле deprecated, но можно оставить для совместимости
    type_name access_log
    
    # Название поля, в которое будет записан тег
    # По умолчанию используется поле @log_name
    tag_key @log_name
    
    # Интервал сброса буфера в секундах
    # Каждую секунду Fluentd будет отправлять накопленные логи в Elasticsearch
    flush_interval 1s
  </store>

  # Второе хранилище — дублируем логи в стандартный вывод (консоль)
  <store>
    # Плагин stdout выводит логи в консоль
    @type stdout
    
    # По умолчанию выводит в формате:
    # 2024-01-15 10:30:00 +0000 app.access: {"message":"Request processed"}
  </store>
</match>
```

Покажем в терминале:

<img width="1543" height="681" alt="image" src="https://github.com/user-attachments/assets/83a03142-abf7-4bb0-8253-03ba9f8e1d35" />

Запустим сервис с именем проекта `rbm31`:

```bash
docker-compose -p rbm31 up -d fluentd elasticsearch kibana
docker-compose -p rbm31 up -d nginx voting mysql redis
docker ps
```

Покажем в терминале:

<img width="1624" height="705" alt="image" src="https://github.com/user-attachments/assets/c901b038-f272-4725-bb0b-4d2057e77393" />

Сконфигурируем приложение, выполнив команды из раздела `Migration` и `Seeding` в `README` репозитория:

```bash
docker exec rbm31-voting-1 php artisan migrate --force
docker exec rbm31-voting-1 php artisan db:seed --force
```

Покажем в терминале:

<img width="1491" height="133" alt="image" src="https://github.com/user-attachments/assets/7ddb06f6-17e7-40ab-8fe0-c84daae9932b" />

Обратимся к сервису по `localhost:20000/polls` (мы должны увидеть `json`-объект):


```bash
 curl http://localhost:20000/polls
```

Покажем в терминале:

<img width="1635" height="316" alt="image" src="https://github.com/user-attachments/assets/5b8ad6e1-03ac-44d0-9078-316f81267cc3" />

Откроем `Kibana` в браузере и проверим, что логи запроса присутствуют, введя следующий `URL`:

```bash
http://localhost:5601/app/discover#/ 
```

Нажмём `Create data view`

<img width="879" height="608" alt="image" src="https://github.com/user-attachments/assets/8f1feb8b-1881-404b-b77a-5f6344634c4f" />

Введём `fluentd-*` в `Index pattern` и нажмём `Save data view to Kibana`. Покажем в терминале:

<img width="879" height="606" alt="image" src="https://github.com/user-attachments/assets/e5aeea54-d3cc-478c-855c-2885daa2b2fd" />
<img width="1904" height="916" alt="image" src="https://github.com/user-attachments/assets/97b025bc-0798-48a6-bbf7-53367b0dcec8" />

Загрузим новую ветку с изменениями в репозиторий:

```bash
git add docker-compose.yml fluentd/
git commit -m "Add EFK stack for logging: Fluentd, Elasticsearch, Kibana"
git status
git push inno30 dkr-31-voting-efk
```

Покажем в терминале:

<img width="1425" height="202" alt="image" src="https://github.com/user-attachments/assets/c002e3db-3793-4e0a-bd4a-d556b448d669" />
<img width="1485" height="334" alt="image" src="https://github.com/user-attachments/assets/fa41a8c7-872f-4ee1-96fa-9ef22c122183" />


