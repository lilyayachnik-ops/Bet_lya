# Checkpoints

Установим консольный `git`-клиент с помощью команды

```bash
sudo apt install git
```
<img width="652" height="149" alt="image" src="https://github.com/user-attachments/assets/3cebe3a7-c17f-41f9-bc55-680b71894bad" />

Выполним настройку `git`-клиента, указав параметры `user.name` и `user.email`, соответствующие нашим данным.

```bash
# git config — команда для настройка Git, --global — параметр, означающий для всех репозиториев, user.name —— имя пользователя, ilya — значение, которое устанавливается
git config --global user.name ilya
# user.email — название почты пользователя, "lilyayachnik@gmail.com" — значение, которое устанавливается
git config --global user.email "lilyayachnik@gmail.com"
# Просмотреть имя пользователя, установленное для текущего репозитория
git config user.name
# Просмотреть название почты, установленное для текущего репозитория
git config user.email
```

Покажем в терминале:

<img width="858" height="61" alt="image" src="https://github.com/user-attachments/assets/5955f2fb-872b-441a-bbea-2741aafaab43" />
<img width="860" height="58" alt="image" src="https://github.com/user-attachments/assets/649c33e4-c00f-42cb-853d-c45e96ca2912" />

Создадим репозиторий `devops-task1` с помощью `git init`:

```bash
# Создадим директорию devops-task1, где -p — создать все недостающие родительские директории, если директория уже существует, то ошибки не будет
mkdir -p ~/devops-task1
# Перейдём в данную директорию
cd devops-task1
# Команда используется для создания нового репозитория Git. Она инициализирует пустой репозиторий в указанной директории, создавая все необходимые файлы и директории для работы с Git.
git init
```

Покажем в терминале:

<img width="860" height="89" alt="image" src="https://github.com/user-attachments/assets/297734e5-fa12-4970-9060-bb7fbea055b4" />

Создадим файл `nginx.conf`, содержащий базовую конфигурацию, которая появляется после установки пакета в `Linux` в `/etc/nginx/nginx.conf`

```bash
#Скопируем в репозиторий
cp /etc/nginx/nginx.conf ~/devops-task1
```
Покажем в терминале:

<img width="867" height="22" alt="image" src="https://github.com/user-attachments/assets/9dcdd2db-7d28-4d00-8300-98dd90ab1561" />

Создадим файл `README.md` с описанием того, что находится в репозитории:

```bash
vim README.md
```

Покажем в терминале:

<img width="859" height="27" alt="image" src="https://github.com/user-attachments/assets/b7a8ea91-37d1-4ee6-85d9-5d43d3cea7c2" />
<img width="862" height="44" alt="image" src="https://github.com/user-attachments/assets/fdffbdd2-d33d-473d-b835-96be99d30b35" />

Добавим файл `README.md` и `nginx.conf` в репозиторий:

```bash
# Добавляем файл README.md в staging area — промежуточная область подготовки к коммиту
git add README.md
# Создаем коммит - постоянную запись в истории репозитория с нашим изменением, -m — флаг для указания сообщения коммита
git commit -m "Add README.md with repository description"
# Добавляем файл nginx.conf в staging area
git add nginx.conf
# Создаем коммит
git commit -m "Add nginx.conf with nginx settings"
```

Покажем в терминале:

<img width="854" height="101" alt="image" src="https://github.com/user-attachments/assets/7eb39c0c-b872-47c7-abae-464bcd194c1f" />
<img width="861" height="105" alt="image" src="https://github.com/user-attachments/assets/2f67f4ce-9e15-42b1-94ce-da1afa86a4eb" />

Проверим историю коммитов, чтобы посмотреть отображаются ли в истории наши коммиты:

```bash
# Показывает историюю коммитов
git log
# Показывает текущее состояние рабочей директории и индекса Git
git status
```

Покажем в терминале:

<img width="860" height="302" alt="image" src="https://github.com/user-attachments/assets/3d4de1cd-d7aa-4c5e-8124-7fcc48c2dd42" />

Зарегистрируемся в двух публичных хостингах `git`-репозиториев `GitLab` и `GitHub`.

**Важно**:

