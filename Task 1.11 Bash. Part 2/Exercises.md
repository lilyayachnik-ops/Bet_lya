# Checkpoints

Выведем размеры разделов диска в отдельный файл. Отсортируем количество столбцов до трех, оставив только `Filesystem`, `User%`, `Mounted on`:

```bash
#!/bin/bash

# Function to check disk space
# Declares a function named check_disk_space
check_disk_space() {
        # Declare local variables that are only visible inside this function
        # $1 - first argument passed to the function (output file path)
        # $2 - second argument passed to the function (threshold percentage)
        local output_file="$1"
        local threshold="$2"

        echo "Checking the available space:"
        echo "============================="
        # Command that does the main work
        # 1. df -h - shows disk space information in human-readable format
        # 2. | - passes the output of df command to awk command
        # 3. awk - text processing tool
        # '{printf "%-20s %-10s %-10s\n", $1,$5,$6; print "-------------------"}'
        # - printf formats the output:
        # %-20s - first field ($1) with width 20 characters, left-aligned
        # %-10s - fifth field ($5) with width 10 characters
        # %-10s - sixth field ($6) with width 10 characters
        # \n - newline character
        # - print "--------------" prints a separator line
        # 4. &> redirects both standard output (stdout) and standard error (stderr)
        # 5. "$output_file" - file where the output will be saved
        df -h | awk '{printf "%-20s %-10s %-10s\n",$1,$5,$6; print "----------------"}' &> "$output_file"

        echo "Results saved to: $output_file"
        echo "Content:"
        cat "$output_file"
        printf "\n"

        echo "Partitions using >${threshold}%:"
        printf "\n"

        # Second disk space check commant
        # 1. df -h - get disk information again
        # 2. awk -v thr="$threshold" - pass threshold variable value to awk as "thr" variable
        # 3. '{if ($5+0 > thr) - check the fifth field (usage percentage)
        # $5+0 - converts string (e.g "45%") to number (45)
        # > thr - compare with threshold
        # 4. {print "Почти " $1 " - " $5 " заполнен"; print "------------")
        # If condition is true, print warning message and separator
        df -h | awk -v thr="$threshold" '{if ($5+0 > thr) {print "Почти " $1 " - " $5 " заполнен"; print "-------------"}}'

}

#End of check_disk_space function

#Main function with parameter processing
# main function - entry point of the script
main() {
        # NO default values - both parametres are required
        # Declare local variables to store parametres
        # They are empty initially - user must specify them
        local output_file=""
        local threshold=""

        # Process named parametres
        # while loop will execute as long as there are arguments ($# - number of arguments)
        # $# - special variable containing count of passed arguments
        while [[ $# -gt 0 ]]; do
                # case construct to compare first argument with various patterns
                case $1 in
                        # Handle -f parameter or its long version
                        -f|--file)
                        # Check that a value follows -f
                        # [[ -z "$2" ]] - checks if string is empty
                        # [[ "$2" == -* ]] - checks if next argument starts with dash
                        if [[ -z "$2" ]] || [[ "$2" == -* ]]; then
                                echo "Error: -f requires a file path"
                                exit 1
                        fi
                        # Save the value to output_file variable
                        output_file="$2"
                        # Remove two processed arguments from the list
                        # shift 2 remove $1 and $2, shifting remaining arguments
                        shift 2
                        ;;

                        # Handle -t parameter or --threshold
                        -t|--threshold)
                        # Similar check for value presence
                        if [[ -z "$2" ]] || [[ "$2" == -* ]]; then
                               echo "Error: -t requires a numeric value"
                               exit 1
                        # Additional check: value must consist only of digits
                        # =~ - regex matching operator
                        # ^[0-9]+$ - regular expression: from start (^) to end ($) only digits ([0-9]) one or more (+)
                        elif ! [[ "$2" =~ ^[0-9]+$ ]]; then
                                echo "Error -t requires a numeric value..."
                                exit 1
                        fi
                        # Save the value to threshold variable
                        threshold="$2"
                        shift 2
                        ;;

                        # Handle help parameter -h or --help
                        -h|--help)
                        # Print detailed usage information
                        echo "Usage: $0 -f FILE -t THRESHOLD"
                        printf "\n"
                        echo "Required options:"
                        echo " -f, --file FILE Output file path"
                        echo "-t, --threshold NUM Alert threshold percentage"
                        printf "\n"
                        echo "Optional:"
                        echo "-h, --help Show this help message"
                        printf "\n"
                        echo "Description:"
                        echo "Saves disk usage info (Filesystem, Use%, Mounted on) to a file"
                        echo "Shows partitions with usage above threshold"
                        printf "\n"
                        echo "Examples:"
                        echo "$0 -f ~/disk_info.txt -t 80"
                        echo "$0 --file /var/log/df.log --threshold 90"
                        echo "$0 -f ./output.txt -t 75"
                        exit 0
                        ;;

                        # Handle any other paametres
                        # * - matches any value
                        *)
                        echo "Unknown option: $1"
                        echo "Use $0 --help for usage information"
                        exit 1
                        ;;
        esac
done
# After processing all parametres, verify that both required parametres were specified
# Check: was -f parameter specified (output_file not empty)
if [[ -z "$output_file" ]]; then
        echo "Error: Output file not specified (use -f option)"
        exit 1
fi

# Check: was -t parameter specified (threshold not empty)
if [[ -z "$threshold" ]]; then
        echo "Error: Threshold not specified (use -t option)"
        exit 1
fi

# Check directory for the output file
# dirname extracts directory path from full file path
local dir_name=$(dirname "$output_file")
if [[ ! -d "$dir_name" ]] && [[ "$dir_name" != "." ]]; then
        echo "Error:Directory '$dir_name' does not exist"
        exit 1
fi

# Call the main disk check function
# Pass two arguments: file path and threshold
check_disk_space "$output_file" "$threshold"
}

# Execute the main function
# "$@" - special variable containing all command line arguments
main "$@"
```

