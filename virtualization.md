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
# Create a qcow image (with NTFS filesystem) from a directory:
#
virt-make-fs --verbose --format=qcow2 --type=ntfs $input_dir $dest.qcow2

# Create a raw file, with a partition, with standard linux tools
# 8300: Partition type: native Linux FS
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

# Create a diff image of a "backing" one.
# If the diff exists, it will be overwritten.
# WATCH OUT! The backing image path is relative to the diff one.
#
qemu-img create -f qcow2 -b $backing.qcow2 $diff.qcow2
```

Copy:

```sh
virt-copy-in -a $image.qcow2 $source_dir $image_dest_dir
```

Convert:

```sh
vboxmanage clonehd --format VDI $source.vmdk $dest.vdi

# Options: [p]rogress, [O]utput format
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
sudo virt-filesystems --long -h --all -a $image  # partition infos
sudo virt-df -h -a $image                        # partitions usage
```

Resize a partition:

```sh
# Resize to an exact destination size
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
