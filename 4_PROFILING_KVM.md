# Profiling KVM

In some edge cases, obscure performance issues may need be diagnosed via profiling.

There is yet another very insightful post by [tholin](https://www.reddit.com/user/tholin):

> The overhead from emulation depends on the workload. If the game tries to access hardware that has to be emulated in software the emulation will suffer.
>
> You can try running `perf kvm --host top -p $(pidof qemu-system-x86_64)` on the host while playing and check how much time is spent in vmx_vcpu_run. That's the function making the switch into guest space and perf account for all time in guest space there. If the game use 100% cpu you want `vmx_vcpu_run` to be close to that. If it's less than say... 70% you have a lot of emulator overhead. 70% is just a value I made up but you can compare a game that runs well with one that don't to see if there is any big difference.
>
> If you run `perf stat -e 'kvm:*' -p $(pidof qemu-system-x86_64) --interval-print 1000` then every second you get a print of kvm related tracepoints. The `kvm_exit` triggers every time the guest tries to access some hardware that has to be emulated by kvm. kvm_exit should preferably be below 50,000 sec.
>
> If you have high emulator overhead it's possible that you'll not be able to do anything about it. It depends on what hardware the guest is trying to access. You can use:
>
> `perf kvm --host stat live`
> `perf kvm --host stat live --event=ioport`
> `perf kvm --host stat live --event=mmio`
>
> to check where kvm spends time during each of the kvm exits.

The `perf` tool can be installed on Ubuntu via `apt-get install linux-tools`.

[Previous: Possible improvements](3_POSSIBLE_IMPROVEMENTS.md)
[Next: QEMU Disk utils/LibGuestFS handy commands](5_USEFUL_TOOLS.md)
