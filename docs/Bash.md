## General Code Style

While you should follow the code style that's already there for files that you're modifying, the following are required for any new code.

### Indentation

Indent 4 spaces. No tabs.

Use blank lines between blocks to improve readability. Indentation is four spaces. Whatever you do, don't use tabs. For existing files, stay faithful to the existing indentation.

### Pipelines

Pipelines should be split one per line if they don't all fit on one line. 

If a pipeline all fits on one line, it should be on one line.

If not, it should be split at one pipe segment per line with the pipe on the newline and a 2 space indent for the next section of the pipe. This applies to a chain of commands combined using '|' as well as to logical compounds using '||' and '&&'.

##### _Bad:_
```shell
command1 | command2 | command3 | command4 | command5 | command6 | command7
```

##### _Good:_
```shell
command1 \
    | command2 \
    | command3 \
    | command4
```

##### _Good:_ All fits on one line
```shell
command1 | command2
```

When possible, use environment variables instead of shelling out to a command.

##### _Bad:_
```shell
$(pwd)
```

##### _Good:_
```shell
$PWD
```

TODO: Add a list of all environment variables you can use.

###  If / For / While

Put `; do` and `; then` on the same line as the `while`, `for` or `if`.

##### _Good:_
```shell
for dir in ${dirs_to_cleanup}; do
    if [[ -d "${dir}/${ORACLE_SID}" ]]; then
        log_date "Cleaning up old files in ${dir}/${ORACLE_SID}"
        rm "${dir}/${ORACLE_SID}/"*
        if [[ "$?" -ne 0 ]]; then
            error_message
        fi
    else
        mkdir -p "${dir}/${ORACLE_SID}"
        if [[ "$?" -ne 0 ]]; then
            error_message
        fi
    fi
done
```

## Variables

### Use local variables

Ensure that local variables are only seen inside a function and its children by using `local` or other `typeset` variants when declaring them. This avoids polluting the global name space and inadvertently setting or interacting with variables that may have significance outside the function.

##### _Bad:_
```shell
function func_bad() {
    global_var=37  #  Visible only within the function block
                   #  before the function has been called. 
}

echo "global_var = $global_var"  # Function "func_bad" has not yet been called,
                                 # so $global_var is not visible here.

func_bad
echo "global_var = $global_var"  # global_var = 37
                                 # Has been set by function call.
```

##### _Good:_
```shell
func_good() {
    local local_var=""
    local_var=37
    echo $local_var
}

echo "local_var = $local_var" # local

func_good
echo "local_var = $local_var" # still local

global_var=$(func_good)
echo "global_var = $global_var" # move function result to global scope
```

In the next example, lots of global variables are used over and over again, but the script "unfortunately" works anyway. The `parse_json()` function does not even return a value and the two functions shares their variables. You could also write all this without any function; this would have the same effect.

##### _Bad:_ with global variables
```shell
#!/bin/bash

parse_json() {
    parent_prop=$1
    prop=$2

    # "'TODO: fix this hack' is the Snooze button of development" - @iamdevloper
    result=`echo $json \
        | sed 's/\\\\\//\//g' \
        | sed 's/^[\ ]*//g' \
        | sed 's/[{}]//g' \
        | awk -v k="$parent_prop" '{n=split($0,a,"\","); for (i=1; i<=n; i++) print a[i]}' \
        | sed 's/\"\:\"/\|/g' \
        | sed 's/"[\,]/ /g' \
        | sed 's/\"//g' \
        | grep "$prop|" \
        | sed "s/^$prop|//g"`
}

parse_ubuntuusers_json() {
    json=`curl -s -X GET 'http://suckup.de/planet-ubuntuusers-json/json.php?callback='`

    parse_json "posts" "title"
    mapfile -t titles_array <<< "$result"

    parse_json "posts" "date"
    mapfile -t dates_array <<< "$result"

    counter=0
    for i in "${titles_array[@]}"; do
        echo "${titles_array[$counter]} | ${dates_array[$counter]}"
        let counter+=1
    done
}

parse_ubuntuusers_json

echo "foobar: $counter - $i"
```

In shell scripts, it is less common that you really want to reuse the functionality, but the code is much easier to read if you write small functions with appropriate return values and parameters.

