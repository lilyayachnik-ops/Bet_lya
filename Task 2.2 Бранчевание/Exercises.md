# Checkpoints

В репозитории `devops-task1`, загруженном на хостинг в предыдущем задании, создадим с помощью `git checkout` и `git branch` две новые ветки: `develop` и `feature/new-site`:

```bash
# Создает ветку develop
git branch develop
# Переключается на ветку develop
git checkout develop
```

Покажем в терминале:

<img width="954" height="23" alt="image" src="https://github.com/user-attachments/assets/38ef4d75-517e-4265-a903-54ee0dc3ae91" />
<img width="953" height="42" alt="image" src="https://github.com/user-attachments/assets/96fd1a1f-411c-4124-800d-2382fc756184" />


Также есть другие способы создания ветки:

```bash
# Создает ветку name_branch
git branch name_branch
# Переключается на ветку name_branch
git switch name_branch
# -b указывает на создания ветки name_branch
git checkout -b name_branch
# -c указывает на создания ветки name_branch
git switch -c name_branch 
```

Переключимся на ветку `master`, создадим ветку `feature/new-site`:

```bash
# Переключается на ветку master
git checkout master
# Создадим новую ветку feature/new-site
git branch feature/new-site
# Переключается на ветку feature/new-site
git checkout feature/new-site
```

Покажем в терминале:

<img width="954" height="101" alt="image" src="https://github.com/user-attachments/assets/fcb8e89c-4a32-4617-82cb-8ba6843acbea" />

В ветке develop измени настройки в nginx.conf: замени `worker_connections` на `16384` и закоммить изменения. В ветке `develop` измени `nginx.conf`, включи сжатие ответа для типа `application/json` и закоммить изменения:

```bash
git checkout develop
# Отредактируем nginx.conf
vim nginx.conf
```
В поле `worker_connections` напишем `16384`:

<img width="954" height="91" alt="image" src="https://github.com/user-attachments/assets/84177e9d-47c3-4606-a44b-75d0e8220902" />

Закоммитим внесённые изменения:

```bash
git add nginx.confg
git commit -m "Changed worker_connection to 16384 in nginx and enabled"
```

Покажем в терминале:

<img width="954" height="23" alt="image" src="https://github.com/user-attachments/assets/f839ce64-3282-4368-bc5b-5bad342964c4" />
<img width="951" height="60" alt="image" src="https://github.com/user-attachments/assets/6dffaaae-3f7b-4525-9f86-f9922ae1667f" />

Включим сжатие ответа для типа `application/json`. Для этого в файле `nginx.conf` пропишем `gzip_types application/json`:

<img width="952" height="204" alt="image" src="https://github.com/user-attachments/assets/2dff1248-3830-4caf-8ecd-89e5fe0a1972" />

Закоммитим внесённые изменения:

```bash
git add nginx.conf
git commit -m "Enabled compression for application/json in nginx.conf"
```
Покажем в терминале:

<img width="953" height="102" alt="image" src="https://github.com/user-attachments/assets/0fc71e33-d684-4795-a86a-fd3821c1dbdb" />

Проверим историю коммитов:

```bash
git log
```

Покажем в терминале:

<img width="958" height="263" alt="image" src="https://github.com/user-attachments/assets/6032d79e-6438-42fe-a8ca-41e747a0cd65" />

В ветке `feature/new-site` добавь файл `conf.d/mysite.domain.com.conf` с базовым описанием статического сайта и закоммить изменения.

```bash
git checkout feature/new-site
mkdir -p conf.d
cp /etc/nginx/sites-available/default ~/develop-task1/conf.d/mysite.domain.com.conf
```

Покажем в терминале:

<img width="936" height="64" alt="image" src="https://github.com/user-attachments/assets/8dadb6dd-ee9f-47ff-adbf-189521bce9dd" />
<img width="937" height="302" alt="image" src="https://github.com/user-attachments/assets/32a0f50d-4a94-4389-859e-25b5be5ef1af" />
<img width="942" height="740" alt="image" src="https://github.com/user-attachments/assets/7b77c4dc-76eb-48c4-a496-b47b480a6eb5" />

Закоммитим изменения:

```bash
git add conf.d/mysite.domain.com.conf
git commit -m "Add nginx configuration for mysite.domain.com static site"
```

Покажем в терминале:

<img width="935" height="121" alt="image" src="https://github.com/user-attachments/assets/a50f84e6-03c3-4cbf-abb5-7553051d563c" />

