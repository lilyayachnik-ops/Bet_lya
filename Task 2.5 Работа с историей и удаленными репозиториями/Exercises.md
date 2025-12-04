# Checkpoints

Клонируй репозиторий: `https://gitlab.com/andrew1test-group/git-squash.git`. Создай в своем аккаунте репозиторий `devops-task-force`. Залей весь репозиторий `git-squash` в `devops-task-force`. После этого в ветке `main` используй `squash` для объединения всех коммитов в один. После этого попробуй выполнить `git push` в ваш репозиторий. Сохрани текст ошибки. Выполни `git push --force` в ваш репозиторий.

```bash
git clone https://gitlab.com/andrew1test-group/git-squash.git
git remote add devops-task https://gitlab.com/lilyayachnik/devops-task-force.git
git log --oneline --graph --all
git push -u devops-task main
# rebase -i — интерактивный режим, --root — обрабатываем всю историю с самого первого коммита
git rebase -i --root
```

Покажем в терминале:

<img width="884" height="45" alt="image" src="https://github.com/user-attachments/assets/9d5d8d15-fa23-4619-a82b-b4a2ad462404" />
<img width="885" height="102" alt="image" src="https://github.com/user-attachments/assets/1f7542a1-986d-4cc8-a0d0-fdfa0e4a67f0" />
<img width="886" height="262" alt="image" src="https://github.com/user-attachments/assets/1fd5bf25-598b-4620-ae24-fcd593f15ce7" />

Откроется текстовый редактор, где мы увидим следующее:

<img width="887" height="726" alt="image" src="https://github.com/user-attachments/assets/1f3054ef-a912-4e12-b9f6-209a5e43eb50" />

Для коммита `abac525` оставим `pick`, который оставляет его без изменений, а коммитам `5e166bf`, `70774aa`, `a60ddff` напишем `squash`, который добавляет изменения из коммита к предыдущему, то есть все три `fix`-коммита будут будут добавлены к `initial`:

<img width="888" height="727" alt="image" src="https://github.com/user-attachments/assets/164898a5-f886-46fd-a755-a05b8db0932f" />

Сохраняем изменения и выходим. Затем откроется снова текстовый редактор, где предложено по умолчанию содержание `squash`-коммита :

<img width="889" height="724" alt="image" src="https://github.com/user-attachments/assets/98392c69-a3fb-444d-8c84-2907a8d50b4a" />

Изменим содержание `squash`-коммита, сохраним изменения и выйдем:

<img width="889" height="728" alt="image" src="https://github.com/user-attachments/assets/93f521ac-480c-4a37-8b56-235548aaea36" />

Посмотрим на результат:

<img width="885" height="162" alt="image" src="https://github.com/user-attachments/assets/a2dbb162-e4dd-4a60-9b3a-6ae22d665046" />

Выполним команду `git push devops-task main`, чтобы закинуть ветку `main` в удаленный репозиторий:

<img width="884" height="204" alt="image" src="https://github.com/user-attachments/assets/bf973d15-3132-4e44-9f53-7aec2381da75" />

**Как видим, появилась ошибка. Эта ошибка означает, что наша локальная история коммитов расходится с удалённой, и `Git` не может автоматически их объединить.**

Запишем эту ошибку в файл, использовав команду `git push devops-task main 2> push_error.txt`, где `2>` перенаправляет стандартный вывод ошибок:

<img width="885" height="65" alt="image" src="https://github.com/user-attachments/assets/5fc04b99-aa17-4467-b4e0-847036cf23e8" />

Выполним команду `git add push_error.txt`, чтобы добавить файл `push_error.txt` в `staging area`, затем выполним команду `git commit -m "Add push error log after squash"`, чтобы создать новый коммит с подготовленным файлом и заданным сообщением:

<img width="885" height="105" alt="image" src="https://github.com/user-attachments/assets/134ee3cd-0ce7-431b-b8ac-b0989fc0ed58" />

Выполним команду `git push --force devops-task main`, где `git push` отправляет коммиты, `--force` — ключ принуждения (отключает проверки безопасности, игнорирует расхождения в истории, перезаписывает удалённую ветку полностью):

<img width="887" height="264" alt="image" src="https://github.com/user-attachments/assets/409e4885-3d89-4c91-9acc-634230a536c8" />

Перейдем в настройки репозитория и разрешим `force push`:

<img width="952" height="726" alt="image" src="https://github.com/user-attachments/assets/e47083a2-56b1-43fc-90ca-dc9bf24d213a" />

Попробуем снова выполнить команду:

<img width="885" height="222" alt="image" src="https://github.com/user-attachments/assets/b5712fa9-baa5-4400-96cf-6092380d44fc" />

