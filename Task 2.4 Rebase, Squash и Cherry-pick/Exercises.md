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
# Выведим историю коммитов
git log --oneline --graph -all
# git rebase -i — запускает интерактивный режим, который помогает сжимать коммиты, HEAD~3 — ,берём три коммита назад от HEAD
git rebase -i HEAD~3
```

Проверим в терминале: 

<img width="883" height="125" alt="image" src="https://github.com/user-attachments/assets/292dd527-98cd-43b4-92f9-a1da9124a49a" />

Далее откроется текстовый редактов. `rebase -i` охватывает только последние три коммита. Приведем все к одному коммиту. Отметим для сжатия последние два коммита как `squash` или `s`, который объединяет коммит с предыдущим, а первый коммит — `reword`, который позволяет изменить текст коммита. В данном примере коммиты, предназначенные для сжатия (`s`), будут слиты с основным коммитом — тем, который отмечен командой `reword`. Отметив коммиты, сохраним изменения в редакторе:

<img width="887" height="662" alt="image" src="https://github.com/user-attachments/assets/e3a251df-b3b5-4753-9e04-56026533a608" />

После того, как сохраним изменения и выйдем, откроется снова текстовый редактор, чтобы поменять название первого коммита: 

<img width="888" height="725" alt="image" src="https://github.com/user-attachments/assets/382d2493-7358-4cd4-8eab-d6fe1d3f24ae" />

Сделаем это, сохраним изменения и выйдем:

<img width="887" height="728" alt="image" src="https://github.com/user-attachments/assets/4397ad5b-ea18-4bf4-b7eb-842a18885331" />

Далее `rebase -i` снова откроет редактор для ввода сообщения о коммите:

<img width="888" height="726" alt="image" src="https://github.com/user-attachments/assets/9cbed4ab-dbde-4b88-b45d-73a5c8d16ec6" />

Отредактировав и сохранив сообщения, закроем редактор:

<img width="887" height="724" alt="image" src="https://github.com/user-attachments/assets/ff4f7d24-d688-4d34-8797-54b7cce94664" />

Выведем историю коммитов с помощью команды `git log --oneline --graph --all`:

<img width="879" height="703" alt="image" src="https://github.com/user-attachments/assets/402fe164-404a-4a29-b3ea-e7a08c1214c2" />

Закинем на удаленный репозиторий:

```bash
# git remote add — создает псвевдоним, sqsh — имя, которое даю удаленному репозиторию, https:/gitlab.com/lilyayachnik/git-squash.git — URL репозитория
git remote add sqsh https:/gitlab.com/lilyayachnik/git-squash.git
# Отправим все локальные ветки в удаленный ропезиторий
git push sqsh --all
```

Покажем в терминале:

<img width="945" height="248" alt="image" src="https://github.com/user-attachments/assets/c4119cbb-c9ed-4578-bb81-893510cfbf6d" />

Клонируй репозиторий: `https://gitlab.com/andrew1test-group/git-cherry-pick.git`. Перенести коммит `Formatted code` в ветку `main`. Создай репозиторий в своем аккаунте и залейте туда получившийся репозиторий:

```bash
# Выведем историю коммитов, чтобы узнать хеш коммита Formatted code
git log --oneline --graph --all
# Перейдем в ветку main
git checkout main
# Эта команда позволяет добавлять произвольные коммиты из ветви в HEAD другой ветви без необходимости полного слияния. В данном случае коммит e4a9855bf663ab8ffd78b232cc6f429b693bb35f из ветви develop в ветвь main
git cherry-pick e4a9855bf663ab8ffd78b232cc6f429b693bb35f
```

Покажем в терминале:

<img width="887" height="485" alt="image" src="https://github.com/user-attachments/assets/cd861240-21a6-4978-a6a9-d69a61751c8c" />
<img width="885" height="207" alt="image" src="https://github.com/user-attachments/assets/ab3e1639-7465-482f-b7dc-dd0adb0e8c89" />

Исправим конфликт. Содержимое файла `main.go`:

