# Chekpoints

Давайте сделаем синхронизацию с облачным хранилищем `GCP Storage` с указанием `NAME_SURNAME` в имени папки.

`GCP Storage` — сервис хранения неструктурированных данных. Внутри можно хранить все что захотим, но в основном используется как файловое хранилище. Каждый файл представлен в виде объекта, сами же объекты лежал в бакетах.

**Основные понятия**

- Бакет — коллекция объектов, все файлы должны лежать внутри. Является центральным местом управления жизненным циклом объектов и доступа к ним.
  Каждый бакет имеет свое уникальное имя, регион и класс хранения данных. Для работы с бакетом используют следующую команду: `gs://название_бакета`

- Объекты — технически `Cloud Storage` не является файловым хранилищем или файловой системой, это объектное хранилище. Каждый файл представляет из себя объект.
  Он состоит из двух частей: сам файл и метаданные. Метаданные описывают характеристики объекта в виде ключ-значение.
  
- Иммутабельность. Объекты можно только перезаписать или удалить.

  `gsutil` — это консольная утилита `GCP Storage`. Большинство команд похожи на те, что есть в `Linux` для работы с `File System`.

  ```bash
  # Отобразить список объектов в бакете
  gsutil ls gs://some_bucket
  # Копирование файлов
  gsutil cp -r <source> <target>

  ```

  `GCP Storage` поддерживает два механизма обеспечения безопасности: `IAM` и `ACL`.

  Для решения нашей задачи для начала установим утилиту `rclone`:

  ```bash
  # Распаковываем архим с утилитой rclone для работы с облачными хранилищами
  unzip rclone-current-linux-amd64.zip
  #  Переходим в данную папку
  cd rclone-current-linux-amd64
  # Копируем программу в системную директорию для исполняемых файлов, доступных всем пользователем
  sudo cp rclone /usr/bin
  # Меняем владельца файла и группу
  sudo chown root:root /usr/sbin/rclone
  # Устанавливаем права доступа u=rwx,g=r-x,o=r-x
  sudo chmod 755 /usr/sbin/rclone
  # Подготовим место для документации
  sudo mkdir -p /usr/local/share/man/man1
  # Копируем rclone.1 (руководство по rclone) в заранее подготовленное место
  sudo cp rclone.1 /usr/local/share/man/man1/
  # Обновляем базы данных man-страницы
  sudo mandb
  ```

  После завершения установки настроим `rclone` для работы с облачным хранилищем `GCP Storage`:

  ```bash
  # Интерактивный режим для настройки подключений в утилите rclone
  rclone config
  ```
  
 Выбираем `n) New remote`:
 <img width="445" height="203" alt="image" src="https://github.com/user-attachments/assets/e79b9e17-d592-4801-bf22-1a0e3f105960" />

 Дадим имя нашему соединению `ilya_bet`:
 <img width="311" height="52" alt="image" src="https://github.com/user-attachments/assets/75041f57-55e0-4c30-a330-bc144c941565" />

 Дальше выберем тип хранилища. В нашем случае `GCP Storage` — это 21:
 <img width="734" height="402" alt="image" src="https://github.com/user-attachments/assets/47915607-716f-43de-9613-c4f16bece918" />
<img width="498" height="47" alt="image" src="https://github.com/user-attachments/assets/3d09a9ad-956e-4395-aa30-5dff1efdc737" />

Следующее поле `client-id`, `client_secret` оставляем пустым:
<img width="492" height="213" alt="image" src="https://github.com/user-attachments/assets/a42f3601-dc6b-45f6-a363-b6f5ed29f90b" />

Дальше поле `project_number`. Это вы сможешь узнать у себя в `Google Cloud`. Вводим 12-е число: 
<img width="732" height="112" alt="image" src="https://github.com/user-attachments/assets/30959f37-f350-475b-8b83-ca7abc42e428" />

То же значения для `user_project`:
<img width="438" height="98" alt="image" src="https://github.com/user-attachments/assets/450e8dd8-eb5c-4dc0-ba2f-3f011ed368dd" />

`service_account_file`, `anonymous` оставляем пустыми. Далее для поля `object_acl` выбираем пункт `4`:
<img width="591" height="418" alt="image" src="https://github.com/user-attachments/assets/e66c0db0-004e-48d0-a3a6-32b79fe245f0" />

Для `bucket_acl` выбираем пункт `2`:
<img width="593" height="343" alt="image" src="https://github.com/user-attachments/assets/b999597e-f36b-41dd-a654-3cd2c8f56c50" />

Почему это важно. Опция `bucket_policy_only` (также известная как "Единый доступ на уровне бакета") заставляет `rclone` использовать только политики `IAM` для проверки прав доступа к бакету и объектам, полностью игнорируя устаревшие списки контроля доступа `ACL`, поэтому ставим `true`:

<img width="694" height="206" alt="image" src="https://github.com/user-attachments/assets/ef66efab-2210-49dc-92fb-137bf905d1ee" />

**Если использовать `ACL` и `IAM` вместо, то стреляешь есть шанс попасть себе в колено.**

Для `location` выбираем `4`:
<img width="660" height="396" alt="image" src="https://github.com/user-attachments/assets/65265fd8-879a-4a6b-bfdc-69970b1506ef" />

`storage_class` ставим `1`, что значит по умолчанию:
<img width="640" height="346" alt="image" src="https://github.com/user-attachments/assets/6c9489de-ebb0-4e5b-ba6b-88609e8377aa" />

Для `env_auth` по умолчанию значение:
<img width="732" height="207" alt="image" src="https://github.com/user-attachments/assets/e5ebae09-94de-4e6e-bcb1-777a64c75aaf" />

И, наконец-то, последний шаг:

