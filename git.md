# Git

- [Git](#git)
  - [Log](#log)
    - [More complex examples](#more-complex-examples)
  - [Format](#format)
  - [Merging](#merging)
  - [Bisect](#bisect)
  - [Export (`archive`)](#export-archive)
  - [Informations gathering](#informations-gathering)

## Log

```sh
log --branches=$branch $file        # search in another branch

log --merges --first-parent		      # search only [merges] commits, only in the [first-parent] (eg. master when run from master)

# commits reachable by $r2 but not by $r1
# alias: `^$r1 $r2`
log $r1..$r2 

# commits reachable by either $r1 or $r2, but not both
# alias: `$r1 $r2 --not $(git merge-base --all $r1 $r2)`
log $r1...$r2

log --diff-filter=[M|D] -- $file		# search for [M]odifications/[D]eletions of a file (`D` requires `--`)

log --grep=$regex                   # search by log message (case insensitive); simple regex, supports at least: '.*[]'; doesn't support: '+(){}'
log --author $email_pattern         # search by author; pattern is a substring, and also accepts `*`
log -G $regex [$path]		            # search $regex in the diff
log -S $string [$path]              # search the deletions for $string
```

### More complex examples

```sh
log --merges v0.1.8...v0.1.9	      # search merges between two tags
```

## Format

```sh
--format="%h %b"        # $commit $body; if a body has more body lines, they will be shown, so some trickery is required
```

## Merging

```sh
checkout (--ours|--theirs) $files   # resolve conflict using ours/theirs version
```

## Bisect

Run using the command passed exit code in order to discern failures:

```sh
bisect run <command>'
```

If the command has aliases/etc, use `bash -c '<commands>'` as command.

## Export (`archive`)

Export in tar format:

```sh
# Example export->import into another directory.
archive master Gemfile Gemfile.lock gems_dir | tar x -f - -C /export
```

## Informations gathering

Low(er) level branch information; can be used to find the remote branch

```sh
# The output listed is exact.

$ git status -b --porcelain
## test...origin/test [gone]

$ git status -b --porcelain
## master...origin/master
```