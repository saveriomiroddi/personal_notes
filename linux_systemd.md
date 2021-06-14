# Linux Systemd

- [Linux Systemd](#linux-systemd)
  - [Generic commands](#generic-commands)
  - [Systemctl](#systemctl)
  - [journalctl](#journalctl)
  - [Configuring a unit](#configuring-a-unit)
    - [Overrides](#overrides)
    - [Hooks](#hooks)
  - [Timers (scheduled events)](#timers-scheduled-events)
    - [Once-off (transient)](#once-off-transient)
  - [Event triggers](#event-triggers)

## Generic commands

List all the services, and their running status: `service --status-all`. Great for disabling unnecessary services!!

## Systemctl

```sh
systemctl enable|disable $service     # enable/disable start on boot
systemctl start|stop $service         # start/stop immediately
systemctl reload $service             # reload; not always functional
systemctl reload-or-restart $service  # if reload is not defined (or has no effect), restart; WATCH OUT! don't define `ExecReload`
systemctl status $service
systemctl cat $service                # print unit file
systemctl edit --full $service        # edit unit file (--full: edit the service, instead of creating an override)
systemctl mask $service               # "mask": disable a service, by symlinking it to /dev/null; WATCH OUT! doesn't fail if the service doesn't exis

systemctl daemon-reload               # invoke this after updating a unit
systemctl daemon-reexec               # required to reload Sytemd's own configuration (e.g. changes to `/etc/systemd/system.conf`)

systemctl is-(active|enabled|failed) $service  # query status
systemctl list-units                  # active loaded units
systemctl list-unit-files [$pattern]  # all units, with their states; glob pattern
systemctl [--user] list-timers        # list timers
systemctl --failed                    # show units that failed to start
```

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

## Configuring a unit

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
cat > /etc/systemd/system/myservice.service <<UNIT
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
# Generally, this is the setting.
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

Timers are represented by `.timer` files accompanying the main service (a service is required).

See [overrides](#overrides) for overriding notes.

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
