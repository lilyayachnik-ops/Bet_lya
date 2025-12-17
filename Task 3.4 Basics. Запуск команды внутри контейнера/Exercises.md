# Checkpoints

Скачай конфигурационный файл nginx. Запусти контейнер со следующими параметрами: должен работать в фоне, слушает на хосте `127.0.0.1:8890`,
имеет имя `inno-dkr-03`, должен пробрасывать скачанный вами конфигурационный файл внутрь контейнера как основной конфигурационный файл, образ — `nginx:stable`. Проверь работу, обратившись к `127.0.0.1:8890`, — в ответ должно возвращать строку `Welcome to the training program Innowise: Docker`. Выведи список запущенных контейнеров — контейнер должен быть запущен. Выполни команду `docker exec -ti inno-dkr-03 md5sum /etc/nginx/nginx.conf`. Скачай новый конфигурационный файл `nginx`. Измени проброшенный конфигурационный файл через `cp`, размещенный на хостовой системе, на новый. Выполни `reload nginx` без остановки контейнера при помощи команды `exec`. Проверь работу, обратившись к `127.0.0.1:8890`, — в ответ должно возвращать строку `Welcome to the training program Innowise: Docker! Again!`. Выполни команду `docker exec -ti inno-dkr-03 md5sum /etc/nginx/nginx.conf` (вывод в консоли сохраняем).:


```bash
cp ~/docker_3.2/nginx.conf ~/docker_3.3/
ls -la
cat nginx.conf
docker run -d -p 127.0.0.1:8890:80 --name inno-dkr-03 --mount type=bind,source="$(pwd)/nginx.conf",target=/etc/nginx/nginx.conf nginx:stable
curl http://127.0.0.1:8890
docker ps
docker exec -ti inno-dkr-03 md5sum /etc/nginx/nginx.conf
cp ~/docker_3.2/nginx-new.conf ~/docker_3.3/nginx.conf


# nginx -s reload, где -s — сигнал, reload — перезагрузить конфигурацию

$ docker exec inno-dkr-03 nginx -s reload
curl http://127.0.0.1:8890
docker exec -ti inno-dkr-03 md5sum /etc/nginx/nginx.conf
```

Покажем в терминале:

<img width="880" height="139" alt="image" src="https://github.com/user-attachments/assets/d5b799ee-63f3-43c5-9034-f766bdf4ca51" />
<img width="889" height="355" alt="image" src="https://github.com/user-attachments/assets/c134e9c9-0d8d-45bc-860e-0b2529c9a52b" />
<img width="1746" height="136" alt="image" src="https://github.com/user-attachments/assets/4f086b17-5ed7-4a56-987c-8c64df90ea41" />
<img width="1369" height="114" alt="image" src="https://github.com/user-attachments/assets/075cd5f0-54b3-494c-89de-948e599a8bc3" />
<img width="711" height="50" alt="image" src="https://github.com/user-attachments/assets/c119d0c2-55e3-4b54-8d54-58cb30c9bc18" />
<img width="1177" height="48" alt="image" src="https://github.com/user-attachments/assets/d360a711-58c8-4cca-9c8f-1b3cc175f36d" />


Проведи эксперименты с различными способами изменения: через `vim`, `nano`. Сравни результаты изменения файла `nginx.conf` разными командами. Найди информацию, почему в некоторых случаях изменение не отображалось в контейнере. Выведи список запущенных контейнеров — контейнер должен быть запущен.

Давайте через редактор `nano`:

```bash
nano nginx.conf
cat nginx.conf
docker exec inno-dkr-03 nginx -s reload
curl http://127.0.0.1:8890
docker exec -ti inno-dkr-03 md5sum /etc/nginx/nginx.conf
docker ps
```

Покажем в терминале:

<img width="1202" height="734" alt="image" src="https://github.com/user-attachments/assets/c0332ca9-c16b-4271-a6d8-a7bdd0002c75" />
<img width="847" height="92" alt="image" src="https://github.com/user-attachments/assets/6f4060e1-ebd4-4c21-8a3a-cda64d334ca7" />
<img width="1343" height="112" alt="image" src="https://github.com/user-attachments/assets/00ef8b23-8b43-41de-883f-bc4277608051" />

Давайте через редактор `vim`:

```bash
cat nginx.conf
vim nginx.conf
cat nginx.conf
docker exec inno-dkr-03 nginx -s reload
curl http://127.0.0.1:8890
docker exec -ti inno-dkr-03 md5sum /etc/nginx/nginx.conf
docker ps
```

Покажем в терминале:

<img width="1321" height="732" alt="image" src="https://github.com/user-attachments/assets/1114bc7f-69ef-4d33-aa75-68253843282b" />
<img width="873" height="86" alt="image" src="https://github.com/user-attachments/assets/f90a8171-39f4-43ff-868d-31791d008f5d" />
<img width="1629" height="72" alt="image" src="https://github.com/user-attachments/assets/b7a2dcc0-2909-458f-981f-aa9ae4775231" />
<img width="1379" height="115" alt="image" src="https://github.com/user-attachments/assets/c9eb2475-579a-40da-aeff-339284a918ef" />




