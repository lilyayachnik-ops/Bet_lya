# Checkpoins

Загрузим себе образ `nginx:stable-alpine`, чтобы он был доступен локально:

```bash
docker pull nginx:stable-alpine
```

Покажем в терминале:

<img width="1226" height="331" alt="image" src="https://github.com/user-attachments/assets/a7aa9ec8-1b7a-4a2f-839b-aafef1863d4d" />

Добавим к загруженному образу новый тег `inno-dkr-12`:

```bash
docker tag nginx:stable-alpine nginx:inno-dkr-12
```

Покажем в терминале

<img width="855" height="23" alt="image" src="https://github.com/user-attachments/assets/d1b90a29-04d5-45c1-8793-a4386a00a18a" />

Выведим список образов на нашем хосте — оба образа должны быть с одним `ID`. Удалим образ `nginx:stable-alpine`:

```bash
docker image ls
docker rmi nginx:stable-alpine
```

Покажем в терминале:

<img width="1139" height="155" alt="image" src="https://github.com/user-attachments/assets/ae6bd846-bb0b-43d0-b1f3-610dacab9027" />

Выведим список образов на нашем хосте — наш переименованный образ все еще должен быть в списке:

```bash
docker image ls
```

Покажем в терминале:

<img width="1190" height="92" alt="image" src="https://github.com/user-attachments/assets/25d97d22-3542-4a17-9aec-e7cbbfc47e58" />

Еще раз выполним загрузку образа `nginx:stable-alpine`. Выведим список образов на вашем хосте и сохраните их в файл ( `docker images | tee /home/user/images.txt` ) — его `ID` остался прежним, хотя `Docker` и пришлось загрузить его, поскольку он не знает, что с таким `ID` у нас уже есть образ.
Удалим все имеющиеся образы `nginx` одной командой (любое удаление разрешено — через `xargs`, прямое указание тегов или другой метод). Выведим список образов на нашем хосте:

```bash
docker pull nginx:stable-alpine
docker image ls | tee ~/docker_3.12/images.txt
docker rmi $(docker image -aq)
docker image ls
```

Покажем в терминале:

<img width="1312" height="223" alt="image" src="https://github.com/user-attachments/assets/7dd8ce00-68f7-4360-946c-5609a343ff0d" />
<img width="1409" height="180" alt="image" src="https://github.com/user-attachments/assets/9911a404-a874-43eb-98b6-2419e538b53a" />

**Так как два образа имеют один и тот же IMAGE_ID**.

Чтобы их удалить,  воспользуемся командой `docker rmi -f $(docker images -aq)`. Покажем в терминале:

<img width="1269" height="182" alt="image" src="https://github.com/user-attachments/assets/879fa006-3a27-442f-9545-aca6af6bd0d0" />

Запустим контейнер со следующими параметрами: должен работать в фоне, образ — `nginx:stable-alpine`, имя — `inno-dkr-12`:

```bash
docker run -d --name inno-dkr-12 nginx:stable-alpine
```

Покажем в терминале:

<img width="1214" height="354" alt="image" src="https://github.com/user-attachments/assets/895e1d96-c4b9-4d86-a054-42a91573b8ba" />

Попробуем удалить образ `nginx:stable-alpine` без флагов: 

```bash
docker rmi nginx:stable-alpine
```

Покажем в терминале:

<img width="1509" height="48" alt="image" src="https://github.com/user-attachments/assets/2286e914-7ed1-43d1-badc-17eec3736f0b" />

Удалим его с флагом `--force`. Покажем  в терминале:

<img width="1167" height="50" alt="image" src="https://github.com/user-attachments/assets/ee31a4ec-8556-42cf-9cff-c5edb143dd51" />

Выведим список запущенных контейнеров, используя команду `docker ps`, — контейнер должен продолжать работать. Покажем в терминале:

<img width="1292" height="72" alt="image" src="https://github.com/user-attachments/assets/66efaec4-eaa2-4233-88d5-10a313d27986" />

Перезапустим контейнер, используя команду `docker restart inno-dkr-12`. Покажем в терминале:

<img width="1210" height="48" alt="image" src="https://github.com/user-attachments/assets/e7c3b07a-cc17-42cd-8040-907480989e71" />

Выведим список запущенных контейнеров, используя команду `docker ps`, — контейнер должен работать. Покажем в терминале:

<img width="1213" height="72" alt="image" src="https://github.com/user-attachments/assets/3b5202c6-f776-4fdd-8119-1a12d05fb16e" />

