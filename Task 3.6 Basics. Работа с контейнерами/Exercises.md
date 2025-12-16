# Checkpoints

Запусти контейнер со следующими параметрами: должно работать в фоне, имеет имя `inno-dkr-05-run-X`, где `X` — набор из `10` случайных букв и/или цифр которая должна генерироваться в момент запуска контейнера (можно использовать команду `cat /dev/urandom | tr -cd 'a-f0-9' | head -c 10`),
образ — `nginx:stable`, команда для запуска обоих контейнеров должна быть одинаковой (выполнить одинаковую команду два раза подряд). Запусти контейнер со следующими параметрами: должно работать в фоне, имеет имя `inno-dkr-05-stop`, образ — `nginx:stable`:

```bash
# cat /dev/urandom — читает поток случайных байтов,
# tr -cd 'a-f0-9', где -c — действует на символы, которые не указаны в наборе, -d — удаляет указанные символы
# head -c 10 — берёт первые 10 символов
# Удаляет все символы, которые не являются a-f,0-9.

$ docker run --name inno-dkr-05-$(cat /dev/urandom | tr -cd 'a-f0-9' | head -c 10) nginx:stable
$ docker run --name inno-dkr-05-$(cat /dev/urandom | tr -cd 'a-f0-9' | head -c 10) nginx:stable

docker run -d --name inno-dkr-05-stop nginx:stable
```
Покажем в терминале:

<img width="1417" height="47" alt="image" src="https://github.com/user-attachments/assets/a98af058-fc9b-4e0b-98cb-2d1660a68180" />
<img width="1490" height="46" alt="image" src="https://github.com/user-attachments/assets/5aa4413f-b090-4a75-be10-814d1d1de54e" />
<img width="1538" height="48" alt="image" src="https://github.com/user-attachments/assets/1c288034-772c-4e3f-b918-aa3e306280be" />
<img width="1564" height="114" alt="image" src="https://github.com/user-attachments/assets/99b7848b-6a40-4565-80ab-072ca3fe9cb6" />

Выполни команду `docker ps`, вывод перенаправь в файл `/home/user/ps.txt` (`docker ps | tee /home/user/ps.txt`). Останови контейнер `inno-dkr-05-stop`. Выведи список всех контейнеров. Одной командой останови все запущенные контейнеры. Выведи список всех контейнеров. Одной командой удали все контейнеры, любой из разобранных методов подходит:

```bash
# tee ~/ps.txt —  записывает результат команды docker ps в файл ~/ps.txt, параллельно выводит эти данные на экран
# tee [options] [file]
# Options:
# -a/-append — используется для записи вывода в конец существующего файла
# -i/-ignore-interrupts — используется, чтобы игнорировать превышающие сигналы
# -help — справочник
# -version — показывает текущую версию команды 

$ docker ps | tee ~/ps.txt 

# docker stop inno-dkr-05-stop — останавливает запущенный контейнер inno-dkr-05-stop.
# Сначала отправляет сигнал SIGTERM процессу внутри контейнера.
# По умолчанию, 10 секунд на корректное завершение.
# Если процесс не завершился, отправляется сигнал SIGKILL.

$ docker stop inno-dkr-05-stop
docker ps

# docker ps -q — получает список ID всех запущенных контейнеров.
# Результат подставляется в docker stop.
# Останавливаются все контейнеры.

$ docker stop $(docker ps -q)

# docker ps -aq — получает список ID всех контейнеров.
# Результат подставляется в docker rm, который удаляет контейнеры по ID.
# Удаляются все контейнеры

$ docker rm $(docker ps -aq)
```

Покажем в терминале: 

<img width="1467" height="311" alt="image" src="https://github.com/user-attachments/assets/92a2362c-bed1-4a7b-b4f8-fcedbb1994d7" />
<img width="1204" height="244" alt="image" src="https://github.com/user-attachments/assets/485eb489-74d0-41ce-9961-58cbbe09fd75" />
