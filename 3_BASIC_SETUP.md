# Basic setup

## General notes

All the commands must be run with sudo permissions.

This guide assumes a reasonably modern machine and O/S, since several things have changed in the last years (CPU power saving, bug fixes in the kernel, etc.).

Two GPUs are required; the combination integrated + discrete is the best option. In some cases, it's possible to obtain the passthrough with a single GPU, however, it requires patching the card bios, and it's outside the scope of this guide.

## Install/prepare required software

For a full installation, the following software is required:

- the UEFI firmware (OVMF - open source UEFI firmware);
- the Virtio drivers, which accelerate various emulated hardware;
- and of course, QEMU.

Install the latest OVMF Ubuntu package:

```sh
latest_ovmf_package_link=$(wget http://archive.ubuntu.com/ubuntu/pool/universe/e/edk2 -O- | perl -ne 'print "$1\n" if /"(ovmf.+deb)"/' | tail -n 1)
wget "http://archive.ubuntu.com/ubuntu/pool/universe/e/edk2/$latest_ovmf_package_link" -O /tmp/ovmf.deb
dpkg -i /tmp/ovmf.deb
```

Download the Windows Virtio Drivers ISO image:

```sh
wget https://fedorapeople.org/groups/virt/virtio-win/direct-downloads/stable-virtio/virtio-win.iso
```

the Virtio website is https://fedoraproject.org/wiki/Windows_Virtio_Drivers.

Install the QEMU package, plus utilities:

```sh
apt-get install qemu-system-x86 qemu-utils
```

## Enable IOMMU

Set the IOMMU kernel parameter, according to the CPU manufacturer:

```sh
# Intel
perl -i -pe 's/(GRUB_CMDLINE_LINUX_DEFAULT=.*)"/\1 intel_iommu=on"/' /etc/default/grub
# or AMD
perl -i -pe 's/(GRUB_CMDLINE_LINUX_DEFAULT=.*)"/\1 amd_iommu=on"/' /etc/default/grub

# Common
update-grub
```

## Check IOMMU, and set the data

Check that the MMU has been enabled:

```sh
dmesg | grep -e DMAR -e IOMMU
```

Check the IOMMU group:

```sh
for iommu_group in $(ls -dv /sys/kernel/iommu_groups/*/); do
  echo "IOMMU group $(basename "$iommu_group")"
  for device in $(ls -1 "$iommu_group"/devices/); do
    echo -n $'\t'
    lspci -nns "$device"
  done
done
```

IOMMU groups are sets of devices which can be virtualized only atomically; this is a sample of a good system:

```
IOMMU group 1
  00:02.0 Host bridge [0600]: Advanced Micro Devices, Inc. [AMD] Device [1022:1424]
  00:02.1 PCI bridge [0604]: Advanced Micro Devices, Inc. [AMD] Device [1022:1425]
  01:00.0 VGA compatible controller [0300]: NVIDIA Corporation GM206 [GeForce GTX 960] [10de:1401] (rev a1)
  01:00.1 Audio device [0403]: NVIDIA Corporation Device [10de:0fba] (rev a1)
```

Ignoring the briges, there are no other devices on the same IOMMU group. If other devices are found, tricks (beyond the scope of this document) need to be used.

From the above configuration, these are the values to take note of:

```sh
VGAPT_VGA_ID='10de:1401'
VGAPT_VGA_AUDIO_ID='10de:0fba'
VGAPT_VGA_BUS=01:00.0
VGAPT_VGA_AUDIO_BUS=01:00.1
```

## Assign VGA devices to VFIO

Bind the vfio driver to the graphic card before any other driver:

```sh
cat > /etc/modprobe.d/vfio.conf << CONF
options vfio-pci ids=$VGAPT_VGA_ID,$VGAPT_VGA_AUDIO_ID
CONF

cat > /etc/modules-load.d/vfio.conf << CONF
vfio
vfio_pci
CONF

update-initramfs -u
```

The approach above implies that the peripheral *can't* be used except for the VGA Passthrough.

While it's possible that on some systems the device can be bound at any moment, on a general basis, doing so will increase the risk of instability.

## Ensure that the VGA drivers are not taking control of the card

It's crucial to ensure that the vfio driver takes the card (components) over before the standard graphic driver does it.

There are a few approaches to solve this problem, which depend on the O/S.

### Kernels with an off-kernel VFIO module

On systems whose kernels has an off-kernel VFIO module (e.g. Ubuntu 18.04), there are a few approaches.

