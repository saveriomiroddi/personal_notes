## Table of contents

- [Table of contents](#table-of-contents)
- [Virtual disks](#virtual-disks)
  - [VirtualBox-specific](#virtualbox-specific)

## Virtual disks

Compact a VDI (manual clearing):

```sh
# Guest space zeroing, Linux
dd if=/dev/zero of=/var/tmp/zerofile bs=4096k; rm /var/tmp/zerofile

# Zeroing, Windows (see https://docs.microsoft.com/en-us/sysinternals/downloads/sdelete)
# It's possible to squeeze more space by deleting the Windows memory-related files (C:\*.sys),
# but it makes sense only for storing the compressed file, as the files are recreated on the next
# session.
sdelete64 -z <drive>

# Host
vboxmanage modifymedium --compact /path/to/disk.vdi
```

Compact a VDI (automated, via `libguestfs-tools` package):

```sh
# May require sudo.
#
virt-sparsify --tmp /path/to /path/to/disk.vdi{,.sparse}
```

Compress (/compact) a qcow image:

```sh
# Options: [p]rogress, [O]utput format, [c]ompress
#
qemu-img convert -p -c -O qcow2 source.qcow2 compressed.qcow2

# This also compacts it!
#
virt-sparsify --convert qcow2 --compress source.raw dest.qcow2
```

Convert a VMDK disk to VDI:

```sh
vboxmanage clonehd original.vmdk new.vdi --format VDI
```

Create a qcow image (with NTFS filesystem) from a directory:

```sh
virt-make-fs --verbose --format=qcow2 --type=ntfs input_dir output_image.qcow2
```

### VirtualBox-specific

Change a disk UUID:

```sh
vboxmanage internalcommands sethduuid /path/to/disk.vdi
```

Clone a disk:

```sh
vboxmanage clonehd source.vdi dest.vdi
```

Create VMDK disk pointing to a physical disk!:

```sh
vboxmanage internalcommands createrawvmdk -filename disk/vdi -rawdisk /dev/disk
```
