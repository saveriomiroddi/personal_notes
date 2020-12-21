# Git

- [Git](#git)
  - [Document notes](#document-notes)
  - [General concepts](#general-concepts)
  - [Configuration](#configuration)
    - [Aliases](#aliases)
    - [Ignore commands/Gitignore](#ignore-commandsgitignore)
  - [Repository](#repository)
  - [Log](#log)
    - [Prettifications](#prettifications)
    - [More complex examples](#more-complex-examples)
  - [Format](#format)
  - [Merging](#merging)
  - [Rebase](#rebase)
  - [Stash](#stash)
  - [Bisect](#bisect)
  - [Export (`archive`)](#export-archive)
  - [Informations gathering](#informations-gathering)
  - [Batch operations  (message/tree filtering)](#batch-operations--messagetree-filtering)
  - [Patching](#patching)
  - [Useful operations](#useful-operations)
    - [Correct whitespaces problems](#correct-whitespaces-problems)

## Document notes

The `git` command is implicit in all the examples, except where:

- it's a sequence of commands
- it's a single command with pipe

## General concepts

- `Target branch` : git merge <target_branch>
- `First parent`  : (of a merge) the current branch when doing a merge

Conventions:

- `$branch~[$n]` : reference to `$branch` minus `$n` (0 is also valid, and it refers to `$branch`)
- `branch^`      : `$branch` - 1
- `HEAD`         : head of the current branch (current commit)
- `ORIG_HEAD`    : last value of HEAD before dangerous operations
- `$ref@{-$n}`   : nth checkout previous to the last one; eg. checkout master => checkout feature => now `@{-1}` refers to master

## Configuration

`git config [--global]`: stores config in `.git/config`; [global] stores in `~/.gitconfig`

| Parameter                 |    Value    | Comment                                            |
| ------------------------- | :---------: | -------------------------------------------------- |
| `user.name`               |    name     |                                                    |
| `user.email`              |    email    |                                                    |
| `core.editor`             |   editor    |                                                    |
| `diff.tool`               | merge_tool  |                                                    |
| `difftool.merge_tool.cmd` |   cmdline   | example: "winmerge.sh \"$LOCAL\" \"$REMOTE\""      |
| `difftool.prompt`         |    false    |                                                    |
| `alias.alias`             | 'arguments' | if arguments is only one, quotes can be omitted    |
| `branch.autosetuprebase`  |   always    | automatically rebase instead of merge when pulling |

### Aliases

General notes:

- always surround complex aliases with double quotes; internal double quotes must be escaped
- the default shells is dash (sh)
- the current path is the project dir; see the `GIT_PREFIX` workaround below
- see `man git-config` for other infos

```
# Use bash and a complex command; note that without `sh` at the end, the positional parameters don't behave as expected
# IMPORTANT: doesn't work with filenames with spaces.
#
cs = "!bash -c 'cd \"${GIT_PREFIX:-.}\"; git checkout --$1 \"${@:2}\" && git add \"${@:2}\"' sh"

# Alternative form for complex commands (uses dash)
#
stsh = "!f() { git stash show -p stash@{${1-0}}; }; f"
```

### Ignore commands/Gitignore

```sh
config --global core.excludesfile ~/.gitignore_global       # create a global gitignore

update-index --[no-]assume-unchanged $file                  # ignore changes to a file in the index
echo '/.ruby-version' >> .git/info/exclude                  # ignore an untracked file only in the local repository
```

Ignore a directory but not a file levels below it; in order for this to work, all the directories in the middle must be excluded.

```
/application/**                           # this must end with `/**`
!/application/language/                   # files in this directory will not be included
!/application/language/gr/
!/application/language/gr/SomeFile.txt    # `/**` also works
```

For one level, it's easier:

```
/terraform/**/.terraform/plugins/*/*
!/terraform/**/.terraform/plugins/*/terraform-provider-archive_v*
```

## Repository

```sh
init       # init a repository
clone $url # clone a remote repos
```

## Log

```sh
log --branches=$branch $file        # search in another branch

log --merges --first-parent         # search only [merges] commits, only in the [first-parent] (eg. master when run from master)

# commits reachable by $r2 but not by $r1
# alias: `^$r1 $r2`
log $r1..$r2

# commits reachable by either $r1 or $r2, but not both
# alias: `$r1 $r2 --not $(git merge-base --all $r1 $r2)`
log $r1...$r2

log --diff-filter=[M|D] -- $file    # search for [M]odifications/[D]eletions of a file (`D` requires `--`)

log --grep=$regex                   # search by log message (case insensitive); simple regex, supports at least: '.*[]'; doesn't support: '+(){}'
log --author $email_pattern         # search by author; pattern is a substring, and also accepts `*`
log -G $regex [$path]               # search $regex in the diff
log -S $string [$path]              # search the deletions for $string
```

### Prettifications

- `--pretty='%h %s'`   : `%h`: abbreviated hash, %s: subject (first line)
- `--pretty="format:"` : empty format, to avoid the cruft at the beginning of logs
- `--oneline`          : same as `--pretty='%h %s'`

### More complex examples

```sh
log --merges v0.1.8...v0.1.9        # search merges between two tags
```

## Format

```sh
--format="%h %b"        # $commit $body; if a body has more body lines, they will be shown, so some trickery is required
```

## Merging

```sh
checkout (--ours|--theirs) $files   # resolve conflict using ours/theirs version
```

## Rebase

Add a root (initial) commit:

```sh
git add --allow-empty -m "Initial (empty) commit"
git rebase -i --root                               # now reorder the commits
```

Programmatically run an interactive rebase:

```sh
# Edit all the commits of a branch!
#
GIT_SEQUENCE_EDITOR="sed -i -re 's/^pick /e /'" git rebase -i "$(git merge-base HEAD master)"
```

## Stash

```sh
stash                         # save current changes to the stash
stash save [--patch]          # !! `--patch` = interactive stashing (can choose what to save) !!
stash (pop|apply)             # restore with/out clearing the entry
stash list
stash clear
stash show -p $stash_entry    # shows a stash entry content
```

## Bisect

```sh
bisect good $rev      # Mark an old enough rev as good: git automatically checks out the middle revision
bisect run $command   # Bisect using command exit code to find failures; if the command has aliases/etc, use `bash -c '<commands>'` as command.
```

## Export (`archive`)

```sh
fast-export --all      # export repo
fast-import            # import repo
archive $rev           # export a rev to stdout (defaults to tar format)

# Example export->import into another directory.
#
archive master Gemfile Gemfile.lock gems_dir | tar x -f - -C /export
```

## Informations gathering

Low(er) level branch information; can be used to find the remote branch

```sh
# The output listed is exact.

status -b --porcelain
## test...origin/test [gone]

status -b --porcelain
## master...origin/master
```

## Batch operations  (message/tree filtering)

Mass-change all the messages in the current branch.

`--msg-filter` receives the commit message from stdin, and set as new message the output of the user-specified command.

Note that the [git filter-repo](https://github.com/newren/git-filter-repo) tool should be used instead. In order to take this approach, `export FILTER_BRANCH_SQUELCH_WARNING=1`.

```sh
# Rename commit titles.
git filter-branch --force --msg-filter 'perl -pe "s/^(\[Chef\] )?/[Chef] /i"' -- $(git merge-base master HEAD)..HEAD

# Delete a file
git filter-branch --force --tree-filter 'rm -f terraform/terraform.tfstate' master..HEAD

# Delete a Ruby method.
git filter-branch --force --tree-filter 'ag "def mymethod" -l | xargs -r perl -0777 -i -pe "s/^(\s+)def mymethod.*?^\g1end\n\n//sm"' master..HEAD
```

## Patching

```sh
# Create standard patch, to be applied with `am`. `diff` can be used, but the metadata (e.g. author) will be lost.
#
# - produce diff for $file only
# - creates a series of patches starting from $start_commit until HEAD or $end_commit
#
format-patch -1 $start_commit^[..$end_commit] [$file] [--stdout]

# Apply a patch and commit.
#
am $infile

# Apply without commit
#
# - [check] instead of apply
#
apply [--check] $infile
```

## Useful operations

### Correct whitespaces problems

```sh
git add .
git diff --cached -w --no-color | git apply --cached -R
git commit -m 'Whitespace changes'
git commit -am 'All other changes'
```

Update git history, cleaning whitespace problems!:

```sh
rebase $base_commit --whitespace=fix
```
