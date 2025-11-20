# Chekpoints
Поднимем Ubuntu на VirtualBox. Создадим там пользователей Raymond и John, используя следующие команды:

<img width="958" height="641" alt="image" src="https://github.com/user-attachments/assets/a1f76fec-6a07-42c5-83cb-ccf2ea59c097" />


```bash
# sudo useradd -m -d /home/Raymond -U Raymond
# sudo useradd -m -d /home/John -U John

m — создаст домашний каталог пользователя, если он не существует
d — домашний каталог пользователя, в котором он будет размещаться
U — создаст группу с тем же именем, что и пользователя, и сделает её основной группой пользователя
```

Выполним эти команды у себя на VM:

<img width="737" height="162" alt="image" src="https://github.com/user-attachments/assets/f1458db6-cc75-4fa6-9753-a02361ceb8ed" />

**_Важно:_** SSH — сетевой протокол прикладного уровня, позволяющий производить удалённое управление операционной системой и туннелирование TCP-соединений (например, для передачи файлов). Схож по функциональности с протоколами Telnet и rlogin, но, в отличие от них, шифрует весь трафик, включая и передаваемые пароли. SSH допускает выбор различных алгоритмов шифрования. 

Установим SSH-сервер на VM, используя команду:

```bash
# sudo apt update && sudo apt install openssh-server
```
Сервер SSH обозначается `sshd`, а клиент SSH — `ssh`.

**Пользовательские файлы настроек SSH**

- файл `config` — файл настройки клиента. Позволяет настраивать параметры соединений, чтобы не было необходимости каждый раз указывать опции для каждого конкретного сервера.
- файл `authorized_keys` — содержит список ключей для доступа к этому пользователю через сервер `sshd` на этой машине.
- файл `known_hosts` — содержит список ключей для хостов, к которым ранее уже было произведено подключение.
- файлы `id_*` — содержат закрытые ключи доступа для текущего устройства. Вместо `*` в имени файла присутствует имя используемого алгоритма шифрования.
- файлы `id_*.pub`— содержат публичные ключи доступа, которые используются для подключения к другим устройствам. Вместо `*` в имени файла присутствует имя используемого алгоритма шифрования.

Сделаем так, чтобы Raymond заходил только по SSH ключу на инстанс, а John должен будет каждый раз вводить свой пароль при попытки зайти на инстанс и не иметь возможности зайти по ssh ключу. Для этого внесем изменения в файл `sshd_config`. Используем текстовый редкатор `vim /etc/ssh/sshd_config` на удаленном хосте:

```bash
Match User Raymond
PasswordAuthentication no
PubkeyAuthentication yes

Match User John
PasswordAuthentication yes
PubkeyAuthentication no
```

С помощью нажатия клавиши `ESC` выходим из режима набора текста. Нажимаем `Shift+:`, чтобы перейти в командный режим. Вводим `wq` для того, чтобы сохранить изменения и выйти из редактора vim.

Создадим пароль для пользователей Raymond и John с помощью команды `sudo passwd user_name`. Дальше узнаем `IP_address` нашей VM. Для этого введём следующую команду `ip a`:

<img width="726" height="413" alt="image" src="https://github.com/user-attachments/assets/58f0e4db-24c1-4108-8d71-367fd4736c71" />

Сгенерируем приватный и публичный ключи для пользователя John и Raymond на основном хосте:

```bash
# ssh-keygen -t rsa -b 4096 -f ~/.ssh/raymond_key
-t rsa — тип ключа
-b 4096 — длина ключа
-f ~/.ssh/raymond_key — путь к файлу с ключами
```

<img width="867" height="403" alt="image" src="https://github.com/user-attachments/assets/c150abf2-5ed0-41cd-af2e-ed7fc01edefc" />

```bash
# ssh-keygen -t rsa -b 4096 -f ~/.ssh/john_key
```

<img width="867" height="404" alt="image" src="https://github.com/user-attachments/assets/30e15c79-4bd9-4654-9cf6-469801cd1489" />

Прокинем наши публичные ключи на наш хост с использованием команды `scp`:

```bash
scp ~/.ssh/raymond_key.pub Raymond@192.168.0.116:/tmp/raymond_key.pub
```

Выполним в терминале

<img width="872" height="69" alt="image" src="https://github.com/user-attachments/assets/5a89f3dd-2b2b-4e3c-b5f8-89e44f257084" />


```bash
scp ~/.ssh/john_key.pub John@192.168.0.116:/tmp/john_key.pub
```

Выполним в терминале

<img width="866" height="68" alt="image" src="https://github.com/user-attachments/assets/a60d0c29-2712-42da-8bc0-08bd83500313" />

Далее выполним следующие команды:

```bash
# Скопируем публичный ключ в папку authorized_keys
cp /tmp/raymond_key.pub ~/.ssh/authorized_keys
# Меняем владельца и группу папки .ssh и всех файлов внутри неё
sudo chown -R Raymond:Raymond ~/.ssh
# Меняем права доступа, чтобы владелец файла мог r,w,e
sudo chmod 700 ~/.ssh
# Меняем права доступа нашего файла authorized_keys, чтобы владелец мог r,w
sudo chmod 600 ~/.ssh/authorized_keys
# Удалим файл /tmp/raymond_key.pub
rm /tmp/raymond_key.pub
```

Покажем в терминале

<img width="721" height="235" alt="image" src="https://github.com/user-attachments/assets/0f904f6d-1a6b-454b-abde-2ece0f9fbfa6" />

