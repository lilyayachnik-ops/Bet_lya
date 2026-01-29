# Checkpoints

Выведем размеры разделов диска в отдельный файл. Отсортируем количество столбцов до трех, оставив только `Filesystem`, `User%`, `Mounted on`:

```bash
#!/bin/bash

RESULT_DF=$1

if [[ -z "$RESULT_DF"  ]]; then
        echo "Please provide path to the file"
        exit 1
fi

echo "Cheking the available space:" 
echo "=========================="

df -h | awk '{printf "%-20s %-10s %-10s\n",$1,$5,$6; print "-----------------"}' &> "$RESULT_DF"

echo "Results saved to: $RESULT_DF"
echo "Content:"
cat "$RESULT_DF"

printf "\n"
echo "Partitions using >80%:"
printf "\n"
df -h | awk '{if ($5+0 >80) {print "Почти " $1 " - " $5 " заполнен"; print "---------------"}}'
```

Далее сделаем скрипт исполняемым, используя команду `sudo chmod 755 script_bash1.sh`. Далее запустим наш скрипт `./script_bash1.sh`, тогда мы получим предупреждение. Если мы `./script_bash1.sh /home/ilya/df.txt`:

<img width="849" height="604" alt="image" src="https://github.com/user-attachments/assets/fdee6dc5-408a-4370-953a-f3ee53fe70ab" />
<img width="848" height="599" alt="image" src="https://github.com/user-attachments/assets/3818b3fb-232b-44e0-a4ea-4e659730d023" />

Узнаем размеры всех файлов и папок в директории /etc. Далее отсортируем вывод так, чтобы показывало только 10 самых больших файлов:

```bash
#!/bin/bash

NAME_DIR=$1

if [[ -z "$NAME_DIR" ]]; then
        echo "Please provide the name of directory"
        exit 1
fi

if [[ ! -d "$NAME_DIR" ]]; then
        echo "Error: Directory $NAME_DIR doesn't exist"
        exit 1
fi

echo "Script is running ..."
printf "\n"
echo "Top 10 largest files in $NAME_DIR:"
echo "=================================="
printf "\n"
sudo find "$NAME_DIR" -type f -exec du -h {} \; | sort -rh | head -n 10 | awk '{print $0; print "---------"}'
```
**_Важно помнить_**
```bash
find "$NAME_DIR" — ищет в директории, переданной в качестве аргумента
-type f — только файлы
-exec du -h {} \; — для каждого найденного файла выполняем du -h
{} — заменяет на путь к найденному файлу
\; — указывает конец команды для выполнения 
```

Аналогично, делаем скрипт исполняемым. Проверяем его работу в терминале

<img width="1015" height="600" alt="image" src="https://github.com/user-attachments/assets/0167eddc-54c5-480c-b3fe-8f01f327507b" />

Нам нужно создать файл со следующим содержанием, сделаем это с помощью команды `cat > nsa.txt`:

```bash
#!/bin/bash
# Создание файла nsa.txt с описанным содержимым
cat << 'EOF' > nsa.txt
NDS/A
NSDA
ANS!D
NAD/A
EOF

# либо

#!/bin/bash
echo "NDS/A" > nsa.txt
echo "NSDA" >> nsa.txt
echo "ANS!D" >> nsa.txt
echo "NAD/A" >> nsa.txt

```
И посмотрим содержимое файла `cat nsa.txt` в терминале:

<img width="791" height="201" alt="image" src="https://github.com/user-attachments/assets/7281ab2a-06d8-4a6f-bca5-906dbbc03a26" />


Вывести строки `NDS/A` и `NAD/A` из файла, используя `awk` и `sed`:

```bash
#!/bin/bash

# Prompt to enter the file name
FILENAME=$1

if [[ -z "$FILENAME"  ]]; then
        echo "Please provide file name."
        exit 1
fi

if [[ ! -f "$FILENAME"  ]]; then
        echo "Please enter the file name."
        exit 1
fi

echo "Contents of file $FILENAME:"
echo "==========================="
cat $FILENAME

printf "\n"
# Prompt to select a tool
echo "Please specify how to find the words. Using awk(1) or sed (2)."
read CHOICE


printf "\n"
echo "The required lines:"
echo "==================="

case $CHOICE in
        1)
               echo "Using awk:"
               awk '$0 == "NDS/A" || $0 == "NAD/A"' $FILENAME
               ;;
        2)
                echo "Using sed:"
                sed -n '/NDS\/A/p; /NAD\/A/p' $FILENAME
                ;;
        *)
                echo "Wrong choice. Using 1 or 2."
                ;;

esac
```