Проверим в терминале:

<img width="894" height="365" alt="image" src="https://github.com/user-attachments/assets/9b443790-51ca-4a08-a034-ca479db474dd" />
<img width="894" height="618" alt="image" src="https://github.com/user-attachments/assets/3feeae58-4cd2-411d-b698-d560c8ee04af" />
<img width="893" height="476" alt="image" src="https://github.com/user-attachments/assets/9b6f740c-552e-4d5b-9b4a-48ff227e6e1e" />

Узнать размеры всех файлов и папок в директории `/etc`. Далее отсортировать вывод так, чтобы показывало только 10 самых больших файлов:

```bash
#!/bin/bash

# Function to find top largest files
find_largest_files() {
        local directory="$1"
        local count="$2"

        echo "Script is running ..."
        printf "\n"
        echo "Top $count largest files in '$directory':"
        echo "========================================="
        printf "\n"

        sudo find "$directory" -type f -exec du -h {} \; | sort -rh | head -n "$count" | awk '{print $0; print "--------"}'

}

show_help() {
        echo "Usage: $0 -d DIRECTORY [-c COUNT]"
        printf "\n"
        echo "Options:"
        echo "-d, --directory DIR Directory to scan"
        echo "-c, --count NUM Number of files to show (default: 10)"
        echo "-h, --help Show this help message"
        printf "\n"
        echo "Examples:"
        echo "$0 -d /etc"
        echo "$0 --directory /var/log --count 20"
        echo "$0 -d /home -c 5"
        exit 0
}

main() {
        local directory=""
        local count=10

        # Parse command line arguments
        while [[ $# -gt 0 ]]; do
                case $1 in
                        -d|--directory)
                                if [[ -z "$2" ]] || [[ "$2" == -* ]]; then
                                       echo "Error: -d requires a directory path"
                                       exit 1
                                fi
                                directory="$2"
                                shift 2
                                ;;

                        -c|--count)
                                if [[ -z "$2" ]] || [[ "$2" == -* ]]; then
                                       echo "Error: -c requires a numeric value"
                                       exit 1
                                elif ! [[ "$2" =~ ^[0-9]+$ ]]; then
                                       echo "Error: Count must be a positive number"
                                       exit 1
                                fi
                                count="$2"
                                shift 2
                                ;;

                        -h|--help)
                                show_help
                                ;;

                        *)
                                echo "Unknown option: $1"
                                echo "Use $0 --help for usage information"
                                exit 1
                                ;;
                esac
        done

        if [[ -z "$directory" ]]; then
                echo "Error: Directory not specified (use -d option)"
                echo "Use $0 --help for usage information"
                exit 1
        fi

        # Check if directory exists
        if [[ ! -d "$directory" ]]; then
                echo "Error: Directory '$directory' does not exist"
                exit 1
        fi

        # Check if user has permission to access directory
        if [[ ! -r "$directory" ]]; then
                echo "Warning: Limited access to directory '$directory'"
                echo "Some files may not be accessible"
                printf "\n"
        fi

        # Call the main function
        find_largest_files "$directory" "$count"
}

# Execute main function
main "$@"
```

