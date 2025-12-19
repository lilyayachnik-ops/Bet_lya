# Checkpoints

Создадим тестовый файл `testfile` размером `10МБ` при помощи команды:

```bash
# dd — копирует блоки данных из одного места в другое
# if=the_input_file
# of=the_output_file
# bs=the_size_of_block
# count=the_count_of_blocks

$ dd if=/dev/zero of=./testfile bs=1M count=10
```

Покажем в терминале:

<img width="862" height="89" alt="image" src="https://github.com/user-attachments/assets/7973d177-6355-44a0-893d-04a6b32debe6" />

Напишем следующий `dockerfile`:

```bash
FROM ubuntu:20.04
ENV testenv1=env1
# Create user
RUN groupadd --gid 2000 user && useradd --uid 2000 --gid 2000 --shell /bin/bash --create-home user
# Check the apt cache status before installing nginx
RUN ls -lah /var/lib/apt/lists/
RUN apt-get update && \
    apt-get install nginx -y
# Re=check the status of the apt cache
RUN ls -lah /var/lib/apt/lists
# Clearing the cache
RUN rm -rf /var/lib/apt/lists/*
RUN ls -lah /var/lib/apt/lists/
# Copy our test file
COPY testfile .
# Change the owner
RUN chown user:user testfile
USER user
CMD ["sleep infinity"]
```

Покажем в терминале:

<img width="1224" height="496" alt="image" src="https://github.com/user-attachments/assets/1f3da940-2e69-4ac1-84ca-7ece9f54f164" />


Соберём образ на основе указанного `dockerfile` с тэгом `inno-dkr-11:default` (если будут проблемы со скачиванием пакетов, нам может помочь флаг `--network=host`):

```bash
docker build -t inno-dkr-11:default .
```

Покажем в терминале:

<img width="1241" height="621" alt="image" src="https://github.com/user-attachments/assets/85ceda61-018e-4796-9fa8-b4924d5384f1" />

Используя команду `docker inspect inno-dkr-11:default`, внимательно изучим вывод команды, она позволяет получать дополнительную информацию об образе/контейнере:

```bash
docker inspect inno-dkr-11:default
```

Покажем в терминале:

<img width="1222" height="600" alt="image" src="https://github.com/user-attachments/assets/64deda79-2815-4f45-a1c2-10c96e3c4e69" />
<img width="1603" height="596" alt="image" src="https://github.com/user-attachments/assets/2dd36128-077d-4d05-a5b7-11f3549260cb" />

Используя команду `docker history inno-dkr-11:default --no-trunc`, изучим список слоёв в собранном контейнере. Обратим внимание: у всех ли слоёв есть размер ? Нет, не у всех слоёв есть размер:

```bash
# docker history — показывает историю слоёв конкретного образа Docker
# inno-dkr-11:default — имя образа и тег
# --no-trunc — флаг, который отключает обрезвание вывода
docker history inno-dkr-11:defautl --no-trunc
```

Покажем в терминале:

<img width="1641" height="755" alt="image" src="https://github.com/user-attachments/assets/46a102f2-fba9-483b-b798-c5fa82f4849b" />
<img width="1585" height="45" alt="image" src="https://github.com/user-attachments/assets/c14ba75d-8f0b-445b-91e1-d789134f0eaa" />

Изучим документацию и ответим на вопрос, какие директивы могут создавать слои (полезные ссылки "Best practices for writing Dockerfile", раздел "Minimize the number of layers" ):

**Важно**: 
- RUN — создаёт новый слой
- COPY — копирование файлов из хоста в контейнер
- ADD — аналогично COPY, но с дополнительными возможностями

Обратим внимание на директиву `RUN chown user:user testfile` — посмотрим, сколько она добавила в итоговый образ:

<img width="1387" height="50" alt="image" src="https://github.com/user-attachments/assets/24c4d40d-1ff6-43ac-853a-8f2532d808b1" />

Используя команду `docker images ls`, оценим размер получившегося образа:

```bash
docker image ls
```

Покажем в терминале:

<img width="1166" height="112" alt="image" src="https://github.com/user-attachments/assets/c3558da9-debd-45fe-b2e7-429346d3edf0" />

Внесём изменения в `dockerfile`: 

- заменим директиву `RUN apt-get update -y && apt-get install nginx -y` на `RUN apt-get update -y && apt-get install nginx -y && rm -rf /var/lib/apt/lists/*`

- используя дополнительные флаги директивы `COPY`, назначим пользователя `user` владельцем скопированных файлов

- уберём директиву `RUN chown user:user testfile`

Покажем содержимое `dockerfile`:

```bash
FROM ubuntu:20.04
ENV testenv1=env1
# Create user
RUN groupadd --gid 2000 user && useradd --uid 2000 --gid 2000 --shell /bin/bash --create-home user
# Check the apt cache status before installing nginx and clearing the cache
RUN ls -lah /var/lib/apt/lists/
RUN apt-get update && \
    apt-get install nginx -y && \
    rm -rf /var/lib/apt/lists/*
# Re=check the status of the apt cache
RUN ls -lah /var/lib/apt/lists/
# Copy our test file
COPY --chown=user:user testfile .
# Change the owner
USER user
CMD ["sleep infinity"]
```

Покажем в терминале:

<img width="1400" height="410" alt="image" src="https://github.com/user-attachments/assets/11d22062-c3b6-48a8-aa29-9d64cfe95023" />

Соберём образ с тэгом `inno-dkr-11:optimized`:

```bash
docker build -t inno-dkr-11:optimized .
```

Покажем в терминале:

<img width="1270" height="557" alt="image" src="https://github.com/user-attachments/assets/5342e643-a665-47dd-833c-afb9b56a0574" />

Используя команду `docker inspect inno-dkr-11:optimized` и `docker history inno-dkr-11:optimized --no-trunc`, изучим список слоёв в собранном контейнере:

```bash
docker inspect inno-dkr-11:optimized
docker history inno-dkr-11:optimized --no-trunc
```

Покажем в терминале:

<img width="1521" height="600" alt="image" src="https://github.com/user-attachments/assets/5c823600-6aed-41a6-87b6-af17bda88c8d" />
<img width="1452" height="534" alt="image" src="https://github.com/user-attachments/assets/9989de2c-01a2-4908-9d58-6748ef11bb1f" />
<img width="1314" height="356" alt="image" src="https://github.com/user-attachments/assets/2bc9121f-848c-474c-a47e-dd4b2021a669" />

Сравним количество и размер с `inno-dkr-11:default`: в образе `inno-dkr-11:default` — у нас `9` слоёв, а во втором образе `inno-dkr-11:optimized` — у нас `6` слоёв.
Используя команду `docker images ls`, посмотрим на размеры образов:

<img width="928" height="133" alt="image" src="https://github.com/user-attachments/assets/1cae4a55-a6bc-42f1-ad9f-7a7b82093f18" />

Образ `inno-dkr-11:optimized` меньше по размеру.
