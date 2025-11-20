# Checkpoints

Создадим одну виртуальную машину `set_dom`. В настройках `VM` выберим тип подключения `Сетевой мост (Bridged adapter)`:

<img width="1441" height="653" alt="image" src="https://github.com/user-attachments/assets/8934fd38-ec3f-4bdd-9157-e95d1d394373" />

Запустим `VM` и подключимся к ней по `SSH`:

<img width="701" height="284" alt="image" src="https://github.com/user-attachments/assets/3f8ce6a5-be4f-475f-a3ef-90489a942b9a" />

Установим `Nginx` с помощью команды `sudp apt update && sudo apt install nginx`. Далее с помощью команды `ip a` узнаем `IP`-адерс нашей `VM`.
Создадим главную `HTML`-страницу, вписав в ней наш `IP`:

```bash
echo "My private IP is $(hostname -I)" | sudo tee /var/www/html/index.html
```

Перезагрузим `nginx` командой `sudo systemctl restart nginx`.

 Так как у нас нет публичного домена, мы "обманем" наш основной компьютер, чтобы он знал, куда обращаться.

 Так как у нас `OC Windows`, то откроем блокнот от имени администратора и откроем файл `C:\Windows\System32\drivers\etc\hosts`.

 Добавим в конец этого файла новую строку, связав `IP`-адрес нашей `VM` с выдуманным доменом:

 <img width="1417" height="55" alt="image" src="https://github.com/user-attachments/assets/d6a3025a-c9eb-46ef-87fd-07aabd4207b2" />
 <img width="419" height="28" alt="image" src="https://github.com/user-attachments/assets/870935fa-978d-4ac1-852b-438e3ff035f4" />

Теперь при вводе `my-secure-server.local` в браузере наш компьютер будет обращаться напрямую к нашей `VM`.

В терминале нашей `VM` создадим директории для сертификата:

```bash
sudo mkdir -p /etc/ssl/private
# Для владельца r,w,x
sudo chmod 700 /etc/ssl/private
```
<img width="533" height="139" alt="image" src="https://github.com/user-attachments/assets/907bdccb-e62a-466e-a72d-2696fe342f51" />


Дальше сгенерируем ключ и сертификат одной командой с помощью `OpenSSL`:

```bash
# openssl — утилита для работы с SSL/TLS сертификатами
# req — команда для создания запросов на сертификаты
# -x509 — создание самоподписанного сертификата
# -nodes — не шифровать приватный ключ паролем
# -days 365 — срок действия сертифката
# -newkey rsa:2048 — создать новый ключ RSA длиной 2048 бит
# -keyout /etc/ssl/private/nginx-selfsigned.key — путь для сохранения приватного ключа
# -out /etc/ssl/certs/nginx-selfsigned.crt — путь для сохранения сертификата
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/ssl/private/nginx-selfsigned.key -out /etc/ssl/certs/nginx-selfsigned.crt
```

Программа задаст нам несколько вопросов. Большинство можно оставить по умолчанию, но обязательно заполнить поле `Common Name (e.g. server FQDN or YOUR name)`. Введём сюда домен, который мы придумали ранее: `my-secure-server.local`.

Настроим `Nginx` для работы с `HTTPS`. Для этого создадим новый файл конфигурации для нашего сайта на `VM`:

```bash
sudo vim /etc/nginx/sites-available/my-secure-server
```
<img width="584" height="103" alt="image" src="https://github.com/user-attachments/assets/5a86668c-80da-4744-b189-dd05fe8f2b51" />


Напишем следующее:

```bash
# Блок для HTTP, который делает редирект на HTTPS
server {
        listen 80;
        server_name my-secure-server.local;
        return 301 https://$host$request_uri;
}
# Основной блок для HTTPS
server {
        listen 443 ssl;
        server_name my-secure-server.local;
        # Указываем путь к нашему самоподписанному сертификату и ключу
        ssl_certificate /etc/ssl/certs/nginx-selfsigned.crt;
        ssl_certificate_key /etc/ssl/private/nginx-selfsigned.key;
        root /var/www/html;
        index index.html;
        location / {
        try_files $uri $uri/ =404;
        }
}
```

Создадим символическую ссылку на этот конфиг и перезапустим `Nginx`:

```bash
# Создание символической ссылки
sudo ln -s /etc/nginx/sites-available/my-secure-server /etc/nginx/sites-enabled/
# Проверяем конфиг на ошибки
sudo nginx -t
# Перезапускаем nginx
sudo systemctl restart nginx
```

Убедимся, что `/etc/nginx/sites-enabled/` является символической ссылкой:

```bash
ls -l /etc/nginx/sites-enabled/
```
<img width="832" height="78" alt="image" src="https://github.com/user-attachments/assets/2cc7a29b-73a4-48b6-95a8-da230b71ac41" />

Проверим себя. Для этого введем в браузере `https://IP_addr`:

<img width="958" height="142" alt="image" src="https://github.com/user-attachments/assets/242a5163-a9e0-4a76-a40d-7545f180c096" />

Отредактируем наш файл `/etc/nginx/sites-available/my-secure-server` на `VM`, добавив в него новый `location` для редиректа:

