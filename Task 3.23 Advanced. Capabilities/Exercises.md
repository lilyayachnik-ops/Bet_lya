# Checkpoints

Запустим контейнер со следующими параметрами: 

- должен работать в фоне
- имеет имя `inno-dkr-24`
- образ - `nginx:stable-alpine`
- убраны все `capabilities`,
- добавлены только необходимые `capabilities` для работы контейнера (в зависимости от пути решения у тебя будет `2-4 capabilites`):

```bash
# --cap-drop=ALL — удаляет все Linux capabilities у контейнера
# --cap-add=CAP_NET_BIND_SERVICE — позволяет процессу привязываться (bind) к TCP/UDP портам ниже 1024
# --cap-add=CAP_CHOWN — позволяет менять владельца (owner) и группу (group) файлов и директорий.
# --cap-add=CAP_SETGID — позволяет процессу изменять свою группу (Group ID).
# --cap-add=CAP_SETUID — позволяет процессу изменять своего пользователя (User ID).

$ docker run -d --name inno-dkr-24 --cap-drop=ALL --cap-add=CAP_NET_BIND_SERVICE --cap-add=CAP_CHOWN --cap-add=CAP_SETGID --cap-add=CAP_SETUID -p 80:80 nginx:stable-alpine
```

Покажем в терминале:

<img width="1600" height="71" alt="image" src="https://github.com/user-attachments/assets/ad358139-05e7-4ae8-8454-967a06520885" />

Выведим список контейнеров, чтобы показать, что требуемый контейнер работает:

```bash
docker ps
```

Покажем в терминале:

<img width="1594" height="70" alt="image" src="https://github.com/user-attachments/assets/eadb8496-2e52-492d-ac77-3e5481b5ecd5" />

Выведим информацию о контейнере через `docker inspect`:

```bash
docker inspect inno-dkr-24
```

Покажем в терминале:

<img width="1597" height="29" alt="image" src="https://github.com/user-attachments/assets/15ebc1cb-3fef-4dad-8120-615e40bea327" />
<img width="1535" height="203" alt="image" src="https://github.com/user-attachments/assets/13552656-e2b1-4bb7-97b5-d9c8f003f2a0" />