Проверим в терминале:

<img width="894" height="544" alt="image" src="https://github.com/user-attachments/assets/f4fd7d64-0119-442d-ac0b-eef0e988b67f" />
<img width="894" height="522" alt="image" src="https://github.com/user-attachments/assets/1b4e38b9-a120-4a7d-88b4-f3f1ca6b71b4" />
<img width="893" height="543" alt="image" src="https://github.com/user-attachments/assets/a277db7c-fc05-4017-bcac-8a913f7661d6" />

Нам нужно создать файл со следующим содержанием, сделаем это с помощью команды `cat > nsa.txt`:

```bash
NDS/A
NSDA
ANS!D
NAD/A
```

Вывести строки `NDS/A` и `NAD/A` из файла, используя `awk` и `sed`:

```bash
#!/bin/bash

# Function to find lines using awk
# awk compares each entire line ($0) with search values
find_with_awk() {
        # Declare local variable for filename
        local filename="$1"
        echo "Using awk:"
        awk '$0 == "NDS/A" || $0 == "NAD/A"' "$filename"
}

# Function to find lines using sed
# sed uses regular expressions for searching
find_with_sed() {
        local filename="$1"
        echo "Using sed:"
        sed -n '/NDS\/A/p; /NAD\/A/p' "$filename"
}

# Function to display script usage help
show_help() {
        echo "Usage: $0 -f FILE [-t TOOL]"
        printf "\n"
        echo "Options:"
        echo "-f, --file FILE Input file name"
        echo "-t, --tool NUM Tool to use: 1 for awk, 2 for sed"
        echo "-h, --help Show this help message"
        printf "\n"
        echo "Examples:"
        echo "$0 -f nsa.txt #Will ask for tool choice"
        echo "$0 --file data.txt -t 1 #Use awk automatically"
        echo "$0 -f input.txt --tool 2 #Use sed automatically"
        exit 0
}

# Function to display file contents
show_file_contents() {
        local filename="$1"
        echo "Contents of file $filename:"
        echo "==========================="
        cat "$filename"
        printf "\n"
}

#  Main function of the script
main() {
        # Declare local variables
        local filename=""
        local tool=""
        local choice=""
        # Process command line arguments
        while [[ $# -gt 0 ]]; do
                case $1 in
                        # Process -f or --file parameter
                        -f|--file)
                                if [[ -z "$2" ]] || [[ "$2" == -* ]]; then
                                        echo "Error: -f requires a file name"
                                        exit 1
                                fi
                                filename="$2"
                                shift 2
                                ;;
                        # Process -t or --tool parameter
                        -t|--tool)
                                if [[ -z "$2" ]] || [[ "$2" == -* ]]; then
                                        echo "Error: -t requires a value (1 for awk, 2 for sed)"
                                        exit 1
                                # Check that value is 1 or 2
                                elif ! [[ "$2" =~ ^[12]$ ]]; then
                                        echo "Error: Tool must be 1 (awk) or 2 (sed)"
                                        exit 1
                                fi
                                tool="$2"
                                shift 2
                                ;;
                        # Process help parameter
                        -h|--help)
                                show_help
                                ;;

                        # Process unknown parameter
                        *)
                                echo "Unknown parameter: $1"
                                echo "Use $0 --help for usage information"
                                exit 1
                                ;;
                esac
        done

        # Check if filename was specified
        if [[ -z "$filename" ]]; then
                echo "Error: File not specified (use -f option)"
                exit 1
        fi

        # Check if file exists
        if [[ ! -f "$filename" ]]; then
               echo "Error: File '$filename' does not exist"
               exit 1
        fi

        show_file_contents "$filename"

        # if tool wasn't specified via -t parameter, ask user
        if [[ -z "$tool" ]]; then
                echo "Please specify how to find the words. Using awk(1) or sed(2)."
                read -r choice

                # Validate user input (should be 1 or 2)
                if ! [[ "$choice" =~ ^[12]$ ]]; then
                       echo "Wrong choice. Using default: awk"
                       choice=1
                fi
        else
               choice="$tool"
        fi
        printf "\n"
        echo "The required lines:"
        echo "==================="

        # Perform search based on selected tool
        case $choice in
                1)
                        find_with_awk "$filename"
                        ;;
                2)
                        find_with_sed "$filename"
                        ;;
                *)
                        echo "Invalid tool choice"
                        exit 1
                        ;;
        esac
}

main "$@"
```

