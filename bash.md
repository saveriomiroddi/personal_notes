# Bash

- [Bash](#bash)
  - [Shellopts](#shellopts)
  - [Variables](#variables)
  - [Switch/case](#switchcase)
  - [Test conditions](#test-conditions)
    - [Regular expressions](#regular-expressions)
    - [Applications](#applications)
  - [Jobs management](#jobs-management)
  - [Cycle a string tokens based on separator (IFS)](#cycle-a-string-tokens-based-on-separator-ifs)
  - [String operations](#string-operations)
    - [Examples](#examples)
  - [printf/escaping](#printfescaping)
  - [Arithmetic operations](#arithmetic-operations)
  - [Date operations](#date-operations)
  - [Redirections](#redirections)
  - [Process substitution](#process-substitution)
  - [Brace expansion](#brace-expansion)
  - [Arrays](#arrays)
    - [Snippets](#snippets)
  - [Associative arrays](#associative-arrays)
  - [Script operational concepts](#script-operational-concepts)
    - [Ask for input (keypress)](#ask-for-input-keypress)
    - [Trapping errors](#trapping-errors)
    - [Log a script output/Enable debugging [log]](#log-a-script-outputenable-debugging-log)
    - [Switching to root user inside a script](#switching-to-root-user-inside-a-script)
    - [Check if there's data in stdin](#check-if-theres-data-in-stdin)
    - [Check if a script is `source`d](#check-if-a-script-is-sourced)
    - [Workaround sudo expiry during a long-running script](#workaround-sudo-expiry-during-a-long-running-script)
    - [Interesting/useful time-related functionalities](#interestinguseful-time-related-functionalities)
    - [Poor man's configfile parser](#poor-mans-configfile-parser)
  - [Shell colors](#shell-colors)

## Shellopts

Basic (important):

```sh
set -o pipefail
set -o errexit
set -o nounset            # WATCH OUT! Unset arrays don't cause errors
set -o errtrace           # `-E`: trap errors also inside functions
shopt -s inherit_errexit  # subshells inherit errexit (Bash 4.4+)
```

Extra:

```sh
set -o monitor            # `-m`: enable job control
shopt -s nullglob         # IMPORTANT: when globs don't match anything, expand to null string, rather than to themselves
set -o xtrace             # `-x`: debugging mode; prints all the statements
shopt -s nocasematch      # case insensitive matches
```

See http://linuxcommand.org/lc3_man_pages/seth.html.

## Variables

```sh
myvar=value                         # no spaces around equal; if value includes spaces, it must be quoted
myvar=$myvar2                       # $myvar2 doesn't require quotes
unset myvar                         # delete myvar
export myvar=value                  # makes available to subshells. `local export` is allowed, but doesn't work as intended
((var+=1))                          # increment variable value
((var++)) || true                   # increment variable value. WATCH OUT!!! `|| true` is required if var=0 before the increment/decrement!!!!

# Indirect variable referencing
#
declare -n myvar=var_name_ref       # bash only; watch out! `unset` will unset also the source var!
echo ${!var_name_ref}               # bash only
eval local myvar2=\$$var_name_ref   # works both on bash/zsh; this shows how to use local modifier

myvar=$(nonexiting_command 2>&1)    # the assignment value is stdout's output; for stderr we need redirection (eg. see whiptail)

declare -g                          # declare variable as global (eg. from a function)
declare -x                          # export; can add to `-g`
```

## Switch/case

```sh
# switch/case. Double semicolon is required (can be put at the end of a statement); space before `)` isn't.
#
case $env_number in
*[0-9][0-9][0-9]*r )
  foo
  ;;
[yY] | [yY][Ee][Ss] )
  bar
  ;;
[nN] | [n|N][O|o] )
  baz
  ;;
*)
  boom
  ;;
esac
```

## Test conditions

Note that the left side of a condition (or the only one on prefix operators) doesn't need to be escaped.

Operators:

```
-z string                 string is empty
-n string                 string is not empty

-v var_no_dollar          for scalars: test if variable has been declared; WATCH OUT: don't prefix with the dollar!

-e <filename>             file/directory/symlink exists; symlink is followed before checking!
-f <filename>             file exists
-x filename               file exists and it's executable
-d <dirname>              directory exists
-L <filename>             file/directory is a symlink
-b <filename>             file is a block device

! -s <filename>           file is empty

-t 0                      input is terminal (false if stdin)
```

Note that functions can return a condition result!!:

```sh
function myTest {
  [[ "$1" == "foo" ]]
}

myTest "foo" && echo "bar"      # prints `bar`!
```

### Regular expressions

!! WATCH OUT !!

1. They're not PCRE;
2. The expressions don't need to be quoted;
3. Use a temporary variable (or cmd subsitution) when using metacharacters with backslash;
4. `^` and `$` refer to the *whole string*, not the line;
5. DON'T WRAP with ruby regex slashes!!!

Metacharacters:

- `\w`: not supported; use `[:alnum:]`, but *doesn't match underscore*
- `\d`: not supported; use `[0-9]` or `[:digit:]`
- `\b`: supported
- `\s`: `[:space:]`
- `[:xdigit:]`: hexadecimal!

References:

- Overview: http://tldp.org/LDP/abs/html/x17129.html
- Chacter classes: https://www.gnu.org/software/grep/manual/html_node/Character-Classes-and-Bracket-Expressions.html
- Limitations: https://stackoverflow.com/a/12696899

The captured groups are available in the `$BASH_REMATCH` array (with entry 0 being the full match).

Examples:

```sh
[[ 192.168.1.2 =~ ^[0-9]{1,3}(\.[0-9]{1,3}){3}$ ]] && echo MATCHES
re='\bword\b'; [[ ' word ' =~ $re ]] && echo MATCHES
[[ ' word ' =~ $(echo '\bword\b') ]] && echo MATCHES
```

### Applications

Check if a binary/executable is in PATH/exists (**don't** use which):

```sh
[[ -x "$(command -v <command>)" ]]
```

Execute a command if a process is not running:

```sh
! pgrep -fa mysqld && (mysqld &)                       # Zsh doesn't need brackets for this semantics
if [[ -z "$(pgrep -fa mysqld)" ]]; then mysqld & fi
```

## Jobs management

```sh
jobs                # job currently running; use tipically when `there are stopped jobs`
kill %[-]<n>        # kill nth (1-based) job; if `-`, start from the end (-1 = last)
kill $!              # kill latest backgrounded job; note that "$!" is the PID of the latest background job
```

## Cycle a string tokens based on separator (IFS)

Use IFS to tokenize a string with alternative characters (they're considered a set of **single** chars); **must** unset after:

```sh
input="a,b c;d"

IFS=",;"
for token in $input; do echo "<$token>"; done
# <a>
# <b c>
# <d>

unset IFS
```

While the `while` loop has scoping:

```sh
while IFS= read -r token; do : done <<< $input
```

it doesn't work as intended for this purpose.

## String operations

Substitutions generally have the single (single operator) or global (double operator) version; where this is the case, the global version is presented.

Parameter expansion supports internal expansion, and escape chars: `line=${line//$'\t'/ }`

```sh
${str:-<expr>}                    # default a parameter: set if undefined or blank
${str:+<expr>}                    # if the var is set, replace with <expr> (which can include the $param itself!)

${str//search/replace}            # substitution (!! NOT REGEX !!) ->
${str//[a-e]/ }                   # -> but supports at least character classes!
                                  # WATCH OUT! In <search>, `~` must be escaped (not not in <replace>).

${str:[<start>][:<end_expr>]}     # substring (0-based)
                                  # `start`: inclusive; blank: first char
                                  # `end_expr`: not inclusive; positive: length; negative: !! -1 = beforelast !!; blank: until end

${str^^}                          # upper case (single `^` applies only once)
${str,,}                          # down case (single `^` applies only once)
```

(g)sub (applied to filenames):

```sh
${filename%<.ext>}                # strip specific <ext>ension; `*` can be used [replace suffix]
${filename%%.*}                   # strip any extension (anything after the first dot)
${filename##*.}                   # extract (last) extension [replace prefix]
${filename##*/}                   # extract basename, eg. `"${PWD##*/}"`

${filename%/*/*}                  # parent dir of a file (watch out - requires at least two slashes!)
```

### Examples

```sh
${str:1}                          # substring: remove the first charater
${str::-1}                        # substring: remove the last character
```

## printf/escaping

```sh
printf "%q" "$str"                # escape a string
```

## Arithmetic operations

```sh
(( <expr> ))                      # if <expr> == 0, evaluates to false!
let param=<expr>                  # if <expr> == 0, evaluates to false!
```

**Important**: arithmetic operations will cause exit if they evaluate to 0 and the `errexit` shellopt is set:

```sh
(( val ))                         # exits if `val` is 0
(( val++ ))                       # postfix exits if the old `val` value is 0
param=$(( val++ ))                # this doesn't exit
```

## Date operations

Add/subtract dates: convert to seconds, then perform an operation via `date --date`, then convert back:

```sh
timestamp_in_secs=`date -d "2015-12-12 13:13:28" +"%s"`
seconds_to_subtract=90
formatted_result=`date '+%Y-%m-%d %H:%M:%S' --date="@$((timestamp_in_secs – seconds_to_subtract))"`
```

## Redirections

```sh
&> "$filename"                                          # redirect both stdout and stderr to <filename>
>&2 echo "error"                                        # write to stderr

{ mycommand 3>&2 2>&1 1>&3 | grep -v "skipme" >&3; } 3>&2 2>&1  # filter out stderr message

exec 200> "$filename"                                   # associate a file to a file descriptor (create if not existing)
```

## Process substitution

Creates a file (descriptor) from the command.

For input file; typically used to feed output of multiple commands to another command:

```sh
meld <(command_list_1) <(command_list_2)>
```

For output file; typically used with tee, to both print to stdout and redirect to a pipeline:

```sh
# If we do `| tee | xsel -ib`, the output is not printed!
#
echo "abc" >(xsel -ib)
```

Extension to input: if the command is just a filename, the command is repliced with the file content:

```sh
echo $(< "$filename")
```

## Brace expansion

```sh
{$start..$end [..$incr]}            # generate a sequence (range) via brace expansion (!!)
```

## Arrays

Note that arrays in zsh work in slightly different ways, so scripts involving them should be shell-specific.

Creation:

```sh
declare -a $array_name              # create an empty array; local by default
myarray=("entry 1" entry2)          # create a filled array; space/newline is separator
myarray2=("$myarray")               # copy an array

coordinates=(${1//./ })             # split by `.` (bash) !!! use only for unspaced stuff !!!

# More solid way of creating an array from a string.
# `mapfile` is synonym of `readarray`.
#
# `-t`: remove trailing delimiter
#
# in order to make the variable local, declare it as local before executing `mapfile`
# When using this pattern, don't forget `-n` on the echo!!
# here-string appends a newline, so it can be used only when mapfile has a newline delimiter.
#
mapfile -td. coordinates < <(echo -n "$1")
mapfile -t coordinates < <(printf "a a\nb b")
mapfile -t coordinates <<< "$myvar"

IFS=, read -ra coordinates <<< $1    # create via `read`; note how `echo -n` is not required

files=(scripts/*.cnf)                # create from list of files; **must use nullglob shopt!**
```

Operations; indexes are 0-based:

```sh
coordinates+=("abc")                 # append an entry
echo ${coordinates[0]}               # access an array; last entry: -1
unset 'coordinates[1]'               # delete an entry
coordinates[1]=b                     # set an indexed value
echo "${coordinates[@]:2}"           # array slicing! ([2..-1]) (see https://stackoverflow.com/a/1336245)
echo "${@:2}"                        # slice the `$@` variable
echo ${#coordinates[@]}              # size (length)
printf '%s\n' "${pizza[@]}"          # print the entries (one per line); echo print all in one line
echo $(IFS=,; echo "${pizza[*]}")    # join the entries. !!! don't forget the `;` and the quotes !!!

# Iteration (regular vars/$@)
#
for entry in "${coordinates[@]}"; do echo $entry; done
for param in "$@"; do echo $param; done
```

### Snippets

Sort an array (/keys of an associative array):

```sh
# For entries with newlinews, `sort -z` and mapfile `-d ''` should be used.
#
mapfile -t sorted_keys < <(printf '%s\n' "${!my_ass[@]}" | sort)
```

Iterate a multiple lines output, assigning it to an array:

```sh
# `IFS=` prevents leading/trailing line whitespace to be trimmed
# `-r` prevents interpreting the escape character (`\`)
# `-a` splits the tokens (by space) into an array; not supported by Zsh.
#
binlogs=$(mysql -u"$my_user" -p"$my_pwd" -h"$my_host" -BNe 'SHOW BINARY LOGS')

while IFS= read -r -a binlog; do
  echo Filename: "${binlog[0]}", Position: "${binlog[1]}"
done <<< "$binlogs"
```

## Associative arrays

See https://www.artificialworlds.net/blog/2012/10/17/bash-associative-array-examples.

```sh
declare -A MYHASH                    # explicit creation
declare -A MYHASH=([foo]=1 [bar]=2)  # explicit creation with multiple values (it's possible to quote both keys and values)
MYHASH[foo]=bar                      # implicit creation
unset MYHASH                         # destruction

echo ${MYHASH[foo]}                  # access an entry
unset MYHASH[foo]                    # remove an entry
[[ -v MYHASH[foo] ]]                 # test if a key exists

echo ${MYHASH[@]}                    # values (!!!)
echo ${!MYHASH[@]}                   # keys
echo ${#MYHASH[@]}                   # size

# Iterate.
# in order to iterate the sorted keys, use `mapfile` as in the Arrays section
#
for key in ${!MYHASH[@]}; do echo "$key => ${MYHASH[$key]}"; done

# Test if a variable is an associate array (!!)
# `declare -p` prints the declaration; writes to stderr if the variable doesn't exist
# watch out! if a variable was defined via `declare -p`, it will be printed.
#
[[ "$(declare -p MYHASH 2> /dev/null)" == "declare -A"* ]] && echo "is associative array"
```

## Script operational concepts

### Ask for input (keypress)

Input one char; if `variable_name` is specified, the input is stored in the variable:

```sh
read -rsn1 [<variable_name>]
```

### Trapping errors

Execute a (explicit) command on exit; generally convenient to trap ERR and INT:

```sh
LOCKFILE=/var/lock/makewhatis.lock

# Explicit command form:
#
trap "{ rm -f $LOCKFILE; }" ERR INT EXIT

# Function form:
#
function remove_lockfile {
  rm -f "$LOCKFILE"
}

trap remove_lockfile ERR INT EXIT
```

### Log a script output/Enable debugging [log]

```sh
# General log
#
exec > >(tee -i "$logfile")
exec 2>&1

# Debugging log
#
exec 5> "$logfile"
BASH_XTRACEFD="5"
set -x
```

### Switching to root user inside a script

This is not possible, but there is a workaround:

```sh
if [[ $(id -u) -ne 0 ]]; then
  sudo "$0" "$@"
  exit $?
fi
```

### Check if there's data in stdin

Check if there is data in stdin (can't be done in Bash natively; see https://unix.stackexchange.com/q/33049):

```sh
first_byte=$(dd bs=1 count=1 2>/dev/null | od -t o1 -A n | tr -dc 0-9)

if [ -z "$first_byte" ]; then
  # stuff to do if the input is empty
else
  {
    printf "\\$first_byte"
    cat
  } | {
    # stuff to do if the input is not empty
  }
fi
```

### Check if a script is `source`d

```sh
[[ "${BASH_SOURCE[0]}" != "$0" ]] # true if script is sourced
```

### Workaround sudo expiry during a long-running script

`$$` is the parent script, which is the script running this snippet.

```sh
sudo -v

while true; do
    sudo -n true
    sleep 60
    kill -0 "$$" || exit
done 2>/dev/null &
```

### Interesting/useful time-related functionalities

```sh
# Check time passed.
SECONDS=0; while [[ $SECONDS -lt 10 ]]; do sleep 1; done

# Wait until next beginning of second
sleep 0.$(printf '%04d' $((10000 - 10#$(date +%4N))))
```

### Poor man's configfile parser

Source (modified): https://stackoverflow.com/a/20815951.

Supports empty lines and single-line comments; spaces around the equal sign are trimmed.

```sh
while IFS='= ' read -r key value || [[ -n $key ]]; do   # `-n`: include the last line, if it hasn't a newline
  if [[ -n $key && ! $key =~ ^\ *# ]]; then
    declare -g "$key"="$value"                          # `-g`: declare as global (otherwise, it's local); add `-x` to also export them
  fi
done < "$configuration_file"
```

A useful functionality is to interpret all values starting with `$`:

```sh
    if [[ $value == \$* ]]; then
      eval declare -g $key=$value
    else
      declare -g $key="$value"
    fi
```

## Shell colors

See https://www.lihaoyi.com/post/BuildyourownCommandLinewithANSIescapecodes.html.

Ruby convenience script:

```ruby
STANDARD_FOREGROUND_COLORS = [
  BLACK_FG = "\u001b[30m",
  RED_FG = "\u001b[31m",
  GREEN_FG = "\u001b[32m",
  YELLOW_FG = "\u001b[33m",
  BLUE_FG = "\u001b[34m",
  MAGENTA_FG = "\u001b[35m",
  CYAN_FG = "\u001b[36m",
  WHITE_FG = "\u001b[37m",
]

BRIGHT_FOREGROUND_COLORS = [
  BRIGHT_BLACK_FG = "\u001b[30;1m",
  BRIGHT_RED_FG = "\u001b[31;1m",
  BRIGHT_GREEN_FG = "\u001b[32;1m",
  BRIGHT_YELLOW_FG = "\u001b[33;1m",
  BRIGHT_BLUE_FG = "\u001b[34;1m",
  BRIGHT_MAGENTA_FG = "\u001b[35;1m",
  BRIGHT_CYAN_FG = "\u001b[36;1m",
  BRIGHT_WHITE_FG = "\u001b[37;1m",
]

# The bright background versions make the foreground color bright (!).
#
BACKGROUND_COLORS = [
  BLACK_BG = "\u001b[40m", # good with BRIGHT_WHITE_FG
  RED_BG = "\u001b[41m",
  GREEN_BG = "\u001b[42m",
  YELLOW_BG = "\u001b[43m",  # good with [BRIGHT_]BLACK_FG
  BLUE_BG = "\u001b[44m",
  MAGENTA_BG = "\u001b[45m",
  CYAN_BG = "\u001b[46m",
  WHITE_BG = "\u001b[47m",
]

RESET_COLOR = "\e[0m"

def colorize_warning_thresholds(value, red_threshold, yellow_threshold)
  if value < red_threshold
    prefix, suffix = "#{BRIGHT_WHITE_FG}#{RED_BG}", RESET_COLOR
  elsif value < yellow_threshold
    prefix, suffix = "#{BRIGHT_BLACK_FG}#{YELLOW_BG}", RESET_COLOR
  end

  "#{prefix}#{value}#{suffix}"
end

def print_color_combinations
  BACKGROUND_COLORS.each do |bg_color|
    STANDARD_FOREGROUND_COLORS.each do |fg_color|
      print "#{bg_color}#{fg_color} 90 #{RESET_COLOR}"
    end

    print "   "

    BRIGHT_FOREGROUND_COLORS.each do |fg_color|
      print "#{bg_color}#{fg_color} 90 #{RESET_COLOR}"
    end

    puts "", ""
  end

  nil
end
```
