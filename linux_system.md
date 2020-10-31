# Linux system

- [Linux system](#linux-system)
  - [Filesystems/partitions](#filesystemspartitions)
  - [Packages](#packages)
    - [Apt/dpkg installation hooks (`etc/apt/apt.conf.d`)](#aptdpkg-installation-hooks-etcaptaptconfd)
  - [Repositories](#repositories)
    - [Keys handling](#keys-handling)
  - [Debconf](#debconf)
  - [Debian alternatives](#debian-alternatives)
  - [Environment](#environment)
    - [Available variables](#available-variables)
    - [Shell (initscripts)](#shell-initscripts)
      - [Example cases](#example-cases)
      - [sudo -i, login shell test, and bash](#sudo--i-login-shell-test-and-bash)
  - [Cron](#cron)
  - [Systemd](#systemd)
    - [Systemctl](#systemctl)
    - [sournalctl](#sournalctl)
  - [Configuring a unit](#configuring-a-unit)
  - [fstab](#fstab)
  - [Logging/syslog/tools](#loggingsyslogtools)
  - [Terminal emulator](#terminal-emulator)
  - [Desktop Environment: windows](#desktop-environment-windows)
  - [Dconf/Gsettings](#dconfgsettings)
  - [MIME (extensions) (file associations) handling](#mime-extensions-file-associations-handling)
    - [Adding and associating a new application](#adding-and-associating-a-new-application)
    - [Split MAFF association](#split-maff-association)
  - [Hardware](#hardware)

## Filesystems/partitions

```sh
fdisk -l                        # list the partitions

cfdisk -z $disk_device          # GUI: create an empty partition

mkfs.ext4 $part_device          # create the FS

# Via sgdisk (more modern approach).
#
# - `-t<partnum>:type` is the partition type (see https://askubuntu.com/q/703443 and https://en.wikipedia.org/wiki/Partition_type).
# - `n<partnum>:start_pos:end_pos`
#   - `partnum`: if 0, the first available partition is chosen
#   - `+start_pos` is relative to the previous partition
#   - `+end_pos` is relative to `start_pos`
#
sgdisk -n1:1M:+512M -t1:EF00 "$selected_disk" # start at absolute 1M, ends at absolute (1+512)M
sgdisk -n2:0:-2G    -t2:BF01 "$selected_disk" # start after last partion, end 2G before end of available space
sgdisk -n3:0:0      -t3:BF01 "$selected_disk" # start after last partion, ends at end of available space

# Resize to given size/expand to fill empty space.
# Watch out the `--` required when passing a negative reference (e.g. `-18G`).
#
parted -s $disk_device unit s resizepart $part_number_1_based -- -18G
parted -s $disk_device resizepart $part_number_1_based 100%

# Remove a partition

parted -s $disk_device rm $part_number_1_based
```

## Packages

Informations:

```sh
# Show package info; for this operation, `apt` is stable [enough].
#
apt show $package

# Check if a package is installed (**with pipefail**), in the most standard way possible
#
if grep -qP "^trash-cli\s+install$" <<< $(dpkg --get-selections); then echo installed; fi
dpkg --get-selections | grep '^trash-cli ' # interactive

# Advanced filtering with aptitude.
#
# Aptitude uses regexes by default, but it doesn't support all the patterns.
# In order to filter out i386 packages, use the suffix `~rnative`.
# The option `-F %p` prints only the package name.
#
aptitude search -F %p 'suld-driver2-[0-9.]+$~rnative'

# Show dependencies of a package
#
aptitude why -v $package

# Display the sources of given package
#
apt-cache policy $package

# Find the repository of a package
#
apt-cache showpkg $package

# (Extra tool) Check dependencies of installed package.
#
apt-rdepends $package
```

Package/file operations:

```sh
# Downgrade to previous version of a package
#
apt-get install $package=$version

# Mark a package as manually installed (apt[itude])
#
aptitude install "${package}&m"

# Install/remove package ignoring dependencies, via dpkg
#
dpkg (-i|-r) --ignore-depends=libdbd-mysql-perl libmysqlclient18

# Download the packages without installing (dependencies are not downloaded).
#
aptitude download ($packages)

# Download source code for package
#
apt-get source $package

# List the contents of a deb file
#
dpkg-deb -c $package.deb

# Unpack deb file
#
ar -vx $package.deb
tar xvf data.tar.gz

# Find which package a file belongs to
#
dpkg -S $file
```

Useful generic snippets:

```sh
# Remove linux kernels other than the current one
# Note that if there is a version newer than the current (uname -r), it will be deleted.
#
dpkg -l 'linux-*' | sed '/^ii/!d;/'"$(uname -r | sed "s/\(.*\)-\([^0-9]\+\)/\1/")"'/d;s/^[^ ]* [^ ]* \([^ ]*\).*/\1/;/[0-9]/!d' | xargs sudo apt-get -y purge

# Purge packages matching a pattern, only if they're installed.
# apt can't search for a pattern and an installation status in one pass
#
aptitude purge '~i ruby1.9'
apt list --installed | perl -ne 'print "$1\n" if /^(<pattern>)\//' | xargs apt purge

# Find packages installed via PPA
#
apt-cache policy $(dpkg --get-selections | grep -v deinstall$ | awk '{ print $1 }') | perl -e '@a = <>; $a=join("", @a); $a =~ s/\n(\S)/\n\n$1/g;  @packages = split("\n\n", $a); foreach $p (@packages) {print "$1: $2\n" if $p =~ /^(.*?):.*?500 http:\/\/ppa\.launchpad\.net\/(.*?)\s/s}'

# Remove PPA versions of packages: use `ppa-purge` utility.
# Comments out the PPA file content after purging.
#
ppa-purge ppa:oibaf/graphics-drivers
```

### Apt/dpkg installation hooks (`etc/apt/apt.conf.d`)

Add the following to `/etc/apt/apt.conf.d/99-post-upgrade`:

- `Dpkg::Post-Invoke {"command";};`: Runs for every dpkg execution (package essentially) invoked by apt
- `APT::Update::Post-Invoke-Success {"command";};`: Run after successful apt update (**update only**)
- `APT::Update::Post-Invoke {"command";};`: Run after apt update (independently of exit status)

See [apt repository](https://salsa.debian.org/apt-team/apt.git) (search `RunScripts`).

Comments can be added with `//` and `/* */`.

## Repositories

### Keys handling

List keys:

```sh
apt-key list

# /etc/apt/trusted.gpg
# --------------------
# pub   rsa4096 2014-11-17 [SC]
#       7053 72CB 1DF3 976C 44B7  B8A6 BBE4 75EB A386 83E4
# uid           [ unknown] Scout Packages (archive.scoutapp.com) <support@scoutapp.com>
# uid           [ unknown] Scout (scoutapp.com) <support@scoutapp.com>
# sub   dsa2048 2014-11-17 [S]
# sub   rsa4096 2014-11-17 [E]
#
# pub   rsa4096 2019-07-05 [SC]
#       394F 883E 0C43 5694 50FD  FB92 A9AF 1C7C 2ED6 5CC0
# uid           [ unknown] Fullstaq Ruby <info@fullstaq.com>
# sub   rsa4096 2019-07-05 [E]
```

Remove key:

```sh
# WATCH OUT: Returns OK!
#
apt-key del NOTPRESENT

# Deletes the Fullstaq Ruby key; use the last 8 digits of the displayed key.
#
apt-key del 2ED65CC0
```

## Debconf

```sh
debconf-show packagename                      # show properties of an installed package
debconf-get-selections | grep packagename/    # more detailed, but requires `debconf-utils package`
```

Example of seeding:

```sh
echo "zfs-dkms zfs-dkms/note-incompatible-licenses note true" | debconf-set-selections
```

## Debian alternatives

Display the current info (for machine parsing).

```sh
# Sample:
#
#   Name: ruby
#   Link: /usr/bin/ruby
#   Status: auto
#   Best: /usr/bin/ruby2.6
#   Value: /usr/bin/ruby2.6
#   
#   Alternative: /usr/bin/ruby2.5
#   Priority: 9999
#   
#   Alternative: /usr/bin/ruby2.6
#   Priority: 9999
#
update-alternatives --query <program>
```

Install an alternative:

```sh
# - format: <file path> <alternative name> <alternative path> <priority>
# - 9999 is high priority, and will (likely) set the alternative as default.
# - If the specified alternative exists, it will be updated.
# - "Slaves" are alternatives that depend on the "master" (e.g. irb -> ruby)
#
update-alternatives --install /etc/mysql/my.cnf my.cnf "$(pwd)/config/my.travis.cnf" 9999 \
                    --slave   /path/to/slave    slave  /path/to/slave_file
```

Clear the alternatives:

```sh
update-alternatives --remove-all ruby
```

Example: set `/etc/alternatives/editor` to vim (requires the alternative to be installed):

```sh
update-alternatives --set editor /usr/bin/vim
```

## Environment

### Available variables

Current user run dir:

```sh
$XDG_RUNTIME_DIR
```

### Shell (initscripts)

There are two aspects of a shell:

- interactive <> non interactive
- login <> non login

Best explanation: https://askubuntu.com/a/376386/46091.

Additionally, there are per-O/S specific compiled configurations. On Debian-based systems, there is `/etc/bash.bashrc` which is run on *non login* shells.

Files involved:

- `/etc/profile`: for login shells; conditionally sources `/etc/bash.bashrc`; always sources `/etc/profile.d/*.sh`
- `$HOME/.profile`: for login shells; sources `$HOME/.bashrc` (!)
- `/etc/bash.bashrc`: for interactive non-login shells (see [here](https://askubuntu.com/a/815083)), however, it's not always loaded when `$HOME/.bashrc` is (!!)
- `$HOME/.bashrc`: not always sourced; part of the body is optionally executed.

All in all, it'a bloody mess; see the next subsection after the examples.

#### Example cases

Example cases (the tracker has been set before the interactive test in every file):

```sh
# Interactive SSH session: /etc/profile /etc/bash.bashrc /etc/profile.d/track.sh /home/user1/.bashrc
ssh myserver.ts.int
env | grep ^ENV_TRACKER

# Non-interactive SSH session: /etc/bash.bashrc /home/user1/.bashrc
ssh myserver.ts.int 'env | grep ^ENV_TRACKER'

# `sudo -u`: none, [explanation](https://unix.stackexchange.com/a/472226): $HOME is not changed!
ssh -t myserver.ts.int 'sudo -u user2 bash -c "env | grep ^ENV_TRACKER"'

# `sudo -iu` + non interactive shell (`bash -c`): /etc/profile /etc/profile.d/track.sh /home/user2/.bashrc
ssh -t myserver.ts.int 'sudo -iu user2 bash -c "env | grep ^ENV_TRACKER"'

# `sudo -iu` -> interactive shell: /etc/profile /etc/bash.bashrc /etc/profile.d/ticketsolve.sh /home/app/.bashrc
ssh -t myserver.ts.int sudo -iu user2
env | grep ^ENV_TRACKER

# `sudo -Eu` + non interactive shell (`bash -c`): /etc/bash.bashrc /home/user1/.bashrc
ssh -t myserver.ts.int 'sudo -Eu user2 bash -c "env | grep ^ENV_TRACKER"'
```

The above are **not** all the possible cases (!!); in a tested Capistrano workflow, `$HOME/.bashrc` was not sourced (wondering if it was using another shell).

If one wants to cover every case (except the bare `sudo -u`), the best is to use `$HOME/.bashrc` (modified at the top) **and** `/etc/profile.d/` scripts.

On a configuration managements system, a clean solution is likely to create a `$HOME/.bashrc.d`-style solution, for drop-in scripts.

On a server system, each login loads the new init scripts, so there's no need to reboot in order to test.

Don't forget that system services don't run any script. In the case of Systemd, use `Environment=...` for this purpose.

#### sudo -i, login shell test, and bash

The `-i` does **not** stand for `i`nteractive; it stands for `i`nitial login` (=login shell).

Joint usage of sudo and bash is confusing, because bash has its own login/interactive options.

Behavior:

```sh
sudo -iu app bash -c 'shopt -q login_shell && echo login shell'
# none

sudo -iu app bash
shopt -q login_shell && echo login shell
# none

sudo -iu app
shopt -q login_shell && echo login shell
# login shell
```

Bash options:

```
-i          : If the -i option is present, the shell is interactive.
-l, --login : Make bash act as if it had been invoked as a login shell (see INVOCATION below).
```

## Cron

Cron doesn't need to be restarted when files are changed.

Note that besides the `/etc/cron*` files/dirs, there are also files in the locations `/var/spool/cron/crontabs/$user`.

User operation:

```sh
crontab -l   # list
crontab -e   # edit
crontab -d   # delete
```

## Systemd

### Systemctl

```sh
systemctl enable $service             # execute on boot
systemctl start $service              # start immediately
systemctl daemon-reload               # invoke this after updating a unit
systemctl daemon-reexec               # required to reload Sytemd's own configuration (e.g. changes to `/etc/systemd/system.conf`)

systemctl disable $service.service    # disable a systemd service autostart
systemctl --failed                    # show units that failed to start
systemctl --all                       # list everything (with their states), including timers
```

### sournalctl

```sh
journalctl --pager-end --unit=$service.service     # show unit log; `page-end`: go to end
journalctl --vacuum-time=1d                        # clean systemd journal (/var/log/journal)
```

## Configuring a unit

See:

- https://www.freedesktop.org/software/systemd/man/systemd.service.html
- https://www.freedesktop.org/software/systemd/man/systemd.kill.html

Almost all the entries are optional. Escaping is not standard; in order to escape, use `systemd-escape`.

```sh
# If entries need escaping, use double quotes, but not slashes. In at least the `ExecStart` entry, if,
# for example, a space is escaped with a slash, both the slash and the space will be part of the string
# (!).
#
cat > /etc/systemd/system/myservice.service <<UNIT
[Unit]
Description=My Service

After=network.target

[Service]
# If ExecStart is specified (and `BusName` not specified), `simple` is implicit.
# Use `forking` when the process forks (e.g. nmon).
#
Type=simple

# Convenient.
#
StandardOutput=syslog
StandardError=syslog
SyslogIdentifier=my-service

User=app
Group=app

WorkingDirectory=/path/to/base_dir

Environment=BUNDLE_GEMFILE=/path/to/Gemfile
Environment=RAILS_ENV=my_rail_env

ExecStart=/usr/bin/bundle exec my_command
ExecReload=/bin/kill -HUP $MAINPID

# Other options: `mixed`, `process`, `none`
# Better not to specify, unless required.
#
KillMode=control-group

# Other options: `no` (default), `on-failure`, ...
#
Restart=always

[Install]
# Generally, this is the setting.
#
WantedBy=multi-user.target
UNIT

# now enable and start
```

## fstab

Columns:

- `options` (-3)
  - `X-mount.mkdir[=mode]`: create the mountpoint (as `mkdir -p`) if not exists
  - `nofail`: don't report error if the device is not present; best used with timeout (`x-systemd.device-timeout`)
  - `x-systemd.device-timeout=<time>`: time can be specified with suffixes, eg. `1ms`; `0` is infinite; defaults to 90"
- `dump` (-2): `1` if the partition is included when using dump tool
- `pass` (-1): order when checking with fsck

Test the content of fstab (not reliable for actual mounting; for example, doesn't detect if the nfs tools are installed, and reports success):

```sh
# [f]ake, [a]ll, [v]erbose
mount -fav
```

## Logging/syslog/tools

```sh
logger [-t $tag] $message           # write to system logger; --[t]ag
logrotate -d -f $config_file        # test configuration; --[d]ebug: dry run; --[f]orce: force file rotation
```

## Terminal emulator

Open a new tab, execute a command, and leave the tab open:

```sh
mate-terminal --tab --command 'sh -c "echo abc; zsh"'
```

## Desktop Environment: windows

Bringing a window to the front:

```sh
# Sample window names
#
xdotool getwindowname "$window_id"
# Google - Mozilla Firefox (Private Browsing) # private with website
# Mozilla Firefox                             # standard, with blank page

# Search only in the current desktop.
#
desktop_id=$(xdotool get_desktop)

# For a brief moment, xdotool changes the terminal name to the command, so if we search for a
# string, the terminal will be found as well! So one needs to make sure that the name regex doesn't
# match itself (any metacharacter will do).
#
# `--all` is important, otherwise, each token of the `--name` parameter is an independent match!
#
# The windows are listed is reverse access order (last accessed is at the bottom)
#
# Don't forget the `|| true`!
#
window_id=$(xdotool search --all --desktop "$desktop_id" --name 'Mozilla Firefox \(Private Browsing\)$' | tail -n 1 || true)

# Bring to front.
#
xdotool windowactivate "$window_id"
```

## Dconf/Gsettings

```sh
# Dump the whole tree.

dconf dump /
gsettings list-recursively

# Watch the whole tree.
#
# gsettings can't watch the whole tree; only one schema.
# but the second works fine 🙂

dconf watch /

gsettings monitor org.mate.caja

gsettings list-recursively > /tmp/before.gsettings
watch -n 1 -x bash -c 'diff /tmp/before.gsettings <(gsettings list-recursively)'
```

## MIME (extensions) (file associations) handling

```sh
# Find the MIME type.
# More accurate than `file --mime-type`;  on Markdown docs for example, the type found was `text/plain`.
#
mimetype "file"

# Open a file using MIME.
# On types which are not set yet, default will be asked.
#
xdg-open "file_or_uri" # supports URI (see https://wiki.archlinux.org/index.php/default_applications)
mimeopen "file"

# Find/set application default.
#
xdg-mime query default "x-scheme-handler/http"
xdg-mime default "sublime_text.desktop" "text/markdown"

# Reset custom associations.
#
rm "~/.local/share/applications/wine*.desktop"
rm "~/.local/share/applications/mimeinfo.cache"
```

### Adding and associating a new application

```sh
# Partial; see `/usr/share/applications/firefox.desktop` for a full example.
#
cat > ~/.local/share/applications/ << 'CONF'
[Desktop Entry]
Version=1.0
Name=Sav's Browser
...
CONF

# Optional: add to MATE menu.
#
perl -i -lpe '$. == 1 && print "location:/home/saverio/.local/share/applications/browser.desktop"' ~/.config/mate-menu/applications.list

# Register!
#
xdg-mime default "browser.desktop" "x-scheme-handler/http"
xdg-mime default "browser.desktop" "x-scheme-handler/https"
```

### Split MAFF association

Associate an extension based on the filename rather than the MIME type:

```sh
cat > /usr/share/mime/packages/maff.xml << 'XML'
<?xml version="1.0" encoding="UTF-8"?>
<mime-info xmlns="http://www.freedesktop.org/standards/shared-mime-info">
<mime-type type="application/x-maff">
<comment>Maff File</comment>
<glob pattern="*.maff"/>
</mime-type>
</mime-info>
XML

update-mime-database /usr/share/mime
```

## Hardware

Get hardware information:

```sh
# Without options, all the system information is reported.
#
dmidecode -s \
  system-product-name   # (laptop) model
  bios-version
  chassis-serial-number # (dell) service tag
```
