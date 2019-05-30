# QEMU Disk utils/LibGuestFS handy commands

1. Create a diff disk:

```sh
qemu-img create -f qcow2 -b "$DISK_IMAGE" "$DISK_IMAGE.diff"
```

2. Create a compressed copy of a disk (can also be used to merge a diff image):

```sh
# [-p]rogress, [-O]utput format, [-c]ompressed
qemu-img convert -p -O qcow2 -c "$DISK_IMAGE" "$DISK_IMAGE.merged"
```

3. Create a compressed copy without unallocated space (use `--tmp`, otherwise `/tmp` is used!!):

```sh
virt-sparsify --compress --tmp $(dirname "$DISK_IMAGE") "$DISK_IMAGE" "$DISK_IMAGE.sparse.compressed"
```

4. Import a directory in a disk:

```sh
virt-copy-in -a "$DISK_IMAGE" /tmp/pizza /
```

5. Mount a disk using `qemu-nbd`:

```sh
PARTITION_NUMBER=4                           # 1-based
modprobe nbd max_part=8
qemu-nbd --connect=/dev/nbd0 "$DISK_IMAGE"   # wait a second after this
partprobe /dev/nbd0
mount "/dev/nbd0p"$PARTITION_NUMBER" /mnt
```

6. and unmount:

```sh
umount /mnt
qemu-nbd --disconnect /dev/nbd0
rmmod nbd
```

7. Mounting a disk can also be used to shrink an image:

```sh
# mount the disk (5.)
dd -status=progress if=/dev/zero of=/mnt/ZEROFILE
rm /mnt/ZEROFILE
# now umount the disk (6.)
# now compress the image (2.)
```

[Previous: Profiling KVM](7_PROFILING_KVM.md) | [Next: References](9_REFERENCES.md)
