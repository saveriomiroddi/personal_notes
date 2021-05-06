# Linux tools

- [Linux tools](#linux-tools)
  - [ls](#ls)
  - [find](#find)
    - [Examples](#examples)
      - [Search text inside multiple PDFs](#search-text-inside-multiple-pdfs)
  - [xargs](#xargs)
  - [tar](#tar)
  - [mkfifo](#mkfifo)
  - [Files](#files)
    - [lsof](#lsof)
    - [Find/restore deleted but open files](#findrestore-deleted-but-open-files)
  - [Parallel execution](#parallel-execution)
    - [Using GNU Parallel](#using-gnu-parallel)
    - [Using xargs](#using-xargs)
  - [Profiling (perf)](#profiling-perf)
    - [Waits on condvar](#waits-on-condvar)
    - [Sleep times](#sleep-times)
  - [GNU Screen](#gnu-screen)
  - [Clonezilla](#clonezilla)
  - [Sleep](#sleep)
  - [Watch](#watch)
  - [Displaying messages](#displaying-messages)
  - [Dates](#dates)
    - [Formatting](#formatting)
    - [Operations](#operations)
    - [Calendar](#calendar)
  - [Images handling](#images-handling)
    - [Imagemagick](#imagemagick)
    - [Raw to JPEG conversion](#raw-to-jpeg-conversion)
  - [Formatting/diff operations](#formattingdiff-operations)
  - [Encoding](#encoding)
  - [Benchmarking](#benchmarking)
  - [Mounting images](#mounting-images)
  - [Create patches (diff)/restore them](#create-patches-diffrestore-them)
  - [Remote desktop](#remote-desktop)
  - [Partclone](#partclone)
  - [Create an iso from a directory](#create-an-iso-from-a-directory)

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
-printf '%P\0'                    # print the relative path, and also append null-char

# exec: in both, `{}` must be the last argument
-exec $command {} \;              # execute command for every file found, passing the name as {}; multiple -exec can be passed
-exec $command {} +               # execute command with as many files as possible (see https://unix.stackexchange.com/a/389706)
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

Note that, when grouping args, they must be at the end of the command (with -I, one invocation per arg is performed). Empty lines are ignored.

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

### lsof

Funky lsof stuff; note that a program may not always keep the files open while is running!

```sh
lsof $filename            # specific open file[s]; also works on mountpoints
lsof +D $directory/       # files open in a specific directory (end slash required!); also works on mountpoints
lsof -c $process_prefix   # files opened by a process starting whose name starts with a given prefix
lsof -p $pid              # files opened by a process with the a given pid
lsof -i :$port            # processes listening on a given port
```

### Find/restore deleted but open files

```sh
lsof -s | grep deleted

lsof | grep fileName					     # get the process id (col 2) and the file descriptor (col 4, remove the letter)
cp /proc/$pid/fd/$fd $dest         # recover the file
cat /dev/null > /proc/$pid/fd/$fd  # truncate a deleted open file!!
```

## Parallel execution

### Using GNU Parallel

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

Other options:

- `--delimiter $delim`    : input delimiter; WARNING! Newline is still considered a delimiter
- `-P|--max-procs <N>[%]` : max procs; `%` (optional) accepts decimal; watch out! uses the ceiling (e.g.13% of 32 yields 5)

In order to programmatically confirm the citation, must run as `echo "will cite" | parallel --citation || true`.

### Using xargs

Differently from GNU Parallel, the command can't be quoted.

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

## Profiling (perf)

```sh
sudo mkdir /usr/share/doc/perf-tip
curl https://raw.githubusercontent.com/torvalds/linux/master/tools/perf/Documentation/tips.txt | sudo tee /usr/share/doc/perf-tip/tips.txt

# Display stats (to stderr!!)
#
# -e                   : events recorded
# -x|--field-separator : print in tabular format, with given separator
# -p                   : attach to process
#
perf stat -e L1-dcache-load-misses,context-switches,migrations,cycles,sched:sched_switch --per-thread -x, -p $(pgrep -f qemu-sys) 2>&1 | tee perf.txt

# Record in-depth profile (to `perf.data`).
#
# This can take lots of space; recording 64 threads for around a minute created more than 1 GB of data.
#
# -e, -s, -p         : same as `stat`
# -s|--stat          : record per-thread event counts (NOT --per-thread!!!)
# -c|--count         : event period
# -g                 : enable call graph
# --call-graph dwarf : use when `-g` doesn't work (see end of this section); implies `-g`
# -o $file           : specify output file
# -F|--freq $hz      : specify the frequency (default=1000)
#
perf record -e sched:sched_stat_sleep,sched:sched_switch,sched:sched_process_exit -s --call-graph dwarf -p $(pgrep -f qemu-sys)

# Display the record results; includes the call graph.
# It's not clear why the "PID TID" section at the end is empty.
#
# -T|--thread     : show per-thread (requires `record -s`); see `prev_pid=$tid` entries
# --tid           : show only a single tid
# --stdio         : text output (instead of GUI)
# --no-call-graph : display in tabular format, without call graph
# -n              : add samples; doesn't seem to be particularly useful.
# --sort comm     : sort by command (easiest to read)
#
perf report --thread --stdio
perf report --stdio --no-call-graph --tid=mytid --sort comm

# Terminate, with data saving.
#
kill -INT $(pgrep perf)
```

General information:

- `record` and `stat` can yield similar results, but they can't be used interchangeably (see https://stackoverflow.com/questions/49216628/perf-stat-vs-perf-record).
- `$event:u` monitors only at user-level privilege; in some cases, kernel-level monitoring is important

Record/report:

- generally speaking, if the call graph is insufficient:
  - enable the debug information for the given program (e.g. `--enable-debug-info` for QEMU)
  - disable optimizations: `-Og` or `-O0`
    - `-Og` adds extra info, but also treats more warnings as errors (use if certain that it's ok `-Wno-error`)
    - on paper, `-O0` produces less debug info, but with QEMU, it produces more
  - set `-fno-omit-frame-pointer`
  - macros seem not to affect the production of call stacks
- `--call-graph dwarf` can get more information than the default, however, it doesn't help anyway with optimizations/lack of debug info
  - it can take 100x as much space (!!)

Events:

- `cpu-cycles` (synonym of `cycles`) is better choice than `cpu-clock`; use the latter only if the first can't be used

### Waits on condvar

Requires recording `sched:sched_switch`; can also try `sched:*switch` for all the context switches.

```sh
# Convenient statement for checking the context switches caused by a thread waiting on a condvar.
#
grep __pthread_cond_wait $report_output -w -B10 | grep prev_comm=qemu | sed 's/%.*//' | awk '{i+=$1} END {printf("%.2f%%\n",i)}'
```

Reference: https://easyperf.net/blog/2019/10/12/MT-Perf-Analysis-part2#find-expensive-synchronization-with-perf-

### Sleep times

Requires recording `sched:sched_stat_sleep,sched:sched_switch,sched:sched_process_exit`.

WATCH OUT! For unclear reasons, it fails, even with the official [patch code](https://lwn.net/Articles/510120).

```sh
# -v|--verbose
# -s|--stat    : special mode to merge sched events in order to perform sleep analysis
#
perf inject -v -s -i perf.data -o perf.sleep.data
perf report --stdio --show-total-period -i perf.sleep.data
```

Reference: https://perf.wiki.kernel.org/index.php/Tutorial#Profiling_sleep_times

In order to use the LWN example, add:

```c
#include <time.h>
#include <sys/time.h>

struct timespec ts1;
struct timeval tv1;
```

## GNU Screen

Commandline params:

- `-ls`                : list session
- `-r [$name]`         : resume screen session; if an exact match is not found, prefix search is performed (but must match only one)
- `-x [$name]`         : join named session (see `-r` for matching rules)
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

In some workflows, it's convenient to set `IGNOREEOF`, in order to prevent accidental exits from the screen session.

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

## Displaying messages

```sh
# Simple popup window with button.
# WATCH OUT! Don't forget `--text`, otherwise, a cryptic message 'All updates are complete.' is shown (!?).
#
# - `--no-markup`       : disable text interpretation ("Pango" markup)
# - `width`, `--height`
#
zenity --info --no-markup --text 'Text message!'
```

## Dates

General format: `date +<format>`

Interpret a date: `date -d <input>`

### Formatting

General format: `date $format --date=$expression`

Symbols:

- `%a/%A`    : short/full day of the week
- `%b/%B`    : short/full month
- `%H:%M:%s`
- `%Y-%m-%d`
- `%F %T`    : same as `%Y-%m-%d %H:%M:S` (time without seconds: `%R`)
- `+%s`      : date in seconds

The output language is the system one; in order to change, set `LC_ALL`:

```sh
LC_ALL=en_GB date # ...
```

Checking: `date -d $date` can be used to check if a date is valid.

### Operations

Operations via natural language expressions:

```sh
date                                # `now` is implied
date --date='yesterday'
date --date='2 years ago'
date --date='2 days'                # now plus days
date --date='next tue'
date --date='@1'                    # epoch time
```

The option `--debug` shows the logic applied.

WATCH OUT!! When performing operations with a date, the timezone must be specified, otherwise, the '+/- ...' will be interpreted as the time zone:

```sh
$ date '+%R'; date '+%R' --date '-1 hour'
22:07
21:07

# WRONG!
$ date '+%R' --date='2021/04/19 22:07 - 1 hour'
02:07

# Correct!
$ date '+%R' --date="2021/04/19 22:07 $(date +%Z) - 1 hour"
21:07
```

Arithmetic via conversion to seconds:

```sh
timestamp_secs=$(date -d "2015-12-12 13:13:28" +"%s")
result_secs=$(( timestamp_secs - 90 ))
date '+%F %R' --date="@$result_secs"

data_start_secs=$(date -d "$data_start" +"%s")
highlight_start_secs=$(date -d "$highlight_start" +"%s")
expr $highlight_start_secs - $data_start_secs
```

### Calendar

!!!

```sh
cal [[$month] $year]
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

## Formatting/diff operations

```sh
# Table (columns) formatting
#
# `-t`: automatically determine columns based on whitespace
#
column -s $separators -t

# "subtract" lines of one files from another
#
grep -Fvx -f $file_1 $file_2

# Find common lines between two files; must be both sorted (lexicographycally)
#
comm -12 $file_1_sorted $file_2_sorted
```

## Encoding

```sh
enca -L none filename                   detects file encoding
iconv -f UCS-2 -t UTF-8 filename        converts file encoding

# convert binary input to something human readable (eg. hexadecimal)
#
hexdump -C
# `-e`: custom format; every byte (`/1`) the lowercase hex value), and every 32 bytes (`/32`) the newline (NL is not printed by default)
#
hexdump -e '/1 "%02x"' -e '/32 "\n"'

# convert binary to base64, and back [hexadecimal,human readable].
#
base64 [-i <input>] [-o <output>]
base64 -d [-i <input>] [-o <output>]
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

## Remote desktop

Remove desktop on a headless server.

Broken (disconnects) - just use Nomachine; kept here for reference.

References:

- https://stackoverflow.com/a/40678605
- https://www.richud.com/wiki/Ubuntu_Fluxbox_GUI_with_x11vnc_and_Xvfb

Server configuration

```sh
 fluxbox: simple diplay manager

sudo apt install x11vnc xvfb fluxbox

# Short way
#
# - `-no6 -noipv6 -rfbportv6 -1`: for systems with ipv6 disabled (see https://bugs.debian.org/cgi-bin/bugreport.cgi?bug=672449)
# - `-forever`                  : without it, the service will terminate after the first session
# - `-ncache`                   : suggested, but corrupts the screen

x11vnc -create -env FD_PROG=/usr/bin/fluxbox \
  -env X11VNC_FINDDISPLAY_ALWAYS_FAILS=1 \
  -env X11VNC_CREATE_GEOM=${1:-1280x800x16} \
  -gone 'killall Xvfb' \
  -nopw \
  -forever \
  -bg
  -no6 -noipv6 -rfbportv6 -1
```

Client:

```sh
sudo apt install tigervnc-viewer

# `-N`                              : no command; just proxy
# `-T`                              : disable pseudo-terminal allocation
# `-L $local_socket:$remote_socket` : proxy!
#
ssh -N -T -L 5900:localhost:5900 arcade &

xtigervncviewer localhost:5900
```

## Partclone

Dump a single partition:

```sh
# Find the partition type
#
mount /dev/sdc1
df -T /dev/sdc1
umount /dev/sdc1

# [c]lone, [d]ebug info, [s]ource, [o]utput
#
partclone.vfat -c -d -s /dev/sdc1 -o - | pixz -2 > sdc1.vfat-ptcl-img.xz.aa
```

Restore a single partition:

```sh
# [d]ebug info, [s]ource, [o]utput
#
gunzip -c pizza.gz.a* | partclone.restore -d -s - -o /dev/sdc1
```

## Create an iso from a directory

```sh
# Use [J]oliet FS and [R]ock Ridge, for better compatibility with filesystem features
#
mkisofs -JR -o $outfile.iso $dir
```