`GitLab` — это `self-hosted`-решение, то есть его можно развернуть на своей собственной инфраструктуре, в то время как `GitHub` — облачное решение, которое хранит все данные на стороне компании `Microsoft`.

`GitLab` сильно «заточен» под использование в `DevOps`-процессах, а именно в `CI/CD`, и позволяет использовать их без настройки дополнительного функционала, в то время как в `GitHub` необходимо использовать дополнительный инструмент `GitHub Actions`.

После того, как зарегистрировались на каждом из хостингов, создадим репозиторий `devops-task1`:

<img width="923" height="171" alt="image" src="https://github.com/user-attachments/assets/b18d52b0-8b1b-4885-9027-6bbee5c43f9c" />
<img width="899" height="39" alt="image" src="https://github.com/user-attachments/assets/798431bd-013e-4335-843a-c70e951d6de2" />

В репозиторий, созданным ранее, добавим 2 удаленных сервера:

```bash
# Добавляем удаленный репозиторий в конфигурацию нашего локального Git-репозитория, add — добавить, github — псевдоним для удаленного репозитория, https://github.com/lilyayachnik-ops/devops-task1 — URL удаленного репозитория github
git remote add github https://github.com/lilyayachnik-ops/devops-task1
# Добавляем удаленный репозиторий в конфигурацию нашего локального Git-репозитория, add — добавить, gitlab — псевдоним для удаленного репозитория, https://gitlab.com/lilyayachnik/devops-task1 — URL удаленного репозитория gitlab
git remote add gitlab https://gitlab.com/lilyayachnik/devops-task1
# Покажет имена и URLs
git remote -v
# Покажет только имена без URLs
git remote
# Подробная информация о конкретном name
git remote show name
# Удаляем удаленный репозиторий из конфигурации нашего локального репозитория
git remote remove name
```

Покажем в терминале:

<img width="954" height="40" alt="image" src="https://github.com/user-attachments/assets/03485e2c-d547-4d2c-a5b1-8abcdcf92010" />
<img width="961" height="40" alt="image" src="https://github.com/user-attachments/assets/5a2dda13-bdcd-43cd-9e18-3dd11d5632ee" />

Загрузим репозиторий во все `git`-хостинги:

```bash
# Отправляем наши локальные коммиты в удаленный репозиторий, github — имя удаленного репозитория, master — название ветки, которую вы отправляете 
git push github master
# Отправляем наши локальные коммиты в удаленный репозиторий, gitlab — имя удаленного репозитория, master — название ветки, которую вы отправляете
git push gitlab master
```

Покажем в терминале:

<img width="959" height="220" alt="image" src="https://github.com/user-attachments/assets/0aa73e3e-6087-42d2-81d9-6be9da913ccf" />
<img width="957" height="225" alt="image" src="https://github.com/user-attachments/assets/4b0658a4-7954-45e2-86fc-9f97203b6373" />

Клонируем репозиторий:

```bash
# Создаем локальную копию удаленного репозитория на нашем компьютере, git clone — команда для клонирования, https://gitlab.com/andrew1test-group/git-checkout.git — URL удаленного репозитория на GitLab
git clone https://gitlab.com/andrew1test-group/git-checkout.git
```

Покажем в терминале:

<img width="956" height="184" alt="image" src="https://github.com/user-attachments/assets/e5bda98e-ab43-4392-be54-1536ec7798b9" />

Глянем историю коммитов:

```bash
# Вывод истории в одну строку
git log --online
# Блее подробный вывод
git log
```

Покажем в терминале:

<img width="959" height="645" alt="image" src="https://github.com/user-attachments/assets/542d149f-b1d2-4a23-9175-447f0a3cb15b" />
<img width="953" height="167" alt="image" src="https://github.com/user-attachments/assets/8e453f49-90f3-40b0-be34-4a88c29b7498" />

Переключимся на 3-й от начала коммит. Посмотрим содержимое файла `deleted.txt`:

```bash
# git checkout — команда для переключения между коммитами, ветками, e8b3ec06b — хеш-коммит
git checkout e8b3ec06b
# Чекним содержимое файла
cat deleted.txt
```

Покажем в терминале:

<img width="953" height="421" alt="image" src="https://github.com/user-attachments/assets/336c42a9-1070-4d78-adf2-15552c2ecf0f" />
