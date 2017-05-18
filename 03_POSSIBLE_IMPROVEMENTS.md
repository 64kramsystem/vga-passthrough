# Possible improvements #

## General improvements ##

In this guide is not known:

- whether there is a way to ensure that `vfio-pci` takes over before other drivers - maybe this is not required on any configuration;
- whether `mem-merge=off` has overhead when only one VM is running at a time, or it doesn't (therefore, it's not meaningful to disable it);
- the functionality of `kernel_irqchip=on`.

## General optimizations ##

The optimizations tried didn't yield any significant improvement, with a single exception for AMD platforms:

- using the `performance` CPU governors yielded a negligible improvement
- use CPU pinning (requires patch) yielded a negligible improvement
- hugepages (use with `-mem-prealloc -mem-path /dev/hugepages`):

  `echo 'vm.nr_hugepages = 5120' > /etc/sysctl.d/50-hugepages-vfio.conf`

  `$QEMU_BINARY -mem-prealloc -mem-path /dev/hugepages`

Note that hugepages need to be locked at boot time, which will reduce the memory available for other uses.

## Optimization issues/limitations ##

The enlightenment `hv_spinlocks=0x1fff` causes Windows 8.1 to reboot before completing the boot.

Windows 7 doesn't support enlightenments with OVMF (see https://bugzilla.redhat.com/show_bug.cgi?id=1185253, Additional info #2).

[Previous: Basic setup](02_BASIC_SETUP.md)
[Next: Useful tools](04_USEFUL_TOOLS.md)
