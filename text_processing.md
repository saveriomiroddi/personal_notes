# Text processing

- [Text processing](#text-processing)
  - [Grep](#grep)
  - [Perl](#perl)
    - [Commandline args](#commandline-args)
    - [Operators](#operators)
    - [Data types and conversions](#data-types-and-conversions)
      - [Arrays](#arrays)
      - [Hashes](#hashes)
    - [Flow control](#flow-control)
    - [Functions/APIs](#functionsapis)
    - [Search/replace](#searchreplace)
      - [Regex extra backslash sequences](#regex-extra-backslash-sequences)
    - [Formatting/printing](#formattingprinting)
    - [Line numbers/Position-based operations](#line-numbersposition-based-operations)
    - [Priority](#priority)
    - [Useful examples](#useful-examples)
  - [Awk](#awk)
    - [Commandline args](#commandline-args-1)
    - [Syntax](#syntax)
    - [Base commands](#base-commands)
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
  - [Silver searcher (ag)](#silver-searcher-ag)
  - [Generic snippets](#generic-snippets)
    - [Sorting versions](#sorting-versions)
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

### Operators

```perl
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
### Data types and conversions

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

#### Arrays

```perl
(1, 2, 3)                             # array literal
(1..3)                                # array literal, as range; inclusive

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

printf "0\n0\n0" | perl -pe '$_ .= "abc\n" if /0/ && ++$cnt == 2'  # 2nd occurrence of the pattern (!!) => 0 0 abc 0
```

### Priority

Watch out the priority!!!

```sh
# Prints only "Line 1" and an empty line
printf 'Line 1\nLine 2' | perl -lne 'print $_; $line = readline && print $line'

# Prints "Line 1" and "Line 2"
printf 'Line 1\nLine 2' | perl -lne 'print $_; $line = readline and print $line'
printf 'Line 1\nLine 2' | perl -lne 'print $_; ($line = readline) && print $line'
```

### Useful examples

Replace only the first occurrence in a file:

```sh
perl -pe '!$found && s/.../.../ && ($found=1)'
```

Strip trailing file spaces:

```sh
perl -pe 'chomp if eof'    # only last whitespace
perl -0777 -pe 's/\s+$//'  # all the whitespaces
```

Print part of line if there is a match:

```perl
# Conventional way.
#
print $1 if /^Host: (.*)/

# Clever way! If there isn't a capturing group, all the match is printed, otherwise, only the
# capturing group. Since this is a print, without newline, lines not matching don't print anything!
#
print /^Host: (.*)/
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
=~                    # regex comparison
!~                    # negative regex comparison

/regex/               # regular expression

expr { statement }    # execute `statement` if `expr` is true
```

Expressions apply by default to the current string:

```
/^pizza / { print $1 } # print the first token if the line starts with "pizza "
```

### Base commands

```awk
print expr
getline           # consume one line
exit
```

### APIs

```sh
# match(string, regex[, captures]): gawk-only; `captures` is a standard-structured capturing array

echo 'a 8 9' | gawk '{match($0, /a ([0-9]) ([0-9])/, a); print a[1], "-", a[2]}'  # `8 - 9`
echo 'a 8 9' | gawk '{match($0, /a ([0-9]) ([0-9])/, a); print length(a)}'        # `9` ????
echo 'a 8 x' | gawk '{match($0, /a ([0-9]) ([0-9])/, a); print a[1], "-", a[2]}'  # ` - `
echo 'a 8 x' | gawk '{print match($0, /a ([0-9]) ([0-9])/, a)}'                   # 0 (equals to false in conditionals)
echo 'a 8 9' | gawk '{print match($0, /a ([0-9]) ([0-9])/, a)}'                   # 1

# substr(string, start[, number]): returns `number` chars from `string``, starting at `start` (1-based!!):

echo "Every good boy. " | awk '{print substr($1, 1, 1)}'   # `E` (first char)
echo "Every good boy. " | awk '{print substr($1, 3)}'      # `ery`
```

### Useful examples

```sh
# Print a line after a match
#
echo $'1\n2' | awk '/1/ { getline; print }' # 2

# Add a line to Nth line (0-based)
#
awk -i inplace 'NR==1{print; print "export LD_LIBRARY_PATH=/usr/local/mysql/lib:$LD_LIBRARY_PATH"} NR!=1' /etc/init/mysql.server
```

## Sed

Cmdline params:

- `-i` : in-place editing

### Syntax

- `<n><operation>[<param>]` : execute `operation` on line `n`, optionally with operation `params`
- `<pattern> <operation>`   : execute `operation` when `pattern` matches
- `/pattern/[modifier]`     : pattern with modifier(s)

### Regexes

Must use `-E`/`-r` to support extended regular expressions; !!! BETTER, BUT NOT CLEAR/COMPLETE !!!

Basic support:

- `[...]` (character sets)
- `*`

Some supported by `-E`:

- `\w`
- `+`

Capturing groups are not supported.

### Operators/variables

- `$`            : last line
- `/from/,/to/p` : flip flop

### Operations

General:

- `p`: print
- `q`: quit

Operations on numbered lines (1-based):

- insert : `i<content>` (before match; includes newline)
- insert : `a<content>` (after match; includes newline)
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
ag -G '_spec.rb$' <pattern> [directory]
```

## Generic snippets

### Sorting versions

Options:

- `dpkg --compare-versions $v1 $op $v2`
  - operators: `lt`, `ge`, etc
  - there are alternative operators, e.g `ge-nl`, that treat "empty versions" (e.g. `5.8` <> `5.8.0`) differently
- `sort -V`
  - see [sort](#sort) section

Dpkg has more complex rules for comparing empty versions, so they should be avoided if possible.

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
