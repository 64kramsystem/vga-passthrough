# Input handling

## Introduction

Conventionally, VFIO guides instruct the user to either pass through the input devices (keyboard+mouse), or to use a sharing software (typically Synergy).

This is a bit puzzling, because neither of those strategies are required; this guide in fact uses a different approach.

In this chapter I'll give some context and explain all the strategies.

## Setups and strategies

### Standard (without VFIO)

In the standard QEMU invocation, the user typically chooses an emulated video adapter (e.g. `-vga qxl`) and a user interface (e.g. `-display gtk`).

When a user interface is specified, QEMU performs the typical hypervisor input trapping functionality - when the user clicks on the interface, any input from that moment on is forwarded to the guest.

In order to stop trapping, a key combination is used (on Linux, `Ctrl+Alt+G`).

### VFIO, with devices passthrough

In this mode, QEMU is executed without user interface, and the input devices are configured to be passed through to the guest:

```sh
# Find the ids from `lsusb`
export VGAPT_KEYBOARD_VEND_ID=abcd
export VGAPT_KEYBOARD_PROD_ID=ef0a
export VGAPT_MOUSE_VEND_ID=1234
export VGAPT_MOUSE_PROD_ID=5678

-device usb-host,vendorid=0x$VGAPT_KEYBOARD_VEND_ID,productid=0x$VGAPT_KEYBOARD_PROD_ID \
-device usb-host,vendorid=0x$VGAPT_MOUSE_VEND_ID,productid=0x$VGAPT_MOUSE_PROD_ID \
```

If the guest hangs, there is no way send any input to the host, and a reset is required. If the user tries to disconnect and reconnect those devices, they are immediately trapped by QEMU and passed through, preventing any interaction with the host.

#### VFIO, with devices passthrough, and The Poor Man's Kill Switch™

A curious way to handle the unexpected guest problems is to set a "kill switch": something that will terminate QEMU at will, without the need for mouse/keyboard. This implementation is performed with the help of a USB drive.

**Requirements:**

- a flash key (or any USB drive);
- sudo permissions, if you create/edit the script under `/usr/local/bin`;
- an O/S with removable media automounting configured (eg. in XFCE, use `thunar-volman-settings`).

**Instructions:**

Plug a flash key, and create an empty file on it, with an arbitrary name, then take note of its full path, and unmount the partition:

```sh
touch /media/myuser/MYKEY/.kill_qemu
umount /media/myuser/MYKEY
```

Create a script (say, `/usr/local/bin/kill_qemu.sh`), with the following content:

```sh
#!/bin/bash

kill_switch_file=/media/myuser/MYKEY/.kill_qemu    # use the filename noted above
sleep_time=2

while [[ ! -e "$kill_switch_file" ]]; do
  sleep $sleep_time
done

echo 'Killing QEMU!!'
pkill -f qemu-system-x86_64
```

And give it executable permissions:

```sh
chmod +x /usr/local/bin/kill_qemu.sh
```

Now, before executing QEMU, run the script in the background (sudo required):

```sh
sudo -b kill_qemu.sh
```

Likely, you will add the execution above in your QEMU execution script.

If QEMU would happen to hang, just plug the flash key, and the script will kill QEMU as soon as the key is (auto)mounted.

### VFIO, with server/client input setup

In this setup, a server/client software is used. With [Synergy](https://symless.com/synergy) (the most common input sharing software), the server is installed on the host, and the client on the guest; the software will take care of sending the input where required.

If the guest or QEMU hang, it's possible to use the input devices on the host without any problem.

### VFIO, with dual graphics adapter

This is the simplest, and still perfectly working solution, which is curiously never mentioned.

Since nothing prevents QEMU to have a user interface (with an emulated video adapter) *and* VFIO at the same time, one just starts QEMU with both the emulated and VFIO adapter options, without input passthrough:

```sh
-vga qxl -display gtk -device vfio-pci
# no `-device usb-host ...`
-device vfio-pci,host=$VGAPT_VGA_BUS \
-device vfio-pci,host=$VGAPT_VGA_AUDIO_BUS \
```

The machine will start with both displays. If the guest hangs, it's not problem, as QEMU still traps the exit hotkey, which can be used to restore control to the host.

This strategy doesn't work for the case where QEMU hangs (crashes are instead ok), although this is very rare (if it ever happens); if a machine has this particular problem, the kill switch strategy can be applied.
[Previous: Basic setup](3_BASIC_SETUP.md) | [Next: Monitors and speakers](5_MONITORS_AND_SPEAKERS.md)
