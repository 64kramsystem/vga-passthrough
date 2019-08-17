# Possible improvements

This section includes a list of possible improvements both to the configuration and the VM performance.

With the proper configuration, covered fully until this point, my system yield already the "95% of native" performance, without stuttering; however, below I point some tweaks that may improve the performance of other users' setups.

I find the best reference to be the [Arch passthrough wiki page](https://wiki.archlinux.org/index.php/PCI_passthrough_via_OVMF), so users may want to check it out.

Thanks to Reddit user [tholin](https://www.reddit.com/user/tholin) for contributing specific details and improvements to this section.

## Configuration improvements

In this guide is not known:

- whether there is a way to ensure that `vfio-pci` takes over before other drivers - maybe this is not required on any configuration;
- whether `mem-merge=off` has overhead when only one VM is running at a time, or it doesn't (therefore, it's not meaningful to disable it);

### CPU pinning

Simply enabling it (using [my patched QEMU fork](https://github.com/saveriomiroddi/qemu-pinning)) yielded a negligible improvement.

It is reported that it can reduce latency and stuttering in certain edge cases.

Users that intend to try such tweak should (but not necessarily) reserve the CPU cores to the VM for exclusive use.

Reference: https://www.redhat.com/archives/vfio-users/2017-February/msg00010.html

### Hugepages

Enabling them didn't yield any improvement (note that in my setup I have no swap; this may be related, or not).

In order to use them:

    echo 'vm.nr_hugepages = 5120' > /etc/sysctl.d/50-hugepages-vfio.conf
    $QEMU_BINARY -mem-prealloc -mem-path /dev/hugepages

In their basic usage, since hugepages require contiguous space, they generally have to be locked at boot time, which will reduce the memory available for other uses.

It's possible to work around this by allocating them dinamycally and free them when the VM shuts down; it's not optional but it works. Reference: https://www.redhat.com/archives/vfio-users/2016-July/msg00017.html

## Optimization issues/limitations

The following are limitations I've found during my VFIO experimentation stage (2016 or earlier); possibly, things may have improved in the meanwhile:

- The enlightenment `hv_spinlocks=0x1fff` causes Windows 8.1 to reboot before completing the boot.
- Windows 7 doesn't support enlightenments with OVMF (see https://bugzilla.redhat.com/show_bug.cgi?id=1185253, Additional info #2).

[Previous: Troubleshooting](5_TROUBLESHOOTING.md) | [Next: Profiling KVM](7_PROFILING_KVM.md)
