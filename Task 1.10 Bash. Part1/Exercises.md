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

Написать скрипт для поиска заданной пользователем строки во всех файлах заданного каталога и всех его подкаталогах (строка и имя каталога задаются пользователем в качестве первого и второго аргумента командной строки). На консоль выводятся полный путь и имена файлов, в содержимом которых присутствует заданная строка, и их размер. Если к какому-либо каталогу нет доступа, необходимо вывести соответствующее сообщение и продолжить выполнение:

```bash
#!/bin/bash

# Store command line arguments in variables
STRING_FIND=$1  # First argument - string to search for
DIR_NAME=$2     # Second argument - directory to search in

# Check if exactly 2 arguments are provided
if [[ $# -ne 2 ]]; then
        echo "Usage: $0 <string_searching> <directory>"
        exit 1  # Exit with error code 1
fi

# Check if search string is not empty
if [[ -z "$STRING_FIND" ]]; then
        echo "Please enter the search string"
        exit 1
fi

# Check if directory name is not empty
if [[ -z "$DIR_NAME" ]]; then
        echo "Please enter the name of the directory where we will search"
        exit 1
fi

# Check if the directory actually exists
if [[ ! -d "$DIR_NAME" ]]; then
        echo "Please enter the correct name of the directory where we will search"
        exit 1
fi

# Recursively search for the string in all files within the directory
# grep -r searches through all files in the specified directory and its subdirectories
# Results are passed line by line to the while loop
grep -r "$STRING_FIND" "$DIR_NAME" | while read -r LINE; do
   
   # Check if the current line is a "Permission denied" error message
   if [[ "$LINE" == *"Permission denied"* ]]; then
           # Output error message to stderr stream (for errors)
           echo "No access: $LINE" >&2
   
   else
           # If it's not an error, it's a found match
           # Line format: "path/to/file:found_text"
           
           # Extract file path (everything before the first colon)
           FILE=$(echo "$LINE" | cut -d":" -f1)
           
           # Get file size in bytes
           # stat --printf='%s' outputs only the file size
           # 2>/dev/null hides possible file access errors
           SIZE=$(stat --printf='%s' "$FILE" 2>/dev/null)
           
           # Output information about the found file:
           # - file path
           # - file size in bytes
           # - found text (everything after the first colon)
           echo "File: $FILE, Size: $SIZE byte, String: ${LINE#*:}"
   fi
done
```

Проверим в терминале:

<img width="999" height="567" alt="image" src="https://github.com/user-attachments/assets/3ee9951a-1171-4224-bff1-10f65074d60f" />

Скрипт для генерации:

```bash
#!/bin/bash

# Create directories for testing
# -p flag prevents errors if directories already exist
mkdir -p mytexts myreports "logs_system"

# Array of fruit names to use for generating random content
fruits=("orange" "apple" "banana" "melon" "lemon")

# Function to generate files with fruit content
# Takes directory name as parameter
generate_files() {
    # Local variable to store directory name (only visible within function)
    local dir=$1
    
    # Loop to create 4 files in each directory
    for i in {1..4}; do
        # Construct file path: directory/file1.txt, directory/file2.txt, etc.
        file="$dir/file$i.txt"
        
        # Create file with initial header line (overwrite if exists)
        echo "This is file $i in directory $dir." > "$file"
        
        # Add 5 random fruit-related lines to the file
        for j in {1..5}; do
            # Select random fruit from array
            # $RANDOM generates random number, % gives remainder when divided by array length
            # This ensures index stays within array bounds (0-4)
            fruit=${fruits[$RANDOM % ${#fruits[@]}]}
            
            # Append fruit line to file
            echo "Fruit line $j: I really like $fruit." >> "$file"
        done
    done
}

# Generate files in each directory by calling the function
generate_files "mytexts"
generate_files "myreports" 
generate_files "logs_system"

# Print success message
echo "Folders and files with fruit content have been created."
```

У нас есть лог-файл, имитирующий логи сервера. Необходимо извлечь все IP-адреса, с которых произошел успешный вход (код ответа 200). Нужно извлечь все уникальные пользователи (usernames), которые пытались авторизоваться, но получили ошибку (код ответа 403), то есть написать Bash-скрипт, который:

1) Находит все IP-адреса с кодом статуса 200.

2) Находит всех уникальных пользователей, у которых был статус 403.

Пример вывода `Bash`-скрипта:

```bash
Successful logins (IP addresses):
192.168.1.10
172.16.0.1
10.0.0.5

Users with failed logins:
johndoe
alice
charlie
```

Вот наш `Bash`-скрипта:

```bash
#!/bin/bash

echo "Successful logins (IP addresses):"
awk '/status=200/{print $5}' server.log | cut -d= -f2 | sort -u

printf "\n"
echo "Users with failed logins:"
awk '/status=403/{print $4}' server.log | cut -d= -f2 | sort -u
```

Проверим в терминале:

<img width="522" height="200" alt="image" src="https://github.com/user-attachments/assets/e273a818-d041-4297-b1d2-5c75a68fc020" />