Проверим в терминале:

<img width="894" height="81" alt="image" src="https://github.com/user-attachments/assets/40c7eb27-48a0-4780-9959-77c5a69444c5" />
<img width="893" height="124" alt="image" src="https://github.com/user-attachments/assets/a3f828bb-d297-485e-ab98-c7d6f82b7957" />
<img width="887" height="560" alt="image" src="https://github.com/user-attachments/assets/b48670a5-c5d9-4910-b3ef-4c384a017f69" />
<img width="893" height="644" alt="image" src="https://github.com/user-attachments/assets/0b4d8445-c9f1-46ae-bf93-ac4851cb8622" />
<img width="891" height="346" alt="image" src="https://github.com/user-attachments/assets/b0cb361c-63d3-40f5-9362-bb04b38cc735" />

Вывести пронумерованные строчки из /etc/passwd, в которых есть оболочка /bin/bash, и перенаправить вывод в файл.

Так как у меня нет изначально `/bin/sh`, то я заменю `/bin/bash` на `/bin/sh`, а затем наоборот. Так как нам нужно проводить изменения в файле `/etc/passwd`, то нам нужно сделать его копию `~/passwd.bak`, поскольку этот файл — содержит список пользовательских учётных записей. Является первым и основным источником информации о правах пользователя ОС:

