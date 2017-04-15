# Introduction #

This repository includes my notes for setting up a VGA Passthrough on a Linux machine (it's targeted to Ubuntu; more advanced users can adapt it to other distributions).

VGA passthrough is a setup which allows virtualized environments (QEMU, in this case) to perform 3D acceleration at near native speed.

The rationale for this document is that the information on the subject is spread, confusing and outdated, so I've decided to create a single, consistent document - a reference.

Note that this is not meant to be a generic source of help (although discussion is welcome), but rather, a documentation of the steps used to obtain a successful setup, and general considerations about the concept.

# Table of contents #

- [Procedure](#procedure)
- [Improvements/Optimizations](#improvementsoptimizations)
- [QEMU Disk utils/LibGuestFS handy commands](#qemu-disk-utilslibguestfs-handy-commands)
- [References](#references)

# Procedure #

## General notes ##

All the commands must be run with sudo permissions.

Reference system:

- Ubuntu 16.04 x86-64
- ASRock Mini ITX FATAL1TY Z170 Gaming-ITX/AC
- Intel i7-6770K
- Gigabyte GTX 960 GV-N960IXOC-2GD

## Install/prepare required software ##

For a full installation, the following software is required:

- the UEFI firmware (OVMF - open source UEFI firmware);
- the Virtio drivers, which accelerate various emulated hardware;
- and of course, QEMU.

Download and installl the OVMF Ubuntu package:

    wget http://de.archive.ubuntu.com/ubuntu/pool/universe/e/edk2/ovmf_0~20160813.de74668f-1_all.deb
    dpkg -i ovmf_0~20160813.de74668f-1_all.deb

note that the Xenial version is buggy and must not be used; the Yakkety version (above) never caused issues on my system.

Download the Windows Virtio Drivers ISO image:

    wget https://fedorapeople.org/groups/virt/virtio-win/direct-downloads/stable-virtio/virtio-win.iso

the Virtio website is https://fedoraproject.org/wiki/Windows_Virtio_Drivers.

## Intel only: Enable IOMMU ##

Only on Intel systems, set the IOMMU kernel parameter:

    perl -i -pe 's/(GRUB_CMDLINE_LINUX_DEFAULT=.*)"/\1 intel_iommu=on"/' /etc/default/grub
    update-grub

## Check IOMMU, and set the data ##

Check that the MMU has been enabled:

    dmesg | grep -e DMAR -e IOMMU

Check the IOMMU group:

    for iommu_group in $(find /sys/kernel/iommu_groups/ -maxdepth 1 -mindepth 1 -type d); do echo "IOMMU group $(basename "$iommu_group")"; for device in $(ls -1 "$iommu_group"/devices/); do echo -n $'\t'; lspci -nns "$device"; done; done

IOMMU groups are sets of devices which can be virtualized only atomically; this is a sample of a good system:

    IOMMU group 1
      00:02.0 Host bridge [0600]: Advanced Micro Devices, Inc. [AMD] Device [1022:1424]
      00:02.1 PCI bridge [0604]: Advanced Micro Devices, Inc. [AMD] Device [1022:1425]
      01:00.0 VGA compatible controller [0300]: NVIDIA Corporation GM206 [GeForce GTX 960] [10de:1401] (rev a1)
      01:00.1 Audio device [0403]: NVIDIA Corporation Device [10de:0fba] (rev a1)

Ignoring the briges, there are no other devices on the same IOMMU group. If other devices are found, tricks (beyond the scope of this document) need to be used.

From the above convfiguration, these are the values which need to be noted:

    VGAPT_VGA_ID='10de:1401'
    VGAPT_VGA_AUDIO_ID='10de:0fba'
    VGAPT_VGA_BUS=01:00.0
    VGAPT_VGA_AUDIO_BUS=01:00.1

## Assign VGA devices to VFIO ##

Bind the vfio driver to the graphic card before any other driver:

    echo options vfio-pci ids=$VGAPT_VGA_ID,$VGAPT_VGA_AUDIO_ID > /etc/modprobe.d/vfio.conf
    printf "vfio\nvfio_pci\n" > /etc/modules-load.d/vfio.conf

    update-initramfs -u

The approach above implies that the peripheral *can't* be used except for the VGA Passthrough.

While it's possible that on some systems the device can be bound at any moment, on a general basis, doing so will cause instability.

## Execute QEMU! ##

Reboot, then execute QEMU! Sample invocation:

    $QEMU_BINARY \
      -drive if=pflash,format=raw,readonly,file=$VGAPT_FIRMWARE_BIN \
      -drive if=pflash,format=raw,file=$VGAPT_FIRMWARE_VARS_TMP \
      -enable-kvm \
      -machine q35,accel=kvm,mem-merge=off \
      -cpu host,kvm=off,hv_vendor_id=vgaptrocks,hv_relaxed,hv_spinlocks=0x1fff,hv_vapic,hv_time \
      -smp $CORES_NUMBER,cores=$CORES_NUMBER,sockets=1,threads=1 \
      -m $VGAPT_MEMORY \
      -vga none \
      -rtc base=localtime \
      -serial none -parallel none \
      -usb \
      -usbdevice host:$VGAPT_KEYBOARD_ID \
      -usbdevice host:$VGAPT_MOUSE_ID \
      -device vfio-pci,host=$VGAPT_VGA_BUS,multifunction=on \
      -device vfio-pci,host=$VGAPT_VGA_AUDIO_BUS \
      -device virtio-scsi-pci,id=scsi \
      -drive file=$VGAPT_DISK_IMAGE,id=disk,format=qcow2,if=none,cache=writeback -device scsi-hd,drive=disk \
      -net nic,model=virtio \
      -net user,smb=$VM_SHARED_FOLDERS \
      -drive file=$VGAPT_VIRTIO_DRIVERS_CD,id=virtiocd,format=raw,if=none -device ide-cd,bus=ide.1,drive=virtiocd \
    ;

Notes (will be expanded):

- using `-bios` in place of the two `pflash` will prevent booting from the DVD;
- uses the video card audio (audio card emulation is poor; between the other things, it caused windows to hang on boot);
- keyboard and mouse as passed to the machine; if it hangs, it's not possible to switch back to the host
- VirtIO is used to accelerate the network card emulation
- shared folders are enabled (on the host, Samba is required)

# Improvements/Optimizations #

## Possible improvements ##

- it's not clear whether there is a way to ensure that `vfio-pci` takes over before other drivers - maybe this is not required on any configuration?
- can anybody confirm that `mem-merge=off` is useless when only one VM is running at a time? or disabling merging removes a possible overhead?
- understand/test `kernel_irqchip=on`

## General optimizations ##

The optimizations tried did't yield any significant improvement, with a single exception for AMD platforms:

- it's been reported (also verified on my previous AMD system) that nested paging must be disabled on AMD 4ᵗʰ generation CPUS (Bulldozer, ...), otherwise the system will go very slow (loss of ~70% of performance):

    echo "options kvm-amd npt=0" > /etc/modprobe.d/kvm-amd.conf

- using the `performance` CPU governors yielded a negligible improvement
- use CPU pinning (requires patch) yielded a negligible improvement
- hugepages (use with `-mem-prealloc -mem-path /dev/hugepages`):

    echo 'vm.nr_hugepages = 5120' > /etc/sysctl.d/50-hugepages-vfio.conf
    $QEMU_BINARY -mem-prealloc -mem-path /dev/hugepages

Note that hugepages need to be locked at boot time, which will reduce the memory available for other uses.

## Optimization issues/limitations ##

The enlightenment `hv_spinlocks=0x1fff` causes Windows 8.1 to reboot before completing the boot.

Windows 7 doesn't support enlightenments with OVMF (see https://bugzilla.redhat.com/show_bug.cgi?id=1185253, Additional info #2).

# QEMU Disk utils/LibGuestFS handy commands #

Create a diff disk:

    qemu-img create -f qcow2 -b $VGAPT_DISK_IMAGE $VGAPT_DISK_IMAGE.diff

Create a merged copy a disk (of any type):

    qemu-img convert -p $VGAPT_DISK_IMAGE.diff -O qcow2 $VGAPT_DISK_IMAGE.merged

Create a compressed copy without unallocated space (use `--tmp`, otherwise `/tmp` is used!!):

    virt-sparsify --compress --tmp $(dirname $VGAPT_DISK_IMAGE) $VGAPT_DISK_IMAGE $VGAPT_DISK_IMAGE.sparse.compressed

Import a directory in a disk:

    virt-copy-in -a $VGAPT_DISK_IMAGE /tmp/pizza /

# References #

1. https://wiki.debian.org/VGAPassthrough
2. https://bufferoverflow.io/gpu-passthrough
3. https://wiki.archlinux.org/index.php/PCI_passthrough_via_OVMF
4. http://www.se7ensins.com/forums/threads/how-to-setup-a-gaming-virtual-machine-with-gpu-passthrough-qemu-kvm-libvirt-and-vfio.1371980 (synergy/tweaks)
5. https://bbs.archlinux.org/viewtopic.php?id=162768
6. https://www.redhat.com/mailman/listinfo/vfio-users
7. http://forum.ipfire.org/viewtopic.php?t=12642 (other optimizations)
8. https://help.ubuntu.com/community/KVM%20-%20Using%20Hugepages (enable hugepages)
9. https://bbs.archlinux.org/viewtopic.php?pid=1270311#p1270311 (NPT et al.)
