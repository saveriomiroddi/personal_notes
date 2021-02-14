# Virtualization

- [Virtualization](#virtualization)
  - [Virtual disks](#virtual-disks)
    - [Creation/copy/conversion:](#creationcopyconversion)
    - [Zeroing/compacting/compressing an image](#zeroingcompactingcompressing-an-image)
    - [Working with an image content](#working-with-an-image-content)
      - [Filesystem operations](#filesystem-operations)
      - [Mount an image](#mount-an-image)
    - [VirtualBox-specific](#virtualbox-specific)
  - [QEMU](#qemu)

## Virtual disks

### Creation/copy/conversion:

Create:

```sh
# The $format defaults to raw.
# The $image is overwritten, if it exists!
#
# Backing:
# - if the backing image path is relative, it's relative to the image.
# - it's possible to create a diff image of a diff image
# - WATCH OUT!: It's possible to specify a size, but in at least one instance, it caused subtle issues.
#
qemu-img create [-f $format] [-b $backing_iamge] $image [20G]

# Create a qcow image (with NTFS filesystem) from a directory:
#
virt-make-fs --verbose --format=qcow2 --type=ntfs $input_dir $dest.qcow2

# Create a raw file, with a partition, with standard linux tools
# 8300: Partition type: native Linux FS
# Guestmount is easier (see [Mount an image](#mount-an-image)).
#
dd if=/dev/zero bs=1M count=10240 of=disk.img
sgdisk -n1:0:0 -t1:8300 disk.img
loop_device=$(sudo losetup --show --find --partscan disk.img)
sudo mkfs.ext4 ${loop_device}p1
sudo mount ${loop_device}p1 /mnt
# ...copy stuff etc...
sudo umount /mnt
sudo losetup -d $loop_device
# now, use losetup (see below)
```

Copy:

```sh
virt-copy-in -a $image.qcow2 $source_dir $image_dest_dir
```

Convert:

```sh
vboxmanage clonehd --format VDI $source.vmdk $dest.vdi

# Options: [p]rogress, [O]utput format (not autodetected)
#
qemu-img convert -p -O qcow2 $source.qcow2 $dest.qcow2
```

### Zeroing/compacting/compressing an image

Zero an image from a guest; a strategy for zeroing via (Linux) host is to mount the image.

```sh
# From a Linux guest
#
dd if=/dev/zero of=/var/tmp/zerofile bs=4096k; rm /var/tmp/zerofile

# From a Windows guest (see https://docs.microsoft.com/en-us/sysinternals/downloads/sdelete).
#
# It's possible to squeeze more space by deleting the Windows memory-related files (C:\*.sys),
# but it makes sense only for storing the compressed file, as the files are recreated on the next
# session.
#
sdelete64 -z drive:
```

Compact a VDI image (must be pre-zeroed):

```sh
vboxmanage modifymedium --compact $image.vdi
```

Zero/compact/compress an image with dedicated tooling:

```sh
# Zero
#
sudo virt-sparsify --tmp $tmp_path $image.vdi{,.sparse}

# Compress
# Options: [p]rogress, [O]utput format, [c]ompress
#
qemu-img convert -p -c -O qcow2 $source.qcow2 $dest.qcow2

# Zero, compact, and compress!!:
# If `--convert` is not specified, the same format as the source is used.
#
sudo virt-sparsify --convert qcow2 --compress $source.raw $dest.qcow2
```

### Working with an image content

#### Filesystem operations

Print partition informations:

```sh
# fs/partition infos
#
# `--all`               : both FS and partitions
# `-a|--add $image`
# `-h|--human-readable`
#
sudo virt-filesystems --long --human-readable --all --add $image

# partitions space usage
#
sudo virt-df -h -a $image
```

Resize a disk/partition:

```sh
qemu-img resize $source.qcow2 +20G

# Resize a disk/partition to an exact destination size.
#
# There is a "resize" action (e.g. add 40G), but it's dumb, as it requires the output disk to be at least
# the required size, which can't be exactly calculated (even for a qcow destination).
#
# - `-v`: verbose
# - `-x`: trace underlying calls, for better error messages
#
truncate -s 40G $dest.raw
sudo virt-resize -v -x --expand /dev/sda4 $source $dest.raw
```

#### Mount an image

Easy way, via libguestfs-tools:

```sh
# $block_device is /dev/sdaN; typically a partition, but can be the entire disk (e.g. busybear image).
#
sudo guestmount -a $image -m $block_device $mountpoint

# WATCH OUT!
#
# The libguestfs stack is functionally poor.
# Unmounting (also via umount), causes an odd `fuse: mountpoint is not empty` error; the guestunmount
# help seems to acknowledge this (ie. retries option), so we don't display errors.
# Additionally, on unmount, the sync is tentative, so need to manually check that the file is closed.
#
if sudo mountpoint -q "$c_local_mount_dir"; then
  sudo guestunmount -q "$c_local_mount_dir"

  # Just ignore the gvfs warning.
  while [[ -n $(sudo lsof -e "$XDG_RUNTIME_DIR/gvfs" "$1" 2> /dev/null) ]]; do
    sleep 0.5
  done
fi
```

It's possible to use qemu-nbd, but the events management (for automation) is insane.

Mount:

```sh
modprobe nbd max_part=8

if [[ $image == *.vhd ]]; then
  format_option=(-f vpc)
fi

qemu-nbd --connect=/dev/nbd0 "${format_option[@]}" "$1"

# Complete clusterduck. Even waiting via:
#
#     while [[ ! -b /dev/nbd0p4 ]]; do sleep 0.1; done
#
# will cause:
#
#     ls -l /dev/nbd0p4
#     ls: cannot access '/dev/nbd0p4': No such file or directory
#
# if there is no sleep.
#
# Therefore, we use `ls` instead.
#
# Fdisk doesn't need to wait (in case one wants to present a list via `fdisk /dev/nbd0 -l`).

partprobe /dev/nbd0

partition_device=/dev/nbd0p$partition_number

while ! ls -l "$partition_device" > /dev/null 2>&1; do
  sleep 0.1
done

mount "$partition_device" /mnt
```

Unmount:

```sh
umount /mnt
qemu-nbd --disconnect /dev/nbd0
rmmod nbd
```

### VirtualBox-specific

Change an image UUID:

```sh
vboxmanage internalcommands sethduuid $image.vdi
```

Clone a disk:

```sh
vboxmanage clonehd $source.vdi $dest.vdi
```

Create VMDK disk pointing to a physical disk!:

```sh
vboxmanage internalcommands createrawvmdk -filename $image.vdi -rawdisk /dev/$device
```

## QEMU

Run a machine in the background, and shut it down:

```sh
QEMU_MONITOR_FILE=$(mktemp)

qemu-system-riscv64 \
   -daemonize -monitor unix:"$QEMU_MONITOR_FILE",server,nowait \
   -display none \
   # ...

echo system_powerdown | socat - UNIX-CONNECT:"$QEMU_MONITOR_FILE"
```

Other options:

- `-nographic`: runs in the foreground, in the terminal (without window)