<img width="275" height="80" alt="image" src="https://github.com/user-attachments/assets/f5b816bd-af8a-460d-9d53-9331e989f9d1" />

<img width="728" height="156" alt="image" src="https://github.com/user-attachments/assets/06dfb7bd-58f0-44f2-98e8-55bbf52b0713" />

Установили синхронизацию с нашей удаленной `VM` и облачным хранилищем `GCP Storage`:
<img width="734" height="175" alt="image" src="https://github.com/user-attachments/assets/ba0457db-0a43-4245-8e94-083bad524bc3" />


В меню появилось наше соединение. Всё работает:
<img width="506" height="363" alt="image" src="https://github.com/user-attachments/assets/ada38c07-ad49-456d-9e9c-7c460e41bf50" />

Дальше выполним следующие команды:

```bash
# Перейдем в домашнюю директорию
cd ~
# Создадим директории, p говорит команда создать все родительские директории в указанном пути, если они не существуют
mkdir -p ~/ILYA_BET/archives
# Перейдем в ILYA_BET
cd ILYA_BET
# Создадим пару файлов в ILYA_BET 
touch file1.txt gile2.txt file3.txt readme.txt
# Перейдем в archives
cd archives
# Создадим файл в archives
touch testing.txt testing2.txt
```
<img width="691" height="182" alt="image" src="https://github.com/user-attachments/assets/a7125c80-2213-4d87-a354-fa64f33b1f54" />
<img width="644" height="132" alt="image" src="https://github.com/user-attachments/assets/3065465c-dfaa-4950-85bf-adabdd82fb20" />


После того, как подготовили песочницу, выполним синхронизацию с `GCP Storage`:

```bash
# Синхронизация файлов на локальной машине и в хранилище, где
# ilya_bet — имя соединения
# ilya_bet_storage — название бакета, куда мы хотим прокинуть наши данные
# --interactive — запускает режим синхронизации в интерактивном режиме
rclone sync --interactive ~/ILYA_BET ilya_bet:ilya_bet_storage
```
<img width="739" height="306" alt="image" src="https://github.com/user-attachments/assets/9beaebc5-4a2d-46e5-97e7-f52283fa6145" />

Перейдем в `GCP Storage`:

<img width="938" height="483" alt="image" src="https://github.com/user-attachments/assets/71aea300-6cb2-4ee2-8a1f-0da2b83b1089" />

Всё есть. Всё работает.

**Полезные команды для работы с rclone**

```bash
# Просмотр списка контейнеров в хранилище:
rclone lsd [remote]:
# Создание нового контейнера:
rclone mkdir [remote]:[имя_контейнера]
# Просмотр списка файлов в контейнере:
rclone ls [remote]:[имя_контейнера]
# Копирование файлов с локальной машины в хранилище:
rclone copy /home/local/directory [remote]:[имя_контейнера]
# Синхронизация файлов на локальной машине и в хранилище:
rclone sync /home/local/directory [remote]:[имя_контейнера]
# Синхронизация файлов в хранилище с файлами на локальной машине:
rclone sync [remote]:[имя_контейнера] /home/local/directory

!!!При выполнении операций копирования и синхронизации rclone проверяет все файлы по дате и времени изменения или md5-сумме. Из директории-источника в директорию назначения передаются те файлы, которые были изменены.
```

Давайте синхронизируем между собой две папки на двух разных `VM` автоматически по дате последнего редактирования. Для этого будем использовать `rsync`. 

`rsync` —  утилита, которую можно использовать для синхронизации файлов и папок с локального компьютера на удаленный и наоборот. Примечательная особенность — возможность передавать зашифрованные файлы с помощью `SSH` и `SSL`. Кроме того, передача файлов выполняется в один поток. Применяется сжатие и шифрование.

```bash
# В качестве источника и приемника может выступать удаленная или локальная директория, ssh, rsync сервер
rsync опции источник приемник

Парочка опций

-v — Выводить подробную информацию о процессе копирования;
-q — Минимум информации;
-c — Проверка контрольных сумм для файлов;
-a — Режим архивирования, когда сохраняются все атрибуты оригинальных файлов;
-R — Относительные пути;
-b — Создание резервной копии;
-u — Не перезаписывать более новые файлы;
-l — Копировать символьные ссылки;
-L — Копировать содержимое ссылок;
-H — Копировать жесткие ссылки;
-p — Сохранять права для файлов;
-g — Сохранять группу;
-t — Сохранять время модификации;
-x — Работать только в этой файловой системе;
-e — Использовать другой транспорт, например, ssh;
-z — Сжимать файлы перед передачей;
--delete — Удалять файлы которых нет в источнике;
--include — Включить файлы по шаблону
--exclude — Исключить файлы по шаблону;
--recursive — Перебирать директории рекурсивно;
--no-recursive — Отключить рекурсию;
--progress — Выводить прогресс передачи файла;
--stat — Показать статистику передачи;
--version — Версия утилиты.
```


Для выполнения нашей задачи используем следующую команду:

```bash
# Синхронизация двух папок с помощью rsync, где
# a — сохраняются все атрибуты оригинальных файлов
# v — выводит подробную информацию
# z — сжимает файлы перед передачей
# u — пропускает файлы, которые новее на приемнике

rsync -avzu /home/bet/sync remote@IP_addr_remote:/home/remote/remote_sync
```

Напишем простой скрипт:

```bash
#!/bin/bash
rsync -avzu /home/bet/ilya_bet_sync/ remote@IP_addr_remote:/home/remote/ilya_bet_backup

```

Чтобы процесс выполнялся автоматически пропишем в `cron` следующее правило:

```bash
# Отредактируем файл для расписания задач
crontab -e
# 
```

 


  
