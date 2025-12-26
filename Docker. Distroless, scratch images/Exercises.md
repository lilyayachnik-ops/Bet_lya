# Checkpoint

Собрать минимальный образ (`distroless` / `scratch`) для приложения: взять простое приложение (`Go`, `Python`, `Node`), собрать образ размером `< 20` MB, убрать `shell`, `package manager`, всё лишнее, настроить `healthcheck` без `bash`. Содержимое `Dockerfile`:

```bash
FROM golang:1.24-alpine AS builder

# Create and set /app as the working directory for all following commands
WORKDIR /app

# Copy only main.go from the local directory to the container's /app directory
COPY main.go .

# Compile the Go program, creating an executable named 'myapp' from main.go
RUN go build -o myapp main.go

# Start a new stage using a minimal security-focused base image without shell or package manager
FROM gcr.io/distroless/static

# Copy the compiled 'myapp' executable from the builder stage to /app in the final image
COPY --from=builder /app/myapp /app

# Configure container health checks to run every 30s, with timeout after 10s, 
# wait 5s before first check, and retry 3 times before marking as unhealthy
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \ 
  CMD [ "/app", "--healthcheck" ]

# Set the default command to execute the application when container starts
CMD ["/app"]
```

Содержимое файла `main.go`:

```bash
package main

import (
    "flag"
    "fmt"
    "net/http"
    "os"
)

func main() {
    // Флаг для healthcheck режима
    healthCheck := flag.Bool("healthcheck", false, "Run health check")
    flag.Parse()
    
    if *healthCheck {
        // В режиме healthcheck проверяем сервер и выходим
        resp, err := http.Get("http://localhost:8080/health")
        if err == nil && resp.StatusCode == 200 {
            os.Exit(0)  // Здоров
        }
        os.Exit(1)  // Не здоров
    }
    
    // Обычный режим - запуск сервера
    http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
        fmt.Fprint(w, "Hello World!")
    })
    
    http.HandleFunc("/health", func(w http.ResponseWriter, r *http.Request) {
        w.WriteHeader(http.StatusOK)
        w.Write([]byte("OK"))
    })
    
    port := os.Getenv("PORT")
    if port == "" {
        port = "8080"
    }
    
    fmt.Printf("Server starting on port %s\n", port)
    http.ListenAndServe(":"+port, nil)
}
```

Соберём образ:

```bash
docker build -t scratch:v1.0
```

Покажем в терминале:

<img width="1594" height="598" alt="image" src="https://github.com/user-attachments/assets/8d196e23-e65a-4758-8c5e-1b8901017887" />

Запустим контейнер на базе этого образа:

```bash
docker run -d -p 8080:8080 --name go-app scratch:v1.0
```

Покажем в терминале:

<img width="1058" height="45" alt="image" src="https://github.com/user-attachments/assets/0d231177-e675-48b7-97ef-f06108c85165" />

Глянем логи контейнера `go-app`:

```bash
docker logs go-app
```

Покажем в терминале:

<img width="1557" height="48" alt="image" src="https://github.com/user-attachments/assets/f01aaa87-ee70-4eb6-8d6f-3ad0178c0ba0" />

Посмотрим `healthcheck`. Для этого введём команду `docker inspect go-app` и найдем секцию `Health`:

<img width="1044" height="839" alt="image" src="https://github.com/user-attachments/assets/0ede70a2-9f82-4f2d-90f2-fd4362a8a392" />
