# Monitors and speakers

<!-- - [Monitors and speakers](#monitors-and-speakers) -->
  - [Introduction](#introduction)
  - [No switch, one monitor](#no-switch-one-monitor)
  - [No switch, two monitors](#no-switch-two-monitors)
  - [Passing the integrated audio device](#passing-the-integrated-audio-device)

## Introduction

Using a guest with the VGA passthrough presents a minor inconvenience - handling separate outputs.

Since the outputs - at least the video signal - are separated, and one expects to fully use peripherals at least on the host, it's convenient to find a practical way to switch outputs.

With modern peripherals, one can take advantage of the multiple inputs, and avoid hardware or software switches.

For the setups, we assume the following hardware ports:

- integrated GPU, used by the host, with HDMI output
- discrete GPU, used by the guest, with Display Port output
- on-board SPDIF (digital) output
- monitor(s) with HDMI and Display Port inputs, and jack (analog) audio output
- speakers with SPDIF (digital) and jack (analog) inputs

The strategies below can be applied independently of using one or two monitors (of course, conditionally to the ports availability).

## No switch, one monitor

Without switch, one can take advantage of the multiple inputs of the peripherals.

In this case, the setup will be:

- integrated GPU HDMI + discrete GPU Display Port outputs -> monitor inputs
- on-board SPDIF + monitor jack outputs -> speakers inputs

When the host starts, the default is the monitor set to use the HDMI input, and the speakers to use the SPDIF input.

When switching to the guest, one sets the monitor to use the Display Port input, and the speakers to use the jack input.  

The audio works because the guest uses the discrete GPU audio device; the signal is carried via Display Port, and from the monitor, it goes to the speakers.

## No switch, two monitors

The advantage of using a switch is that it avoids using the monitor switch - the first is generally a single button push, while the second involves navigating through menus; additionally, some monitors go into sleep shortly after losing the signal, which introduces an annoying delay while switching.

In this case, the setup will be:

- integrated GPU Display Port + discrete GPU Display Port outputs -> switch -> gaming monitor input
- integrated GPU HDMI -> secondary monitor input
- on-board SPDIF + monitor jack outputs -> speakers inputs

The strategy is essentially the same. The difference is that when using the guest, the input is switched via switch.

## Passing the integrated audio device

In some systems, the integrated audio device is on its own group, for example, on the MSI MAG B550 Mortar:

```
IOMMU group 19
	2d:00.4 Audio device [0403]: Advanced Micro Devices, Inc. [AMD] Starship/Matisse HD Audio Controller [1022:1487]
```

This allows passing the device through (although it will cause a warning due to dependency on another group - see end of section).

The convenience of this setup is that one can have a single audio pipeline; when the VM is started, the audio device can be unbound from the audio driver and bound to the vfio one, and when the VM is shut down, the opposite.

On the tested system, this worked stably, but it needs to be kept in mind that hot swapping is not necessarily supported by the component/driver.

For those who want to try, the procedure is simple:

```sh
# This is necessary, at least on the tested setup. It may be possible to run it only once per host boot;
# if scripted, running it on each VM startup doesn't hurt.
#
# Its purpose is documented here: https://www.kernel.org/doc/Documentation/ABI/testing/sysfs-bus-pci:
#
# > This may allow the driver to support more hardware than was included in the driver's static device
# > ID support table at compile time.
#
echo "1022 1487" | sudo tee /sys/bus/pci/drivers/vfio-pci/new_id

# On VM startup
echo "0000:2d:00.4" | sudo tee /sys/bus/pci/drivers/snd_hda_intel/unbind
echo "0000:2d:00.4" | sudo tee /sys/bus/pci/drivers/vfio-pci/bind

# Start QEMU as usual, passing through also this device:
qemu-system-x86_64 -device vfio-pci,host=2d:00.4 # etc.etc.

# On VM shutdown
echo "0000:2d:00.4" | sudo tee /sys/bus/pci/drivers/vfio-pci/unbind
echo "0000:2d:00.4" | sudo tee /sys/bus/pci/drivers/snd_hda_intel/bind
```

The status can be checked via `lspci`:

```sh
$ lspci -v | perl -ne 'print if (/^2d:00.4/ .. /^$/) && /Kernel/'
	Kernel driver in use: snd_hda_intel
	Kernel modules: snd_hda_intel
```

After the switch, it will look like this:

```sh
$ lspci -v | perl -ne 'print if (/^2d:00.4/ .. /^$/) && /Kernel/'
	Kernel driver in use: vfio-pci
	Kernel modules: snd_hda_intel
```

On the test setup, QEMU displays a warning `Cannot reset device <BUS_ID>, depends on group <GROUP_NUM> which is not owned.`; this delays the QEMU startup time, but can be ignored.

[Previous: Input handling](4_INPUT_HANDLING.md) | [Next: Troubleshooting](6_TROUBLESHOOTING.md)
