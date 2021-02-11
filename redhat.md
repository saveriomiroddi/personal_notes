# RedHat

- [RedHat](#redhat)
  - [Packages](#packages)
  - [System conveniences](#system-conveniences)

## Packages

```sh
# Equivalent to Debian "build-essential" package.
#
dnf groupinstall -y "Development Tools" "Development Libraries"

# Find out which packages a file belongs to. rpm is much faster.
#
rpm -qf /usr/lib64/liblzma.so
dnf provides /usr/lib64/liblzma.so
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
