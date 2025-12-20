# Checkpoints

Возьмём файл `main.go`, содержащий следующее:

```bash
package main

import (
	"database/sql"
	"fmt"
	_ "github.com/lib/pq"
	"log"
	"net/http"

	"github.com/caarlos0/env"
	"github.com/prometheus/client_golang/prometheus"
	"github.com/prometheus/client_golang/prometheus/promhttp"
)

type config struct {
	PostgresUri   string `env:"POSTGRES_URI" envDefault:"postgres://root:pass@127.0.0.1/postgres"`
	ListenAddress string `env:"LISTEN_ADDRESS" envDefault:":7000"`
}

var (
	db          *sql.DB
	errorsCount = prometheus.NewCounterVec(
		prometheus.CounterOpts{
			Name: "gocalc_errors_count",
			Help: "Gocalc Errors Count Per Type",
		},
		[]string{"type"},
	)

	requestsCount = prometheus.NewCounter(
		prometheus.CounterOpts{
			Name: "gocalc_requests_count",
			Help: "Gocalc Requests Count",
		})
)

func main() {
	var err error

	// Initing prometheus
	prometheus.MustRegister(errorsCount)
	prometheus.MustRegister(requestsCount)

	// Getting env
	cfg := config{}
	if err = env.Parse(&cfg); err != nil {
		fmt.Printf("%+v\n", err)
	}

	// Connecting to database
	db, err = sql.Open("postgres", cfg.PostgresUri)
	if err != nil {
		log.Fatalf("Can't connect to postgresql: %v", err)
	}
	defer db.Close()

	err = db.Ping()
	if err != nil {
		log.Fatalf("Can't ping database: %v", err)
	}

	http.HandleFunc("/", handler)
	http.Handle("/metrics", promhttp.Handler())
	log.Fatal(http.ListenAndServe(cfg.ListenAddress, nil))
}

func handler(w http.ResponseWriter, r *http.Request) {
	requestsCount.Inc()

	keys, ok := r.URL.Query()["q"]
	if !ok || len(keys[0]) < 1 {
		errorsCount.WithLabelValues("missing").Inc()
		log.Println("Url Param 'q' is missing")
		http.Error(w, "Bad Request", 400)
		return
	}
	q := keys[0]
	log.Println("Got query: ", q)

	var result string
	sqlStatement := fmt.Sprintf("SELECT (%s)::numeric", q)
	row := db.QueryRow(sqlStatement)
	err := row.Scan(&result)

	if err != nil {
		log.Println("Error from db: %s", err)
		errorsCount.WithLabelValues("db").Inc()
		http.Error(w, "Internal Server Error", 500)
		return
	}

	fmt.Fprintf(w, "query %s; result %s", q, result)
}
```

Напишим и добавим в директорию `/home/user/gocalc/Dockerfile` с `Multi-stage build`, в котором сборка выполняется в одном образе (пусть это будет образ `golang:1.19.1-alpine`), а исполнение в другом образе — `alpine:3.10.3`. Просьба учесть, что в `Go 1.16` и выше изменилось дефолтное значение переменной `GO111MODULE`, которая отвечает за поиск модулей в каталоге `GOPATH`. Для корректной работы, необходимо указать значение `auto`. Для установки `golang` зависимостей используем команду `go get`. Для сборки бинарного файла используйте команду `go build`. Используя дополнительные флаги, задайте имя выходного файла `app`. В итоговом образе укажите директиву, для запуска вашего файла при старте контейнера:

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

Покажем в терминале:

<img width="1269" height="761" alt="image" src="https://github.com/user-attachments/assets/dc704390-556c-468f-a6b4-fa028c4b4dff" />

Соберём образ. Выведем список образов. Изучим `history` собранного образа. Запушим результаты задания в репозиторий на `Gitlab`:

```bash
docker build -t goapp:v1.0 .
docker images
docker history goapp:v1.0
git init
git remote add docker314 https://gitlab.com/lilyayachnik/docker_3.14.git
git add main.go dockerfile
git commit -m "Add Dockerfile with multi-stage build and main.go"
git push -u docker314 master
```

Покажем в терминале:

<img width="1610" height="669" alt="image" src="https://github.com/user-attachments/assets/e747caae-97c1-48d2-8b3d-a7949e608062" />
<img width="1578" height="752" alt="image" src="https://github.com/user-attachments/assets/c7160143-efd3-4bb9-8dca-b8953b6b951d" />
<img width="1281" height="227" alt="image" src="https://github.com/user-attachments/assets/9fb97bf7-8e79-4a61-8c4b-92853a9b6475" />