Сделаем то же самое для John

```bash
# Скопируем публичный ключ в папку authorized_keys
cp /tmp/john_key.pub ~/.ssh/authorized_keys
# Меняем владельца и группу папки .ssh и всех файлов внутри неё
sudo chown -R John:John ~/.ssh
# Меняем права доступа, чтобы владелец файла мог r,w,e
sudo chmod 700 ~/.ssh
# Меняем права доступа нашего файла authorized_keys, чтобы владелец мог r,w
sudo chmod 600 ~/.ssh/authorized_keys
# Удалим файл /tmp/john_key.pub
rm /tmp/john_key.pub
```

Покажем в терминале

<img width="727" height="188" alt="image" src="https://github.com/user-attachments/assets/935c5170-ceac-4980-93f5-b295f78efc27" />

Проверим, может ли John подключиться по ключу и паролю:

```bash
# По ключу
ssh -i ~/.ssh/john_key John@192.168.0.122
i — файл идентификации
# По паролю
ssh John@192.168.0.122
```

Проверим в терминале 

<img width="865" height="384" alt="image" src="https://github.com/user-attachments/assets/1b67d16d-92ae-42d8-b4a9-d74e56867b3f" />

Проверим, может ли Raymond подключиться по ключу и паролю:

```bash
# По ключу
ssh -i ~/.ssh/raymond_key Raymond@192.168.0.122
# По паролю
ssh Raymond@192.168.0.122
```

Проверим в терминале

<img width="863" height="201" alt="image" src="https://github.com/user-attachments/assets/b0088cac-f504-46c4-a61b-5eaee46c4fd7" />

Давайте сделаем, чтобы пользователь Raymond имел доступ к sudo, а John — нет:

```bash
# Добавим пользователя Raymond в группу sudo
sudo usermod -a -G sudo Raymond
a — добавить группу, а не заменять
G sudo — добавить в группу sudo
# Посмотрим информацию о пользователе Raymond
id Raymond
# Посмотрим информацию о пользователе John
id John
```

Посмотрим в терминале

<img width="720" height="107" alt="image" src="https://github.com/user-attachments/assets/e1d6dde5-77de-456f-982c-bf873c28699c" />

Зайдем из-под пользователя Raymond с основного хоста, создадим документ. Используя права доступа, сделаем так, чтобы пользователь John не смог редактировать содержимое документа, но смог его прочитать:

```bash
# Зайдем из-под пользователя Raymond
ssh -i ~/.ssh/raymond_key Raymond@192.168.0.122
# Создадим файл и запишем в него какую-нибудь информацию
echo "Never give up" > raymond.txt
# Помяняем права доступа к файлу так, чтобы владелец мог r,w,e, группа — r, остальные — r
chmod 744 raymond.txt
```

Введём в терминале

<img width="873" height="241" alt="image" src="https://github.com/user-attachments/assets/61cbb81b-c020-4ac3-a700-725b5e5feecb" />

Покажем, что всё настроено правильно

<img width="728" height="113" alt="image" src="https://github.com/user-attachments/assets/fb7aa4b5-e89a-4842-8131-55cd74c4fa6c" />

Создадим скрипт `john_script.sh`, выдадим ему следующие права `755`:

```bash
vim john_script.sh
```

```bash
#!/bin/bash
echo "Hello World"
```

```bash
chmod 755 john_script.sh
./john_script.sh
```

Запустим данный скрипт от пользователя Raymond:

<img width="867" height="220" alt="image" src="https://github.com/user-attachments/assets/770b5412-c4e0-41fe-89d8-05adeceb0bff" />

Поменяем пользователю John интерпретатор на bash и создадим нового пользователя, чтобы у него был интерпретатор sh:

```bash
# Переключимся на пользователя root
su -
# Поменяем командную оболочку John
chsh -s /bin/bash John
s — явно указывает, что следующий аргумент - это путь к командной оболочке 
```

<img width="735" height="60" alt="image" src="https://github.com/user-attachments/assets/67c3a04f-6c4e-4409-bd71-796c52e8b745" />

```bash
# Создание нового пользователя с интерпретатором sh
useradd -m -d /home/Daniil -s /bin/sh Dannil
# Выполним проверку
cat /etc/passwd
```

<img width="736" height="42" alt="image" src="https://github.com/user-attachments/assets/2feb8ab8-002a-4bc4-be01-925758cf1627" />
<img width="728" height="38" alt="image" src="https://github.com/user-attachments/assets/cd2e8902-13ad-41f1-9e12-0170f2806526" />

Добавим Daniil в группу John, чтобы проверить, сможет ли он запустить john_script.sh:

```bash
# Добавим Daniil в группу John
usermod -a -G John Daniil
# Проверим, к каким группа принадлежит Daniil
groups Daniil
```

<img width="725" height="75" alt="image" src="https://github.com/user-attachments/assets/05a9f9bf-d25e-409b-bc1d-223ce38f203d" />

Запустим скрипт `john_script.sh` от имени Daniil:

<img width="723" height="111" alt="image" src="https://github.com/user-attachments/assets/c70f747d-e45a-4d72-b81b-12488feecdf3" />

Создадим файл, закинем его на VM:

```bash
echo "I will be a champion" > ilya.txt
scp -i ~/.ssh/raymond_key ~/ilya.txt Raymond@192.168.0.122:/home/Raymond
```

<img width="868" height="63" alt="image" src="https://github.com/user-attachments/assets/fb4276b8-41a4-4af2-98bc-433ee4cf9464" />