```bash
package main

import (
        "fmt"
        "log"
        "net/http"
        "strconv"
)

func sendResponse(w http.ResponseWriter, r *http.Request) {
<<<<<<< HEAD
  keys, ok := r.URL.Query()["a"]
  if !ok || len(keys[0]) < 1 {
      log.Println("Url Param 'a' is missing")
      return
  }

  a, err := strconv.Atoi(keys[0])

  if err != nil {
    return
  }

  keys, ok = r.URL.Query()["b"]
  if !ok || len(keys[0]) < 1 {
      log.Println("Url Param 'b' is missing")
      return
  }
  b, err := strconv.Atoi(keys[0])
  if err != nil {
    log.Println("Bad number!")
    return
  }

  fmt.Fprintf(w, "%d + %d = %d", a, b, sum(a,b))
=======
        keys, ok := r.URL.Query()["a"]
        if !ok || len(keys[0]) < 1 {
                log.Println("Url Param 'a' is missing")
                return
        }

        a, err := strconv.Atoi(keys[0])

        if err != nil {
                return
        }

        keys, ok = r.URL.Query()["b"]
        if !ok || len(keys[0]) < 1 {
                log.Println("Url Param 'b' is missing")
                return
        }
        b, err := strconv.Atoi(keys[0])
        if err != nil {
                log.Println("Bad number!")
                return
        }

        fmt.Fprintf(w, "%d + %d = %d", a, b, sumValues(a, b))
>>>>>>> e4a9855... Formatted code
}

func main() {
        http.HandleFunc("/", sendResponse)
        log.Fatal(http.ListenAndServe(":7000", nil))
}

<<<<<<< HEAD
func sum (a, b int) int {
  t := b + a
  return t
}

=======
func sumValues(a, b int) int {
        t := b + a
        return t
}
>>>>>>> e4a9855... Formatted code
```

Откроем файл в текстовом редакторе `vim main.go`. Оставим код, который относится к `Formatted code`:

```bash
package main

import (
        "fmt"
        "log"
        "net/http"
        "strconv"
)

func sendResponse(w http.ResponseWriter, r *http.Request) {
        keys, ok := r.URL.Query()["a"]
        if !ok || len(keys[0]) < 1 {
                log.Println("Url Param 'a' is missing")
                return
        }

        a, err := strconv.Atoi(keys[0])

        if err != nil {
                return
        }

        keys, ok = r.URL.Query()["b"]
        if !ok || len(keys[0]) < 1 {
                log.Println("Url Param 'b' is missing")
                return
        }
        b, err := strconv.Atoi(keys[0])
        if err != nil {
                log.Println("Bad number!")
                return
        }

        fmt.Fprintf(w, "%d + %d = %d", a, b, sumValues(a, b))

}

func main() {
        http.HandleFunc("/", sendResponse)
        log.Fatal(http.ListenAndServe(":7000", nil))
}

func sumValues(a, b int) int {
        t := b + a
        return t
}
```

Далее:

```bash
# Добавим файл main.go в staging area
git add main.go
# Продолжим cherry-pick, --continue — так после того, как встретил конфликт, он приостанавливается и ждет
git cherry-pick --continue
```

Покажем в терминале:

<img width="888" height="141" alt="image" src="https://github.com/user-attachments/assets/015ebfdc-ba27-4809-bb36-4ee2dc360b77" />

Открылся текстовый редактор:

<img width="889" height="729" alt="image" src="https://github.com/user-attachments/assets/ef992542-785b-4dff-9579-9b9c373dd717" />


Глянем историю коммитов:

<img width="886" height="145" alt="image" src="https://github.com/user-attachments/assets/75046774-bd9a-431c-afc0-3c2af6869880" />

Закинем в удаленный репозиторий:

```bash
git remote add pick https://gitlab.com/lilyayachnik/git-cherry-pick.git
git push pick --all
```

Покажем в терминале:

<img width="889" height="224" alt="image" src="https://github.com/user-attachments/assets/0d2be354-e001-4119-991c-133a94bfd004" />



