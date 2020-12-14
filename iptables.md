## Table of contents

- [Table of contents](#table-of-contents)
- [Commands](#commands)
  - [Examples](#examples)
- [System configuration](#system-configuration)
  - [Test](#test)
  - [Ubuntu startup](#ubuntu-startup)
  - [Sample configuration](#sample-configuration)

## Commands

Iptables commands:

- `-A chain`             (A)ppend a rule; not convenient, as usually there's a deny all at the end.
- `-I chain [position]`  (I)nsert; at position, 1-based
- `-L`                   (L)ist rules
- `-D chain [position]`  (D)elete; at position, 1-based

- `-F -X`                (F)lush(delete) all the rules; (X) delete all the non builtin rules

- `-p protocol`
- `-m type`              (m)atch by type

- `--dport destPort`
- `-j (ACCEPT|DROP)`

Save/restore:

```sh
iptables-save > fileName           # export rules
iptables-restore < fileName        # import rules
```

### Examples

```sh
# drop input from a specific IP
iptables -I INPUT -j DROP -s 255.255.255.255

# accept TCP traffic on port 25
iptables -I INPUT -p tcp --dport 44033 -j ACCEPT

# local redirect from port 3306 to 4040; change '-I' to '-D' to rollback the change
iptables -t nat -I PREROUTING -s ! 127.0.0.1 -p tcp --dport 3306 -j REDIRECT --to-ports 4040

# iptables: log initiated output connections
iptables -I OUTPUT 1 -m state --state NEW -j LOG --log-prefix "iptables: output: "
```

## System configuration

### Test

To remove the configuration every two minutes, add to crontab:

```cron
*/2 * * * * root /usr/sbin/iptables -F
```

### Ubuntu startup

Reference: https://help.ubuntu.com/community/IptablesHowTo

File `/etc/network/interfaces`:

```
iface eth0 inet dhcp
pre-up iptables-restore « /etc/iptables.rules
post-down iptables-save -c » /etc/iptables.rules
```

### Sample configuration

```
*filter
:INPUT ACCEPT [0:0]
:FORWARD ACCEPT [0:0]
:OUTPUT ACCEPT [0:0]

# accept input loopback
-A INPUT -i lo -j ACCEPT
# reject dest loopback from input not loopback
-A INPUT -d 127.0.0.0/255.0.0.0 -i ! lo -j REJECT --reject-with icmp-port-unreachable
# log "invalid" packets
-A INPUT -m state --state INVALID -j LOG --log-prefix "DROP INVALID" --log-tcp-options --log-ip-options
# drop "invalid" packets
-A INPUT -m state --state INVALID -j DROP
# accept "related"/"established" packets; needed for outgoing ping
-A INPUT -m state --state RELATED,ESTABLISHED -j ACCEPT

# accept connections from jool for MySQL replication
-A INPUT -s 82.195.158.85 -p tcp -m tcp --dport 3306 -m state --state NEW,ESTABLISHED -j ACCEPT
# ... mongrel
-A INPUT -s 82.195.158.83 -p tcp -m tcp --dport 8000:8019 -m state --state NEW,ESTABLISHED -j ACCEPT
# ... mongrel
-A INPUT -s 82.195.158.84 -p tcp -m tcp --dport 8000:8019 -m state --state NEW,ESTABLISHED -j ACCEPT
# ... memcached
-A INPUT -s 82.195.158.84 -p tcp -m tcp --dport 11211 -m state --state NEW,ESTABLISHED -j ACCEPT

# accept SSH from everywhere on 4444
-A INPUT -p tcp -m state --state NEW -m tcp --dport 4444 -j ACCEPT
# ??? log the following with a max of 5 per min ???
-A INPUT -m limit --limit 5/min -j LOG --log-prefix "DENIED " --log-level 7
# accept echo request
-A INPUT -p icmp -m icmp --icmp-type 8 -j ACCEPT
# reject all other input
-A INPUT -j REJECT --reject-with icmp-port-unreachable
# reject all forwards
-A FORWARD -j REJECT --reject-with icmp-port-unreachable
# accept outputs
-A OUTPUT -j ACCEPT

COMMIT
```
