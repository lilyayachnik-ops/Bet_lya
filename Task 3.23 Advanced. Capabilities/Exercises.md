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

**Важно**:

1) **USER**:
The default user within a container is root (`uid = 0`). You can set a default user to run the first process with the Dockerfile `USER` instruction. When starting a container, you can override the `USER` instruction by passing the `-u` option.

The `USER` instruction sets the user name (or `UID`) and optionally the user group (or `GID`) to use as the default user and group for the remainder of the current stage. The specified user is used for RUN instructions and at runtime, runs the relevant ENTRYPOINT and CMD commands.

2) **--privileged**:
The --privileged flag gives all capabilities to the container. When the operator executes docker run --privileged, Docker enables access to all devices on the host, and reconfigures AppArmor or SELinux to allow the container nearly all the same access to the host as processes running outside containers on the host.

3)  **--devices**:
If you want to limit access to a specific device or devices you can use the --device flag. It allows you to specify one or more devices that will be accessible within the container.

By default, the container will be able to read, write, and mknod these devices. This can be overridden using a third :rwm set of options to each --device flag.
