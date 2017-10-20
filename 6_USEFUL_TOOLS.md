# QEMU Disk utils/LibGuestFS handy commands

Create a diff disk:

    qemu-img create -f qcow2 -b $VGAPT_DISK_IMAGE $VGAPT_DISK_IMAGE.diff

Create a merged copy a disk (of any type):

    qemu-img convert -p $VGAPT_DISK_IMAGE.diff -O qcow2 $VGAPT_DISK_IMAGE.merged

Create a compressed copy without unallocated space (use `--tmp`, otherwise `/tmp` is used!!):

    virt-sparsify --compress --tmp $(dirname $VGAPT_DISK_IMAGE) $VGAPT_DISK_IMAGE $VGAPT_DISK_IMAGE.sparse.compressed

Import a directory in a disk:

    virt-copy-in -a $VGAPT_DISK_IMAGE /tmp/pizza /

[Previous: Profiling KVM](5_PROFILING_KVM.md) | [Next: References](7_REFERENCES.md)