```bash
#!/bin/bash

# Default values
FILE_SAVE=""
BACKUP_FILE="passwd.bak"
VERBOSE=false

# Parsed named parametres

while [[ $# -gt 0 ]]; do
        case $1 in
                -o|--output)
                        FILE_SAVE="$2"
                        shift 2
                        ;;
                -b|--backup)
                        BACKUP_FILE="$2"
                        shift 2
                        ;;
                -v|--verbose)
                        VERBOSE=true
                        shift
                        ;;
                -h|--help)
                        echo "Usage: $0 -o <output_file> [options]"
                        echo "Options:"
                        echo "-o, --output FILE Output file for results"
                        echo "-b, --backup FILE Backup file name (default: passwd.bak)"
                        echo "-v, --verbose Enable verbose output"
                        echo "-h, --help Show this help message"
                        exit 0
                        ;;
                *)
                        echo "Unknown parameter: $1"
                        echo "Use $0 -h for help"
                        exit 1
                        ;;
        esac
done

# Validation
check() {
        if [[ -z "$FILE_SAVE" ]]; then
                echo "Error: Output file is requeired!"
                echo "Usage: $0 -o <output_file>"
                exit 1
        fi

        if [[ ! -f "/etc/passwd" ]]; then
                echo "Error: /etc/passwd file not found!"
                exit 1
        fi
}

# Create backup
backup() {
        echo "Copying /etc/passwd to '$BACKUP_FILE'..."
        sudo cp /etc/passwd "$BACKUP_FILE"

        if [[ $? -ne 0 ]]; then
                echo "Error: Backup failed!"
                exit 1
        else
                echo "Backup created successfully"
        fi
}

# Find shell users
find_shell() {
        local shell_name="$1"
        local output_file="$2"
        local description="$3"

        if [[ "$VERBOSE" = true ]]; then
                echo "Searching for: $description"
        fi

        # grep searches lines, -n adds line numbers
        grep -n "$shell_name" "$BACKUP_FILE" > "$output_file"

        if [[ -s "$output_file" ]]; then
                local count=$(wc -l < "$output_file")
                echo "Found $count lines with $shell_name"

                if [[ "$VERBOSE" = true ]]; then
                        echo "Contents of '$output_file':"
                        cat "$output_file"
                fi
        else
                echo "No lines found with $shell_name"
        fi
        printf "\n"
}

# Replace shell
replace() {
        local from="$1"
        local to="$2"

        if [[ "$VERBOSE" = true ]]; then
                echo "Executing: sed -i 's|$from|$to|g' '$BACKUP_FILE'"
        fi

        echo "Replacing $from with $to"
        sed -i "s|$from|$to|g" "$BACKUP_FILE"
}

# Validate parameters
check

# Create backup
backup

# Find /bin/bash and save results
find_shell "/bin/bash" "$FILE_SAVE" "users with /bin/bash"

# Replace bash with sh
replace "/bin/bash" "/bin/sh"

# Find /bin/sh after replacement
find_shell  "/bin/sh" "${FILE_SAVE}_sh" "users with /bin/sh (after replacement)"

# Restore original
replace "/bin/sh" "/bin/bash"

# Verify restoration
find_shell "/bin/bash" "${FILE_SAVE}_final" "users with /bin/bash (after restoration)"

echo "=== SUMMARY ==="
ls -la "$FILE_SAVE"* 2>/dev/null || echo "No output files found"
printf "\n"
echo "Backup file: '$BACKUP_FILE'"
```

Покажем в терминале:

<img width="811" height="207" alt="image" src="https://github.com/user-attachments/assets/2ac2da1c-9bde-4eb1-9bf4-f31150dde15b" />
<img width="809" height="339" alt="image" src="https://github.com/user-attachments/assets/7ebbade1-b6ea-48df-89af-5329b6458885" />
<img width="811" height="344" alt="image" src="https://github.com/user-attachments/assets/335e31b7-8bbd-4b0f-861e-7a858eea6724" />
<img width="808" height="546" alt="image" src="https://github.com/user-attachments/assets/77b3b560-7044-4609-8829-e6b08ef2e908" />
<img width="811" height="119" alt="image" src="https://github.com/user-attachments/assets/15f8ffaf-97bf-4bae-8bca-f811b2a5cbc6" />

Нужно написать скрипт, выводящий в файл (имя файла задаётся пользователем в качестве первого аргумента командной строки) имена всех файлов с заданным расширением (третий аргумент командной строки) из заданного каталога (имя каталога задаётся пользователем в качестве второго аргумента командной строки):

