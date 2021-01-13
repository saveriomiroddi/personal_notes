# Virtualization

- [Virtualization](#virtualization)
  - [Virtual disks](#virtual-disks)
    - [Creation/copy/conversion:](#creationcopyconversion)
    - [Zeroing/compacting/compressing an image](#zeroingcompactingcompressing-an-image)
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

Zero and compact an image:

```sh
sudo virt-sparsify --tmp $tmp_path $image.vdi{,.sparse}
```

Compress an image:

```sh
# Options: [p]rogress, [O]utput format, [c]ompress
#
qemu-img convert -p -c -O qcow2 $source.qcow2 $dest.qcow2

# Zero, compact, and compress!!:
# If `--convert` is not specified, the same format as the source is used.
#
sudo virt-sparsify --convert qcow2 --compress $source.raw $dest.qcow2
```

### Mount an image

Mount:

```sh
PARTITION_NUMBER=4                          # 1-based
modprobe nbd max_part=8
qemu-nbd --connect=/dev/nbd0 $image.qcow2   # wait a second after this
partprobe /dev/nbd0
mount "/dev/nbd0p"$PARTITION_NUMBER" /mnt
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