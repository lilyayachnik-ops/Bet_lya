# Checkpoints

Клонировать репозиторий: `https://gitlab.com/andrew1test-group/git-rebase.git`. Переместить коммиты из ветки `feature` в ветку `develop` с помощью `rebase`. Переместить коммиты из ветки `develop` в ветку `master` с помощью `rebase`. Создать репозиторий в своем аккаунте `gitlab` и загрузить в него получившийся локальный репозиторий.

```bash
# Перейти в ветку feature
git checkout feature
# Перебазировать feature на develop
git rebase develop
# Переключиться на develop
git checkout develop
# Сделать fast-forward merge (просто переместить указатель develop)
git merge --ff-only feature
# Выведем историю коммитов develop
git log --oneline --graph develop
# Выведем историю коммитов feature
git log --oneline --graph feature
```

Покажем в терминале:

<img width="885" height="69" alt="image" src="https://github.com/user-attachments/assets/c9231685-a107-42e9-91b3-5e4eb24c5fdd" />
<img width="887" height="67" alt="image" src="https://github.com/user-attachments/assets/c1f3a87a-f9b7-4e5e-9a8f-ebffb8ba1c36" />
<img width="887" height="181" alt="image" src="https://github.com/user-attachments/assets/8fcfe8b5-252a-4815-beb5-5fb046b53514" />
<img width="887" height="248" alt="image" src="https://github.com/user-attachments/assets/d5452a3a-43ef-45b5-b0fe-80a9ca9f901c" />

```bash
# Перейти в ветку develop
git checkout develop
# Перебазировать develop на master
git rebase master
# Переключиться на master
git checkout master
# Сделать fast-forward merge (просто переместить указатель master)
git merge --ff-only develop
# Выведем историю коммитов develop
git log --oneline --graph develop
# Выведем историю коммитов master
git log --oneline --graph master
# Выведем историю коммитов feature
git log --oneline --graph feature
```

Покажем в терминале:

<img width="884" height="367" alt="image" src="https://github.com/user-attachments/assets/784e455f-6a04-4e64-a6ab-5cfffe1676d9" />
<img width="884" height="288" alt="image" src="https://github.com/user-attachments/assets/f80acd90-29c9-4659-be17-5603f44c62f8" />
<img width="884" height="123" alt="image" src="https://github.com/user-attachments/assets/746706c1-cc90-4ce6-890b-398353704ee5" />

История коммитов до:

<img width="885" height="183" alt="image" src="https://github.com/user-attachments/assets/7dad9ea5-bca5-4960-834c-6d476f5a122a" />

```bash
git remote add greb https://gitlab.com/lilyayachnik/git-rebase.git
# Отправить все ветки удаленного репозитория и сделать их отслеживаемыми
git push -u greb --all
```

Покажем в терминале:

<img width="883" height="44" alt="image" src="https://github.com/user-attachments/assets/9f57028d-eb82-4e25-9fee-47c143589e58" />
<img width="885" height="523" alt="image" src="https://github.com/user-attachments/assets/849db2e3-fffe-440d-a07d-6f8619f6f8c4" />


Клонируй репозиторий: `https://gitlab.com/andrew1test-group/git-squash.git`. Просмотри историю коммитов в ветке `main`. Объедини все `FIX`-коммиты c родительским `(Simplified sum function)` в один коммит. Создай репозиторий в своем аккаунте и загрузите в него получившийся репозиторий. 

```bash

```



