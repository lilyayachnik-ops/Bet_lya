# Checkpoints

Проверить наличие файла `~/.gitconfig` (только на `Linux` системе, файл будет присутствовать, если до этого ты настраивал `git config --global`). В случае отсутствия можешь создать его сам:

```bash
ls -la
```

Покажем в терминале:

<img width="809" height="24" alt="image" src="https://github.com/user-attachments/assets/87f266b9-7b7b-4ea3-a313-5b0c228b7df5" />

Настроить `.gitconfig` так, чтобы он подключал другой конфиг для определенной директории. Ниже мы для всех подпапок и файлов директории `~/innowise/` подключаем конфиг `.gitconfig-innowise`:

```bash
vim .gitconfig
```

В редакторе `vim` прописываем следующее:

```bash
# [includeIf] — секция для условного включения; "gitdir:~/innowise/" — условие: "если текущий git репозиторий находится в директории ~/innowise/ или её поддиректориях"
# path — путь к файлу с дополнительной конфигурацией; ~/.gitconfig-innowise — файл с настройками Git для проектов Innowise

[includeIf "gitdir:~/innowise/"]
        path = ~/.gitconfig-innowise
```

Покажем в терминале:

<img width="810" height="119" alt="image" src="https://github.com/user-attachments/assets/93725eb5-fc72-4bd4-b2c1-22cc459c3cda" />


Написать сам .gitconfig-innowise. Здесь мы просто добавляем нужные нам креденшиалы:

```bash
vim .gitconfig-innowise
```

В редакторе `vim` прописываем следующее:

```bash
# [user] — секция конфигурации. Означает, что следующие строки относятся к настройкам пользователя.
# Ключ: name. Значение: Аркадий Петров Устанавливает имя автора коммитов
# Ключ: email. Значение: arkadiy_parovozov@innowise-group.com. Устанавливает email автора коммитов

[user]
        name = Аркадий Петров
        email = arkadiy_parovozov@innowise-group.com  
```

Покажем в терминале: 

<img width="811" height="65" alt="image" src="https://github.com/user-attachments/assets/97392b4d-56d7-42bc-b5da-0c335f04f3cd" />

Теперь дело за малым. Создай репозиторий внутри директории `~/innowise/`, после чего внеси какие-нибудь изменения и закоммить. Если автором коммита является Аркадий Паровозов, то ты справился:

```bash
mkdir -p ~/innowise
cd innowise
mkdir test_project
cd test_project
git init
touch file1.txt
git add file1.txt
git commit -m "Add new file file1.txt"
git log --graph master
```
Покажем в терминале:

<img width="810" height="364" alt="image" src="https://github.com/user-attachments/assets/177215e6-39f7-4ccb-9815-a0819dcb994c" />
