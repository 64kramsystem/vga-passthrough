# Basic setup

## General notes

All the commands must be run with sudo permissions.

## Install/prepare required software

For a full installation, the following software is required:

- the UEFI firmware (OVMF - open source UEFI firmware);
- the Virtio drivers, which accelerate various emulated hardware;
- and of course, QEMU.

Install the latest OVMF Ubuntu package:

    wget http://de.archive.ubuntu.com/ubuntu/pool/universe/e/edk2/ovmf_0~20170911.5dfba97c-1_all.deb -O /tmp/ovmf.deb
    dpkg -i /tmp/ovmf.deb

Download the Windows Virtio Drivers ISO image:

    wget https://fedorapeople.org/groups/virt/virtio-win/direct-downloads/stable-virtio/virtio-win.iso

the Virtio website is https://fedoraproject.org/wiki/Windows_Virtio_Drivers.

Install the QEMU package, plus utilities:

    apt-get install qemu-system-x86 qemu-utils

## Intel only: enable IOMMU

Only on Intel systems, set the IOMMU kernel parameter:

    perl -i -pe 's/(GRUB_CMDLINE_LINUX_DEFAULT=.*)"/\1 intel_iommu=on"/' /etc/default/grub
    update-grub

## AMD only: disable nested paging

It's been reported (also verified on my previous AMD system) that nested paging must be disabled on AMD 4ᵗʰ generation CPUS (Bulldozer, ...), otherwise the system will go very slow (loss of ~70% of performance):

    echo "options kvm-amd npt=0" > /etc/modprobe.d/kvm-amd.conf

## Check IOMMU, and set the data

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

From the above configuration, these are the values to take note of:

    VGAPT_VGA_ID='10de:1401'
    VGAPT_VGA_AUDIO_ID='10de:0fba'
    VGAPT_VGA_BUS=01:00.0
    VGAPT_VGA_AUDIO_BUS=01:00.1

## Assign VGA devices to VFIO

Bind the vfio driver to the graphic card before any other driver:

    echo options vfio-pci ids=$VGAPT_VGA_ID,$VGAPT_VGA_AUDIO_ID > /etc/modprobe.d/vfio.conf
    printf "vfio\nvfio_pci\n" > /etc/modules-load.d/vfio.conf

    update-initramfs -u

The approach above implies that the peripheral *can't* be used except for the VGA Passthrough.

While it's possible that on some systems the device can be bound at any moment, on a general basis, doing so will increase the risk of instability.

## Ensure that the VGA drivers are not taking control of the card

It's important to uninstall the host VGA drivers, if they are installed, as they may take over the card control before the VFIO driver.

In case the drivers are standard part of the system (eg. nouveau), it's possible to blacklist them.

As of today, this document doesn't cover how to be 100% sure that VFIO takes control of one VGA, on a system with two cards from the same producer.

## Gather keyboard/mouse USB id

We need keyboard and mouse USB id, in order to pass them to the VM:

    lsusb

Sample output:

    ...
    Bus 001 Device 006: ID 05f3:0007 PI Engineering, Inc. Kinesis Advantage PRO MPC/USB Keyboard
    Bus 001 Device 007: ID 1bcf:0824 Sunplus Innovation Technology Inc.
    ...

From the above configuration, these are the values to take note of:

    VGAPT_KEYBOARD_VEND_ID=05f3
    VGAPT_KEYBOARD_PROD_ID=0007
    VGAPT_MOUSE_VEND_ID=1bcf
    VGAPT_MOUSE_PROD_ID=0824

## Create a virtual disk

Create a virtual disk using the QEMU utility:

  qemu-img create -f qcow2 /path/to/virtual_disk.img 128G

In the example above, the size is 128 GiB (dynamically allocated, so it will start with a minimal occupation).

## Execute QEMU, and install/prepare Windows

Reboot, execute QEMU, and install Windows. The VirtIO drivers must also be installed, in order to maximize I/O (in particular, disk) performance.

