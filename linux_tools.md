# Linux tools

- [Linux tools](#linux-tools)
  - [ls](#ls)
  - [find](#find)
    - [Examples](#examples)
      - [Search text inside multiple PDFs](#search-text-inside-multiple-pdfs)
  - [xargs](#xargs)
  - [tar](#tar)
  - [rsync](#rsync)
  - [mkfifo](#mkfifo)
  - [Files](#files)
  - [Processes](#processes)
    - [Parallel execution](#parallel-execution)
      - [Using GNU Parallel](#using-gnu-parallel)
      - [Using xargs](#using-xargs)
  - [Networking](#networking)
    - [wget](#wget)
    - [curl](#curl)
    - [Netcat (nc)](#netcat-nc)
  - [GNU Screen](#gnu-screen)
  - [Clonezilla](#clonezilla)
  - [Sleep](#sleep)
  - [Watch](#watch)
  - [Dates](#dates)
    - [Formatting](#formatting)
    - [Operations](#operations)
  - [Images handling](#images-handling)
    - [Imagemagick](#imagemagick)
    - [Raw to JPEG conversion](#raw-to-jpeg-conversion)
  - [SSH/utilities](#sshutilities)
  - [SSL/Certificates](#sslcertificates)
  - [PGP (GnuPG/gpg)](#pgp-gnupggpg)
    - [Key servers](#key-servers)
  - [Formatting tools](#formatting-tools)
  - [Benchmarking](#benchmarking)
  - [Mounting images](#mounting-images)
  - [Create patches (diff)/restore them](#create-patches-diffrestore-them)

## ls

```sh
ls -d               # don't [d]escend into directories
ls -D               # don't [D]ereference files

ls -r               # [r]everse order

ls -S               # sort by [S]ize (default: desc.)
ls -t               # sort by m[t]ime (default: desc.)
ls -v               # sort numerically (!)

ls --block-size=K   # human-readable; use K,M,G...
```

## find

```sh
! (condition)                     # negates a condition
-name xx -not -name yy            # negate search condition
-name "*.c" -o -name "*.h"        # simple OR condition
\( -name xx -or -name yy \)       # OR condition with brackets

-[i]name <pattern>                # use quotes when using wildcards in pattern; case [i]nsensitive
-wholename <value>                # include the directory name (eg. `.git/config`)
-regextype awk -[i]regex <regex>  # same as name, but with regex; must match the whole name; !! specify the regextype (the default is minimal) !!
                                  # !! uses crap metachars (eg. [[:digit:]]) !!

-type (f|d)                       # [f]iles, [d]irectories
-L <path> -xtype l                # find symlinks
-L <path> -type l                 # find broken symlinks
-maxdepth <value>                 # max number of levels to descend; `1` causes not to descend into directories
-mindepth <value>                 # opposite of `maxdepth`; useful when the root path (eg. `.`) must not be displayed (use `1`)
-follow                           # [follow] symlinks

-perm                                    # parameters can be either in number or literal ("u+w,g+w","o=w") format
      <value>                            # find where permissions are exactly <value>
      -<value>                           # find where all <value> permission are set
      /<value>                           # find where any of <value> permissions are set
-user <user>                      # [user]

-(a|c|m)[-|+](time|min)           # Accessed|Changed|Modified; time = n*hours, min = n*mins; +/- = more/less than
--mtime +7                        # mtime = file data last modified N*24 hours ago; +7 = modified more than 7 days ago
--mtime 0                         # 0 = created today
--mmin -<minutes>                 # mmin = modified in the last <minutes> minutes (don't forget the minus)

-size [+|-]<value>[c|k|M]         # bigger or greater than <value>; c/k/M = bytes/kilo/mega
-empty                            # empty files/directories

-print                            # when passing the content to the pipe, adds the filename to the start of every line
-print0 | xargs -0                # use null-char to separate filenames (useful for filenames with spaces)
-print -quit                      # print one entry and quit; useful to check if there is one or more files in a directory (or matching)

-exec <command> {} \;             # execute command for every file found, passing the name as {}; multiple -exec can be passed
-exec <command> {} +              # execute command with as many files as possible (see https://unix.stackexchange.com/a/389706)
```

### Examples

Execute a certain command for the found directories:

```sh
# Better approach. The null-char approach is needed in order to cover filenames
# with double quotes.
#
find . -name no -print0 | xargs -0 -n 1 bash -c 'mv "$1" ../exclude/"$(basename "$(dirname "$1")")"' {}

# Simpler, but worse approach - fails if there are double quotes in the directory name.
#
find . -mindepth 1 -type d -exec bash -c 'cd "{}" && pwd' \;
```

Excluding paths in find (see https://stackoverflow.com/questions/4210042/exclude-directory-from-find-command).
Note that the -path must be relative to the same path as the search path, and it also must end with '/*'.

```sh
find . -name 1.txt -not -path ./skipdir/* -not -path '*/internal_dir/*'
```

Cycle filenames, with whitespace support.

```sh
# WATCH OUT! "<<<" won't work, because Bash parameter substitution doesn't support null bytes
#
find . -name "*.txt" -print0 | while read -rd $'\0' filename; do
  echo "<$filename>"
done

# If the cycle needs to break on the first find, ie. in case of particular conditions, and it's in
# a function, then it needs special handling, as `return` won't work, because it's in a subshell:
#
function myfunc() {
  local result

  result=$(
    find . -name "*.txt" -print0 | while read -rd $'\0' filename; do
      if [[ $filename == "matching.txt" ]]; then
        echo "$filename"
        break
      fi
    done
  )

  if [[ $result != "" ]]; then
    return
  # etc.etc.
}
```

Test if a directory contains directories:

```sh
! find -mindepth 1 -type d -exec false {} +
```

Execute a command for the content of a directory.
Using `for f in $dir/*` works, however, if there are no files, it raises an error.

```sh
find $dir -name $glob -exec bash -c '
  zpool labelclear -f "$1" 2> /dev/null || true
' _ {} \;
```

#### Search text inside multiple PDFs

```sh
find . -name '*.pdf' -exec sh -c 'pdftotext "{}" - | grep -i --with-filename --label="{}" --color "<pattern>"' \;
```

## xargs

For the parallel concepts, see the specific section.

```sh
-0                                    # specify null-char separator; useful ie. with `-print0`
-r --no-run-if-empty                  # :-)
-L $lines --max-lines=$lines          # maximum input lines per command
[-I pattern]                          # cmd split input on new lines and executes cmd; use of -I is the simplest
                                      # if no args, appends the line at the end
-d '<delimiter>'                      # set delimiter, e.g. "\n" with `ls`

```

Examples:

```sh
ls -1rt | xargs -d "\n" $cmd                  # executes the command, with files sorted by reverse modification time
cat /proc/self/environ | xargs -0 -L 1        # print the env vars (null-terminated), one per line
```

## tar

```sh
tar -C /tmp xvz                                        # Extract to a different directory
tar xvz --transform="s/^parsec-3.0/parsec-benchmark/"  # Rename destination files while extracting!!
tar --exclude='parsec-benchmark/.git' parsec-benchmark # Exclude (glob pattern)
```

## rsync

```sh
# Copy the the structure of a relative path. Use `--relative`, and place a dot path `./`:
#
rsync -av --relative "/target/run/./systemd/resolve" "/mnt/run"

# "Move-merge" a directory into another (mv doesn't allow this)
#
rsync -av --remove-source-files $from $to

# Resume partial files, but obviously must be 100% sure that the file content is the same
#
rsync --append ...

# Exclude (glob)
#
rsync --exclude=.git parsec-benchmark/ /dest                     # exclude at any level
rsync --exclude=parsec-benchmark/.git parsec-benchmark/ /dest    # exclude only the root one
```

## mkfifo

Messages don't need termination; this is abstracted, so empty messages and binary files can be sent:

```sh
# Reads and writes block until there is a write/read on the other end.
#
mkfifo /tmp/fifo

echo -n > /tmp/fifo &

echo "<$(cat /tmp/fifo)>"
# <>
```

## Files

```sh
mktemp --suffix="${filename##*.}"     # Create a temporary filename (using the extension of $filename)
stat $filename --format='%s'          # Get file size
```

## Processes

### Parallel execution

#### Using GNU Parallel

The number of jobs is automatically limited to the number of cores.
Quoting is not required.

```sh
ls -1 *.tar.* | parallel tar xvf      # if unspecified, the argument is automatically appended
ls -1 | parallel zip -m {1}.zip {1}   # parameterized; can also use `{}`
```

Using the `:::` commands separator:

```
parallel echo ::: A B                 # in parallel: (echo A) (echo B)
parallel echo vmsav{1} ::: A B        # in parallel: (echo vmsavA) (echo vmsavB)
```

Examples:

```sh
# multiple commands for each job
ls -1 | parallel "avconv -i {} {}.wav && neroAacEnc -q 0.5 -if {}.wav -of {}.m4a"

# same command, 4 times
parallel 'ruby -e "while true; end"' ::: $(seq 4)
```

Specifying the input delimiter; WARNING! Newline is still considered a delimiter:

```sh
echo 'a b c' | parallel --delimiter ' ' echo
```

#### Using xargs

Differently from GNU Parallel, the command must can't be quoted.

- `-P <processes>`: use 0 for unlimited; based on the manpage, `-n` or `-L` should be used, but they weren't required with the personal use cases.

```sh
ls -1 *.txt | perl -pe 's/\.txt//' | xargs -P 4 -I {} mysql -e 'LOAD DATA INFILE "{}.txt" INTO TABLE "{}"'

# Prints the files, surrounded by double quotes.
# Double quotes needs to be escaped! Additionally, must escape the backslash, otherwise it's interpreted by Perl.
#
ls -1 *.txt | perl -pe 's/(.*)/\\"$1\\"/' | xargs -P 0 -I {} echo {}
```

Ignore failing commands:

```sh
seq 4 | xargs -I {} -P 0 sh -c 'aws ec2 delete-snapshot --snapshot-id {} || true'
```

## Networking

Find process listening on port:

```sh
sudo lsof -i :4000
```

### wget

Options:

- `-P|--directory-prefix $dir`: download to a specific directory (`P`refix)

### curl

Options:

- `-d`, `--data`
- `-X`, `--request <command>`
- `-H`, `--header <header/@file>`
- `-i`, `--include`: include headers in the response

```sh
# Bare form: implies `-X GET`
#
curl "$URL"

# `-d` implies `-X POST`.
# In this case, the data format must be specified.
#
curl \
  -H "Authorization: Bearer $token" \
  -H 'Content-type: application/json; charset=utf-8' \
  -d '
  {
    "channel": "'"$channel"'",
    "text": "test",
  }' \
  "https://slack.com/api/chat.postMessage"

# Send an HTTP request for a specific format
#
curl -H 'Accept: application/halo+json' "$URL"
```

### Netcat (nc)

```sh
nc <address> $port           # simulate telnet
nc -lk -p $port              # listen to port; [k]=keep listening after a connection terminates
```

Examples:

```sh
# Net transfer from 1.1.1.1 via port 666
#
echo 'pizza' | nc -l -p 666 ==> nc highnotes.org 666 > pizza.txt

# Bidirectional proxy (3000→3001→3000)
#
mkfifo loop.pipe && cat loop.pipe | nc -l -p 3000 | nc localhost 3001 > loop.pipe

# Wait until a port is open.
#
while ! nc -z localhost 9200; do sleep 0.5; done
```

## GNU Screen

Commandline params:

- `-r`                 : resume screen session
- `-ls`                : list session
- `-x $name`           : join named session
- `-h $scrollback`     : size of scrollback buffer
- `-dm`                : (special) start session in detached mode
- `-S $name`           : session name
- `-X $command`        : execute a command in the session (see example below)
- `-L`                 : enable logging (flushes every 10" by default); doesn't overwrite existing files
- `-Logfile $filename` : set logfile (does _not_ imply `-L`)

Commands:

```
Ctrl+a                             function shortcut
  0..9                             go to window
  A                                set name
  K                                kill
  c                                create new
  C (:clear)                       clear screen. NOTE: in order to clear the scrollback buffer, one needs to set it to zero, then back to the original size
  d                                detach
  "                                list of windows
  h (:hardcopy)                    dump (hardcopy) screen in file
    -h $filename                   dump entire buffer, not only visible screen
  :                                command mode

Copy-related:
  [                                copy mode; lets the user scroll (Shift+PgUp/Down) and mark paste buffer start/end (using space)
  scrollback lines                 set the scrollback buffer; default=5000
  >                                output paste buffer to file

defscrollback $lines               set scrollback buffer in .screenrc
logfile flush $seconds             change the flush period (nonnegative integer; defaults to 10)
```

Execute commands in a detached session:

```sh
# `stuff` is the command; the terminating newline is necessary.
#
screen -dmS $session_name
screen -r $session_name -X logfile flush 0
screen -r $session_name -X stuff 'ls^M' # ^M is enter
```

## Clonezilla


```sh
# Setup
#
# The Ubuntu package is broken; must install `partclone` manually.
#
apt install -y clonezilla partclone

# Backup (sde)
#
# >>> Try from Ubuntu desktop
#
/usr/sbin/ocs-sr -q2 -c -j2 -z5p -i 409600000000000 -sfsck -scs -senc -p reboot savedisk ubuntu-20.04.1-server-usb sde

# Restore (sde)
#
# The partitions metadata files generated by the live USB version are incompatible with the Ubuntu version;
# this header is not meaningful.
#
sed -i '/^sector-size:/ s/^/# /' /home/partimag/ubuntu-20.04.1-server-usb/*.sf
/usr/sbin/ocs-sr -g auto -e1 auto -e2 -r -j2 -c -scr -p choose restoredisk ubuntu-20.04.1-server-usb sde
```

## Sleep

Sleeping works for decimals, and multiple times!

```sh
# Sleep for 1.5 hours. Valid suffixes: `s`, `m`, `h`, `d`
#
sleep 1.5h
sleep 1h 30m
```

## Watch

Be aware that watch commands are tricky. While not quoting the command works, for more complex commands, it will fail. Additionally, the shell executed it dash.

Example of scripted invocation:

```sh
watch_command=(watch -n 1 -x) # will execute bash
output_format_options=()
query_option=("$(printf "%q" "SELECT * FROM mytable")")

"${watch_command[@]}" bash -c "mysql ${output_format_options[*]} -uroot -h$mysql_host ${schema_option[*]} ${query_option[*]}"
```

## Dates

General format: `date +<format>`

Interpret a date: `date -d <input>`

### Formatting

Sequences:

- `%a/%A`: short/full day of the week
- `%b/%B`: short/full month
- `%H:%M:%s`
- `%Y-%m-%d`
- `%F %R` == `%Y-%m-%d %H:%M`

Operations:

```sh
# Print in a specific language:
#
LC_ALL=en_GB date --date='-1 month' +%B | tr '[:upper:]' '[:lower:]'
```

### Operations

Subtract (result is in seconds):

```sh
data_start_secs=`date -d "$data_start" +"%s"`
highlight_start_secs=`date -d "$highlight_start" +"%s"`

rel_highlight_start=`expr $highlight_start_secs - $data_start_secs`
```

## Images handling

### Imagemagick

Resample to 200 DPI (resample+units)

- Darken midtones (level)
- Converts format (.ext2)
- Uses a compression quality of 85% (quality); JPEG default is 92%

The used values are good for resizing, contrasting and compressing a scanned letter.
See how to execute an operation on multiple files, while changing the extension, for batch conversion.

```sh
convert -resample 200 -units PixelsPerInch -level 0%,100%,0.5 -quality 85% "$input.ext1" "$output.ext2"
```

Other operations:

```sh
convert "$input" -flip -flop "$output"            # Rotate 180°
convert "$input" (-resize|-scale) 50% "$output"   # Resize (with interpolation) or scale (without), 50%
convert -size 1920x1080 xc:white "$output.pdf"    # Create blank pdf page, with given resolution (size)
convert -coalesce animation.gif target.png        # Split an animated gif into its frames; `-coalesce` is required for more complex sources.
```

Use coalesce, then, separately, resize, in order to resize an animated gif.

### Raw to JPEG conversion

Use dcraw, Darktable or RawTherapee:

```bash
# This seems to produce nicer-looking photos than DarkTable.
#
ls -1 *.ORF | parallel 'dcraw -c {} | ppmtojpeg > {}.jpg'

# Can't be parallelized due to global lock (possibly there's a flag to disable this).
#
for f in *.ORF; do darktable-cli "$f" "$f".jpg; done
```

[Reference](https://askubuntu.com/a/1256073/46091).

## SSH/utilities

```sh
openssl rsa -in $private_key -pubout   # Generate a public key from private
ssh-keygen -e [-f $private_key]        # Generate a public key from private, for SSH usage
ssh-keygen -y                          # Same as previous, but requires user input
ssh-copy-id -i $private_key $user@$host # Authorize a user (key) on a host!

ssh-add -D                             # Delete cached keys. Useful if there are problems in authenticating!
ssh-add                                # adds the ssh key password to the authentication agent (e.g. gnome-keyring)

<RETURN><RETURN>~.                     # terminate an ssh session

openssl rsa -in $private_key -out $private_key_dec # decrypt a key
ssh-keygen -p -f $private_key                      # encrypt a key (or change passphrase)
ssh-keygen -t rsa                                  # generate ssh key pair

ssh-keyscan -H $address > ~/.ssh/known_hosts                                     # Programmatically add fingerprints to known_hosts
ssh-keygen -E md5 -lf $public_key                                                # Print public keys fingerprint
openssl pkey -in $private_key -pubout -outform DER | openssl md5 -c              # !!! Print fingerprint, for EC2 !!!

openssl s_client -connect $url:$port < /dev/null | openssl x509  -noout -enddate # Get the expiry of an ssl certificate

# Run a program in the primary display, from an SSH section
DISPLAY=:0 teamviewer
```

sshpass usages:

```sh
sshpass -p 'fedora_rocks!' ssh -p 10000 riscv@localhost

# With sshpass, it's odd.
#
echo -n 'fedora_rocks!' > /tmp/pwdfile
sshfs -p 10000 riscv@localhost: /media/saverio/temp -o ssh_command="sshpass -f /tmp/pwdfile ssh"
```

## SSL/Certificates

For an automatic (and locally trusted) solution, see [mkcert](https://github.com/FiloSottile/mkcert).

Create a self-signed certificate, manual procedure:

```sh
# Generate a key (insert a 4-letters password; it will be discarded in subsequent steps)
openssl genrsa -des3 -out server.key 2048

# Remove the password
openssl rsa -in server.key -out server.key.insecure
mv -f server.key.insecure server.key

# Create the CSR; use `*.$domain.com` as Common Name.
openssl req -new -key server.key -out server.csr

# Create the certificate
openssl x509 -req -days 365 -in server.csr -signkey server.key -out server.crt

# Display informations
openssl x509 -in server.crt -noout -text
```

## PGP (GnuPG/gpg)

Main commands (`gpg...`); `$key_id` can be email or key id. See the [Key Servers section](#key-servers) for important notes.

```sh
--gen-key                                            # generate a gpg keypair (store in the database)

--armor --export[-secret-key] $key_id                # export a key
--import [$pubkey]                                   # import a key (also secret); accepts stdin (don't specify $pubkey)

printf $'fpr\nsign\n'   | gpg --command-fd 0 --edit-key $key_id  # self-sign a key
printf $'trust\n5\ny\n' | gpg --command-fd 0 --edit-key $key_id  # ultimately trust a key [required by some programs]

printf $'passwd' | gpg --command-fd 0 --edit-key $key_id   # remove passphrase -> INTERACTIVE!

--delete[-secret]-key $key_id

--list[-secret]-keys
--fingerprint [$key_id]

--recipient $key_id --encrypt [--output $destfile]   # encrypt from stdin
--decrypt [file] [--output $destfile]

--detach-sign [--default-key $key_id] $doc           # don't include the original; use default key for specifying to key
--clearsign [--default-key $key_id] $doc             # don't compress the original, useful for ascii files
--verify [$sig] $doc                                 # see https://security.stackexchange.com/a/45534 about key not trusted

--keyserver $keyserver_address --search-key $key_id  # search key, by email, on a keyserver
```

WATCH OUT GnuPG private keys are composed of a "master" and a "subordinate" key. Both are necessary. If for some reason, a master key is missing the `sec` entry in the listings will show as `sec#`.

Snippets:

```sh
# Encode a group of files; gpg can also input from stdin.
#
find . -name *.log.gz | xargs -I {} gpg -r phony@recipient.com [--output {}.xxx] --encrypt {}
```

### Key servers

Key servers are surprisingly terrible (timeouts, usability, correct practices...).

WATCH OUT: once a key has been published, the master key is required in order to replace an existing public key; without it, nothing can be done, aside waiting for the expiry.

The best choice is [The HKPS pool](http://hkps.pool.sks-keyservers.net) (see [Stack Overflow](https://superuser.com/a/228033)).

When searching keys in servers via fingerprint, the prefix `0x` must be added.

## Formatting tools

Table (columns) formatting:

```sh
# `-t`: automatically determine columns based on whitespace
#
column [-s "$separators"] -t
```

## Benchmarking

The `time` program is built-in in modern shells. For more userful functionality, use `/usr/bin/time`.

```sh
# `-f`/`--format`: better use `-f`, which is more portable.
#
# %e: walltime
#
# The output goes to stderr!
#
/usr/bin/time -f ">>> WALLTIME:%e"
```

## Mounting images

Mount raw dumps:

```sh
# Attach the image to a loop device (**this is not mounting**)
#
loop_device=$(sudo losetup --show --find --partscan $image)

# List attached devices
#
losetup -l
#
# NAME       SIZELIMIT OFFSET AUTOCLEAR RO BACK-FILE                                               DIO LOG-SEC
# /dev/loop0         0      0         0  0 /home/saverio/code/riscv_images/components/busybear.bin   0     512

# Now we can mount
#
sudo mount $loop_device /mnt

# Detach the loop device
#
sudo losetup -d $loop_device
```

Mount any image type, via `qemu-utils`:

```sh
# Use `-f vpc` for vhd images.
#
qemu-nbd --connect=/dev/nbd0 [-f vpc] $image

# Inform the kernel of the changes.
#
partprobe /dev/nbd0

# List the partitions (optional); doesn't need any wait (only partprobe).
# Can also read a raw image!
#
fdisk -l /dev/nbd0

# Complete clusterduck. Even waiting via:
#
#     while [[ ! -b /dev/nbd0p4 ]]; do sleep 0.1; done
#
# will cause:
#
#     ls -l /dev/nbd0p4
#     ls: cannot access '/dev/nbd0p4': No such file or directory
#
# Therefore, we use `ls` instead.
#
while ! ls -l $partition_device > /dev/null 2>&1; do sleep 0.1 done

mount "$partition_device" /mnt
```

## Create patches (diff)/restore them

```sh
# Create the diff (`u`nified format)
#
diff -u $original $modified > patch.diff

# Create the diff from git (see git notes for details)
#
git diff --no-prefix $file > patch.diff

# `p0`: required for files in subdirs; for safety, best to always use it.
#
patch -p0 $patch.diff
```

If a hunk has been applied, patch will prompt. It's possible to skip this by using `--forward`, although that will return `1` exit status.