Клонируй репозиторий: `https://gitlab.com/andrew1test-group/git-rebase.git`. Создай новый репозиторий у себя на `gitlab` и запушь туда репозиторий. Создай `2 pull requests`: из ветки `feature` в `develop` и из ветки `develop` в `master`.

```bash
git clone https://gitlab.com/andrew1test-group/git-rebase.git
git remote add rebase-pull https://gitlab.com/lilyayachnik/git-rebase-pull.git
git push -u rebase-pull --all
```

Покажем в терминале:

<img width="886" height="49" alt="image" src="https://github.com/user-attachments/assets/789d296d-1d12-477f-8cdb-bf10ae8baccd" />
<img width="883" height="525" alt="image" src="https://github.com/user-attachments/assets/7e699d43-4a16-4095-abd8-ed8d8b07cf2a" />

Далее перейдем в свой удаленный репозиторий `git-rebase-pull`, перейдем во вкладку меню, найдем там `Merge requests`:

<img width="384" height="907" alt="image" src="https://github.com/user-attachments/assets/c9a230d8-7918-4dc7-bd03-b78a04d7a7e0" />

Перейдем и нажмем `Create merge request`:

<img width="939" height="559" alt="image" src="https://github.com/user-attachments/assets/f9c03ba2-7a75-4db8-b87d-61bfcac9a8bb" />

В качестве `Source branch` (что сливаем) выберем ветку `feature`, а в качестве `Target branch` (куда сливаем) — `develop`:

<img width="941" height="627" alt="image" src="https://github.com/user-attachments/assets/133f3bcd-62e4-4ebc-9c1b-93e2135f6bfe" />

Нажимаем кнопку `Compare branches and continue`. Далее оставили настройки по умолчанию и заполнили поле `Description`:

<img width="919" height="768" alt="image" src="https://github.com/user-attachments/assets/1cf51403-18bf-4294-b518-414989be38d0" />
<img width="931" height="772" alt="image" src="https://github.com/user-attachments/assets/337d1105-ee69-4820-b807-1dbe91678250" />

Нажмем кнопку `Create merge request`:

<img width="913" height="863" alt="image" src="https://github.com/user-attachments/assets/dc492fe1-d87f-4a47-8b7f-b750b38e8df2" />
<img width="924" height="775" alt="image" src="https://github.com/user-attachments/assets/811fc3d0-6940-4417-9fc9-8f0ff0f7f310" />

Видим уведомление `The source branch is 1 commit behind the target branch`. Значит, выполним следующие команды:

```bash
git checkout feature
git rebase develop
git push --force rebase-pull feature
```

Покажем в терминале:

<img width="890" height="426" alt="image" src="https://github.com/user-attachments/assets/10cca273-cd3e-4dae-b8b4-df2bd462b341" />

Обновим страницу. Уведомление пропало и заполним `Activity`, чтобы было понятно, что мы сделали:

<img width="921" height="863" alt="image" src="https://github.com/user-attachments/assets/dc50ac1d-9b1a-4ed3-b259-7a216cbb8246" />
<img width="921" height="806" alt="image" src="https://github.com/user-attachments/assets/92fb7921-7e6c-431e-88fc-54265ae48213" />
<img width="924" height="656" alt="image" src="https://github.com/user-attachments/assets/e0b710e1-9989-4480-99bb-5be6d13bcb44" />

Нажмем кнопку `Merge`:

<img width="154" height="66" alt="image" src="https://github.com/user-attachments/assets/1e3c2fa8-b426-474e-aaa9-503943fb12e5" />

Проделаем те же шаги для создания `pull requests` из ветки `develop` в ветку `master`:

<img width="943" height="583" alt="image" src="https://github.com/user-attachments/assets/c08489c1-7416-486a-8356-072b6125ae76" />

Далее:

<img width="943" height="869" alt="image" src="https://github.com/user-attachments/assets/66bc6826-c31a-4e64-b754-e32aa7dfdb97" />
<img width="941" height="801" alt="image" src="https://github.com/user-attachments/assets/7e3f8db3-6132-4038-94db-7d5061196b80" />

Далее:

<img width="929" height="860" alt="image" src="https://github.com/user-attachments/assets/b50c95fb-fc95-47de-8fef-5d416594224a" />
<img width="935" height="509" alt="image" src="https://github.com/user-attachments/assets/d729f2f1-87aa-4881-8953-c82dd1e2b262" />

Видим уведомление `The source branch is 1 commit behind the target branch`. Значит, выполним следующие команды:

```bash
git checkout develop
git rebase master
git push --force rebase-pull develop
```

Покажем в терминале:

<img width="890" height="422" alt="image" src="https://github.com/user-attachments/assets/1f1be95b-24c7-4830-958a-0937f57ed7af" />

Обновим страницу. Уведомление пропало и заполним `Activity`, чтобы было понятно, что мы сделали:

