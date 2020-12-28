# Text processing

- [Text processing](#text-processing)
  - [Grep](#grep)
  - [Perl](#perl)
    - [Flow control/Operators](#flow-controloperators)
    - [Commandline args](#commandline-args)
    - [Line numbers/Position-based operations](#line-numbersposition-based-operations)
    - [Priority](#priority)
    - [Data types and conversions](#data-types-and-conversions)
      - [Arrays](#arrays)
    - [Functions/APIs](#functionsapis)
    - [Search/replace](#searchreplace)
      - [Regex extra backslash sequences](#regex-extra-backslash-sequences)
    - [Formatting/printing](#formattingprinting)
    - [Useful examples](#useful-examples)
  - [Sed](#sed)
    - [Syntax](#syntax)
    - [Regexes](#regexes)
    - [Operators](#operators)
    - [Operations](#operations)
    - [Special characters](#special-characters)

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

### Flow control/Operators

```perl
# If (and blocks)
if (CONDITION) { TRUE_BRANCH } else { ELSE_BRANCH }

# Ternary operator
CONDITION ? TRUE_BRANCH : FALSE_BRANCH

# Flip-flop
print if /DELIMITER ;;$/ .. /DELIMITER ;$/

# String operators (!!! DON'T USE == !!!)
'a' eq 'a' # 1
'a' ne 'b' # 1
```

### Commandline args

- `-0`: use null character as line separator

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

### Data types and conversions

```sh
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

foreach (@Array) { SubRoutine($_); }  # iterate an array
map { SubRoutine($_) } @Array;        # map an array
List::Util qw/sum/ -> sum(@Array)     # sum the elements of an array
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

### Useful examples

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

### Operators

`/from/,/to/p` : flip flop

### Operations

General:

- `p`: print
- `q`: quit

Operations on numbered lines (1-based):

- insert : `i<content>` (includes newline)
- delete : `d`.
- replace : `c<content>`.
- search/replace: `s/<search>/<replace>/[g]`

Operations with matches:

- delete: `/<pattern>/ d`
- insert (add before): `/<pattern>/ i newstring` (in order to add leading spaces, must escape the first)
- add (add after): `/<pattern>/ a newstring`

### Special characters

In order to handle tabs (`\t`), either use `$` quoting or parameter substitution:

```sh
sed $'s/\t/ /'
sed "s/$(printf '\t')/ /"
```
