# Checkpoints

Склонируем репозиторий `fat_free_crm`:

```bash
git init
git clone https://github.com/fatfreecrm/fat_free_crm
```

Покажем в терминале:

<img width="1055" height="266" alt="image" src="https://github.com/user-attachments/assets/f26f0c4e-e64c-4fa9-802a-e77c9fffd811" />
<img width="1404" height="861" alt="image" src="https://github.com/user-attachments/assets/9de96065-ff16-440c-a132-daf04bf3b6cb" />

Добавим в репозиторий файл `/home/user/fat_free_crm/.dockerignore`, который должен содержать:

- директорию `.git`

- все скрытые файлы (начинаются с `.`) с расширением `yml`

- все `yml`-файлы в директории `config`, кроме `settings.default.yml`, `database.sqlite.yml`, `database.postgres.docker.yml`

- все файлы во всех подпапках `./public/avatars/` с расширением `gif`

```bash
.git/**
.*.yml
config/**/*.yml
!config/settings.default.yml
!config/database.sqlite.yml
!config/database.postgres.docker.yml
public/avatars/**/*.gif
```

Покажем в терминале:

<img width="1395" height="221" alt="image" src="https://github.com/user-attachments/assets/5adfb920-bf14-4a02-9319-5f973beffbd7" />

Соберём образ:

```bash
# --progress=auto (по умолчанию) — красивый прогресс-бар
# --progress=plain — сырой текстовый вывод
# --progress=tty — для терминалов

$ docker build --no-cache --progress=plain -t fat_free_crm-app .
```

Покажем в терминале:

<img width="1333" height="288" alt="image" src="https://github.com/user-attachments/assets/f47cf5a3-065a-4c42-b776-f709c4b71042" />