Parameters:

    # Convenience script for assigning all the cores:
    export CORES_NUMBER=$(lscpu --all -p=CORE | grep -v ^# | sort | uniq | wc -l)
    # Assigned memory, in MiB:
    export VGAPT_MEMORY=8192

    export VGAPT_KEYBOARD_VEND_ID=05f3
    export VGAPT_KEYBOARD_PROD_ID=0007
    export VGAPT_MOUSE_VEND_ID=1bcf
    export VGAPT_MOUSE_PROD_ID=0824

    export VGAPT_VGA_ID='10de:1401'
    export VGAPT_VGA_AUDIO_ID='10de:0fba'
    export VGAPT_VGA_BUS=01:00.0
    export VGAPT_VGA_AUDIO_BUS=01:00.1

    # Standard locations from the Ubuntu `ovmf` package; last one is arbitrary:
    export VGAPT_FIRMWARE_BIN=/usr/share/OVMF/OVMF_CODE.fd
    export VGAPT_FIRMWARE_VARS=/usr/share/OVMF/OVMF_VARS.fd
    export VGAPT_FIRMWARE_VARS_TMP=/tmp/OVMF_VARS.fd.tmp

    # Standard location from the Ubuntu `qemu-system-x86` package:
    export QEMU_BINARY=/usr/bin/qemu-system-x86_64

    export VGAPT_DISK_IMAGE=/path/to/virtual_disk.img
    export VGAPT_WINDOWS_ISO=/path/to/win_installation.iso
    export VGAPT_VIRTIO_DRIVERS_ISO=/path/to/virtio-win.iso

    export VGAPT_SHARED_FOLDERS=/path/to/shared_folders

Invocation:

    cp -f $VGAPT_FIRMWARE_VARS $VGAPT_FIRMWARE_VARS_TMP &&
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
      -device usb-host,vendorid=0x$VGAPT_KEYBOARD_VEND_ID,productid=0x$VGAPT_KEYBOARD_PROD_ID \
      -device usb-host,vendorid=0x$VGAPT_MOUSE_VEND_ID,productid=0x$VGAPT_MOUSE_PROD_ID \
      -device vfio-pci,host=$VGAPT_VGA_BUS,multifunction=on \
      -device vfio-pci,host=$VGAPT_VGA_AUDIO_BUS \
      -device virtio-scsi-pci,id=scsi \
      -drive file=$VGAPT_DISK_IMAGE,id=disk,format=qcow2,if=none,cache=writeback -device scsi-hd,drive=disk \
      -drive file=$VGAPT_WINDOWS_ISO,id=scsicd,format=raw,if=none -device scsi-cd,drive=scsicd \
      -drive file=$VGAPT_VIRTIO_DRIVERS_ISO,id=idecd,format=raw,if=none -device ide-cd,bus=ide.1,drive=idecd \
      -net nic,model=virtio \
      -net user,smb=$VGAPT_SHARED_FOLDERS \
    ;

The above command sets two cd drives (and ISOs), one for Windows, and the other for VirtIO.

Notes (will be expanded):

- using `-bios` in place of the two `pflash` will prevent booting from the DVD;
- uses the video card audio (audio card emulation is poor; between the other things, it caused windows to hang on boot);
- keyboard and mouse as passed to the machine; if it hangs, it's not possible to switch back to the host (see next paragraph for a workaround);
- shared folders are enabled (on the host, Samba is required)

## Keyboard/mouse stealing

QEMU will steal the keyboard and mouse devices, therefore, if it hangs, it's not possible to return to the host O/S (without plugging another keyboard).

There are a few approaches to this problem; the most common is to install the commercial software [Synergy](https://symless.com/synergy), which provides the sharing logic.

For those who, for whatever reason, don't want to use such software, a different approach can be used.

### Poor man's QEMU input devices switching workaround

During VFIO virtualization sessions, typically keyboard/mouse switching is not required.  
In real world scenarios, switching is only required if the guest hangs: in this case, QEMU won't release the USB devices, and if there is no mean for switching mouse/keyboard (or if one hasn't got a second keyboard), a host reboot will need to be performed.

A curious way to handle this problem is to set a "kill switch": something that will terminate QEMU at will, without need for mouse/keyboard. This implementation is performed with the help of a USB drive.

**Requirements:**

- a flash key (or any USB drive);
- sudo permissions, if you create/edit the script under `/usr/local/bin`;
- an O/S with removable media automounting configured (eg. in XFCE, use `thunar-volman-settings`).

**Instructions:**

Plug a flash key, and create an empty file on it, with an arbitrary name, then take note of its full path, and unmount the partition:

    touch /media/myuser/MYKEY/.kill_qemu
    umount /media/myuser/MYKEY

Create a script (say, `/usr/local/bin/kill_qemu.sh`), with the following content:

    #!/bin/bash

    kill_switch_file=/media/myuser/MYKEY/.kill_qemu    # use the filename noted above
    sleep_time=2

    while [[ ! -e "$kill_switch_file" ]]; do
      sleep $sleep_time
    done

    echo Killing QEMU!!
    pkill -f qemu-system-x86_64

And give it executable permissions:

    chmod +x /usr/local/bin/kill_qemu.sh

Now, before executing QEMU, run the script in the background (sudo required):

    sudo -b kill_qemu.sh

Likely, you will add the execution above in your QEMU execution script.

If QEMU would happen to hang, you just plug the flash key, and the script will kill QEMU once the key is mounted :-)

[Previous: A word of caution](2_A_WORD_OF_CAUTION.md) | [Next: Troubleshooting](4_TROUBLESHOOTING.md)
