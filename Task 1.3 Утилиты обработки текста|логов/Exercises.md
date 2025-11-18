# Checkpoints

Выведем размеры разделов диска в отдельный файл. Отсортируем количество столбцов до трех, оставив только Filesystem, User%, Mounted on:

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
NDS/A
NSDA
ANS!D
NAD/A
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





