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