Проверим работу скрипта в терминале:

<img width="790" height="405" alt="image" src="https://github.com/user-attachments/assets/3b5f6fd8-7a60-4bdd-8ec9-240c2f5d4354" />
<img width="795" height="601" alt="image" src="https://github.com/user-attachments/assets/f1099491-f9a2-4d0a-b69e-e75934436035" />

Чтобы вывести пронумерованные строчки из `/etc/passwd`, в которых есть оболочка `/bin/bash`, и перенаправить вывод файл, воспользуемся следующими командами

```bash
awk -F":" '{print $7}' /etc/passwd | grep -n "/bin/bash" > bash.txt
cat bash.txt
```

Проверим в терминале:

<img width="849" height="106" alt="image" src="https://github.com/user-attachments/assets/3f5aeda5-8988-4fc0-9232-8a6159b97734" />

Так как у меня нет изначально `/bin/sh`, то я заменю `/bin/bash` на `/bin/sh`, а затем наоборот. Так как нам нужно проводить изменения в файле `/etc/passwd`, то нам нужно сделать его копию `~/passwd.bak`, поскольку этот файл — содержит список пользовательских учётных записей. Является первым и основным источником информации о правах пользователя ОС.

```bash
#!/bin/bash

FILE_SAVE=$1

if [[ -z "$FILE_SAVE" ]]; then
        echo "Please provide the name of file where will be saved data"
        exit 1
fi

if [[ ! -f "/etc/passwd" ]]; then
        echo "Error: Source file /etc/passwd not found"
        exit 1
fi

echo "Creating a backup of /etc/passwd"
sudo cp /etc/passwd ./passwd.bak

if [[ $? -eq 0 && -f "/home/ilya/passwd.bak"  ]]; then
        echo "Backup successfully created: passwd.bak"
else
        echo "Backup creation failed"
        exit 1
fi

printf "\n"
echo "Outputting lines from /bin/bash to the $FILE_SAVE..."
awk -F":" '{print $7; print "-----------"}' ./passwd.bak | grep -n "/bin/bash" > $FILE_SAVE

if [[ -s "$FILE_SAVE" ]]; then
        echo "Data successfully written to $FILE_SAVE"
else
        echo "No data written to $FILE_SAVE"
fi

echo "Contents of the $FILE_SAVE file:"
echo "================================"
cat $FILE_SAVE
echo ""

echo "Replace /bin/bash with /bin/sh in the passwd.bak..."
sed -i 's/bash/sh/g' ./passwd.bak

echo "Checking the changes:"
echo "===================="
echo "Lines with /bin/sh:"
awk -F":" '{print $7; print "---------"}' ./passwd.bak | grep -n "/bin/sh" > $FILE_SAVE

echo "Contents of the $FILE_SAVE file:"
echo "================================"
cat $FILE_SAVE
echo ""
```
Проверим работу в терминале:

<img width="1053" height="469" alt="image" src="https://github.com/user-attachments/assets/aa29d883-c50c-4315-83eb-e59cc5476297" />

**_ Операции проверки файлов _**:
 - e — файл существует
 - f — обычный файл (не каталог)
 - s — ненулевой размер файла
 - d — файл является каталогом
 - b — файл является блочным устройством
 - c — файл является символьным устройством
 - p — файл является каналом
 - h — файл является символической ссылкой
 - S — файл является сокетом
 - t — является ли файл стандартным устройством ввода (-t 0 ) или вывода ( -t 1 )
 - L — файл являетcя символической ссылкой
 - r — файл доступен для чтения
 - w — файл доступен для записи
 - x — файл доступен для выполнения
 - O, G, N — вы являетесь владельцем файла, вы принадлежите к той же группе, что и файл, файл был модифицирован с момента последнего чтения
 - f1 -nt f2 — файл f1 более новый, чем f2
 - f1 -ot f2 — файл f1 более старый, чем f2
 - f1 -ef f2 — файлы f1 и f2 являются жесткими ссылками на один и тот же файл
 - ! — логическое отрицание

**Тоже нужно запомнить:** `script_name=$(name_of_command)` или `echo $script_name`.
**Тоже нужно запомнить:** 
- "" — не выполняется интерпретация большинства служебных символов в строке, '' — экранирует все служебны символы в строке.
- \X — экранирует символ X, но в арифметических операциях — это оператор деления.
- $? — возвращает код выполнения предыдущей команды.
- &# — количество входящих параметров.  

   
