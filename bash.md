## Table of contents

- [Shellopts](#shellopts)
- [Test conditions](#test-conditions)
  - [Applications](#applications)
- [String operations](#string-operations)
- [Redirections](#redirections)
- [Trapping errors](#trapping-errors)

## Shellopts

```sh
set -o errtrace           # `-E`: trap errors also inside functions
set -o xtrace             # `-x`: debugging mode; prints all the statements
shopt -s nocasematch      # case insensitive matches
```

**important!** don't forget to set `errtrace` when trapping errors!

## Test conditions

```
-z string 								string is empty
-n string								  string is not empty
-e <filename>							file/directory/symlink exists; symlink is followed before checking!
-f <filename>							file exists
-x filename								file exists and it's executable
-d <dirname>							directory exists
-L <filename>							file/directory is a symlink
-b <filename>             file is a block device
! -s <filename>						file is empty
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

## String operations

```sh
${param:-<expr>}                                       # default a parameter: set if undefined or blank
${param:+<expr>}                                       # if the var is set, replace with <expr> (which can include the $param itself!)

${param:<start>[:<end>]}                               # substring (0-based); end is included
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

## Redirections

```sh
&> <filename>                                          # redirect both stdout and stderr to <filename>
>&2 echo "error"                                       # write to stderr
{ mycommand 3>&2 2>&1 1>&3 | grep -v "^Message to skip" >&3; } 3>&2 2>&1  # filter out stderr message

exec 200> <filename>                                   # associate a file to a file descriptor (create if not existing)
```

## Trapping errors

Execute a (explicit) command on exit; generally convenient to trap ERR and INT:

```sh
LOCKFILE=/var/lock/makewhatis.lock

# Explicit command form:
#
trap "{ rm -f $LOCKFILE; }" EXIT

# Function form:
function remove_lockfile {
  rm -f "$LOCKFILE"
}

trap remove_lockfile EXIT
```