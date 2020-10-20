# Git

- [Git](#git)
  - [General concepts](#general-concepts)
  - [Log](#log)
    - [More complex examples](#more-complex-examples)
  - [Format](#format)
  - [Merging](#merging)
  - [Rebase](#rebase)
  - [Bisect](#bisect)
  - [Export (`archive`)](#export-archive)
  - [Informations gathering](#informations-gathering)
  - [Ignoring](#ignoring)
  - [Batch operations  (message/tree filtering)](#batch-operations-messagetree-filtering)

## General concepts

```sh
HEAD~$n                             # reference to HEAD minus $n (0 is also valid, and it refers to HEAD)
```

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

## Rebase

Add a root (initial) commit:

```sh
git add --allow-empty -m "Initial (empty) commit"
git rebase -i --root	# and reorder the commits
```

Programmatically run an interactive rebase:

```sh
# Edit all the commits of a branch!
#
GIT_SEQUENCE_EDITOR="sed -i -re 's/^pick /e /'" git rebase -i "$(git merge-base HEAD master)"
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

## Ignoring

```sh
update-index --[no-]assume-unchanged $file                  # ignore changes to a file in the index
echo '/.ruby-version' >> .git/info/exclude                  # ignore an untracked file only in the local repository
git config --global core.excludesfile ~/.gitignore_global   # create a global gitignore
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