An approach is to uninstall the driver \[package\]; for example, in case of an Nvidia card:

```sh
apt purge "xserver-xorg-video-nouveau*"
```

Another approach is to blacklist the driver:

```sh
echo "blacklist nouveau" > /etc/modprobe.d/blacklist_nouveau.conf
```

The best approach, which is not blunt and makes a good use of the modules framework, is to use the [`softdep` modprobe command](https://www.systutorials.com/docs/linux/man/5-modprobe.d/), setting the load order via dependencies.

```sh
# Append this to the previously created file.
#
echo "softdep nouveau pre: vfio_pci" >> /etc/modprobe.d/vfio.conf
```

Note that this approach requires the name of the module as listed by `lsmod`, i.e. `vfio_pci` instead of `vfio-pci`.

### Cards with a built-in USB controller

Some cards, like the RTX 2070 Super, have a USB controller onboard, which causes an interesting problem.

Since the USB driver (`xhci_hcd`) is built in the kernel, it takes control of the devices before the off-kernel modules (like the VFIO one); as a consequence, it can't be blacklisted, nor it can be set as dependency of the VFIO module.

In such conditions, it's not possible to start a passthrough, because the IOMMU group will not have all the peripherals controlled by the VFIO module.

Fortunately, the controller can be unbound from the `xhci_hcd` module and rebound to the VFIO module. What we want to do though, is to make sure this happens as early as possible, as good VFIO practice; this can be accomplished with an initramfs script.

First, take note of the device using the IOMMU groups listing script; for example, this is the outout of an RTX 2070 super:

```sh
lspci -nn  | grep '^26:'

# 26:00.0 VGA compatible controller [0300]: NVIDIA Corporation TU104 [GeForce RTX 2070 SUPER] [10de:1e84] (rev a1)
# 26:00.1 Audio device [0403]: NVIDIA Corporation TU104 HD Audio Controller [10de:10f8] (rev a1)
# 26:00.2 USB controller [0c03]: NVIDIA Corporation TU104 USB 3.1 Host Controller [10de:1ad8] (rev a1)
# 26:00.3 Serial bus controller [0c80]: NVIDIA Corporation TU104 USB Type-C UCSI Controller [10de:1ad9] (rev a1)
```

Then, execute the following, assigning to the `device_address` variable the address just gathered:

```sh
cat > /etc/initramfs-tools/scripts/init-top/vfio-rebind-vga-usb <<'SHELL'
#!/bin/sh

if [ "$1" = "prereqs" ]; then
  echo ""
  exit 0
fi

device_address="26:00.2" # replace here

if [ -f "/sys/bus/pci/devices/0000:$device_address/driver/unbind" ]; then
  echo "vfio: rebinding device at $device_address"
  echo "0000:$device_address" > /sys/bus/pci/drivers/xhci_hcd/unbind
  echo "0000:$device_address" > /sys/bus/pci/drivers/vfio-pci/bind
else
  echo "vfio error: didn't find the device to unbind (at $device_address)"
fi
SHELL

chmod +x /etc/initramfs-tools/scripts/init-top/vfio-rebind-vga-usb

update-initramfs -u
```

### Kernels with a built-in kernel VFIO module

Some systems, e.g. Ubuntu 20.04, have the VFIO module built in the kernel; this makes configuration very simple.

Just run the IOMMU groups listing script, but this time, take note of the device ids (as opposed as the bus addresses), and add a `vfio-pci.ids` kernel parameter:

```sh
lspci -nn | grep '^26:'

# 26:00.0 VGA compatible controller [0300]: NVIDIA Corporation TU104 [GeForce RTX 2070 SUPER] [10de:1e84] (rev a1)
# 26:00.1 Audio device [0403]: NVIDIA Corporation TU104 HD Audio Controller [10de:10f8] (rev a1)
# 26:00.2 USB controller [0c03]: NVIDIA Corporation TU104 USB 3.1 Host Controller [10de:1ad8] (rev a1)
# 26:00.3 Serial bus controller [0c80]: NVIDIA Corporation TU104 USB Type-C UCSI Controller [10de:1ad9] (rev a1)

perl -i -pe 's/(GRUB_CMDLINE_LINUX_DEFAULT=.*)"/$1 vfio-pci.ids=10de:1e84,10de:10f8,10de:1ad8,10de:1ad9"/' /etc/default/grub

update-grub
```

The advantage of this setup is that no initramfs scripts are needed, since `vfio-pci` now runs in the same context as `xhci_hcd`, and is able to take over the control of the device.

## Create a virtual disk

Create a virtual disk using the QEMU utility:

```sh
qemu-img create -f qcow2 /path/to/virtual_disk.img 128G
```

In the example above, the size is 128 GiB (dynamically allocated, so it will start with a minimal occupation).

## Set the `Performance` governor

On at least some systems (eg. the Intel i7-6700k), the CPU frequency may not ramp up when required, causing low performance problems (ie. stuttering). This happens because [KVM cannot actually change the CPU frequency on its own](https://wiki.archlinux.org/index.php/PCI_passthrough_via_OVMF#CPU_frequency_governor).

In the past, typically, tweaking the `ondemand` governor solved the problem, however, nowadays, the scaling system is different, and this governor is not available anymore.

The easiest solution is to force the CPU to use the maximum frequency for all the cores, by using the `performance` governor; this can be enabled and disabled at wish, in this case, before and after the virtualization session.

A governor widget is typically displayed in the system tray, and it allows setting the governors (`Powersave` and `Performance`. Alternatively, it can be enabled with a simple command:

```sh
echo performance | tee /sys/devices/system/cpu/cpu*/cpufreq/scaling_governor
```

## Execute QEMU, and install/prepare Windows

Reboot, execute QEMU, and install Windows. The VirtIO drivers must also be installed, in order to maximize I/O (in particular, disk) performance.

Parameters:

```sh
# Convenience script for assigning all the cores:
export CORES_NUMBER=$(lscpu --all -p=CORE | grep -v ^# | sort | uniq | wc -l)
# Assigned memory, in MiB:
export VGAPT_MEMORY=8192

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
```

Invocation (QEMU 4.0):

```sh
cp -f $VGAPT_FIRMWARE_VARS $VGAPT_FIRMWARE_VARS_TMP &&
$QEMU_BINARY \
  -drive if=pflash,format=raw,readonly=on,file=$VGAPT_FIRMWARE_BIN \
  -drive if=pflash,format=raw,file=$VGAPT_FIRMWARE_VARS_TMP \
  -enable-kvm \
  -machine q35,accel=kvm,mem-merge=off,kernel-irqchip=on \
  -cpu host,kvm=off,hv_vendor_id=vgaptrocks,hv_relaxed,hv_spinlocks=0x1fff,hv_vapic,hv_time \
  -smp $CORES_NUMBER,cores=$CORES_NUMBER,sockets=1,threads=1 \
  -m $VGAPT_MEMORY \
  -display gtk -vga qxl \
  -rtc base=localtime \
  -serial none -parallel none \
  -usb \
  -device vfio-pci,host=$VGAPT_VGA_BUS \
  -device vfio-pci,host=$VGAPT_VGA_AUDIO_BUS \
  -device virtio-scsi-pci,id=scsi \
  -drive file=$VGAPT_DISK_IMAGE,id=disk,format=qcow2,if=none,cache=writeback -device scsi-hd,drive=disk \
  -drive file=$VGAPT_WINDOWS_ISO,id=scsicd,format=raw,if=none -device scsi-cd,drive=scsicd \
  -drive file=$VGAPT_VIRTIO_DRIVERS_ISO,id=idecd,format=raw,if=none -device ide-cd,bus=ide.1,drive=idecd \
  -net nic,model=virtio \
  -net user,smb=$VGAPT_SHARED_FOLDERS \
;
```

The above command sets two cd drives (and ISOs), one for Windows, and the other for VirtIO.

QEMU 3.x users should remove the `kernel-irqchip=on` option (see the [troubleshooting chapter](5_TROUBLESHOOTING.md)).

Notes:

- using `-bios` in place of the two `pflash` will prevent booting from the DVD;
- the commonly set `multifunction=on` device property is a mistake (see [Reddit comment](https://www.reddit.com/r/VFIO/comments/arg25i/issue_using_rtx_2070_for_passthrough/ego6woz));
- uses the video card audio (audio card emulation is poor/unstable);
- shared folders are enabled (on the host, Samba is required);
- this setup has two graphics adapters (emulated and VFIO), with the QEMU user interface; for a description of the input (keyboard/mouse) handling strategies, see the next chapter.

[Previous: A word of caution](2_A_WORD_OF_CAUTION.md) | [Next: Input handling](4_INPUT_HANDLING.md)
