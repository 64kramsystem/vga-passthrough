# Basic setup #

## General notes ##

All the commands must be run with sudo permissions.

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

## Intel only: enable IOMMU ##

Only on Intel systems, set the IOMMU kernel parameter:

    perl -i -pe 's/(GRUB_CMDLINE_LINUX_DEFAULT=.*)"/\1 intel_iommu=on"/' /etc/default/grub
    update-grub

## AMD only: disable nested paging ##

It's been reported (also verified on my previous AMD system) that nested paging must be disabled on AMD 4ᵗʰ generation CPUS (Bulldozer, ...), otherwise the system will go very slow (loss of ~70% of performance):

    echo "options kvm-amd npt=0" > /etc/modprobe.d/kvm-amd.conf

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

Reboot, then execute QEMU!

Sample invocation:

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

[Previous: Introduction to VGA Passthrough](01_INTRODUCTION_TO_VGA_PASSTHROUGH.md)
[Next: Possible improvements](03_POSSIBLE_IMPROVEMENTS.md)
