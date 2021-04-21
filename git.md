# Git

- [Git](#git)
  - [Document notes](#document-notes)
  - [General concepts](#general-concepts)
  - [Configuration](#configuration)
    - [Aliases](#aliases)
    - [Ignore commands/Gitignore](#ignore-commandsgitignore)
  - [Commit](#commit)
  - [Index operations](#index-operations)
  - [Repository](#repository)
  - [Log/Blame/Tree comparison](#logblametree-comparison)
    - [Formatting/Prettifications](#formattingprettifications)
    - [Pretty formats](#pretty-formats)
    - [Finding](#finding)
  - [Metadata](#metadata)
  - [Merging](#merging)
  - [Rebase](#rebase)
    - [Preserve merges when rebasing](#preserve-merges-when-rebasing)
  - [History rewrite](#history-rewrite)
    - [Remove a file](#remove-a-file)
    - [Remove a merge](#remove-a-merge)
  - [Remotes](#remotes)
  - [Stash](#stash)
  - [Bisect](#bisect)
  - [Export (`archive`)](#export-archive)
  - [Batch operations (message/tree filtering, files removal)](#batch-operations-messagetree-filtering-files-removal)
  - [Diffing/Patching/Status](#diffingpatchingstatus)
  - [Useful operations](#useful-operations)
    - [Shell prompt](#shell-prompt)
    - [Correct whitespaces problems](#correct-whitespaces-problems)
    - [Revert common mistakes](#revert-common-mistakes)

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
- `-$n`          : !!! HEAD - `$n` !!!
- `HEAD`         : head of the current branch (current commit)
- `ORIG_HEAD`    : last value of HEAD before dangerous operations
- `$ref@{-$n}`   : nth checkout previous to the last one; eg. checkout master => checkout feature => now `@{-1}` refers to master

## Configuration

```sh
config [--global]   # stores config in `.git/config`; [global] stores in `~/.gitconfig`
help -c             # list all possible configuration keys
```

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

It's possible to ignore a directory also by adding a gitignore to it, containing `*`.

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

## Commit

```sh
--no-commit                     	# doesn't commit for operations that does so by default (e.g. merge)

commit --amend [-m <newMessage>]  # amend the last commit; content of the index are added to the last commit; only [m]essage 

# Set the message/authorship/timestamp of a commit as another [retain timestamp on squash].
# Will also copy the message; [-c] prompts for the new message.
#
commit --amend -C $reference_commit
```

## Index operations

```sh
add -i                          	# interactive adding
rm --cache					              # remove from the index (without deleting the file)

reset --(soft|hard) $commit     	# move the head pointer to the commit. if hard, the index is reset; if soft, the index will
                                  # contain the diff with the old data

git reset --soft $commit; git commit -c ORIG_HEAD    # amend a commit for more complex cases

clean [-d] [-f]                 	# remove untracked files; [d]irectories as well; [f]orce if option `clean.requireForce` is set

# perform reset HEAD and checkout of a file in staging; '--' is optional.
#
checkout HEAD -- $path

checkout -b $name $commit         # create branch from commit
```

## Repository

```sh
init       # init a repository
clone $url # clone a remote repos
```

## Log/Blame/Tree comparison

```sh
log --oneline --date-order --graph --all --decorate   # !! graph/diagram of the commits !!
script --return --quiet -c "git --no-pager log --no-color --graph --oneline" /dev/null > /path/to/out # dump the history to a file

log --branches=$branch $file        # search in another branch

log --merges --first-parent         # search only [merges] commits, only in the [first-parent] (eg. master when run from master)

log -$n                             # show the latest $n commits, e.g. -1 for the last

# commits reachable by $r2 but not by $r1
# alias: `^$r1 $r2`
log $r1..$r2

# commits reachable by either $r1 or $r2, but not both
# alias: `$r1 $r2 --not $(git merge-base --all $r1 $r2)`
log $r1...$r2

log --diff-filter=[M|D] -- $file    # search for [M]odifications/[D]eletions of a file (`D` requires `--`)

log --grep=$regex                   # search by log message (case insensitive); simple regex, supports at least: '.*[]'; doesn't support: '+(){}'
log --author $email_pattern         # search by author; pattern is a substring, and also accepts `*`
log -S $string [$path]              # search the changes (additions/deletions) for $string, as string
log -G $regex [$path]               # search the changes (additions/deletions) for $string, as regex
```

Blaming format:

```sh
blame [$branch] $file

git show $(git blame example.js -L 4,4 | awk '{print $1}') # show the commit of a certain line
```

Tree comparison:

```sh
cherry [-v] $upstream [$head]     # changes against tree (useful for rebasing: cherry master); [v]iew commit subjects
                                  # example commits in branch and not in master: cherry master branch
```

### Formatting/Prettifications

`--format` and `--pretty` are the same.

- `--pretty="format:"` : empty format, to avoid the cruft at the beginning of logs
- `--oneline`          : same as `--pretty='%h %s'`

### Pretty formats

- `%h` : abbreviated hash
- `%s` : subject first line (title)

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

# Git path relative to repository, from an absolute path. Note that the file must be in HEAD.
#
ls-tree --full-name --name-only HEAD $filename
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

### Preserve merges when rebasing

It's possible to preserve merges using `--rebase-merges`, although it's not entirely clear how to do funky manipulations of history.

## History rewrite

### Remove a file

In order to remove a file from the history, it's easier to use the [BFG Repo-Cleaner](https://rtyley.github.io/bfg-repo-cleaner).

WATCH OUT! Don't forget to unprotect branches (e.g. `master`), otherwise, after pushing, the BFGRC will leave the repository in an inconsistent (history-wise) state.

### Remove a merge

To remove the merge `cleaner_pigz_compiling_flags`:

```
*   e1a9575 (HEAD -> master, origin/master) Merge branch 'increase_start_vm_timeout'
|\
| * f92c47a setup_system: Increase timeout when connecting to the VM
|/
*   8473f42 Merge branch 'cleaner_pigz_compiling_flags'
|\
| * 3b9f0df setup_system: Improve static compilation flags strategy
|/
*   fef2766 Merge branch 'add_qemu_log'
```

Remove the ones prefixed with `> `:

```
# Branch cleaner-pigz-compiling-flags
> reset onto
> pick 3b9f0df setup_system: Improve static compilation flags strategy
> label cleaner-pigz-compiling-flags

# Branch increase-start-vm-timeout
reset onto
> merge -C 8473f42 cleaner-pigz-compiling-flags # Merge branch 'cleaner_pigz_compiling_flags'
label branch-point
pick f92c47a setup_system: Increase timeout when connecting to the VM
label increase-start-vm-timeout
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

## Batch operations (message/tree filtering, files removal)

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

## Diffing/Patching/Status

```sh
show --name-[only|status] rev[:file]            # diff rev^..rev; show name only [--name-only] or status only [--name-status]
show -m $merge                                  # show the full diff of the parents of a merge

diff [--cached]                                 # non committed files; [--cached] files in the index
diff --stat                                     # only filenames
difftool                                        # GUI version of diff
git difftool --extcmd='vim -d -c "windo set wrap" $5'    # Convenient vimdiff usage; requires `diffchar` plugin, otherwise, it's ugly

status -sb                                      # more compact version of the status (show [b]ranch; first column: in index

# Run a certain operation for each modified file.
#
git status -s | awk '{print $2}' | xargs -I {} scp {} myserver:mypath/{}
```

Diff/Patching:

```sh
# Patch with metadata retain (and commit)
#
# When specifying $start_commit/$end_commit, several patches will be created.
#
format-patch [--stdout] [$start_commit^[..$end_commit]]
am $patchfile

# "Transfer" commit between two repositories (use `$sha~..$sha` for single commit):
#
git -C ~/code/riscv_images format-patch --stdout -1 |
  sed 's/guest_benchmark_pigz/altscripts\/guest_benchmark_pigz/g' |
  git am

# Patch with metadata loss (and no commit)
#
# WATCH OUT! One can configure `diff.noprefix true`, however, `format-patch` will need the prefixes
# to be specified in one way or another (since they're mandatory, and `format-patch` has no configuration
# entries).
#
# - diff: suitable for standard patch import; with prefix, import using "patch -p1"
# - apply: [check] instead of apply
#
diff --no-prefix
apply [--check] $patchfile
```

## Useful operations

### Shell prompt

```sh
# Shows branch in the terminal
#
export PS1="\h:\W:\$(git branch 2>/dev/null | grep '^*' | colrm 1 2)\$ "
```

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

### Revert common mistakes

See: https://sethrobertson.github.io/GitFixUm/fixup.html
