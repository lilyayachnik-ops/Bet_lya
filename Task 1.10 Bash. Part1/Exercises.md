# Checkpoints

Нужно написать скрипт, выводящий в файл (имя файла задаётся пользователем в качестве первого аргумента командной строки) имена всех файлов с заданным расширением (третий аргумент командной строки) из заданного каталога (имя каталога задаётся пользователем в качестве второго аргумента командной строки).


```bash
#!/bin/bash

FILE_NAME=$1
DIR_NAME=$2
EXTEN=$3

if [[ $# -ne 3 ]]; then
        echo "Usage: $0 <output_file> <directory> <extension>"
        exit 1
fi

if [[ -z "$FILE_NAME" ]]; then
        echo "Please enter the name of the file where we will save the result"
        exit 1
fi

if [[ -z "$DIR_NAME" ]]; then
        echo "Please enter the name of the diectory where we will search"
        exit 1
fi

if [[ ! -d "$DIR_NAME" ]]; then
        echo "Please enter the correct name of the directory where we will search"
        exit 1
fi

if [[ -z "$EXTEN" ]]; then
        echo "Please enter the extension that we will search for in the files"
        exit 1
fi

echo "Starting the searching"
printf "\n"

if find "$DIR_NAME" -type f -name "*.${EXTEN}" > "$FILE_NAME"; then
       echo "The searching completed successfully"
else
       echo "The searching didn't complete successfully"
fi
printf "\n"
echo "Output the contents of the file"
printf "\n"
cat "$FILE_NAME"
```

Проверим в терминале:

<img width="741" height="304" alt="image" src="https://github.com/user-attachments/assets/8a1257e9-9d38-485b-bead-7b2b6aa2f5eb" />
