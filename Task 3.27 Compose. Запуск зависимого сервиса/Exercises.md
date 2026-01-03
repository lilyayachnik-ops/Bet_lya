# Checkpoints

Для этого задания используем наш репозиторий в `gitlab` с приложением `gocalc` из прошлого задания (`dkr-14-gocalc`). Создадим новую ветку с именем `dkr-28-compose`:

```bash
git clone https://gitlab.com/lilyayachnik/docker_3.14.git
cd ./docker_3.14/
ls -la
git checkout -b dkr-28-compose
git branch
```

Покажем в терминале:

<img width="1326" height="467" alt="image" src="https://github.com/user-attachments/assets/19470dec-7582-462e-86cb-873c5fe91fd4" />

Соберём образ, если у нас его не осталось с предыдущего задания:

```bash
# Build stage - uses Go Alpine image for building the application
FROM golang:1.23.1-alpine AS build

# Install git which is required for Go to fetch dependencies from repositories
RUN apk add --no-cache git 

# Set the working directory inside the container to /app
WORKDIR /app

# Copy main.go from the host machine to the container's working directory
COPY main.go .

# Combine all Go-related commands into a single RUN instruction:
# 1. Initialize Go module with name 'gocalc' (creates go.mod file)
# 2. Download and tidy dependencies (updates go.mod and go.sum)
# 3. Build the application into an executable named 'app'
RUN go mod init gocalc && \
    go mod tidy && \
    go build -o app main.go

# Runtime stage - uses minimal Alpine Linux for the final image
FROM alpine:3.10.3

# Set working directory for the runtime container
WORKDIR /app

# Copy only the compiled binary from the build stage
# This excludes Go compiler, source code, and dependencies, making the image smaller
COPY --from=build /app/app .

# Executes the Go application binary
CMD ["./app"]
```

Покажем в термминале:

<img width="1270" height="760" alt="image" src="https://github.com/user-attachments/assets/27b4b9b4-cafc-4401-89dd-9fd1bd55c567" />

Добавим в `/home/user/dkr-14-gocalc/docker-compose.yml` файл, который выполняет следующее:

- запускает базу данных `postgres` (версии `10`) с паролем для базы данных `DatabasePassword` (в качестве базового образа используем `postgres:10`);
- запускает приложение из собранного образа `gocalc`, которое подключается к базе данных (переменные окружения с примерами формата для конфигурации можно найти в начале файла `main.go`);
- запускает контейнер nginx, который проксирует все запросы на `gocalc` по `DNS`-имени (конфигурационный файл требуется написать и пробросить внутрь контейнера).

```bash
services:
  postgres:
    image: postgres:10
    environment:
      POSTGRES_PASSWORD: DatabasePassword
  gocalc:
    build: .
    environment:
      POSTGRES_URI: "postgres://postgres:DatabasePassword@postgres:5432/postgres?sslmode=disable"
      LISTEN_ADDRESS: ":7000"
    restart: always
  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
```

Покажем в терминале:

<img width="1424" height="468" alt="image" src="https://github.com/user-attachments/assets/0018a024-b6cf-46dd-b193-769290ba1ea3" />

Напишем содержимое файла `nginx.conf`:

```bash
events {
    worker_connections 1024;
}

http {
    server {
        listen 80;
        
        # Явно для /metrics
        location = /metrics {
            proxy_pass http://gocalc:7000/metrics;
        }
        
        # Для всего остального
        location / {
            proxy_pass http://gocalc:7000;
        }
    }
}
```

Покажем в терминале:

<img width="932" height="456" alt="image" src="https://github.com/user-attachments/assets/13a2f427-d6a6-4e60-9355-eaba38d105d5" />

Запустим данный сервис с именем проекта `inno28`:

```bash
# -p inno28 — позволяет задать имя проекта
docker-compose -f docker-compose.yml -p inno28 up -d
```

Покажем в терминале:

<img width="1550" height="137" alt="image" src="https://github.com/user-attachments/assets/e68be265-4ff8-4392-9e39-67429e76e5ee" />

Выведим список всех запущенных контейнеров:

```bash
docker ps
```

Покажем в терминале:

<img width="1555" height="108" alt="image" src="https://github.com/user-attachments/assets/569636df-8184-4f76-8f39-90a5d2a620b0" />

Выполним команду `curl 127.0.0.1/metrics` — должен вернуться ответ от сервиса `gocalc`:

```bash
curl 127.0.0.1/metrics
```

Покажем в терминале:

<img width="1534" height="885" alt="image" src="https://github.com/user-attachments/assets/3c7d84c6-7c97-409b-bcaa-7df22c3e9a96" />
<img width="1520" height="860" alt="image" src="https://github.com/user-attachments/assets/351319d9-1d98-4a7e-a461-1dfc3c02119e" />
<img width="1535" height="862" alt="image" src="https://github.com/user-attachments/assets/35645970-f1eb-4bd1-bcc6-aa3c38febf03" />

Загрузим новую ветку с изменениями в репозиторий:

```bash
git remote add inno28 https://gitlab.com/lilyayachnik/docker_3.14.git
git add docker-compose.yml nginx.conf
git commit -m "Add docker-compose configuration for gocalc app, postgress:10 database with Password, nginx proxy configuration, all services run in docker-compose with project name inno28"
git push -u inno28 --all
```

Покажем в терминале:

<img width="1452" height="28" alt="image" src="https://github.com/user-attachments/assets/0593e629-f868-4556-8403-d9d03a7b9aa0" />
<img width="1557" height="176" alt="image" src="https://github.com/user-attachments/assets/27123487-4654-4378-9c75-8e7eff2ee714" />
<img width="1544" height="330" alt="image" src="https://github.com/user-attachments/assets/cc65a99c-e906-40a3-9109-4cb473b29c08" />