##### _Good:_ with local variables
```shell
#!/bin/zsh

parse_json() {
    local json=`cat $1`
    local parent_prop=$2
    local prop=$3

    # "'TODO: this hack' is the Snooze button of development" - @iamdevloper
    echo $json \
        | sed 's/\\\\\//\//g' \
        | sed 's/^[\ ]*//g' \
        | sed 's/[{}]//g' \
        | awk -v k="$parent_prop" '{n=split($0,a,"\","); for (i=1; i<=n; i++) print a[i]}' \
        | sed 's/\"\:\"/\|/g' \
        | sed 's/"[\,]/ /g' \
        | sed 's/\"//g' \
        | grep "$prop|" \
        | sed "s/^$prop|//g"
}

parse_ubuntuusers_json() {
    local temp_file=`mktemp`
    local json=`curl -s -X GET 'http://suckup.de/planet-ubuntuusers-json/json.php?callback=' -o $temp_file`

    local titles=`parse_json "$temp_file" "posts" "title"`
    local titles_array
    mapfile -t titles_array <<< "$titles"

    local dates=`parse_json "$temp_file" "posts" "date"`
    local dates_array
    mapfile -t dates_array <<< "$dates"

    local counter=0 i
    for i in "${titles_array[@]}"; do
        echo "${titles_array[$counter]} | ${dates_array[$counter]}"
        let counter+=1
    done

    rm $temp_file
}

parse_ubuntuusers_json

echo "foobar: $counter - $i"
```

### Constants and Environment Variable Names

All caps, separated with underscores, declared at the top of the file. Constants and anything exported to the environment should be capitalized.

##### _Constant:_
```shell
readonly PATH_TO_FILES='/some/path'
```

##### _Constant and environment:_
```shell
declare -xr ORACLE_SID='PROD'
```

Some things become constant at their first setting (for example, via `getopts`). Thus, it's okay to set a constant in `getopts` or based on a condition, but it should be made `readonly` immediately afterwards. Note that `declare` doesn't operate on global variables within functions, so `readonly` or `export` is recommended instead.

```shell
VERBOSE='false'
while getopts 'v' flag; do
    case "${flag}" in
        v) VERBOSE='true' ;;
    esac
done
readonly VERBOSE
```

### Read-only Variables

Use `readonly` or `declare -r` to ensure they're read only. As globals are widely used in shell, it's important to catch errors when working with them. When you declare a variable that is meant to be read-only, make this explicit.

```shell
zip_version="$(dpkg --status zip | grep Version: | cut -d ' ' -f 2)"
if [[ -z "${zip_version}" ]]; then
    error_message
else
    readonly zip_version
fi
```

## Functions

Private or utility functions should be prefixed with an underscore:

##### _Good:_
```shell
_helper-util()  {
    ...
}
```

### Use and check return values

After a script or function terminates, a `$?` from the command line gives the exit status of the script, that is, the exit status of the last command executed in the script, which is, by convention, 0 on success or an integer in the range 1 - 255 on error.

##### _Bad:_
```shell
my_bad_func() {
    # didn't work with zsh / bash is ok
    #read lowerPort upperPort < /proc/sys/net/ipv4/ip_local_port_range

    for port in $(seq 32768 61000); do
        for i in $(netstat_used_local_ports); do
            if [[ $used_port -eq $port ]]; then
                continue
            else
                echo $port
            fi
        done
    done
}
```

##### _Good:_
```shell
my_good_func() {
    # didn't work with zsh / bash is ok
    #read lowerPort upperPort < /proc/sys/net/ipv4/ip_local_port_range

    for port in $(seq 32768 61000); do
        for i in $(netstat_used_local_ports); do
            if [[ $used_port -eq $port ]]; then
                continue
            else
                echo $port
                return 0
            fi
        done
    done

    return 1
}
```
Example:
```Bash
command &>> log_file || error ${LINENO} $(basename $0) "See log for more information. \ncommand: \n commando + params"

```
### Check return values

Always check return values and give informative error messages. For unpiped commands, use `$?` or check directly via an if statement to keep it simple. Use nonzero return values to indicate errors.

##### _Bad:_
```shell
mv "${file_list}" "${dest_dir}/"
```

##### _Good:_
```shell
mv "${file_list}" "${dest_dir}/" || exit 1
```

##### _Good:_ use "$?" to get the last return value
```shell
mv "${file_list}" "${dest_dir}/"
if [[ "$?" -ne 0 ]]; then
    echo "Unable to move ${file_list} to ${dest_dir}" >&2
    exit 1
fi
```
You can use this function for each important command evaluation:

