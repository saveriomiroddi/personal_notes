# Git

- [Git](#git)
  - [Document notes](#document-notes)
  - [General concepts](#general-concepts)
  - [Configuration](#configuration)
    - [Aliases](#aliases)
    - [Ignore commands/Gitignore](#ignore-commandsgitignore)
  - [Repository](#repository)
  - [Log/Blame](#logblame)
    - [Formatting/Prettifications](#formattingprettifications)
    - [Finding](#finding)
  - [Metadata](#metadata)
  - [Merging](#merging)
  - [Rebase](#rebase)
  - [Remotes](#remotes)
  - [Stash](#stash)
  - [Bisect](#bisect)
  - [Export (`archive`)](#export-archive)
  - [Batch operations  (message/tree filtering)](#batch-operations--messagetree-filtering)
  - [Diffing/Patching](#diffingpatching)
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

## Log/Blame

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

Blaming format:

```sh
blame [$branch] $file
```

### Formatting/Prettifications

`--format` and `--pretty` are the same.

- `--pretty='%h %s'`   : `%h`: abbreviated hash, `%s`: subject first line
- `--pretty="format:"` : empty format, to avoid the cruft at the beginning of logs
- `--oneline`          : same as `--pretty='%h %s'`

### Finding

```sh
merge-base $A $B                                       # find common ancestor
show :/$regex                                          # show the last commit matching a regex in the message
branch --contains $commit                              # shows which branches contains the given commit
name-rev --name-only $commit                           # shows which tag the commit was in. '~N' indicates how many commits (N) before the tag
git rev-list --pretty=oneline --before="%F %R" $branch # show commits before/at given datetime
log --merges v0.1.8...v0.1.9                           # search merges between two tags

[[ $(git cat-file -t $object 2> /dev/null) ]] && echo exists # check if a branch/commit/etc exists
```

## Metadata

```sh
# Reference to upstream (remote) branch of HEAD. Returns a blank line if there isn't one.
#
for-each-ref --format='%(upstream:short)' $(git symbolic-ref -q HEAD)

# Repository name
#
ls-remote --get-url origin | ruby -ne 'print $_[/(\w+)\.git/, 1]'

# Repository root path
#
rev-parse --show-toplevel

# Name of the current branch
#
rev-parse --abbrev-ref HEAD   # on non named branches (including remote ones) prints `HEAD`
symbolic-ref --short -q HEAD  # on non named branches prints nothing
```

Porcelain (low(er) level information, intended to be programmatically parsed); can be used to find the remote branch:

```sh
# The output listed is exact. Show [b]ranch info.

status -b --porcelain
## test...origin/test [gone]

status -b --porcelain
## master...origin/master
```

## Merging

```sh
checkout (--ours|--theirs) $files   # resolve conflict using ours/theirs version
```

Examples:

```sh
# Solve conflicts using always the local/remote copy (assumes no spaces in filenames).
# Use `--ours` (local) or `--theirs` (remote):
#
git st | awk '/both modified:/ { print $3 }' | xargs sh -c 'git checkout --theirs {} && git add {}'
```

## Rebase

```sh
rebase -i $parent_commit                               # interactive rebase from parent of given commit
rebase --onto $dest_branch $feat_branch_parent_commit

# Add a root (initial) commit
#
git add --allow-empty -m "Initial (empty) commit"
git rebase -i --root

# Programmatically run an interactive rebase. Example: edit all the commits of a branch:
#
GIT_SEQUENCE_EDITOR="sed -i -re 's/^pick /e /'" git rebase -i "$(git merge-base HEAD master)"
```

## Remotes

```sh
fetch [--all]                                            # fetch remote changes; [all] remotes
pull [--rebase]                                          # fetch+merge; do [rebase] instead of merging

remote                                                   # list the remotes; a remote is a label for a remote repos url
remote add $name $remote_url                             # add a remote (eg. for forks (upstream))
push [-u] [$remote [$branch[$rem_branch]]] --tags        # push changes; optionally only for a [branch] - if it's a branch, a remote branch is created; push
                                                         # set [u]pstream; set [rem_branch] if name differs from local; push [tags] information

push $remote :$branch                                    # delete a remote branch
push $remote :heads/$branch                              # delete a remote branch
push $remote :refs/tags/$tag                             # delete a remote tag
```

Examples:

```sh
# List status of all the local branches.
#
for-each-ref --format="%(refname:short) %(upstream:track)" refs/heads
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

Examples:

```sh
# Recover dropped stashes
#
gitk --all $(git fsck --no-reflog | awk '/dangling commit/ {print $3}')
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

## Diffing/Patching

```sh
show --name-[only|status] rev[:file]            # diff rev^..rev; show name only [--name-only] or status only [--name-status]

diff [--cached]                                 # non committed files; [--cached] files in the index
diff --stat                                     # only filenames
difftool                                        # GUI version of diff
git difftool --extcmd='vim -d -c "windo set wrap" $5'    # Convenient vimdiff usage; requires `diffchar` plugin, otherwise, it's ugly

status -sb                            more compact version of the status (show [b]ranch; first column: in index
```

```sh
# Suitable for standard patch import; if [--no-prefix] is not specified, import using "patch -p1"
#
diff --no-prefix

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
