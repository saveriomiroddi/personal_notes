# RedHat

- [RedHat](#redhat)
  - [Packages](#packages)
  - [System conveniences](#system-conveniences)

## Packages

```sh
# Equivalent to Debian "build-essential" package.
#
dnf groupinstall -y "Development Tools" "Development Libraries"
```

## System conveniences


```sh
# Passwordless sudo!
#
sed -i '/%wheel.*NOPASSWD: ALL/ s/^# //' "$local_mount_dir/etc/sudoers"

# Disable a long-running service
#
systemctl mask man-db-cache-update
```
