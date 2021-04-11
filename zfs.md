# ZFS

- [ZFS](#zfs)
  - [Basic usage](#basic-usage)
  - [Snapshotting](#snapshotting)
  - [Lightweight per-file rollback workflow](#lightweight-per-file-rollback-workflow)
  - [Cool diffing functions](#cool-diffing-functions)

## Basic usage

For more advanced concepts, see http://j.mp/18tSgvx.

List the pools.

```sh
zpool list
```

More details about the pools.
[-v] displays files with error, if there are any.

```sh
zpool status [-v]
```

Create a pool from a file/device (if file, it must be present).

- `-d` + `-o <features>`: enable only the specified features.

```sh
zpool create [-d {-o feature@<name>=enabled, ...}] <pool> <device>
```

Hackish way of getting the supported features.

```sh
man zpool-features | grep -E '^       \w+$'
```

Don't allow pool mounting.
Useful on creating a pool that is purely a container

```sh
zpool create -O canmount=off
```

Create with a permanent mountpoint (-O mountpoint), but a different temporary one (-R).
WATCH OUT!: With `-R`, filesystem's mountpoint will be relative to it.
The mountpoints are created on pool creation, but not destroyed on pool destruction.

```sh
zpool create -O mountpoint=/ -R /media/disk_a
```

Enable all the supported features in a pool.

```sh
zpool upgrade <pool>
```

Import a pool:

```sh
# Import in the given path.
zpool import -d $path $pool

# Import pool using $alt_root as temporary root.
zpool import -R $alt_root $pool

# Import an encrypted pool. IMPORTANT!:
# - if `-l` is not specified, and the pool is encrypted, it will be imported, with successful exit status (!), and with empty content (!!)
# - if `-l` is specified, and the pool is not encrypted, it will be imported, without prompt
zpool import -l $pool
```

Rename a pool (permanently).

```sh
zpool import <pool> <new_name>
```

Create a mirrored pool.

```sh
zpool create <pool> mirror <device1> <device2>
```

Replace a device.

If the new device is in the same place as the old one, don't specify the new one.

```sh
zpool replace <pool> <old_device> [<new_device>]
```

Add a new device to a pool (doesn't mirror!).

```sh
zpool add <pool> <device>
```

Attach a new device to a mirror (convert the pool to a mirror, if it isn't already so)

Resilvering will start immediately.
Note that $device_in_mirror is the name of the device as defined in the pool status.

```sh
zpool attach $pool $device_in_mirror $new_device
```

Add a new top level mirror.

```sh
zpool add <pool> mirror <device1> <device2>
```

Expand a pool to the entire partition. If performed on a mirror, the operation must be performed on each device.

```sh
parted -s $disk_device resizepart $part_number_1_based 100%
zpool online -e $pool $part_device
```

Scrub a pool. Results are displayed both in :list and :status.

```sh
zpool scrub <pool>
```

Destroy a pool.

```sh
zpool destroy <pool>
```

Unmount datasets (don't use export!, or it won't be automatically autoimported).

```sh
zpool export <device_or_mountpoint>
```

Detach a device from a pool (/mirror).

```sh
zpool detach $pool $device
```

Store the mountpoint and mount a dataset/pool.
The dataset is mounted automatically when importing the pool!
The mountpoint presence is optional, but it won't be deleted when unmounting.

```sh
zfs set mountpoint=<mountpoint> <dataset>
zfs set readonly=on <dataset>
```

Get mountopoint/mount status -> mount.

```sh
zfs get mountpoint <filesystem>
zfs get mounted
zfs mount <filesystem>
```

[re]mount in readonly mode.
if a FS is mounted, use the 'remount' option to change a property.

```sh
zfs mount -o remount,ro <dataset>
```

ZFS mirroring with different sector sizes (9=512, 12=4096).
Note that it has a significant impact on performance.

```sh
fdisk -l                                                         # show the SS for the disks
zdb | grep ashift                                                # show the pool current ashift
zpool attach -o ashift=9 <pool> <device_in_mirror> <new_device>  # force the optimal ashift for the attaching device
```

## Snapshotting

(`$volume` can also be a pool)

Create a subvolume, then a snapshot.

```sh
zfs create -o mountpoint=$mountpoint $pool/$volume
zfs snapshot $volume@$snaphot
```

List snapshots.

```sh
zfs list -t snapshot
```

Rollback after modifications!

```sh
zfs rollback $volume@$snaphot
```

Destroy snapshot (changes are kept), then subvolume.

```sh
# `-r`: destroy subsequent snapshots, if present
#
zfs destroy -r $volume@$snaphot
zfs destroy $volume
```

Check SIMD performance (see https://github.com/zfsonlinux/zfs/issues/9215).

```sh
cat /proc/spl/kstat/zfs/fletcher_4_bench
cat /proc/spl/kstat/zfs/vdev_raidz_bench
```

## Lightweight per-file rollback workflow

```sh
zfs create -o mountpoint=/busy rpool/busy

# Move the file, symlink it, and create the snapshot
mv components/busybear.raw /busy/
ln -s /busy/busybear.raw components/
zfs snapshot rpool/busy@unchanged

# do all the work, and when required, rollback:
zfs rollback rpool/busy@unchanged

# Restore the file and destroy the snapshot/volume
zfs rollback rpool/busy@unchanged
mv -f /busy/busybear.raw components/
zfs destroy rpool/busy@unchanged
zfs destroy rpool/busy
```

## Cool diffing functions

The function don't work with ecryptfs, due to the files at a higher level than the storage layer.

`zdifff` and `zversions` could be improved (simplified) by searching `.zfs` from the bottom up.

```sh
# Recreate a snapshot.
#
# $1: pool, $2: snapshot
#
function zreset {
  zfs destroy "$1@$2"
  zfs snapshot "$1@$2"
}

# Diffs against a snapshot.
#
# $1: pool, $2: snapshot
#
function zdiffs {
  zfs diff "$1@$2"
}

# Diff a file (with `meld`).
#
# $1: filesystem mount (eg. /home), $2: snapshot, $3: file path (relative to mount)
#
function zdifff {
  meld "$1/.zfs/snapshot/$2/$3" "$1/$3"
}

# Creates a `.before` (=at snapshot time) and `.after` versions of a given file.
# Won't create the .before if the file doesn't exist before.
#
# $1: filesystem mount (eg. /home), $2: snapshot, $3: file path (relative to mount, $4: output directory
#
function zversions {
  mkdir -p "$4"

  if [[ -f "$1/.zfs/snapshot/$2/$3" ]]; then
    cp "$1/.zfs/snapshot/$2/$3" "$4/$(basename "$3").before"
  fi

  cp "$1/$3" "$4/$(basename "$3").after"
}
```
