# Checkpoints

Склонируем репозиторий `https://gitlab.com/andrew1test-group/dkr30` в свой аккаунт на `gitlab` с именем `dkr-30-voting`. Перейдем по ссылке:

<img width="923" height="279" alt="image" src="https://github.com/user-attachments/assets/a95c466e-8951-4c4c-84f2-347c5dbd888f" />

Нажмём `Fork`:

<img width="925" height="803" alt="image" src="https://github.com/user-attachments/assets/25dcd8bd-a974-4b6b-98ad-a2c95fc33ba6" />
<img width="924" height="479" alt="image" src="https://github.com/user-attachments/assets/f71c671f-f092-4357-a639-fcfb8e9231f1" />

Нажмём `Fork project`:

<img width="927" height="842" alt="image" src="https://github.com/user-attachments/assets/ca9e6a01-0e7a-4b78-9534-a32078e1c2fc" />

Напишим `docker-compose.yml` файл, который бы собирал приложение, запускал его и все требуемые зависимости:

- `nginx` - проксирует запросы на `voting` и доступен на хосте на порту `20000`. Пример конфигурационного файла находится в папке `nginx`, смонтируем его в `/etc/nginx/conf.d/default.conf`. Используем `alpine`-версию образа.
- `voting` - собирается из репозитория;
- `mysql` - база данных, к которой подключается `voting`;
- `redis` - `In-memory` хранилище для кэша;

```bash
git clone https://gitlab.com/lilyayachnik/dkr-30-voting.git
```

Покажем в терминале:


Запустим сервис с именем проекта `inno30`:

```bash

```

Покажем в терминале:



Сконфигурируем приложение, выполнив команды из раздела `Migration` и `Seeding` в `README` репозитория:

```bash

```

Покажем в терминале:


Обратимся к сервису по `localhost:20000/polls` (в ответе мы должны увидеть `json`-объект):

```bash

```

Покажем в терминале:


Загрузим новые файлы в репозиторий:

```bash

```

Покажем в терминале:
