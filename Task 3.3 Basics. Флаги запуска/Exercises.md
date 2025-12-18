# Checkpoints

Скачаем конфигурационный файл `nginx`. Запустим контейнер со следующими параметрами: должен работать в фоне, слушает на хосте `127.0.0.1:8889`, имеет имя `inno-dkr-02`, должен пробрасывать скачанный нами конфигурационный файл `nginx` внутрь контейнера как основной конфигурационный файл, образ — `nginx:stable`:

```bash
vim nginx.conf

Файл nginx.conf будет содержать следующее:

events {
    worker_connections 1024;
}

http {
    server {
        listen 80;
        server_name localhost;
        
        location / {
            return 200 'Welcome to the training program Innowise: Docker';
            add_header Content-Type text/plain;
        }
    }
}
```

Далее:

```bash
# -v — монтирование файла nginx.conf, где $(pwd)/nginx.conf — это файл, который мы монтируем, а /etc/nginx/nginx.conf — это путь, куда мы монтируем файл

$ docker run -d -p 127.0.0.1:8889:80 --name inno-dkr-02 -v "$(pwd)/nginx.conf:/etc/nginx/nginx.conf" nginx:stable
```

Покажем в терминале:

<img width="1295" height="385" alt="image" src="https://github.com/user-attachments/assets/8bc2a2b1-fd24-41af-8004-047522be2e01" />
<img width="1461" height="72" alt="image" src="https://github.com/user-attachments/assets/73a9ac9c-8459-4de9-8f88-51f7b523950f" />


Проверим работу, обратившись к `127.0.0.1:8889`, — в ответ должно возвращать строку `Welcome to the training program Innowise: Docker`. Выведим в консоли список запущенных контейнеров — контейнер должен быть запущен. Выполним подсчет `md5sum` конфигурационного файла `nginx` командой: `docker exec -ti inno-dkr-02 md5sum /etc/nginx/nginx.conf`:

```bash
curl http://127.0.0.1:8889
docker ps

# docker exec — выполняет новую команду в запущенном контейнере,
-i — сохраняет STDIN открытым,
-t — выделяет псевдотерминал,
md5sum — утилита для вычисления MD5 хеша файла,
/etc/nginx/nginx.conf — путь к файлу внутри контейнера

$ docker exec -ti inno-dkr-02 md5sum /etc/nginx/nginx.conf
```

Покажем в терминале:

<img width="800" height="43" alt="image" src="https://github.com/user-attachments/assets/48fd1f9f-2a99-4e79-af14-ce30a641cabb" />
<img width="1594" height="73" alt="image" src="https://github.com/user-attachments/assets/65c7267f-baa8-497d-90f4-48e8e57f6131" />
<img width="1591" height="55" alt="image" src="https://github.com/user-attachments/assets/496c2e74-6417-4314-a9c1-427f297cca46" />