```bash
#!/bin/bash

FILE_NAME=""
DIR_NAME=""
EXTEN=""

show_usage() {
        echo "Usage: $0 [-o output_file] [-d directory] [-e extension]"
        echo "$0 <output_file> <directory> <extension>"
        exit 1
}

validate_arguments() {

        # Check filename
        if [[ -z "$FILE_NAME" ]]; then
                echo "Error: Please enter the name of the file where we will save the result"
                exit 1
        fi

        # Check directory name
        if [[ -z "$DIR_NAME" ]]; then
                echo "Error: Please enter the name of the directory where we will search"
                exit 1
        fi

        # Check if directory exists
        if [[ ! -d "$DIR_NAME" ]]; then
                echo "Error: Directory '$DIR_NAME' does not exist or is not a directory"
                exit 1
        fi

        # Check extension
        if [[ -z "$EXTEN" ]]; then
                echo "Error: Please enter the extension that we will search for in the files"
                exit 1
        fi
}

parse_arguments() {

        # If first argument is not an option, treat as positional arguments
        if [[ $# -eq 3 ]] && [[ "$1" != -* ]]; then
                FILE_NAME="$1"
                DIR_NAME="$2"
                EXTEN="$3"
                return
        fi

        # Otherwise parse as named arguments
        while [[ $# -gt 0 ]]; do
                case "$1" in
                        -o|--output)
                                if [[ -z "$2" ]] || [[ "$2" == -* ]]; then
                                        echo "Error: $1 requires a file path"
                                        show_usage
                                fi
                                FILE_NAME="$2"
                                shift 2
                                ;;
                        -d|--directory)
                                if [[ -z "$2" ]] || [[ "$2" == -* ]]; then
                                        echo "Error: $1 requires a directory path"
                                        show_usage
                                fi
                                DIR_NAME="$2"
                                shift 2
                                ;;
                        -e|--extension)
                                if [[ -z "$2" ]] || [[ "$2" == -* ]]; then
                                        echo "Error: $1 requires an extension"
                                        show_usage
                                fi
                                EXTEN="$2"
                                shift 2
                                ;;
                        -h|--help)
                                show_usage
                                ;;
                        *)
                                echo "Unknown option: $1"
                                echo "Use $0 --help for usage information"
                                exit 1
                                ;;
                esac
        done
}

search_files() {
        echo "Starting the search"
        printf "\n"

        if find "$DIR_NAME" -type f -name "*.${EXTEN}" > "$FILE_NAME"; then
                echo "Search completed successfully"
        else
                echo "Search did not complete successfully"
                exit 1
        fi
}
show_results() {
        printf "\n"
        echo "Output the contents of the file"

        if [[ -s "$FILE_NAME" ]]; then
                cat "$FILE_NAME"
        else
                echo "No files found with extension .$EXTEN"
        fi
}

main() {
        parse_arguments "$@"
        validate_arguments
        search_files
        show_results
}

main "$@"
```

Покажем в терминале:

<img width="804" height="423" alt="image" src="https://github.com/user-attachments/assets/f00a417a-27d9-4c0f-b928-7e8329b2d6bd" />
<img width="814" height="425" alt="image" src="https://github.com/user-attachments/assets/ec71f29d-225a-461a-b5af-9cb7d8a3c42f" />
<img width="809" height="100" alt="image" src="https://github.com/user-attachments/assets/9af23530-fd1a-4a12-9030-a088fd46e9f9" />

Написать скрипт для поиска заданной пользователем строки во всех файлах заданного каталога и всех его подкаталогах (строка и имя каталога задаются пользователем в качестве первого и второго аргумента командной строки). На консоль выводятся полный путь и имена файлов, в содержимом которых присутствует заданная строка, и их размер. Если к какому-либо каталогу нет доступа, необходимо вывести соответствующее сообщение и продолжить выполнение:

