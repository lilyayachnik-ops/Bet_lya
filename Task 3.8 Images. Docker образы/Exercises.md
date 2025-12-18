# Checkpoints

Выполним загрузку образа `nginx:stable-alpine` на свой локальный хост. Добавим к загруженному образу новый тег `inno-dkr-07` — имя репозитория должно оставаться таким же, поменяться только тег. Выведим список образов на нашем хосте:

```bash
# docker pull nginx:stable-alpine — скачивает образ nginx:stable-alpine из внешнего Docker-реестра 
$ docker pull nginx:stable-alpine
# docker tag — создаёт тег для nginx:inno-dkr-07, которая указывает на nginx:stable-alpine
$ docker tag nginx:stable-alpine nginx:inno-dkr-07
# docker image ls — выводит список образов, которые имеются на хосте
$ docker image ls
```

Покажем в терминале:

<img width="891" height="357" alt="image" src="https://github.com/user-attachments/assets/d3fc5b3f-d3b1-4a40-a72b-ca8554885911" />
<img width="939" height="225" alt="image" src="https://github.com/user-attachments/assets/ac2eed53-2333-49d7-bcf5-71f88e68deb3" />

Запустим контейнер со следующими параметрами: контейнер должен работать в фоне, образ, который должен быть получен в результате переименования образа `nginx:stable-alpine`:

```bash
docker run -d nginx:inno-dkr-07
docker ps
```

Покажем в терминале:

<img width="1325" height="111" alt="image" src="https://github.com/user-attachments/assets/f43d1308-6008-40ab-a807-52cc15e0ba60" />