Модифицируем `dockerfile` таким образом, что не должен использовать именованный этап (директива `FROM` не должна содержать параметр `AS` у образа сборщика, и `COPY --from=` не должен ссылаться на имя):

```bash
# First stage (build) without named stage
FROM golang:1.23.1-alpine

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

# Copy only the compiled binary from the build stage using index 0
# --from=0 refers to the first FROM stage (index starts from 0)
COPY --from=0 /app/app .

# Executes the Go application binary
CMD ["./app"]
```

Покажем в терминале:

<img width="1072" height="757" alt="image" src="https://github.com/user-attachments/assets/ed081feb-9a50-4af2-a00c-3ee7dfb48419" />

Добавим в `/home/user/gocalc/Dockerfile` директиву `ARG` с секретом в первый образ и запишим его значение в файл в конечном образе:

```bash
# First stage (build) without named stage
FROM golang:1.23.1-alpine

# Add ARG directive with secret in the first image
ARG SECRET_PASSWORD=default_secret_value

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
# 4. Write the secret value to a file in the first image
RUN go mod init gocalc && \
    go mod tidy && \
    go build -o app main.go && \
    echo "$SECRET_PASSWORD" > secret.txt

# Runtime stage - uses minimal Alpine Linux for the final image
FROM alpine:3.10.3

# Set working directory for the runtime container
WORKDIR /app

# Copy only the compiled binary from the build stage using index 0
# --from=0 refers to the first FROM stage (index starts from 0)
COPY --from=0 /app/app .

# Copy the secret file from the first stage to the final image
COPY --from=0 /app/secret.txt .

# Executes the Go application binary
CMD ["./app"]
```

Покажем в терминале:

<img width="1041" height="798" alt="image" src="https://github.com/user-attachments/assets/da2c0c68-209b-4bad-8dec-22f7a9d33a41" />
<img width="1019" height="160" alt="image" src="https://github.com/user-attachments/assets/899e223f-6f3c-4075-a3e0-2461c8130ac1" />

Соберём образ:

```bash
docker build --build-arg SECRET_PASSWORD=my_super_secret_123 -t goapp:v2.0 .
```

Покажем в терминале:

<img width="1584" height="750" alt="image" src="https://github.com/user-attachments/assets/bd35fe47-54e7-4620-844b-8f79560e15a3" />

Склонируем на свою машину содержимое из официального репозитория `grafana` из ветки с именем `v6.3.x`:

```bash
# --branch v6.3.x — указываем конкретную ветку
# --depth 1 — скачиваем только последний коммит ветки
git clone --branch v6.3.x --depth 1 https://github.com/grafana/grafana.git
```

Покажем в терминале:

<img width="1295" height="177" alt="image" src="https://github.com/user-attachments/assets/9db3411e-a4ca-4c62-9386-29b73ae2b426" />

Добавим в существующий `/home/user/grafana/Dockerfile` еще один образ на базе `nginx:alpine`, в который копируется скомпилированная на предыдущем шаге статика `(public)`:

```bash
# New stage: Nginx container for serving Grafana static files
# This creates a lightweight image that only contains the compiled frontend assets
FROM nginx:alpine AS grafana-static

# Copy compiled static files from the Node.js build stage (stage 1)
# The Grafana frontend is built during the Node stage and stored in /usr/src/app/public/
# This includes HTML, CSS, JavaScript, images, and other assets
COPY --from=1 /usr/src/app/public /usr/share/nginx/html

# Default command to run nginx in the foreground
# Docker requires the main process to run in the foreground
# The -g "daemon off;" directive prevents nginx from running as a background daemon
CMD ["nginx", "-g", "daemon off;"]
```

Покажем в терминале:

<img width="1232" height="414" alt="image" src="https://github.com/user-attachments/assets/29d23766-a3b6-4015-9dcd-9c438bfe78ba" />

Соберём отдельно образ с `nginx`, отдельно с приложением. Выставим им теги `grafana:app` и `grafana:static`. Выведим список образов:

```bash
docker build --target=grafana-app -t grafana:app .
docker build --target=grafana-static -t grafana:static .
docker images
```

Покажем в терминале:

<img width="1414" height="810" alt="image" src="https://github.com/user-attachments/assets/54eeb0e8-1fe7-4300-9823-70f93291025b" />
<img width="921" height="446" alt="image" src="https://github.com/user-attachments/assets/3ed18191-2096-4ef7-a1ee-de037cbcf2ba" />
<img width="1180" height="740" alt="image" src="https://github.com/user-attachments/assets/14426b82-3c96-44be-9f7e-c958bdfa843b" />
<img width="897" height="268" alt="image" src="https://github.com/user-attachments/assets/edddc321-5d25-4905-bee9-4ffd4caf5bb4" />
