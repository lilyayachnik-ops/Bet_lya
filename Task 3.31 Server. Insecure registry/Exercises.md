# Checkpoints

Запустим контейнер с `Docker Registry`:

```bash
docker run -d -p 5000:5000 --name registry registry:3
```

Покажем в терминале:

<img width="1203" height="286" alt="image" src="https://github.com/user-attachments/assets/ff3ed5b4-323a-40fb-aed8-e14b219cd08e" />

Скачаем образ `nginx:stable`, переименуем образ для локального `registry` (ввиду того, что `localhost` захардкожен в `Docker` как разрешенный, адрес `registry` необходимо указать в виде `ip` адреса отличного от `127.0.0.1`):

```bash
docker pull nginx:stable
hostname -I
docker tag nginx:stable 172.30.151.111:5000/my-nginx:stable
```

Покажем в терминале:

<img width="1122" height="378" alt="image" src="https://github.com/user-attachments/assets/ea492411-7748-4c9a-b574-3efda249b1a5" />

Выведим список образов на твоем хосте:

```bash
docker images
```

Покажем в терминале:

<img width="1189" height="423" alt="image" src="https://github.com/user-attachments/assets/eed32817-eb78-48d1-b18c-0f72e3097b0f" />

Попробуем загрузить образ в `registry`:

```bash
docker push 172.30.151.111:5000/my-nginx:stable
```

Покажем в терминале:

<img width="1638" height="243" alt="image" src="https://github.com/user-attachments/assets/1467d3e6-9f78-4477-a58f-ca4db8a16a66" />

Разрешим `Docker` использовать небезопасные `registry` с помощью настройки `daemon.json`, добавит следующую строку `"insecure-registries": ["172.30.151.111:5000"]`, в настройках `Docker Desktop`. Покажем:

<img width="1157" height="408" alt="image" src="https://github.com/user-attachments/assets/b1723ea6-9cc8-4686-94dc-13f12f58fb83" />

Нажимаем `Apply and Restart`. Попробуем еще раз загрузить образ в `registry`:

```bash
docker start registry
docker push 172.30.151.111:5000/my-nginx:stable
```

Покажем в терминале:

<img width="1630" height="330" alt="image" src="https://github.com/user-attachments/assets/56aeb0d5-cd21-43d1-8e0c-e500cd8d72e2" />

Удалим наш новый образ с хоста:

```bash
docker rmi 172.30.151.111:5000/my-nginx:stable
```

Покажем в терминале:

<img width="990" height="46" alt="image" src="https://github.com/user-attachments/assets/28140957-8783-4a9a-b016-621209402aaa" />

Выведим список образов на нашем хосте:

```bash
docker images
```

Покажем в терминале:

<img width="1203" height="333" alt="image" src="https://github.com/user-attachments/assets/e4d0a4a4-77cf-44ca-b749-dafc172fad4f" />

Скачаем наш образ из `registry`:

```bash
docker pull 172.30.151.111:5000/my-nginx:stable
```

Покажем в терминале:

<img width="1186" height="119" alt="image" src="https://github.com/user-attachments/assets/a36d4f7e-8480-4c0e-84b1-416adee59c71" />

Выведим список образов на нашем хосте:

```bash
docker images
```

Покажем в терминале:

<img width="1407" height="361" alt="image" src="https://github.com/user-attachments/assets/e3f06a12-0819-438a-a645-438478d143a8" />