```bash
# Блок для HTTP, который делает редирект на HTTPS
server {
        listen 80;
        server_name my-secure-server.local;
        return 301 https://$host$request_uri;
}
# Основной блок для HTTPS
server {
        listen 443 ssl;
        server_name my-secure-server.local;
        # Указываем путь к нашему самоподписанному сертификату и ключу
        ssl_certificate /etc/ssl/certs/nginx-selfsigned.crt;
        ssl_certificate_key /etc/ssl/private/nginx-selfsigned.key;
        root /var/www/html;
        index index.html;
        location / {
        try_files $uri $uri/ =404;
        }
        location = /picture {
           return 302 /picture.html;
        }
}
```

Давайте создадим страницу и загрузите картинку с `VM`:

```bash
# Создаем HTML-файл с изображением
echo '<img src="/images/my_cat.jpg">' | sudo tee /var/www/html/picture.html
# Создаем каталог, куда скачаем наше изображение
sudo mkdir -p /var/www/html/images
# Скачаем наше изображение с помощью утилиты curl, где -o — опция сохранения результата в файл
sudo curl -o /var/www/html/images/my_cat.jpg https://http.cat/200
```

Проверим в терминале:

<img width="753" height="325" alt="image" src="https://github.com/user-attachments/assets/2cb5ced3-cd1b-4c03-967a-628169704a8b" />

На главной странице `index.html` добавим ссылку:

```bash
echo "My private IP is $(hostname -I) <br> <a href="/picture">Show Picture</a>" | sudo tee /var/www/html/index.html
```

Перезапустим `nginx`.

Проверим:
<img width="955" height="134" alt="image" src="https://github.com/user-attachments/assets/93b81e74-b8f6-480f-a172-19a04bbc8a06" />
Нажмем на `Show picture`:
<img width="950" height="823" alt="image" src="https://github.com/user-attachments/assets/e44b1d4f-6095-4345-b2a2-b0a95375795e" />

Давайте добавим ещё один редирект:

```bash
# Блок для HTTP, который делает редирект на HTTPS
server {
        listen 80;
        server_name my-secure-server.local;
        return 301 https://$host$request_uri;
}
# Основной блок для HTTPS
server {
        listen 443 ssl;
        server_name my-secure-server.local;
        # Указываем путь к нашему самоподписанному сертификату и ключу
        ssl_certificate /etc/ssl/certs/nginx-selfsigned.crt;
        ssl_certificate_key /etc/ssl/private/nginx-selfsigned.key;
        root /var/www/html;
        index index.html;
        location / {
        try_files $uri $uri/ =404;
        }
        location = /picture {
           return 302 /picture.html;
        }
# alias заменяет часть URL, соответствующую location, на указанный путь
# /download/ -> заменяется на /var/www/html/
# Оставшаяся часть URL добавляется к alias-пути
        location /download/ {
                alias /var/www/html/;
        }
}
```

На главной странице `index.html` добавим ссылку:

```bash
# tee -a — записывает в файл с добавлением, не перезаписывает
echo "... <a href="/download/music.mp3">Download Music</a>" | sudo tee -a /var/www/html/index.html
```

Загрузим `MP3` файл:

```bash
sudo curl -o /var/www/html/music.mp3 https://www.soundhelix.com/examples/mp3/SoundHelix-Song-1.mp3
```

Перезапустим `nginx`. И проверим в браузере:

<img width="957" height="184" alt="image" src="https://github.com/user-attachments/assets/90e11cd2-5020-4a10-b648-b02d326ca50e" />

Перейдем в `Download Music`:

<img width="961" height="721" alt="image" src="https://github.com/user-attachments/assets/59aea9de-1be3-4337-bf2a-fbf702955225" />

Добавим регулярные выражения для картинок и поворот `JPG`:

```bash
# Блок для HTTP, который делает редирект на HTTPS
server {
        listen 80;
        server_name my-secure-server.local;
        return 301 https://$host$request_uri;
}
# Основной блок для HTTPS
server {
        listen 443 ssl;
        server_name my-secure-server.local;
        # Указываем путь к нашему самоподписанному сертификату и ключу
        ssl_certificate /etc/ssl/certs/nginx-selfsigned.crt;
        ssl_certificate_key /etc/ssl/private/nginx-selfsigned.key;
        root /var/www/html;
        index index.html;
        location / {
        try_files $uri $uri/ =404;
        }
        location = /picture {
           return 302 /picture.html;
        }
        location /download/ {
                alias /var/www/html/;
        }
        # Обработка картинок
        # ~* — регистронезависимое выражение
        # \.(png|gif|ico|css|js) — регулярное выражение, которое обрабатывает файлы с расширениями, указанными в скобках
        location ~* \.(png|gif|ico|css|js)$ {
                root /var/www/html;
        # Браузер будет кэшировать файлы на 1 день 
                expires 1d;
        }
        # Обработка JPG с поворотом на 180 градусов
        location ~* \.jpg {
        root /var/www/html;
        image_filter rotate 180;
        }
}
```

**Важно**: для поворота картинок нужно установить `sudo apt install libnginx-mod-http-image-filter`.

Перезагрузим `nginx`. Проверим в браузере:

<img width="950" height="826" alt="image" src="https://github.com/user-attachments/assets/366999c7-db70-4e09-80d9-a800210b80c8" />

Картинка повернулась.