<img width="921" height="798" alt="image" src="https://github.com/user-attachments/assets/faa2d6b1-ec41-4dd0-897f-e41a4934072f" />
<img width="918" height="803" alt="image" src="https://github.com/user-attachments/assets/ea1c4179-0ab8-44a6-bb25-fdb9afafabc6" />
<img width="919" height="133" alt="image" src="https://github.com/user-attachments/assets/cf02fdc8-a5ca-4183-97e0-781beb3fd384" />

Нажмем кнопку `Merge`:

<img width="912" height="483" alt="image" src="https://github.com/user-attachments/assets/48b480e2-17b5-4de8-b665-bc87ffa355ac" />

Проверяем себя:

<img width="931" height="505" alt="image" src="https://github.com/user-attachments/assets/ab7ebc4d-8c9f-4b40-ac01-834760f7b757" />

**Важно**:

Так как прямые коммиты в ветку `master` будут запрещены настройками удаленного репозитория, для работы над каждой задачей в проекте необходимо будет создать отдельную ветку, а потом на основании нее `Pull request` в `GitHub` или `Merge request` в `Gitlab`.

`Pull request`/`Merge Request` — это запрос на слияние вашей ветки в ветку `master`.

Клонируй репозиторий: `https://gitlab.com/andrew1test-group/git-merge.git`. Создай новый репозиторий у себя на `gitlab` и загрузи туда репозиторий. Создай `pull request` из ветки `feature` в ветку `development`. Посмотри, как отображаются конфликты в интерфейсе `gitlab`.

```bash
git clone https://gitlab.com/andrew1test-group/git-merge.git
git remote add  merge https://lilyayachnik/git-merge.git
git push -u merge --all
```

Покажем в терминале:

<img width="886" height="42" alt="image" src="https://github.com/user-attachments/assets/e887dd71-e6a8-4fdb-b983-48f2f1c2500e" />
<img width="888" height="505" alt="image" src="https://github.com/user-attachments/assets/c6f57b84-b09c-4c93-a064-6bfa69c023a2" />

Создадим `Merge request`. Подробно описывать не буду, давайте сразу проделаем шаги и увидим, как отображаются конфликты:

<img width="939" height="656" alt="image" src="https://github.com/user-attachments/assets/60557bf4-da02-4064-8e9d-b7e4511129bb" />
<img width="930" height="753" alt="image" src="https://github.com/user-attachments/assets/3ac7484a-74f3-4a4f-ac3f-5e4454ed9972" />

Появились подсветка `Something went wrong. Please try again` и `Merge blocked: 1 check failed. Merge conflicts must be resolved`.

Решить возникшей конфликт можно двумя способами: `Resolve locally` и `Resolve conflicts`. Выберем `Resolve conflicts`:

<img width="936" height="864" alt="image" src="https://github.com/user-attachments/assets/502f88ed-47e4-452d-8faa-a4cf1704371c" />
<img width="920" height="96" alt="image" src="https://github.com/user-attachments/assets/e2459883-979d-4850-85b0-16a6169886fc" />

Конфликт возник в файле `main.go`. Можно выбрать решение: `Use ours`, которое принадлежит ветке `development`, или `Use theirs`, которое принадлежит ветке `feature`. Выберем `Use ours` и нажмем `Commit to source branch`:

<img width="920" height="769" alt="image" src="https://github.com/user-attachments/assets/fa2aa3e1-2582-49bd-b99a-cb9102004799" />

Далее:

<img width="936" height="861" alt="image" src="https://github.com/user-attachments/assets/bf9a114a-ec27-4b6c-90bd-1417a688f37b" />
<img width="916" height="801" alt="image" src="https://github.com/user-attachments/assets/96419671-3a94-4fd6-bc98-1d7f62db8529" />
<img width="918" height="131" alt="image" src="https://github.com/user-attachments/assets/395b16dc-7e0f-4ec0-95c6-ce2bd7030ee5" />

Нажмем `Merge`:

<img width="917" height="482" alt="image" src="https://github.com/user-attachments/assets/38ea51a8-177c-4935-b6c0-ee270db46a02" />

Проверим себя:

<img width="935" height="395" alt="image" src="https://github.com/user-attachments/assets/f7bee11f-53a6-4638-9f35-7b03788ca778" />

Выполни `fork` репозитория `https://gitlab.com/andrew1test-group/git-checkout.git`. Переключись в ветку `feature` и добавь в нее новый файл с интересной информацией отдельным коммитом. Создай `pull request` в основной проект: `git-checkout`.

<img width="939" height="192" alt="image" src="https://github.com/user-attachments/assets/5969306a-4053-4ea9-a45c-1ffe5170fd84" />
<img width="936" height="865" alt="image" src="https://github.com/user-attachments/assets/c7ebd2df-3bb5-44f7-874e-171cdda28d54" />
