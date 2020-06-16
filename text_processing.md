# Text processing

- [Text processing](#text-processing)
  - [Perl](#perl)
    - [General concepts](#general-concepts)
      - [Priority](#priority)
    - [Regex extra backslash sequences](#regex-extra-backslash-sequences)
    - [Useful examples](#useful-examples)
  - [Sed](#sed)
    - [Useful examples](#useful-examples-1)
    - [Special characters](#special-characters)

## Perl

### General concepts

#### Priority

Watch out the priority!!!

```sh
# Prints only "Line 1" and an empty line
printf 'Line 1\nLine 2' | perl -lne 'print $_; $line = readline && print $line'

# Prints "Line 1" and "Line 2"
printf 'Line 1\nLine 2' | perl -lne 'print $_; $line = readline and print $line'
printf 'Line 1\nLine 2' | perl -lne 'print $_; ($line = readline) && print $line'
```

### Regex extra backslash sequences

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

### Useful examples

Strip trailing file spaces: `chomp if eof`

## Sed

Must use `-E` to support extended regular expressions!

Basic support:

- `[...]` (character sets)
- `*`

Some supported by `-E`:

- `\w`
- `+`

Capturing groups are not supported.

### Useful examples

Operations on numbered lines (1-based):

- insert : `<n>i<content>`
- delete : `<n>d`.
- replace : `<n>c<content>`.

- search/replace on a given line: `<n>s/<search>/<replace>/[g]`

With regexes, put the operation after the pattern:

- delete: `/<pattern>/d`

### Special characters

In order to handle tabs (`\t`), either use `$` quoting or parameter substitution:

```sh
sed $'s/\t/ /'
sed "s/$(printf '\t')/ /"
```
