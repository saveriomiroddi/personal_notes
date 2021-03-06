# Text processing

- [Text processing](#text-processing)
  - [Grep](#grep)
  - [Perl](#perl)
    - [Commandline args](#commandline-args)
    - [Operators](#operators)
    - [Expression-related](#expression-related)
      - [Flip-flop, with examples](#flip-flop-with-examples)
    - [Priority](#priority)
    - [Special variables](#special-variables)
    - [Data types, conversions and contexts](#data-types-conversions-and-contexts)
      - [Arrays](#arrays)
      - [Hashes](#hashes)
    - [Date/times](#datetimes)
    - [Flow control](#flow-control)
    - [Functions/APIs](#functionsapis)
    - [Search/replace](#searchreplace)
      - [Regex extra backslash sequences](#regex-extra-backslash-sequences)
    - [Formatting/printing](#formattingprinting)
    - [Line numbers/Position-based operations](#line-numbersposition-based-operations)
    - [Useful examples](#useful-examples)
  - [Awk](#awk)
    - [Commandline args](#commandline-args-1)
    - [Syntax](#syntax)
    - [Base commands](#base-commands)
    - [Regex](#regex)
    - [APIs](#apis)
    - [Useful examples](#useful-examples-1)
  - [Sed](#sed)
    - [Syntax](#syntax-1)
    - [Regexes](#regexes)
    - [Operators/variables](#operatorsvariables)
    - [Operations](#operations)
    - [Concatenate operations](#concatenate-operations)
    - [Special characters](#special-characters)
  - [Snippets](#snippets)
    - [Extract difference between two files](#extract-difference-between-two-files)
    - [Compute aggregates on a text/log file](#compute-aggregates-on-a-textlog-file)
  - [cut](#cut)
  - [tr (translate tokens)](#tr-translate-tokens)
  - [sort](#sort)
  - [jq](#jq)
    - [Hash](#hash)
    - [Arrays](#arrays-1)
      - [More complex filtering](#more-complex-filtering)
    - [Functions](#functions)
  - [Silver searcher (ag)](#silver-searcher-ag)
  - [Generic snippets](#generic-snippets)
    - [Stop tail when a string matches](#stop-tail-when-a-string-matches)
    - [Compare/sort versions](#comparesort-versions)
    - [Sum/average/etc. values extracted from a textfile](#sumaverageetc-values-extracted-from-a-textfile)

## Grep

```sh
# -P: [P]erl regexes
# -z: process the entire file as a single file
# -o: print only match (in this case, if not specified, the entire file will be printed, due to `-z`
#
grep -Pzo 'void\srb_backtrace.*\b' -r . --include="*.h"

# `(?s)`: match newlines with `.` ("PCRE_DOTALL")
#
grep -Pzo '(?s)void\srb_backtrace\(.*?\n\}' --include="*.c" -r .
```

## Perl

### Commandline args

- `-0`: use null character as line separator
- `-a`: splits the input into the `@F` array (0b.)
- `-F<val>`: separator

### Operators

```perl
# !!!!!!!!!!!!!!!!!!!!!!!!! Custom search delimiter !!!!!!!!!!!!!!!!!!!!!!!!!
s|from|to|

# Ternary operator
CONDITION ? TRUE_BRANCH : FALSE_BRANCH

# Flip-flop; works also with line numbers (!!!!!!)
#
print if /DELIMITER ;;$/ .. /DELIMITER ;$/
print if /match/ .. -1                       # print all the lines after /match/ (included)

# String operators (!!! DON'T USE == !!!)
'a' eq 'a' # 1
'a' ne 'b' # 1
```

For regexes, see the dedicated note file.

### Expression-related

Interpolate an expression via `@{[...]}` (trick):

```sh
echo "abc: 250" | perl -ne '/(\d+)/ && print "@{[$1 + 1]}"' # 251
```

#### Flip-flop, with examples

The flip-flop itself returns the line number (1b.), with an `E0` suffix for the last line.

Manipulate the lines matching the flip-flop operator:

```sh
# Only match
printf 'a\nb\nc\nd\ne' | perl -ne 'print "# $_" if /b/ .. /d/'

# Entire textfile
printf 'a\nb\nc\nd\ne' | perl -pe '/b/ .. /d/ and print "# "'

# Entire textfile, without commenting the last match ('d')
printf 'a\nb\nc\nd\ne' | perl -pe '$ln = /b/ .. /d/; $ln && $ln !~ /E0/ and print "# "'
```

### Priority

Watch out the priority!!!

```sh
# Prints only "Line 1" and an empty line
printf 'Line 1\nLine 2' | perl -lne 'print $_; $line = readline && print $line'

# Prints "Line 1" and "Line 2"
printf 'Line 1\nLine 2' | perl -lne 'print $_; $line = readline and print $line'
printf 'Line 1\nLine 2' | perl -lne 'print $_; ($line = readline) && print $line'

# In cases like flip-flop, either use parenteses, or the low-priority operators `and`/`or`.
#
/start/ .. /end/ and print "match!"
```

### Special variables

Special variables can be modified, depending on the type, on each cycle, or in the `BEGIN` block.

- `$_`: current line; if modified, and `-p` is specified, the new value is printed.
- `$.`: current line number (1-based)
- `$/`: separator; see examples.
- `$ENV`: env variables; don't forget that they need to be exported!!
- `@ARGV`: arguments (not including command)

### Data types, conversions and contexts

```sh
# Strings can be single- or double-quoted
#
'abc' eq "abc"

# True/false: !! there are no `true`/`false` keywords !!
# False values (everything else is true)
0
'0'
''
"()" # empty list
"undef"

# Regex matches and boolean true conditions, evaluate to 1
$counter += /matching_line/
$counter += ($match != "1")
```

Depending on the context, operations return different values. Watch out when using capturing groups!

```sh
# Prints (unexpectedly) 2, because the regex returns a scalar for the match (0/1)
#
printf '1\n2\n3' | perl -ne '$tot += /([23])/; END { print $tot }'

# Prints (correctly) 6, because the array addressing makes the regex return an array with the matches.
#
printf '1\n2\n3' | perl -ne '$tot += (/(.)/)[0]; END { print $tot }'
```

#### Arrays

```perl
(1, 2, 3)                             # array literal
(1..3)                                # array literal, as range; inclusive

$#array                               # length

push(@array, item)                    # append an item to an array
item = pop(@array)                    # pop an item from an array

$elements_sum / @elements             # in a scalar context (the division, in this case), an array yields its length

map { SubRoutine($_) } @Array;        # map an array
List::Util qw/sum/ -> sum(@Array)     # sum the elements of an array
```

#### Hashes

```perl
# Don't need to instantiate hashes!
$totals{'foo'} = 12;
print $totals{'foo'};
```

### Date/times

Date/time conversion:

```perl
# localtime() is a native function, but it doesn't have a pretty output.
#
perl -MPOSIX=strftime -e 'print strftime("%Y", localtime(355701600))'      # 1981
perl -MDateTime -e 'print DateTime->from_epoch(epoch => 355701600)->year'  # 1981
```

### Flow control

```perl
if (CONDITION) { TRUE_BRANCH } else { ELSE_BRANCH }

foreach (@Array) { SubRoutine($_); }           # iterate an array
```

Hash iteration is tricky:

```perl
# keys and value needed
# reset the internal iterator so a prior each() doesn't affect the loop; can ignore in a simple script.
#
keys %hash;
while(my($k, $v) = each %hash) { ... }

# only keys needed
foreach my $val (keys %hash) { ... }

# only values needed
foreach my $val (values %hash) { ... }
```

### Functions/APIs

```perl
length $str                          # length of a string
Math::Complex->sqrt($value)          # square root (but `$value ** 0.5` works as well)
```

Equivalent of Ruby scan/join:

```sh
# Fancy!!: return a list from captured groups (`m/`), and joins it.
#
printf "k1: a\nk2: b\nk3: c" | perl -0777 -ne 'print join(",", m/^k[13]: (.+)$/mg)' # => a,c
```

### Search/replace

(for the flags, see the `regexes.md` document)

```sh
# Replace with code evaluation.
#
perl -pe 's/(@\S+)/" " x length($1)/e'

# Regex matches print the groups.
#
echo 'a_b_c' | perl -lne 'print /(a)_(b)_(c)/' # `abc`

# Assign regex captured groups to variables.
# WATCH OUT! In the context of one or more variables, groups are interpreted as scalar; in order to
# assign captured strings, assign to an array (use one variable for each group).
#
echo a_b_c | perl -lne '$a = /a_(b)_c/; print $a'   # `1`
echo a_b_c | perl -lne '($a) = /a_(b)_c/; print $a' # `b`
```

#### Regex extra backslash sequences

Don't substitute part of the match!!:

```sh
# Don't replace before `\K`.
#
echo abc | perl -pe 's/^a\K.*/xx/' # => axx
```

Apply operations inside regexes (see https://perldoc.perl.org/perlrebackslash.html)!!:

```sh
# Convert next character: `\u`: upper case, `\l`: lower case
#
perl -i -pe 's/- (\w)/- \u$1/'
```

### Formatting/printing

```perl
printf "DB: $db %.1f\%\n", 100*$db/$total  # string interpolation + (float) formatting (`sprintf` also supported)
```

### Line numbers/Position-based operations

```sh
# Insert at specific positions (line number):
#
# - `$.` is 1-based!; it's also valid in the END block
# - `S_` → current line (can be modified)
# - `.=` → append
# - `.`  → concatenation
#
printf "0\n1\n2" | perl -lpe 'BEGIN { print "abc" }'      # !! doesn't work inplace !!
printf "0\n1\n2" | perl -lpe 'END { print "abc" }'        # !! doesn't work inplace !!
printf "0\n1\n2" | perl -ne 'eof && print'                # match and print last line

printf "0\n1\n2" | perl -lpe '$. == 2 && print "abc"'     # print before a numbered line => `0 abc 1 2`
printf "0\n1\n2" | perl -pe '$_ .= "abc\n" if /1/'        # print after a match; !! `/regex/ && ...` doesn't work! !!
printf "0\n1\n2" | perl -pe 'print $_; print "abc\n" if /1/' # print after a match (alternative)

printf "0\n0\n0" | perl -pe '$_ .= "abc\n" if /0/ && ++$cnt == 2'  # 2nd occurrence of the pattern (!!) => 0 0 abc 0
```

### Useful examples

Replace only the first occurrence in a file:

```sh
perl -pe '!$found && s/.../.../ && ($found=1)'
```

Strip trailing file spaces (or arbitrary characters):

```sh
perl -pe 'chomp if eof'    # only last whitespace
perl -0777 -pe 's/\s+$//'  # all the whitespaces
head -c -1                 # last character (!!)
```

Print part of line if there is a match:

```perl
# Conventional and compact way.
# The compact way work only when there is only one match (if there are multiple, either there is no separator, or one separator is printed for each input line).
#
print $1 if /^Host: (.*)/
print /^Host: (.*)/
```

Print the next line after a match (alternatives):

```sh
printf 'a\nb1' | perl -ne '/a/ && print readline =~ /b(\d)/'  # => 1
printf 'a\nb\nc\nd\n' | perl -pe '$_ = readline'              # => b\nd
```

Print progress:

```sh
# Simple, generic format:
#
perl -lne 'BEGIN { $/ = "\r" } print /(\d+)%/'

# Progress tracking with Whiptail gauge + autoflushing (see https://perl.plover.com/FAQs/Buffering.html); equivalent versions (like `stdbuf -o0 awk`):
#
rsync --progress --human-readable bigfile host:/path/ |
  perl -lne 'BEGIN { $/ = "\r"; $|++ } print /(\d+)%/' |
  whiptail --gauge Syncing 20 80 0

rsync --progress --human-readable bigfile host:/path/ |
  perl -lne 'BEGIN { $/ = "\r"; *STDOUT->autoflush } print /(\d+)%/' |
  whiptail --gauge Syncing 20 80 0

# Simulate awk field vars, with slicing support!
# Autosplit (`-a`, 0-based)) + array slicing + array length + automatically add newline to output (`-l`, +chomp) !!!
# `-F` is used for the field separator; defaults to space
#
perl -F: -lane 'print "@F[3..$#F]"'
```

Amusing way to expand tilde (home):

```sh
echo '~/.my.cnf' | perl -pe 's/~/$ENV{HOME}/'
```

## Awk

### Commandline args

- `-i inplace` : edits the file in-place
- `-v RS='\r'` : set a variable (see syntax section below for NR)

### Syntax

```awk
$<N>                  # numbered token; 1-based
$NF                   # last token
NR                    # line number (1-based)
RS                    # separator

==                    # string comparison
~                     # regex comparison
!~                    # negative regex comparison

/regex/               # regular expression; can be prefixed with `!`

expr { statement }    # execute `statement` if `expr` is true

ENVIRON["MYVAR"]      # access environment vars!
```

Expressions apply by default to the current string:

```
/^pizza / { print $1 } # print the first token if the line starts with "pizza "
```

Flip-flop is not supported; it must be simulated via a variable.

### Base commands

```awk
print expr
getline           # consume one line
exit
```

### Regex

- capturing groups are not supported (see [APIs](#apis))
- `\b` is not supported - use `\y`

### APIs

```sh
# match(string, regex[, captures]): gawk-only; `captures` is a standard-structured capturing array

echo 'a 8 9' | gawk '{match($0, /a ([0-9]) ([0-9])/, a); print a[1], "-", a[2]}'  # `8 - 9`
echo 'a 8 9' | gawk '{match($0, /a ([0-9]) ([0-9])/, a); print length(a)}'        # `9` ????
echo 'a 8 x' | gawk '{match($0, /a ([0-9]) ([0-9])/, a); print a[1], "-", a[2]}'  # ` - `
echo 'a 8 x' | gawk '{print match($0, /a ([0-9]) ([0-9])/, a)}'                   # 0 (equals to false in conditionals)
echo 'a 8 9' | gawk '{print match($0, /a ([0-9]) ([0-9])/, a)}'                   # 1

# substr(string, start[, number]): returns `number` chars from `string``, starting at `start` (1-based!!):

echo "Every good boy. " | awk '{print substr($1, 1, 1)}'       # `E` (first char)
echo "Every good boy. " | awk '{print substr($1, length($1))}' # `d` (last char)
echo "Every good boy. " | awk '{print substr($1, 3)}'          # `ery`

# Execute a system command

echo 'abc.txt:12:XXX' | awk -F: '{ system("echo git blame "$1" -L "$2","$2) }' # `git blame abc.txt -L 12,12`
```

### Useful examples

```sh
# Print a line after a match
#
printf '1\n2' | awk '/1/ { getline; print }' # 2

# Add a line to Nth line (0-based)
#
awk -i inplace 'NR==1{print; print "export LD_LIBRARY_PATH=/usr/local/mysql/lib:$LD_LIBRARY_PATH"} NR!=1' /etc/init/mysql.server

# Print $N tokens, starting from the third; `cut` can do the same, but has issues
#
awk '{$1=$2=""; print $0}'
```

## Sed

Cmdline params:

- `-i` : in-place editing

### Syntax

- `<n><operation>[<param>]` : execute `operation` on line `n`, optionally with operation `params`
- `<pattern> <operation>`   : execute `operation` when `pattern` matches
- `/pattern/[modifier]`     : pattern with modifier(s)
- `s|from|to|`              : custom search delimiter

### Regexes

Must use `-E`/`-r` to support extended regular expressions; !!! BETTER, BUT NOT CLEAR/COMPLETE !!!

Basic support:

- `[...]` (character sets)
- `*`
- capturing groups, but require escaping (`s|ab\(c\)|\1|`); `$<n>` reference is not supported

Some supported by `-E`:

- `\w`
- `+`
- capturing groups (`s|ab(c)|\1|`); `$<n>` reference is not supported

Not supported (at all):

- `\d`

### Operators/variables

- `$`            : last line
- `/from/,/to/p` : flip flop

### Operations

General:

- `p`: print
- `q`: quit

Operations on numbered lines (1-based):

- insert : `i<content>` (before match; includes newline)
- append : `a<content>` (after match; includes newline)
- delete : `d`.
- replace : `c<content>`.
- search/replace: `s/<search>/<replace>/[g]`

Operations with matches:

- delete: `/<pattern>/ d`
- insert (add before): `/<pattern>/ i newstring` (in order to add leading spaces, must escape the first)
- add (add after): `/<pattern>/ a newstring`

### Concatenate operations

Examples:

- `/ntp/ s/^/# /` : comment out a line matching `ntp`

### Special characters


```sh
# In order to handle tabs (`\t`), either use `$` quoting or parameter substitution:
#
sed $'s/\t/ /'
sed "s/$(printf '\t')/ /"

# WATCH OUT!! Newlines don't require `$` quoting:
#
sed 's/from/to1\nto2/'
```

## Snippets

### Extract difference between two files

Find the difference file2 - file1 (lines in file2 but not in file1):

```sh
grep -Fxvf file1 file2
```

### Compute aggregates on a text/log file

Unkeyed:

```sh
awk '{total += $4} END {print total / 2**20}' $filename
perl -a -ne '{$total += @F[3]} END {print $total / 2**20}' $filename   # `@F` is 0-based!
```

Keyed:

```sh
# Perl+SQLite version.
# There are different approaches to this (e.g. Perl-only via BEGIN+END), but this is arguably clean.
#
cat $filename |
  perl -ne 'print "INSERT INTO log_values VALUES (\"$2\", $1);" if /Completed in ([0-9.]+)ms - (\w+)/;' |
  (echo "CREATE TABLE log_values(key TEXT, value REAL); $(cat); SELECT key, SUM(value) FROM log_values GROUP BY key;") |
  sqlite3

# GAWK version
#
gawk '
{if (match($0, /Completed in ([[:digit:].]+)ms - ([[:alnum:]]+)/, m)) totals[m[2]] += m[1]}
END {for (key in totals) {print key, totals[key]}}
' $filename

# Perl version.
#
perl -ne '
$totals{$2} += $1 if /Completed in ([\d+.]+)ms - (\w+)/;
END {for $key (keys %totals) {print "$key $totals{$key}\n"}}
' $filename
```

## cut

```sh
cut -d' ' -f3						                      # extract a field from a string; [d]elimiter (default: TAB)
cut -d' ' -fM-                                # Print last N tokens (1-based, M=total-N+1): in awk, this is not trivial; cut is cleaner
git st | tail -n 12 | cut -c -14 | sort		    # cut first N chars of each line
xz -dc dump.sql.xz | cut -c 64- | head -n 10	# cut after N chars of each line
```

## tr (translate tokens)

```sh
tr -s ' '   		    # [s]queeze all ' '
tr -d ' '   		    # [d]elete all ' '
tr ab cd    		    # translate a..c → b..d
```

Examples:

```
tr -cd '\0' | wc -c	# count zero chars (example with `\0`)
```

## sort

```sh
sort -V           # compare by versions! WATCH OUT! '5.8' is sorted before '5.8.0'

# `-t/--field-separator`: separator
# `-n/--numeric-sort`: interpret as number
# `-k/--key <start,end>`: specify sorting column number (1-based); if specifying only <start>, <end>
#                         is assumed to be the last field (can be acceptable, for simplification)
#
sort -t, -n -k1

# Sort by multiple fields; in this case, it's important to use the <start,end> format.
#
# `-u/--unique`: distinct
#
sort -t, -n -k 1,1 -k 2,2 -k 3,3 -u
```

## jq

### Hash

Select from a hash; returns an array:

```sh
jq '.Versions' << JSON
{
  "Versions":      [ { "Key": "key1" } ],
  "DeleteMarkers": [ { "Key": "key2" } ]
}
JSON
# [
#   {
#     "Key": "key1"
#   }
# ]
```

### Arrays

Array iteration/filtering:

```sh
# Iterator; used to apply filters. If printed, the result is that the wrapping brackets are absent.
#
jq ".[]"                 # Iterator

# Array slicing; input: array; returns an array.
#
jq ".[$start:$end]"      # Slice an array (0-based, open end)

# Iterator operations are applied with the `|` operator; they return an iterator. In order to return an array, wrap the expression in `[]` (see git.io/J3roO).
#
jq "[.[] | $filter]"
```

Filters:

```sh
# Filter an array, based on a condition.
#
jq '.[] | select(.IsLatest)' << JSON
[
  {
    "Key": "key1",
    "IsLatest": true
  },
  {
    "Key": "key3",
    "IsLatest": true
  }
]
JSON
# {
#   "Key": "key1",
#   "IsLatest": true
# }
# {
#   "Key": "key3",
#   "IsLatest": true
# }

# Filter an array, selecting only some keys.
#
jq '.[] | {Key,VersionId}' << JSON
[
  {
    "Key": "key1",
    "VersionId": "key2",
    "IsLatest": false
  },
  {
    "Key": "key1",
    "VersionId": "key2",
    "IsLatest": false
  }
]
JSON
# {
#   "Key": "key1",
#   "VersionId": "key2"
# }
# {
#   "Key": "key1",
#   "VersionId": "key2"
# }
```

#### More complex filtering

More complex filters, with string operations:

```sh
aws rds describe-db-instances | jq '
  .DBInstances[] |                                     # Get `DBInstance` key from root hash, and iterate the array values
  {DBInstanceIdentifier,DBInstanceStatus} |            # For convenience, filter those two keys only, from each array entry
  select(.DBInstanceIdentifier | contains("00")) |     # Select only the entries whose `DBInstanceIdentifier` value includes "00"
  select(.DBInstanceStatus != "available")             # Select only the entries whose `DBInstanceStatus` value != "available"
'
```

### Functions

```sh
jq 'length'    # length/size of array/hash
```

## Silver searcher (ag)

WATCH OUT!: `.git` directory and `.log` files are ignored by default

```sh
# Ignore a file/directory.
# WATCH OUT: The pattern is not implicitly surrounded with `.*`
#
ag --ignore=$glob

# [-G] Filter in filenames matching the specified regex.
# WATCH OUT: The pattern is implicitly surrounded with `.*`
#
ag -G '_spec.rb$' $pattern [$directory]
```

## Generic snippets

### Stop tail when a string matches

```sh
# $pattern must quote backslashes, if there are any.
#
sh -c "tail --pid=\$\$ -f $(printf "%q" "$filename") | { sed -n $(printf "%q" "/$pattern/ q") && kill \$\$; }" || true
```

### Compare/sort versions

Options:

- `dpkg --compare-versions $v1 $op $v2`
  - operators: `lt`, `ge`, etc
  - there are alternative operators, e.g `ge-nl`, that treat "empty versions" (e.g. `5.8` <> `5.8.0`) differently
- `sort -V`
  - see [sort](#sort) section

Dpkg:

- has more complex rules for comparing empty versions, so they should be avoided if possible.
- considers valid a blank string, and lower than any non-blank version (including 0)

### Sum/average/etc. values extracted from a textfile

Standard Perl, if there can be non-matching lines:

```sh
perl -ne 'if (/WALLTIME:(\S+)/) { $total += $1 }; END { print $total }' $file
```

Insane version, if all the lines match:

```sh
# paste: `-s`: puts all values on a single line; `-d+`: use `+` as separator
# bc: is a calculator (!)
#
$(perl -lne 'print /WALLTIME:(\S+)/' $file | paste -sd+ | bc)
```
