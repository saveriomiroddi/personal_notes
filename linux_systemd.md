# Linux Systemd

- [Linux Systemd](#linux-systemd)
  - [Generic commands and notes](#generic-commands-and-notes)
  - [Systemctl](#systemctl)
  - [journalctl](#journalctl)
  - [Configuring a service unit](#configuring-a-service-unit)
    - [Overrides](#overrides)
    - [Hooks](#hooks)
  - [Timers (scheduled events)](#timers-scheduled-events)
    - [Systemd time format](#systemd-time-format)
    - [Once-off (transient)](#once-off-transient)
  - [Event triggers](#event-triggers)

## Generic commands and notes

List all the services, and their running status: `service --status-all`. Great for disabling unnecessary services!!

Units are stored under `/etc/systemd/system`, however, it's best practice not to edit them directly, instead, use the `edit` command.

## Systemctl

```sh
systemctl enable|disable $service     # enable/disable start on boot
systemctl start|stop $service         # start/stop immediately
systemctl reload $service             # reload; not always functional
systemctl reload-or-restart $service  # if reload is not defined (or has no effect), restart; WATCH OUT! don't define `ExecReload`
systemctl status $service
systemctl cat $service                # print unit file
systemctl edit --full $service        # create/edit unit file (--full: edit the service, instead of creating an override)
                                      # in order to programmatically edit, workaround by setting `SYSTEMD_EDITOR=tee [-a]` and piping the content.
systemctl mask $service               # "mask": disable a service, by symlinking it to /dev/null; WATCH OUT! doesn't fail if the service doesn't exis

systemctl daemon-reload               # invoke this after updating a unit
systemctl daemon-reexec               # required to reload Sytemd's own configuration (e.g. changes to `/etc/systemd/system.conf`)

systemctl is-(active|enabled|failed) $service  # query status
systemctl list-units                  # active loaded units
systemctl list-unit-files [$pattern]  # all units, with their states; glob pattern
systemctl [--user] list-timers        # list timers
systemctl --failed                    # show units that failed to start
```

If a unit is stored with ill-formed content, it won't be found by commands, which print instead a confusing `No files found` message.

The `--user` is for units belonging to the user (eg. transient timers).

A static service can't be manually enabled or disabled; use mask for that.

WATCH OUT! Even if a service is disabled, if other enabled services depend on it; in this case, find and disable them:

```sh
systemctl list-dependencies --reverse snapd.socket
# ... now disable the dependencies ...
```

## journalctl

```sh
journalctl -b -xp 3 [-k]                           # view log for current Boot; with e[x]tra information, only errors (`-p 3` = level); only [k]ernel messages
journalctl --pager-end --unit=$service.service     # show unit log; `page-end`: go to end
journalctl --vacuum-time=1d                        # clean systemd journal (/var/log/journal)
```

Limit the max use:

```sh
mkdir -p /etc/systemd/journald.conf.d
cat > /etc/systemd/journald.conf.d/00-journal-size.conf <<'CFG'
[Journal]
SystemMaxUse=128M
CFG

# Restart the journal service (won't purge the existing journal)
systemctl kill --kill-who=main --signal=SIGUSR2 systemd-journald

# Purge the existing journal
journalctl --vacuum-size=128M
```

## Configuring a service unit

See:

- https://www.freedesktop.org/software/systemd/man/systemd.service.html
- https://www.freedesktop.org/software/systemd/man/systemd.kill.html

General location of systemd units (not the only one): `/lib/systemd/system/$name.service`.

Almost all the entries are optional. Escaping is not standard; in order to escape, use `systemd-escape`.

```sh
# If entries need escaping, use double quotes, but not slashes. In at least the `ExecStart` entry, if,
# for example, a space is escaped with a slash, both the slash and the space will be part of the string
# (!).
#
SYSTEMD_EDITOR=tee systemctl edit --full myservice.service << UNIT
[Unit]
Description=My Service

# `Requires` states only dependency; `After` mandates ordering.
#
Requires=network.target
After=network.target

# Example of waiting for a network device to be online.
#
# After=sys-subsystem-net-devices-wlan0.device
# BindsTo=sys-subsystem-net-devices-wlan0.device

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

# In other to extend $PATH, use something like:
#
#     ExecStart=/bin/bash -c 'PATH=/new/path:$PATH exec /bin/mycmd'
#
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
# General-purpose setting.
#
WantedBy=multi-user.target
UNIT

# now enable and start
```

### Overrides

Overrides are created via `systemctl edit` (without `--full`); they create the file+dir `/etc/systemd/system/myservice.$type.d/override.conf`.

At least certain entries are additive, e.g. `OnCalendar`; in order to clear the existing entries, set an empty one (eg. `OnCalendar=`) and the new one.

### Hooks

If the `Type=oneshot`, `ExecPostStart` can be used to specify a command to execute on successful exit; the entry is additive.

## Timers (scheduled events)

Reference: https://www.freedesktop.org/software/systemd/man/systemd.timer.html.

Timers are represented by `.timer` files accompanying the main service (a service is required).

See [overrides](#overrides) for overriding notes.

Example:

```sh
SYSTEMD_EDITOR=tee systemctl edit --full myservice.timer << UNIT
[Unit]
Description=myservice timer

[Timer]
OnCalendar=*-*-* 0/5:*:*

# Other options/configurations:
#
# AccuracySec=1s          # defaults to 1m
# OnCalendar=*-*-* 6:00
# RandomizedDelaySec=30m  # defaults o 0
# Persistent=true         # stores the last run on disk, so jobs are not missed; requires OnCalendar

[Install]
WantedBy=timers.target
UNIT

systemctl enable myservice.timer
systemctl start myservice.timer
```

### Systemd time format

Reference: https://www.freedesktop.org/software/systemd/man/systemd.time.html.

Examples:

```
  Sat,Thu,Mon..Wed,Sat..Sun → Mon..Thu,Sat,Sun *-*-* 00:00:00
      Mon,Sun 12-*-* 2,1:23 → Mon,Sun 2012-*-* 01,02:23:00
                    Wed *-1 → Wed *-*-01 00:00:00
           Wed..Wed,Wed *-1 → Wed *-*-01 00:00:00
                 Wed, 17:48 → Wed *-*-* 17:48:00
Wed..Sat,Tue 12-10-15 1:2:3 → Tue..Sat 2012-10-15 01:02:03
                *-*-7 0:0:0 → *-*-07 00:00:00
                      10-15 → *-10-15 00:00:00
        monday *-12-* 17:00 → Mon *-12-* 17:00:00
  Mon,Fri *-*-3,1,2 *:30:45 → Mon,Fri *-*-01,02,03 *:30:45
       12,14,13,12:20,10,30 → *-*-* 12,13,14:10,20,30:00
            12..14:10,20,30 → *-*-* 12..14:10,20,30:00
  mon,fri *-1/2-1,3 *:30:45 → Mon,Fri *-01/2-01,03 *:30:45
             03-05 08:05:40 → *-03-05 08:05:40
                   08:05:40 → *-*-* 08:05:40
                      05:40 → *-*-* 05:40:00
     Sat,Sun 12-05 08:05:40 → Sat,Sun *-12-05 08:05:40
           Sat,Sun 08:05:40 → Sat,Sun *-*-* 08:05:40
           2003-03-05 05:40 → 2003-03-05 05:40:00
 05:40:23.4200004/3.1700005 → *-*-* 05:40:23.420000/3.170001
             2003-02..04-05 → 2003-02..04-05 00:00:00
       2003-03-05 05:40 UTC → 2003-03-05 05:40:00 UTC
                 2003-03-05 → 2003-03-05 00:00:00
                      03-05 → *-03-05 00:00:00
                     hourly → *-*-* *:00:00
                      daily → *-*-* 00:00:00
                  daily UTC → *-*-* 00:00:00 UTC
                    monthly → *-*-01 00:00:00
                     weekly → Mon *-*-* 00:00:00
    weekly Pacific/Auckland → Mon *-*-* 00:00:00 Pacific/Auckland
                     yearly → *-01-01 00:00:00
                   annually → *-01-01 00:00:00
                      *:2/3 → *-*-* *:02/3:00
```

### Once-off (transient)

Once-off events can be scheduled; mind that they're not second-accurate. They belong to the user (therefore need `--user` to be listed).

```sh
# Execute after certain interval, with output.
# `--unit` is the optional name; without, a long random name is set.
#
$ systemd-run --user --on-active=10min --unit=test-example /bin/systemctl suspend
Running timer as unit: test-example.timer
Will run service as unit: test-example.service

# Execute at given time
#
$ systemd-run --user --on-calendar=09:12 /bin/systemctl suspend

# Cancel
#
$ systemctl --user stop test-example
```

## Event triggers

Run a script on suspend/hibernate/thaw/resume (the change applies immediately):

```sh
cat > /lib/systemd/system-sleep/50_myscript << 'SH'
#!/bin/bash
set -o errexit

case "$1" in
  pre)
    # ACTION BEFORE SUSPEND/HIBERNATE
    ;;
  post)
    # ACTION AFTER THAW/RESUME
    ;;
esac
SH

chmod +x /lib/systemd/system-sleep/50_myscript
```

Run a script on screen lock/unlock:

```sh
dbus-monitor --session "type='signal',interface='org.gnome.ScreenSaver'" |
  while true; do
    read X;
    if echo $X | grep "boolean true" &> /dev/null;
      then echo locked >> /tmp/pizza.txt;
    elif echo $X | grep "boolean false" &> /dev/null;
      then echo unlocked >> /tmp/pizza.txt;
    fi
  done
```