```bash
#!/bin/bash

# Global variables
SEARCH_STRING=""
SEARCH_DIR=""

# Function to display usage
show_usage() {
        echo "Usage: $0 [-s|--string <search_string>] [-d|--directory <directory>]"
        echo "$0 <search_string> <directory>"
        printf "\n"
        echo "Options:"
        echo "-s, --string String to search for"
        echo "-d, --directory Directory to search in"
        echo "-h, --help Show this help message"
        exit 1
}

# Function to validate arguments
validate_arguments() {
        # Check if search string is not empty
        if [[ -z "$SEARCH_STRING" ]]; then
                echo "Error: Please enter the search string"
                show_usage
        fi

        # Check if directory name is not empty
        if [[ -z "$SEARCH_DIR" ]]; then
                echo "Error: Please enter the name of the directory where we will search"
                show_usage
        fi

        # Check if the directory actually exists
        if [[ ! -d "$SEARCH_DIR" ]]; then
                echo "Error: Directory '$SEARCH_DIR' does not exist or is not a directory"
                exit 1
        fi
}

# Function to parse command line arguments
parse_arguments() {
        # If first argument is not an option, treat as positional arguments
        if [[ $# -eq 2 ]] && [[ "$1" != -* ]]; then
                SEARCH_STRING="$1"
                SEARCH_DIR="$2"
                return
        fi

        # Otherwise parse as named arguments
        while [[ $# -gt 0 ]]; do
                case "$1" in
                        -s|--string)
                                if [[ -z "$2" ]] || [[ "$2" == -* ]]; then
                                        echo "Error: $1 requires a search string"
                                        show_usage
                                fi
                                SEARCH_STRING="$2"
                                shift 2
                                ;;
                        -d|--directory)
                                if [[ -z "$2" ]] || [[ "$2" == -* ]]; then
                                        echo "Error: $1 requires a directory path"
                                        show_usage
                                fi
                                SEARCH_DIR="$2"
                                shift 2
                                ;;
                        -h|--help)
                                show_usage
                                exit 0
                                ;;
                        *)
                                echo "Error: Unexpected argument '$1'"
                                show_usage
                                ;;
                esac
        done
}

# Function to process grep output line
process_line() {
        local line="$1"

        # Check if the current line is a "Permission denied" error message
        if [[ "$line" == *"Permission denied"* ]]; then
                echo "No access: $line"
        else
                # If it's not an error, it's a found match
                # Line format: "path/to/file:found_text"

                # Extract file path
                local file=$(echo "$line" | cut -d":" -f1)

                # Get file size in bytes
                local size=$(stat --printf='%s' "$file")

                # Output information
                echo "File: $file, Size: $size byte, String: ${line#*:}"
        fi
}

# Function to search for string in files
search_files() {
        echo "Searching for string: '$SEARCH_STRING'"
        echo "Directory: $SEARCH_DIR"
        echo "====================================="

        # Recursively search for the string in all files within the directory
        # grep -r searches through all files in the specified directory and its subdirectories
        # Results are passed line by line to the while loop
        grep -r "$SEARCH_STRING" "$SEARCH_DIR" 2>/dev/null | while read -r line; do
        process_line "$line"
        done

# Handle permission denied errors from grep
grep -r "$SEARCH_STRING" "$SEARCH_DIR" | grep "Permission denied" | while read -r error_line; do
echo "No access: $error_line" >&2
done

echo "==========================================="
echo "Search completed"
}

main() {
        parse_arguments "$@"
        validate_arguments
        search_files
}

main "$@"
```

Проверим в терминале:

<img width="802" height="563" alt="image" src="https://github.com/user-attachments/assets/434df450-bf25-45a0-b2af-b7686df181e3" />
<img width="809" height="583" alt="image" src="https://github.com/user-attachments/assets/ad16712b-544e-4f14-ac15-d23cec237553" />
<img width="811" height="289" alt="image" src="https://github.com/user-attachments/assets/7769aeee-40b6-4f6e-ae7d-14104463e7cd" />
<img width="809" height="424" alt="image" src="https://github.com/user-attachments/assets/2bbf7f4a-ef39-45aa-bdc0-eb4061ef43b6" />


