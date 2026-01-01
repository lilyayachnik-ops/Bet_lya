# Checkpoints

Создадим сеть с драйвером `bridge` и именем `inno-25-bridge`:

```bash
# docker network create — создаёт сеть Docker
# -d bridge — указывает драйвер сети. Драйвера: host, none, overlay, macvlan
# inno-25-bridge — имя сети

$ docker network create -d bridge inno-25-bridge
```

Покажем в терминале:

<img width="1191" height="43" alt="image" src="https://github.com/user-attachments/assets/5d8e160b-45fe-407d-a005-e261f7df079b" />

Выведим список всех сетей:

```bash
docker network ls 
```

Покажем в терминале:

<img width="1344" height="135" alt="image" src="https://github.com/user-attachments/assets/e85515f8-b0b5-4f8b-863a-69de56b19ed0" />

Создадим из образа `nginx:stable` контейнер с именем `inno-dkr-25-nginx`, работающий в фоне и подключенный к новой сети:

```bash
# --network inno-25-bridge — подключает контейнер к сети с именем inno-dkr-25-bridge

$ docker run -d --network inno-25-bridge --name inno-dkr-25-nginx nginx:stable
```

Покажем в терминале:

<img width="1435" height="330" alt="image" src="https://github.com/user-attachments/assets/fe71a2eb-1886-4034-bb94-b7c58dfd0a7e" />

Запустим второй контейнер с именем `inno-dkr-25-pinger` в интерактивном режиме из образа `alpine:3.10` и подключенный к новой сети, установим в нем `curl` и обратимся к контейнеру `inno-dkr-25-nginx` по `DNS`-записи:

```bash
docker run -it --rm --network inno-25-bridge --name inno-dkr-25-pinger alpine:3.10
apk update
apk add curl
curl inno-dkr-25-nginx
```

Покажем в терминале:

<img width="1615" height="857" alt="image" src="https://github.com/user-attachments/assets/bc4c6763-1bd4-4346-bc60-dd807e5f0e07" />
<img width="1171" height="176" alt="image" src="https://github.com/user-attachments/assets/b29dc7b4-9aee-41f2-bb46-954f5990eb8a" />
