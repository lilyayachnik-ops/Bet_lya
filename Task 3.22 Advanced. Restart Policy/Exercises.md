# Checkpoints

Запустим `4` контейнера для каждой политики рестарта, со следующими параметрами: должно работать в фоне, имеет имя по шаблону `inno-dkr-23-$RESTART_POLICY`, образ — `nginx:stable-alpine`:

```bash
docker run -d --name inno-dkr-23-no --restart no nginx:stable-alpine
docker run -d --name inno-dkr-23-on-failure --restart on-failure nginx:stable-alpine
docker run -d --name inno-dkr-23-always --restart always nginx:stable-alpine
docker run -d --name inno-dkr-23-unless-stopped --restart unless-stopped  nginx:stable-alpine
```

Покажем в термминале:

<img width="1215" height="184" alt="image" src="https://github.com/user-attachments/assets/9c53a688-4d6f-42ab-981c-070a63c16d9d" />

По каждому из запущенных контейнеров вызовим `docker kill -s 15 $CONTAINER_NAME`:

```bash
docker kill -s 15 inno-dkr-23-no
docker kill -s 15 inno-dkr-23-on-failure
docker kill -s 15 inno-dkr-23-always
docker kill -s 15 inno-dkr-23-unless-stopped
```

Покажем в терминале:

<img width="653" height="184" alt="image" src="https://github.com/user-attachments/assets/7ee4b398-6c32-4ed0-a090-f19c136b0889" />

Выведим список контейнеров с дублированием вывода в файл `kill_15.txt` (`docker ps -a | tee /home/user/kill_15.txt`):

```bash
docker ps -a | tee /home/user/kill_15.txt
ls -la | grep kill_15.txt
```

Покажем в терминале:

<img width="1337" height="178" alt="image" src="https://github.com/user-attachments/assets/a392ddc5-5f98-4509-8c77-fd919f646a13" />

Запустим неактивные контейнеры:

```bash
docker start inno-dkr-23-on-failure inno-dkr-23-no inno-dkr-23-unless-stopped
```

Покажем в терминале:

<img width="1195" height="93" alt="image" src="https://github.com/user-attachments/assets/539bf529-4c25-4179-80cd-fbf5d3968bf4" />

Выведим список контейнеров:

```bash
docker ps
```

Покажем в терминале:

<img width="1372" height="137" alt="image" src="https://github.com/user-attachments/assets/11469480-89c4-497d-9480-432c134b36cb" />

По каждому из запущенных контейнеров вызовим `docker kill $CONTAINER_NAME`:

```bash
docker kill inno-dkr-23-no
docker kill inno-dkr-23-on-failure
docker kill inno-dkr-23-always
docker kill inno-dkr-23-unless-stopped
```

Покажем в терминале:

<img width="1013" height="177" alt="image" src="https://github.com/user-attachments/assets/4d0410a3-6fbb-4737-b6ea-04bb3ff9a716" />

Выведим список контейнеров с дублированием вывода в файл `kill.txt` (`docker ps -a | tee /home/user/kill.txt`):

```bash
docker ps -a | tee ~/kill.txt
```

Покажем в терминале:

<img width="1431" height="136" alt="image" src="https://github.com/user-attachments/assets/6a17738c-1baa-4401-84d5-10204dbce7ef" />

Перезапустим `Docker` через `Docker Desktop`

<img width="1551" height="777" alt="image" src="https://github.com/user-attachments/assets/30f50086-fda3-4a3f-ac83-e95efd6ca04a" />

Выведим список контейнеров:

```bash
docker ps 
```

Покажем в терминале:

<img width="1348" height="91" alt="image" src="https://github.com/user-attachments/assets/aa4d70be-464c-4a59-9619-9181b683394d" />
