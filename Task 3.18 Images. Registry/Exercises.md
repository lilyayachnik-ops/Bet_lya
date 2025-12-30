<img width="734" height="187" alt="image" src="https://github.com/user-attachments/assets/114e55df-1bf4-41c9-98e4-7c2a2a3f8c59" /><img width="960" height="709" alt="image" src="https://github.com/user-attachments/assets/44d447b4-b877-4eb3-9dae-305ea5290f2b" /># Checkpoints

Развернём локальный `registry` согласно официальной документации. Для этого зарегистрируемся в `Docker Hub`:

<img width="960" height="709" alt="image" src="https://github.com/user-attachments/assets/dec2df21-3ac0-4bec-99a2-004cde3f152c" />

Далее создадим свой репозиторий. Для этого переходим во вкладку `Repositories`:

<img width="734" height="187" alt="image" src="https://github.com/user-attachments/assets/8e0efe78-1de8-4117-a343-3d5523b5dc5c" />

Выберем `Create a Repository`:

<img width="457" height="222" alt="image" src="https://github.com/user-attachments/assets/3d64d0ae-8675-441a-9a98-241c9e3dcb4f" />

Настроим: 

<img width="928" height="615" alt="image" src="https://github.com/user-attachments/assets/13622559-389a-46c2-bdd4-899aa1856979" />

Мы создали и настроили локальный репозиторий:

<img width="949" height="432" alt="image" src="https://github.com/user-attachments/assets/4f80f2fd-9dda-4634-a1ee-b99d660f1863" />

Соберём образ приложения `gocalc`, которое мы собирали ранее:

```bash
package main

import (
    "fmt"
    "log"
    "net/http"
	"strconv"
)

func sendResponse(w http.ResponseWriter, r *http.Request) {
  keys, ok := r.URL.Query()["a"]
  if !ok || len(keys[0]) < 1 {
      log.Println("Url Param 'a' is missing")
      return
  }

  a, err := strconv.Atoi(keys[0])

  if err != nil {
    return
  }

  keys, ok = r.URL.Query()["b"]
  if !ok || len(keys[0]) < 1 {
      log.Println("Url Param 'b' is missing")
      return
  }
  b, err := strconv.Atoi(keys[0])
  if err != nil {
    log.Println("Bad number!")
    return
  }

  fmt.Fprintf(w, "%d + %d = %d", a, b, sum(a,b))
}

func main() {
    http.HandleFunc("/", sendResponse)
    log.Fatal(http.ListenAndServe(":7000", nil))
}

func sum (a, b int) int {
  t := b + a
  return t
}
```

Выполним `git pull` в свой локальный репозиторий и там же соберём образ:

```bash
git init
git remote add gocalc https://gitlab.com/lilyayachnik/git-merge.git
git pull gocacl master
```

Покажем в терминале:

<img width="1221" height="494" alt="image" src="https://github.com/user-attachments/assets/bb8ac276-273f-4116-a905-95e4e37e1ea1" />

Создадим `Dockerfile`:

```bash
FROM golang:alpine AS builder
WORKDIR /app
COPY main.go .
ENV GO111MODULE=off
RUN go build -o app .

FROM gcr.io/distroless/base
WORKDIR /app
COPY --from=builder /app/app .
EXPOSE 7000
CMD ["./app"]
```

Покажем в терминале:

<img width="546" height="291" alt="image" src="https://github.com/user-attachments/assets/0508423e-ce91-41ff-80a6-958536dc9616" />

Соберём образ и запустим контейнер, чтобы протестировать работу:

```bash
docker build -t gocalc:v1.0 .
docker run -d --rm -p 7000:7000 --name gocalc-app gocalc:v1.0
curl "http://localhost:7000/?a=8&b=10"
```

Покажем в терминале:

<img width="1423" height="604" alt="image" src="https://github.com/user-attachments/assets/5bc0424e-c5d0-43ea-b15a-30800f101a34" />
<img width="1075" height="42" alt="image" src="https://github.com/user-attachments/assets/ee493a4f-ee81-4869-b5e1-f62d033526a8" />
<img width="778" height="48" alt="image" src="https://github.com/user-attachments/assets/bbe39c59-3b52-4536-bfbf-998b02b91787" />

Всё работает. 
Образ необходимо загрузить  в локальный `registry`. Запушим наш образ `gocalc:v1.0` в наш созданный репозиторий в `Docker Hub`:

```bash
docker login
docker tag gocalc:v1.0 betl123/innowise-training:sum-v1.0

# betl123 — username
# innowise-training — the name of repository
# sum-v1.0 — the name of tag

$ docker push betl123/innowise-training:sum-v1.0
```

Покажем в термниале:

<img width="1147" height="157" alt="image" src="https://github.com/user-attachments/assets/7840cf2f-c938-4d07-b1ae-0c1baa918ff7" />
<img width="1172" height="468" alt="image" src="https://github.com/user-attachments/assets/4b5d78ce-5974-4d44-814a-c042feac605f" />
