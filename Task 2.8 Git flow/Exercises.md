# Checkpoints

Создай новый локальный репозиторий. Создай две ветки: `master/develop`. Добавь пару коммитов в `master`. Перенеси изменения в ветку `develop`. Добавь один коммит в ветку `develop`. Создай из ветки `develop` новую ветку `feature`. Создай по одному коммиту в ветках `develop` и `feature`. Загрузи все изменения на `gitlab`:

```bash
mkdir -p gitflow
cd gitflow
git init
echo "My project" > text.txt
git add text.txt
git commit -m "Initial commit"
git checkout -b develop
git checkout master
echo "One" > file1.txt
git add file1.txt
git commit -m "The second commit"
echo "File2" > file2.txt
git add file2.txt
git commit -m "Add file2"
git checkout develop
git merge master -m "Merge master into develop"
echo "Dev feature" > dev.txt
git add dev.txt
git commit -m "Add dev feature"
git checkout -b feature
echo "Feature code" > feature.txt
git add feature.txt
git commit -m "Implement feature"
git remote add flow https://gitlab.com/lilyayachnik/git-flow.git
git push -u flow --all
```

Покажем в терминале:

<img width="815" height="543" alt="image" src="https://github.com/user-attachments/assets/989aa26a-cbcf-4d81-8bb7-3a82a3d47c86" />
<img width="810" height="592" alt="image" src="https://github.com/user-attachments/assets/a75312a5-94a3-4763-82cc-7764234e73d3" />
<img width="815" height="223" alt="image" src="https://github.com/user-attachments/assets/8d118115-ff43-485b-ae6e-58d774cc2518" />
<img width="815" height="566" alt="image" src="https://github.com/user-attachments/assets/2bed1ddc-3469-4581-98a5-0b19cb20c5d6" />

Создай `pull request` из `feature` в `develop` и замержи его. Создай `pull request` из `develop` в `master` и замержи его. Пометь замерженный коммит в ветке `master` тегом `release_1.1` и загрузи в свой репозиторий:

<img width="922" height="879" alt="image" src="https://github.com/user-attachments/assets/13ac14dd-10d1-4082-b4af-bff850442485" />
<img width="917" height="99" alt="image" src="https://github.com/user-attachments/assets/ff31efe3-a0e7-4479-a80c-fdcd31e348cd" />

Замержим:

<img width="929" height="875" alt="image" src="https://github.com/user-attachments/assets/90273b9c-6c42-4ee5-a1e4-06d91a570e70" />
<img width="922" height="226" alt="image" src="https://github.com/user-attachments/assets/93e040ba-6614-4eea-b8f7-5c67234c1802" />

Выполним команду:

```bash
git checkout develop
git pull flow develop
git log --oneline --graph develop
```

Проверим в терминале:

<img width="815" height="505" alt="image" src="https://github.com/user-attachments/assets/d745f196-65ff-4744-895c-382294d6515d" />

Замержим:

<img width="927" height="833" alt="image" src="https://github.com/user-attachments/assets/0edf9ce1-ef4d-41b8-b58f-d7253f615052" />
<img width="930" height="199" alt="image" src="https://github.com/user-attachments/assets/7e412640-2c37-4834-a94e-da54a03a80e1" />

Выполним команды:

```bash
git checkout master
git pull flow master
git log --oneline --graph master
```

Проверим в терминале:

<img width="818" height="586" alt="image" src="https://github.com/user-attachments/assets/31730f70-e340-4516-82c5-afe54900c770" />
<img width="812" height="25" alt="image" src="https://github.com/user-attachments/assets/f88a077e-8cc0-4767-932b-51eb784d4f08" />

Создадим тег:

```bash
git checkout master
git tag release_1.1
git log --oneline --graph master
git push flow release_1.1
```

Покажем в терминале:

<img width="821" height="469" alt="image" src="https://github.com/user-attachments/assets/8db1b9af-5770-4421-9d2b-31495b5d5826" />