```Bash
# Error handling
error(){
    local parent_lineno="$1"
    local script="$2"
    local message="$3"
    local code="${4:-1}"

	RED='\033[0;31m'
	NC='\033[0m'

    if [[ -n "$message" ]] ; then
        echo -e "\n---------------------------------------\n"
        echo -e "${RED}ERROR${NC} in Script $script on or near line ${parent_lineno}; exiting with status ${code}"
        echo -e "MESSAGE:\n"
        echo -e "$message"
        echo -e "\n---------------------------------------\n"
    else
        echo -e "\n---------------------------------------\n"
        echo -e "${RED}ERROR${NC} in Script $script on or near line ${parent_lineno}; exiting with status ${code}"
        echo -e "\n---------------------------------------\n"
    fi

    exit "${code}"
}

```
## Features and Bugs

### Command Substitution

Use `$(command)` instead of backticks.

Nested backticks require escaping the inner ones with `\`. The `$(command)` format doesn't change when nested and is easier to read.

##### _Bad:_
```shell
var="`command \`command1\``"
```

##### _Good:_
```shell
var="$(command "$(command1)")"
```

### Eval

Eval is evil! Eval munges the input when used for assignment to variables and can set variables without making it possible to check what those variables were. Avoid `eval` if possible.

### Script template
You can use this [file]() as template for creating a new script.

### Dependency check
You can dependency check using this script:
```
#!/bin/bash

#=============================================================
# HEADER
#=============================================================

#INSTITUTION:ISCIII
#CENTRE:BU-ISCIII
#AUTHOR: Pedro J. Sola
VERSION=1.0 
#CREATED: 19 March 2018
#REVISION: 12 July 2018: add formated output and colors
#AKNOWLEDGE: Colored text: https://stackoverflow.com/questions/5947742/how-to-change-the-output-color-of-echo-in-linux
#DESCRIPTION:Short function to evaluate if programs are on path

#================================================================
# END_OF_HEADER
#================================================================

#SHORT USAGE RULES
#LONG USAGE FUNCTION
usage() {
	cat << EOF
Check_dependencies Short function to evaluate if files exist
usage : $0 <program_name1> [program_name2] ...
example: lib/check_dependencies.sh foo bar
EOF
}

if [ $# = 0 ] ; then
    usage >&2
    exit 1
fi

#DECLARE FLAGS AND VARIABLES
missing_dependencies=0

#SET COLORS

RED='\033[0;31m'
GREEN='\033[0;32m'
NC='\033[0m'

printf '\n%s\t%20s\n' "DEPENDENCY" "STATUS"
printf '%s\t%20s\n'   "----------" "------"

for command in "$@"; do
    #dependency_version=$($command --version)
    length_command=$(echo $command | wc -m)
    distance_table=$((30 - $length_command))
    distance_expression=$(echo "%${distance_table}s")

    printf '%s' $command
    if ! [ -x "$(which $command 2> /dev/null)" ]; then
		
		
        printf $distance_expression
        printf "${RED}NOT INSTALLED${NC} \n"
        let missing_dependencies++
    else
        printf $distance_expression
        printf "${GREEN}INSTALLED${NC} \n"
    fi
done

if [ $missing_dependencies -gt 0 ]; then 
    printf "${RED}ERROR${NC}: $missing_dependencies missing dependencies, aborting execution\n" >&2
    exit 1
fi
```
### Mandatory files checking
```
#!/bin/bash

#=============================================================
# HEADER
#=============================================================

#INSTITUTION:ISCIII
#CENTRE:BU-ISCIII
#AUTHOR: Pedro J. Sola
VERSION=1.0 
#CREATED: 19 March 2018
#REVISION:
#DESCRIPTION:Short function to evaluate if files exist

#================================================================
# END_OF_HEADER
#================================================================

#SHORT USAGE RULES
#LONG USAGE FUNCTION
usage() {
    cat << EOF
Check_mandatory_files Short function to evaluate if files exist
usage : $0 <file1> [file2] ...
example: lib/check_mandatory_files.sh foo.txt bar.fasta
EOF
}

if [ $# = 0 ] ; then
    usage >&2
    exit
fi

#DECLARE FLAGS AND VARIABLES
missing_files=0

for file in "$@"; do
    if [ ! -f $file ]; then
        echo "$(basename $file)" "not supplied, please, introduce a valid file" >&2
        let missing_files++
    fi
done

if [ $missing_files -gt 0 ]; then 
    echo "ERROR: $missing_files missing files, aborting execution" >&2
    exit 1
fi
```
## References

- [Shell Style Guide](https://google.github.io/styleguide/shell.xml)
- [BASH Programming - Introduction HOW-TO](http://tldp.org/HOWTO/Bash-Prog-Intro-HOWTO.html)
- [Linux kernel coding style](https://www.kernel.org/doc/Documentation/CodingStyle)