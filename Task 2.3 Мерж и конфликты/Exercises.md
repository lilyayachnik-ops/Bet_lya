# Checkpoints

Объедини ветку `develop` с веткой `master` из предыдущего задания. Запушь ветку в удаленный репозиторий:

```bash
# Глянем удаленные репозитории и их URL
git remote -v
# Переключаемся на ветку master
git checkout master
# Выполним слияние develop в master
git merge develop
# Закидываем ветку master на удаленный репозиторий
git push gitlab master
```

Покажем в терминале:

<img width="1018" height="264" alt="image" src="https://github.com/user-attachments/assets/2c84faec-f9a3-41f5-8e3d-7808199ef0b6" />

Склонируем `https://gitlab.com/andrew1test-group/git-merge.git`. Смержи ветку `feature` в ветку `development`, устранив конфликты. При разрешении конфликта нужно выбрать последнее по времени изменение в ветках `development` и `feature` по выводу `git log`:

```bash
# Склонируем репозиторий
git clone https://gitlab.com/andrew1-test-group/git-merge.git
# Перейдем в репозиторий git-merge
cd git-merge
# Показывает локальные и удаленные ветки
git branch -a
# Перейдем в ветку development
git checkout development
# Выполним слияние feature в development
git merge feature
```
Покажем в терминале:

<img width="1022" height="321" alt="image" src="https://github.com/user-attachments/assets/e0db96ce-e26c-40f4-a5a6-f52dbef270b1" />
<img width="1016" height="146" alt="image" src="https://github.com/user-attachments/assets/5a84f2ae-9b92-45cb-9c97-b8a871fc43d4" />

Появился конфликт в `main.go`. Глянем на него:

<img width="1019" height="730" alt="image" src="https://github.com/user-attachments/assets/2a476367-c0ee-43db-bcff-5e8bf67ce886" />
<img width="1020" height="269" alt="image" src="https://github.com/user-attachments/assets/d5d3e605-cda4-43c5-94e9-9131b3aee8c8" />

```bash
func sum(a, b int) int {
<<<<<<< HEAD #  Начало конфликтующего блока, HEAD — указывает на текущую ветку (ту,в которой находимся). Всё между <<<<<<< HEAD и ======= — код из ветки development 
        return a + b №
=======
        t := b + a
        return t
>>>>>>> feature # Конец конфликтующего блока, feature — название ветки, которую пытаемся смержить. Всё между ======= и >>>>>>> feature — код из ветки feature
}
```

Выясним, когда были внесены изменения в `main.go` в ветке `development` и `feature`:

```bash
# git log — основная команда для просмотра истории коммитов, -1 — ограничить вывод последним коммитом, --format="%ai %H" — тут %ai показывает author date (пример: 2024-03-15 14:30:45 +0300), %H — полный хеш в коммите, development — имя ветки, историю которой мы смотрим, -- — разделитель указывает Git, что дальше идут пути к файлам, main.go — ограничить вывод только теми коммитами, которые изменяли этот файл
```

```bash
git log -1 --format="%ai %H" development -- main.go
git log -1 --format="%ai %H" feature -- main.go
```

Покажем в терминале:

<img width="1017" height="81" alt="image" src="https://github.com/user-attachments/assets/8ab79748-0a1a-48d9-ab41-c07121ab221b" />
<img width="1016" height="399" alt="image" src="https://github.com/user-attachments/assets/58012fee-6e93-4f92-bd75-e76d9172cb48" />

Устраним конфликт:

```bash
# Берет версию файла из ветки feature и заменяет ей текущий конфликтующий файл main.go. Для Git: --ours = ветка development, где мы находимся; --theirs = ветка feature, которую мы мержим

git checkout --theirs main.go
# Делаем коммит
git commit -m "Merge feature into development: resolved conflict by using feature's version of sum()"
# Глянем текущее состояние
git status
# --oneline — короткий хеш + сообщение коммита, --graph — рисует граф связей между коммитами, --all — показывает все ветки
git log --oneline --graph --all
```

Покажем в терминале:

<img width="1020" height="403" alt="image" src="https://github.com/user-attachments/assets/d4c6dcff-76c5-414c-9259-0d34d6ecfacf" />

Смержим получившуюся ветку `development` в ветку `master`:

```bash
git checkout master
git merge development
git log --oneline --graph --all
```

Покажем в терминале:

<img width="1019" height="358" alt="image" src="https://github.com/user-attachments/assets/d6ab119f-80a4-401a-acfe-3696026d60f6" />



Создадим в своем аккаунте в `gitlab` репозиторий `devops-task-merge` и загрузи все изменения в него:

```bash
git remote add merge https://gitlab.com/lilyayachnik/devops-task-merge.git
# git push — основная команда для отправки изменений на удаленный репозиторий, merge — псевдоним удаленного репозитория, --all — отправить все локальные ветки и историю коммитов
git push merge --all
```

Покажем в терминале:

<img width="1018" height="482" alt="image" src="https://github.com/user-attachments/assets/6b768a36-8277-4e79-973e-1348d22dd3c0" />