Чтобы изменить последний коммит:

```bash
git commit --ammend -m "Add nginx configuration for mysite.domain.com static site"
```

Покажем в терминале:

<img width="935" height="247" alt="image" src="https://github.com/user-attachments/assets/9124b6c5-4e96-42d4-8cb4-7ce80930ff6c" />

Добавим легковесный тег `v0.1` на последний коммит в  ветке `develop` (где изменяли `nginx.conf`):

```bash
# Переключается на ветку develop
git checkout develop
# Показывает историю коммитов, --oneline — отображает каждый коммит в одну строку (хеш+сообщение), -1 — показывает только последний коммит
git log --oneline -1
# Создает тег с именем v0.1, по умолчанию создаёт легковесный тег
git tag v0.1
git log
```

Покажем в терминале:

<img width="941" height="102" alt="image" src="https://github.com/user-attachments/assets/0621eb6e-ffc2-49c7-a3d8-aed3b77943e5" />
<img width="940" height="139" alt="image" src="https://github.com/user-attachments/assets/07dd7298-4def-4585-8cd1-50d896d54427" />

Добавь файл `.gitignore` в ветку `feature/new-site`.  Файл `gitignore` должен быть составлен таким образом, чтобы исключать из репозитория папку `tmp` и все ее содержимое. Создай локально папку `tmp` с несколькими файлами в ней, закоммить изменения:

```bash
# Переключается на ветку feature/new-site
git checkout feature/new-site
# Создаем директорию tmp 
mkdir -p ~/devops-task1/tmp
# Выводим содержимое devops-task1
ls -la
# Переходим в tmp
cd tmp
# Создаем файлы file1.txt, file2.txt, file3.txt
touch file1.txt file2.txt file3.txt
# Выводим содержимое
ls -la
# Переходим на один уровень вверх
cd ..
# Создадим файл .gitignore
vim .gitignore
```

Покажем в терминале:

<img width="935" height="42" alt="image" src="https://github.com/user-attachments/assets/b3acd493-27c4-4014-a4c2-d503de6c7361" />
<img width="936" height="422" alt="image" src="https://github.com/user-attachments/assets/a216dce3-347d-482d-871a-db0a9f06caff" />

**Важно**: `.gitignore` помещается в корневой каталог репозитория. Корневой каталог также известен как родительский или текущий рабочий каталог. Корневая папка содержит все файлы и другие папки, из которых состоит проект. Тем не менее, вы можете поместить этот файл в любую папку в репозитории. Если на то пошло, но у вас может быть несколько файлов `.gitignore`. В файл `.gitignore` должны быть добавлены файлы любого типа, которые не нужно фиксировать.

```bash
# Будет игнорировать файл text.txt
/text.txt
# Будет игнорировать файл text.txt, который расположен в папке test корневого каталоге
/test/text.txt
# Будет игнорировать любые файлы text.txt
text.txt
# Будет игнорировать весь каталог со всем его содержимым
test/
# Если мы напишем просто имя каталога без слеша, то этот шаблон будет соответствовать как любым файлам, так и любым каталогам с таким именем:
test
```

Закомментим изменения:

```bash
git add .gitignore
git commit -m "Add .gitignore to exclude tmp directory"
```

Покажем в терминале:

<img width="933" height="102" alt="image" src="https://github.com/user-attachments/assets/57ca68d3-1d24-4810-b0f9-ef6fc4b14535" />

Загрузим обе ветки на `gitlab`. Загрузим тег на хостинг с помощью `git push` и проверим его наличие. Присутствует ли директория `tmp` в ветке `feature/new-site`?(Нет)

```bash
git checkout develop
git push gitlab develop
```

Покажем в терминале:

<img width="937" height="366" alt="image" src="https://github.com/user-attachments/assets/64e08e3d-5122-44b3-8d0f-884a9f2d4064" />

```bash
git checkout feature/new-site
git push gitlab feature/new-site
```

Покажем в терминале:

<img width="945" height="365" alt="image" src="https://github.com/user-attachments/assets/b1fef2aa-5f74-4717-8470-0544edeaed4d" />

```bash
git checkout develop
git push gitlab v0.1
```

Покажем в терминале:

<img width="943" height="163" alt="image" src="https://github.com/user-attachments/assets/b216ba13-b1be-468c-9501-bdf805a92e4d" />

