# Checkpoints

Часто при работе над одним проектом, возникает необходимость использовать в нём другой проект. Возможно, это библиотека, разрабатываемая сторонними разработчиками или вами, но в рамках отдельного проекта, и используемая в нескольких других проектах. Типичная проблема, возникающая при этом — вы хотите продолжать работать с двумя проектами по отдельности, но при этом использовать один из них в другом.

**Git решает эту проблему с помощью подмодулей. Подмодули позволяют вам сохранить один Git-репозиторий, как подкаталог другого Git-репозитория. Это даёт вам возможность клонировать в ваш проект другой репозиторий, но коммиты при этом хранить отдельно.**

**Submodule — это дочерний репозиторий внутри основного.**

Создай пустой репозиторий. Добавь в него `https://github.com/WordPress/WordPress` как сабмодуль с именем wordpress. Далее зайди в этот сабмодуль и сделай `git fetch -a`, просмотри все имеющиеся тэги и верни сабмодуль к состоянию с тэгом `3.4.2`. Сделай коммит основного репозитория, чтобы зафиксировать версию сабмодуля:

```bash
mkdir -p ~/submodule
cd submodule
git init
# git submodule add — добавляет нового подмодуля, https://github.com/WordPress/WordPress — URL проекта, который хотим начать отслеживать, wordpress — имя папки, куда будет клонирован сабмодуль
git submodule add https://github.com/WordPress/WordPress wordpress
```
Покажем в терминале:

<img width="809" height="84" alt="image" src="https://github.com/user-attachments/assets/6d67dc91-b8d3-4148-a5a6-6e0adc16426d" />
<img width="810" height="184" alt="image" src="https://github.com/user-attachments/assets/9dc36b45-8709-4b98-b3ad-8cbdf98b1bc5" />

Перейдем в сабмодуль и выполним команду:

```bash
cd wordpress
# git fetch -a — загружает обновления сразу из всех удаленных репозиториев, связанных с локальным хранилищем.
git fetch -a
# git tag -l — просмотреть список имеющихся тегов
git tag -l
```

Покажем в терминале:

<img width="803" height="367" alt="image" src="https://github.com/user-attachments/assets/09b8bde1-b16b-42d8-9a3c-862f13c2b0cd" />

Вернем сабмодуль к состоянию с тегом `3.4.2`. Сделаем коммит основого репозитория, чтобы зафиксировать версию сабмодуля:

```bash
# --hard: HEAD меняется, индекс сбрасывается, рабочие файлы перезаписываются
git reset --hard tags/3.4.2
cd ..
git add wordpress
git commit -m "Pin WordPress submodule to version 3.4.2"
```
Покажем в терминале:

<img width="810" height="221" alt="image" src="https://github.com/user-attachments/assets/3dbd3f12-1326-4ccb-8ae5-a17c9c916421" />

**Обратите внимание на права доступа 160000 у wordpress. Это специальные права доступа в Git, которые означают, что вы сохраняете коммит как элемент каталога, а не как подкаталог или файл. А права доступа 100644 у .gitmodules означают, что это обычный файл.**

**.gitmodules — конфигурационный файл, в котором хранится соответствие между URL проекта и локальным подкаталогом, в который вы его выкачали.**

<img width="813" height="81" alt="image" src="https://github.com/user-attachments/assets/beba55f7-72b7-4882-9018-1cb49d3622e9" />

Скопируй из сабмодуля в свой репозиторий следующие файлы, а также очисти от ненужных:
- `cp wordpress/index.php index.php`
- `cp wordpress/wp-config-sample.php wp-config.php`
- `cp -Rf wordpress/wp-content .`
- `rm -Rf wp-content/plugins/hello.php wp-content/themes/twentyten`

```bash
# index.php — основной файл, который обрабатывает ВСЕ запросы к сайту WordPress
cp wordpress/index.php index.php
# wp-config.php —  главный конфигурационный файл WordPress. Содержит настройки базы данных, безопасности и поведения
cp wordpress/wp-config-sample.php wp-config.php
# -R — рекурсивно (копирует папку со всем содержимым), -f — force (принудительно, перезаписывает если существует), . — текущая директория (основной репозиторий). Копирует папку с темами, плагинами, загрузками и т.д.
cp -Rf wordpress/wp-content .
# -Rf — рекурсивно и принудительно (для папок). Удаляет стандартный плагин "Hello Dolly" и стандартную тему "Twenty Ten"
rm -Rf wp-content/plugins/hello.php wp-content/themes/twentyten
```

Покажем в терминале:

<img width="884" height="66" alt="image" src="https://github.com/user-attachments/assets/c65c5b76-04a6-45d8-a901-f207f83d440a" />
<img width="807" height="263" alt="image" src="https://github.com/user-attachments/assets/6682231f-6635-4ff3-9a5d-14f99ce0d601" />:

```bash
vim index.php
:set number
require('wordpress/wp-blog-header.php')
```
**require — PHP-функция, которая включает и выполняет указанный файл. А 'wordpress/' — путь к папке сабмодуля WordPress, где wp-blog-header.php — главный файл-загрузчик WordPress**

Покажем в терминале:

<img width="808" height="24" alt="image" src="https://github.com/user-attachments/assets/af501731-923a-4319-b680-d99b29fe8752" />
<img width="813" height="593" alt="image" src="https://github.com/user-attachments/assets/c9fd0078-4aa6-44bf-a5c9-d4c8cf3320b7" />

Зафиксируй все изменения коммитом и запушь репозиторий:

```bash
git add index.php wp-config.php wp-content/
git commit -m "Add WordPress core files configured for submodule. Copied index.php, wp-config.php, wp-content from submodule. Removed hello.php plugin and twentyten theme. Modified index.php to point to submodule"
git remote add sub https://gitlab.com/lilyayachnik/submodule.git
git branch -a
git push -u sub master
```

Покажем в терминале:

<img width="809" height="590" alt="image" src="https://github.com/user-attachments/assets/c6045cba-4653-47ba-8baf-72dca7999b93" />
<img width="804" height="590" alt="image" src="https://github.com/user-attachments/assets/a2e50b19-c7ee-464c-b5d4-a6ec8eb5fa75" />
<img width="802" height="588" alt="image" src="https://github.com/user-attachments/assets/305f6599-703c-4be8-a80c-fa44f1c794db" />
<img width="804" height="344" alt="image" src="https://github.com/user-attachments/assets/b39485eb-4f7b-4313-886e-7eb56c84ecaa" />
