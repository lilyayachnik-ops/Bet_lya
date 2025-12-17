# Checkpoint

**Цель**: получить практический опыт написания `Dockerfile`, развертывания приложений с использованием `Docker-compose`.

**Задание 1**: Создание `Dockerfile` для приложения веб-сервера. Вам необходимо написать `Dockerfile` для создания контейнера с приложением веб-сервера на основе образа `Ubuntu 20.04`. Приложение должно быть запущено на порту `8080` и должно отдавать статические файлы из каталога `/app/static`.

Шаги, которые необходимо выполнить:
- Создайте новый файл `Dockerfile` в пустой директории на вашем
локальном компьютере.
- Напишите инструкцию `FROM`, которая указывает базовый образ
`Ubuntu 20.04`.
- Установите необходимые зависимости с помощью инструкции `RUN`. Установите пакеты `nginx` и `curl`, а также создайте каталог `/app/static`.
- Скопируйте файл конфигурации `nginx` из вашего локального каталога
внутрь контейнера с помощью инструкции `COPY`.
- Скопируйте статические файлы из каталога `/app/static` на вашем
локальном компьютере внутрь контейнера с помощью инструкции `COPY`.
- Используйте инструкцию `EXPOSE` для открытия порта `8080`.
- Используйте инструкцию `CMD` для запуска команды `nginx` с указанием
пути к файлу конфигурации, который вы скопировали на четвёртом шаге.
- Сохраните файл `Dockerfile` и соберите образ с помощью команды `docker build`.
- Запустите контейнер из образа с помощью команды `docker run` и
проверьте, что веб-сервер отдает статические файлы из каталога `/app/static` на
порту `8080`:

```bash
FROM ubuntu:20.04
RUN apt update && \
    apt install -y curl nginx && \
    mkdir -p /app/static
COPY nginx.conf /etc/nginx/nginx.conf
COPY app/static/ /app/static/
EXPOSE 8080
CMD ["nginx", "-c", "/etc/nginx/nginx.conf", "-g", "daemon off;"]
```

Содержимое файла `nginx.conf`:

```bash
events {
    worker_connections 1024;
}

http {
    include /etc/nginx/mime.types;
    default_type application/octet-stream;

    server {
        listen 8080;
        server_name localhost;
        
        location / {
            root /app/static;
            index index.html;
            try_files $uri $uri/ =404;
        }
    }
}
```

Содержимое файла `index.html`, находящийся в директории `app/static`:

```bash
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Docker Nginx Static Site</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            max-width: 800px;
            margin: 0 auto;
            padding: 20px;
            background-color: #f5f5f5;
        }
        .container {
            background-color: white;
            padding: 30px;
            border-radius: 10px;
            box-shadow: 0 0 10px rgba(0,0,0,0.1);
        }
        h1 {
            color: #333;
        }
        .success {
            color: #4CAF50;
            font-weight: bold;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1> Успех!</h1>
        <p>Nginx успешно запущен в Docker контейнере.</p>
        <p class="success">Статические файлы обслуживаются из каталога /app/static</p>
        <p>Порт: 8080</p>
        <p>Текущее время: <span id="datetime"></span></p>
    </div>

    <script>
        function updateDateTime() {
            const now = new Date();
            document.getElementById('datetime').textContent = 
                now.toLocaleString('ru-RU');
        }
        updateDateTime();
        setInterval(updateDateTime, 1000);
    </script>
</body>
</html>
```

Соберем образ, используя команду `docker build -t mywebserver:v1.0 .`, где `-t` — задает имя и версию образа, `.` — контекст сборки, то есть тут `Docker` ищет `dockerfile`. Запустим в терминале:

<img width="1360" height="512" alt="image" src="https://github.com/user-attachments/assets/d6bbcba6-95e2-4791-b2fb-f0a75bf36858" />

Запустим контейнер, используя команду `docker run -d -p 8080:8080 --name mywebserver1 mywebserver:v1.0`. Покажем в терминале:

<img width="1044" height="47" alt="image" src="https://github.com/user-attachments/assets/6b2485e3-931f-42e1-a64d-8ff6186cb7f9" />

Дернём наш сервер, используя команду `curl http://localhost:8080/`. Покажем в терминале:

<img width="1313" height="804" alt="image" src="https://github.com/user-attachments/assets/2937c1a1-efae-415f-9898-b04d6ec5f1f5" />

Получили в ответ наш `index.html`:

<img width="915" height="337" alt="image" src="https://github.com/user-attachments/assets/2f203f59-2a5f-4db4-bfc6-387023b629db" />

Задание 2 – развертывание приложения с помощью Docker-compose
Шаги, которые необходимо выполнить:
1. Создайте новый файл docker-compose.yml в пустой директории на
вашем локальном компьютере.
2. Напишите инструкцию version в версии 3.
3. Определите сервис для базы данных PostgreSQL. Назовите его ""db"".
Используйте образ postgres:latest, задайте переменные окружения
POSTGRES_USER, POSTGRES_PASSWORD и POSTGRES_DB для
установки пользовательского имени, пароля и имени базы данных
соответственно.
4. Определите сервис для веб-сервера на основе образа NGINX. Назовите
его ""web"". Используйте образ nginx:latest. Определите порт, на котором
должен работать сервер, с помощью инструкции ports. Задайте путь к
файлам конфигурации NGINX внутри контейнера, используя
инструкцию volumes.
5.* Определите ссылку на сервис базы данных в сервисе веб-сервера.
Используйте инструкцию links.
6. Сохраните файл docker-compose.yml и запустите приложение с
помощью команды docker-compose up.
7. Проверьте, что приложение работает, перейдя в браузере на
localhost:80."

