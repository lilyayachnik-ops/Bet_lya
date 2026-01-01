# Checkpoints

Запустим `Docker` в `Docker`, используя первый метод (`dind`). Подключимся к контейнеру c `Docker и выполним внутри него следующие действия:

- получим список текущих образов и всех контейнеров;
- скачаем образ `redis:5`;
- создадим из загруженного образа контейнер с именем `inno-dkr-26`, работающий в фоне;
- ещё раз выведим список всех контейнеров;
- удалим контейнер;
- ещё раз выведи список всех контейнеров.

```bash
docker run --privileged -d docker:dind
docker ps
docker exec -it busy_shirley sh
docker images
docker ps -a
docker pull redis:5
docker run -d --name inno-dkr-26 redis:5
docker ps -a
docker rm -f inno-dkr-26
docker ps -a
```  

Покажем в терминале:

<img width="1397" height="817" alt="image" src="https://github.com/user-attachments/assets/91b331ad-087b-497f-8260-fc92cacd2966" />

Запусти `Docker` в `Docker`, используя второй метод (проброс сокета). Повторим те же самые действия из предыдущего пункта:

- получим список текущих образов и всех контейнеров;
- скачаем образ `redis:5`;
- создадим из загруженного образа контейнер с именем `inno-dkr-26`, работающий в фоне;
- ещё раз выведим список всех контейнеров;
- удалим контейнер;
- ещё раз выведи список всех контейнеров.

```bash
docker run -it -v /var/run/docker.sock:/var/run/docker.sock docker:cli sh
docker images
docker ps -a
docker pull redis:5
docker run -d --name inno-dkr-26 redis:5
docker ps -a
docker rm -f inno-dkr-26
docker ps -a
```

Покажем в терминале:

<img width="1547" height="755" alt="image" src="https://github.com/user-attachments/assets/cf8bdac5-6e3e-4cb5-91ef-7ddfa6934019" />
<img width="1587" height="861" alt="image" src="https://github.com/user-attachments/assets/8942163a-96ed-4d41-bb9a-b459f35e5734" />
