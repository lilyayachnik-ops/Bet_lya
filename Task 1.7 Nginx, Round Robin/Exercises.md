<img width="1077" height="526" alt="image" src="https://github.com/user-attachments/assets/53737eee-4041-48c3-8bdd-f1e7933e5821" /># Checkpoints

Создали три `VM` с названиями `Blue-server`, `Yellow-server`, `balancer-server`.

<img width="965" height="642" alt="image" src="https://github.com/user-attachments/assets/ce7d4db2-3ae1-43d7-be2a-debd80112467" />

Для всех трёх `VM` в настройках VirtualBox выбрали тип сетевого подключения `Сетевой мост (Bridged Adapter)`

<img width="1112" height="623" alt="image" src="https://github.com/user-attachments/assets/44c788eb-126f-4a60-961f-ef39b9fb7e9a" />

Мы хотим настроить первые две `VM`: `Blue-server` и `Yellow-server`, так, чтобы они отдавали статические `HTML`-страницы разного цвета.

Перейдем к настройке `Blue-server`. Создадим `HTML`-файл:

```bash
echo '<body style="background=color: blue;"><h1>Blue Server</h1></body>' | sudo tee /var/www/html/blue.html
```
<img width="546" height="53" alt="image" src="https://github.com/user-attachments/assets/b504f7dd-498d-4eaf-87c8-8f6a720705cc" />


Далее отредактируем стандартный конфигурационный файл `Nginx`, использовав команду `sudo vim /etc/nginx/sites-available/default`:

```bash
server {
         listen 80;
         location / {
         root /var/www/www/html;
         index blue.html;
         }
}
```

Таким образом, мы обрабатываем все запросы с `/` и отдаче файла `blue.html`.

Так как мы внесли изменения в конфигурационный файл, необходимо перезапустить `Nginx`:

```bash
sudo systemctl restart nginx
```

Также узнаем `IP`-адрес нашей машины:

```bash
ip a 
```

Перейдем к настройке `Yellow-server`. Создадим `HTML`-файл:

```bash
echo '<body style="background-color: yellow;"><h1>Yellow Server</h1></body> | sudo tee /var/www/html/yellow.html'
```
<img width="613" height="57" alt="image" src="https://github.com/user-attachments/assets/d3f1aa87-e775-4cc9-8dbd-3841677f30f1" />

Далее отредактируем стандартный конфигурационный файл `Nginx`, использовав команду `sudo vim /etc/nginx/sites-available/default`:

```bash
server {
         listen 80;
         location / {
         root /var/www/www/html;
         index yellow.html;
         }
}
```

Так как мы внесли изменения в конфигурационный файл, необходимо перезапустить `Nginx`:

```bash
sudo systemctl restart nginx
```

Также узнаем `IP`-адрес нашей машины:

```bash
ip a 
```

Теперь настроим нашу третью `VM` с названием `balancer-server`, которая будет распределять запросы между первыми двумя `VM`.
Перейдем к настройке конфигурационного файла `vim /etc/nginx/sites-available/default`:

```bash
# Настройка специального формата логов

log_format balanced '$remote_addr - $remote_user [$time_local] `
                    '"$request" $status $body_bytes_sent '
                    '"$http_referer" "$http_user_agent" "$gzip_ratio" '
                    'uct=$upstream_connect_time'
                    'urt=$upstream_response_time';

# Настройка группы серверов. Объявляем группу серверов с именем backend

upstream backend {
                 server 192.168.0.124;
                 server 192.168.0.125;
         }

         server {
                 listen 80;
# Указываем путь к лог-файлу
                 access_log /var/log/nginx/access.log balanced;
# Настройка основного сервера-прокси
                 location / {
                         proxy_pass http://backend;
                         }
         }
   ```
Не забудем перезапустить `sudo systemctl restart nginx`. Перейдём в браузер и введем `http://ip_addr_balancer`:

<img width="966" height="987" alt="image" src="https://github.com/user-attachments/assets/582ea10a-70bb-41a1-876e-4f1f4edc07ea" />

Обновим страницу:

<img width="958" height="991" alt="image" src="https://github.com/user-attachments/assets/5fcfa4e0-f7c1-4e84-ae16-65462a65c7db" />

Как видим, нас перекидывает между двумя страницами. Это алгоритм `Round Robin`. 
Давайте подключимся по `SSH` к нашему `balancer-server` и посмотрим логи, чтобы убедиться, что запросы распределяются:

```bash
ssh balancer-server@ip_addr_balancer
tail -f /var/log/nginx/access.log
```

Проверим в терминале:

<img width="1083" height="599" alt="image" src="https://github.com/user-attachments/assets/56557a02-b633-4b69-b5ad-659f45d9a9e1" />
<img width="1077" height="526" alt="image" src="https://github.com/user-attachments/assets/2aef2f02-ecc6-4ced-917b-4851e8906116" />

Добавим `Backup-server`. Чтобы не создавать четвертую `VM`, мы сымитируем его на `balancer-server`.

Создадим `HTML`-файл для бэкапа:

```bash
echo '<body style="background-color: grey;"><h1> Backup Server</h1></body>' | sudo tee /var/www/html/grey.html
```
<img width="691" height="47" alt="image" src="https://github.com/user-attachments/assets/8af233fb-9f9b-49e4-b9ff-a2baf8f86ffc" />

Дальше отредактируем наш файл `vim /etc/nginx/sites-available/default`:

```bash
# Настройка специального формата логов

log_format balanced '$remote_addr - $remote_user [$time_local] `
                    '"$request" $status $body_bytes_sent '
                    '"$http_referer" "$http_user_agent" "$gzip_ratio" '
                    'uct=$upstream_connect_time'
                    'urt=$upstream_response_time';

# Настройка группы серверов. Объявляем группу серверов с именем backend

upstream backend {
# Метод балансировки, при котором запрос отправляется на сервер с наименьшим количеством активных соединений
                 least_conn;
                 server 192.168.0.124;
                 server 192.168.0.125;
# Резервный сервер, используется когда оба основных недоступны. 127.0.0.1 - IP-адрес localhost. Слушает порт 8081
                 server 127.0.0.1:8081 backup;
         }

         server {
                 listen 80;
# Указываем путь к лог-файлу
                 access_log /var/log/nginx/access.log balanced;
# Настройка основного сервера-прокси
                 location / {
                         proxy_pass http://backend;
                         }
         }

         server {
                 listen 8081;
                 location / {
                          root /var/www/html;
                          index grey.html;
                 }
         } 
```

Давайте выключим наши `blue-server` и `yellow-server`, чтобы проверить работу `backup-server`:

<img width="954" height="977" alt="image" src="https://github.com/user-attachments/assets/0b74f8bb-57fa-4125-99e1-0f25c09b4b8c" />

Все работает.
