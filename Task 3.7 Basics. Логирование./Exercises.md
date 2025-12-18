# Checkpoints 

Запустим контейнер со следующими параметрами: должен работать в фоне, слушает на хосте `127.0.0.1:8892`, имеет имя `inno-dkr-06-local`, образ — `nginx:stable`, логи должны использовать драйвер `local` и объем файла лога не должен превышать `10 MiB`:

```bash
# --log-driver local, где --log-driver указывает Docker, как именно сохранять логи контейнера, local — это улучшенная версия json-file.
# --log-opt max-size=10m, где --log-opt уточняет работу драйвера логирования, max-size=10m --log-opt max-size=10m — устанавливает максимальный размер одного лог-файа

$ docker run -d -p 127.0.0.1:8892:80 --name inno-dkr-06-local --log-driver local --log-opt max-size=10m nginx:stable
```

Покажем в терминале:

<img width="1475" height="50" alt="image" src="https://github.com/user-attachments/assets/1153fadb-f555-48d1-8657-998c54beeddb" />


Выведим список запущенных контейнеров. Один раз обратимся к запущенному `nginx`, чтобы были записаны логи (например, `curl --silent http://127.0.0.1:8892 > /dev/null`). Выполним вывод содержимого файла на хостовой системе, в который записаны логи контейнера. Настроим глобальное сохранение логов с драйвером `local` и объемом логов в `10 MiB` через файл `/etc/docker/daemon.json`:

```bash
docker ps

# --silent — это опция подавляет весь вспомогательный вывод curl, без этой опции при ошибки соединения она выведтся в терминал, с этим параметром ничего не выведется, но команда вернет не нулевой код выхода, > /dev/null — перенаправляем вывод в специальное устройство, которое отбрасывает всё, что в него записывают, так называемый битоприёмник

$ curl --silent http://127.0.0.1:8892 > /dev/null
docker logs inno-dkr-06-local
docker inspect --format='{{.HostConfig.LogConfig.Type}}' inno-dkr-06-local
```
Покажем в терминале:

<img width="1536" height="92" alt="image" src="https://github.com/user-attachments/assets/424b9715-e600-4ed2-a787-5d3b65b0f6e4" />
<img width="1256" height="714" alt="image" src="https://github.com/user-attachments/assets/a0f8f3d2-492d-469a-b9c7-97cade5c1af3" />
<img width="1079" height="46" alt="image" src="https://github.com/user-attachments/assets/0037b8ad-da4f-4749-927d-7407e8f13cfe" />


Настроим глобальное сохранение логов с драйвером `local` и объёмом логов в `10Mib". Так как у меня `Ubuntu` крутится на `WSL2`, то я использую `Docker Desktop`. Переходим в раздел `Docker Engine`, видим конфигурационный файл:

<img width="1130" height="612" alt="image" src="https://github.com/user-attachments/assets/1d6f3412-e33a-49e2-839c-70b3f7e9956e" />

Допишем следующие строчки и сохраним:

```bash
 "log-driver": "local",
  "log-opts": {
    "max-size": "10m"
   }
```

Запустим контейнер со следующими параметрами: должно работать в фоне, слушает на хосте `127.0.0.1:8893`, имеет имя `inno-dkr-06-global`, образ — `nginx:stable`, в команде запуска НЕ должны присутствовать параметры драйвера:

```bash
docker run -d -p 127.0.0.1:8893:80 --name inno-dkr-06-global nginx:stable
```

Покажем в терминале:

<img width="1152" height="48" alt="image" src="https://github.com/user-attachments/assets/1176504b-4808-4102-b9e6-1b97e7885e35" />

Выведим список запущенных контейнеров. Один раз обратимся к запущенному `nginx`, чтобы были записаны логи (например, `curl --silent http://127.0.0.1:8893 > /dev/null`). Выполним вывод содержимого файла на хостовой системе, в который записаны логи контейнера:

```bash
docker ps
curl --silent http://127.0.0.1:8893 > /dev/null
docker logs inno-dkr-06-global
docker inspect --format='{{.HostConfig.LogConfig.Type}}' inno-dkr-06-global
```

Покажем в терминале:

<img width="1357" height="133" alt="image" src="https://github.com/user-attachments/assets/63ace89b-68ef-4a66-b8ca-07168260e3ee" />
<img width="1186" height="707" alt="image" src="https://github.com/user-attachments/assets/3f97804b-724e-420b-b2cc-09622f5e1bc0" />
<img width="1113" height="45" alt="image" src="https://github.com/user-attachments/assets/092c8865-a8f9-4a33-8a6c-fff4558ea9d5" />





