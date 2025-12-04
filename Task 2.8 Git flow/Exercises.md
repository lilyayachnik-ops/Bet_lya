# Checkpoints

Создай новый локальный репозиторий. Создай две ветки: `master/develop`. Добавь пару коммитов в `master`. Перенеси изменения в ветку `develop`. Добавь один коммит в ветку `develop`. Создай из ветки `develop` новую ветку `feature`. Создай по одному коммиту в ветках `develop` и `feature`. Загрузи все изменения на `gitlab`:

```bash
mkdir -p gitflow
cd gitflow
git init
echo "One" > file1.txt
git add file1.txt
git commit -m "Initial commit"
echo "File2" > file2.txt
git add file2.txt
git commit -m "Add file2"
git checkout -b develop
git merge master
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

<img width="808" height="200" alt="image" src="https://github.com/user-attachments/assets/b18d3dd4-4e90-424e-9976-ac14854cc42d" />
<img width="814" height="260" alt="image" src="https://github.com/user-attachments/assets/9601ffee-c3f1-409e-a428-666852153688" />
<img width="809" height="46" alt="image" src="https://github.com/user-attachments/assets/98bb55af-c1b8-45da-a9a6-747e15dc99e5" />
<img width="811" height="305" alt="image" src="https://github.com/user-attachments/assets/9cf647fb-1411-4beb-bb86-795b6ea24bf3" />
<img width="809" height="587" alt="image" src="https://github.com/user-attachments/assets/37efd562-3640-4089-88ad-f3ea708141ec" />
<img width="810" height="322" alt="image" src="https://github.com/user-attachments/assets/52537b1b-b0fc-4c4f-af26-6b7991666423" />

Создай `pull request` из `feature` в `develop` и замержи его. Создай `pull request` из `develop` в `master` и замержи его. Пометь замерженный коммит в ветке `master` тегом `release_1.1` и загрузи в свой репозиторий:

<img width="923" height="877" alt="image" src="https://github.com/user-attachments/assets/cb06badb-8215-4474-b7a5-58db3321f016" />
<img width="922" height="202" alt="image" src="https://github.com/user-attachments/assets/505fc153-0dc0-4207-8c29-aae730c90a1e" />

Замержим:

<img width="936" height="871" alt="image" src="https://github.com/user-attachments/assets/c693154b-3542-4351-92a3-cc590cf27b4a" />
<img width="916" height="290" alt="image" src="https://github.com/user-attachments/assets/95cab0ee-967a-43a8-b286-7af6e64d2f3d" />

Выполним команду:

```bash
git checkout develop
git pull flow develop
git log --oneline --graph develop
```

Проверим в терминале:

<img width="809" height="66" alt="image" src="https://github.com/user-attachments/assets/17ced2fb-c2db-4514-8aad-f01bfb9c370c" />
<img width="807" height="429" alt="image" src="https://github.com/user-attachments/assets/5db77936-acb4-4372-8a65-e306b6b9f0bb" />

<img width="926" height="881" alt="image" src="https://github.com/user-attachments/assets/200a9075-bc84-4d3b-9cdc-ebaacbedb640" />
<img width="925" height="395" alt="image" src="https://github.com/user-attachments/assets/7a5716e1-2954-4e8d-ad88-5116a57f3483" />

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

<img width="812" height="593" alt="image" src="https://github.com/user-attachments/assets/59e6e988-db4f-42cb-b6f2-2945d4c03c20" />

Создадим тег через коммит. Перейдём в `Repository`, затем `Commits`. Найдём последний коммит в `master` (после мержа). Нажмём на хеш коммита. Нажми `Tags`, затем `New tag`. Заполним:

- `Tag name`: `release_1.1`
- `Message`: `Release version 1.1`
- `Create from`: коммит после мержа

Покажем:

<img width="950" height="583" alt="image" src="https://github.com/user-attachments/assets/635b8804-76a4-4a56-91e2-d30ba5154998" />
<img width="952" height="333" alt="image" src="https://github.com/user-attachments/assets/fe4d410b-8df5-45ae-a7a2-2ebc2e596f31" />

Нажмем `Create release`:

<img width="933" height="880" alt="image" src="https://github.com/user-attachments/assets/c368df8b-f12b-4b93-9a00-861c6dec33fd" />
<img width="926" height="316" alt="image" src="https://github.com/user-attachments/assets/b00a1fad-eb9f-4c46-a6bc-c7bf576ddcd9" />

Нажмем `Create release`:

<img width="916" height="343" alt="image" src="https://github.com/user-attachments/assets/3f3cb0c1-13d0-4da0-8c48-54600b9ffbef" />

Выполним команды:

```bash
git pull flow --tag master
git log --oneline --graph master
```
Проверим в терминале:

<img width="807" height="421" alt="image" src="https://github.com/user-attachments/assets/8cf82375-86da-4b04-8dd0-801cfaf1a396" />
