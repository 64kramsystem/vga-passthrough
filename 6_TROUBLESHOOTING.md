# Troubleshooting

## General virtual machine issues: use a more recent firmware

Sometimes, issues are due to bugs in the OVMF firmware. They typically manifest themselves as hangs on boot, and are sometimes triggered by QEMU updates.

In such cases, a good first debug attempt is to build the latest firmware from source.

Prepare the machine:

    sudo apt-get install build-essential uuid-dev iasl git gcc-5 nasm

Clone the repository and change path:

    git clone --recursive https://github.com/tianocore/edk2.git
    cd edk2

Compile and build the tools:

    make -C BaseTools
    export EDK_TOOLS_PATH=$(pwd)/BaseTools
    source edksetup.sh BaseTools

Configure the build, in this case for an X64 target:

    perl -i -pe 's/^(ACTIVE_PLATFORM).*              /$1 = OvmfPkg\/OvmfPkgX64.dsc/x'  Conf/target.txt
    perl -i -pe 's/^(TOOL_CHAIN_TAG).*               /$1 = GCC5/x'                     Conf/target.txt
    perl -i -pe 's/^(TARGET_ARCH).*                  /$1 = X64/x'                      Conf/target.txt

Build:

    build

Enjoy!:

    $ ls -1 Build/OvmfX64/DEBUG_GCC5/FV/OVMF_*.fd
    Build/OvmfX64/DEBUG_GCC5/FV/OVMF_CODE.fd
    Build/OvmfX64/DEBUG_GCC5/FV/OVMF_VARS.fd
[Previous: Input handling](4_INPUT_HANDLING.md) | [Next: Possible improvements](6_POSSIBLE_IMPROVEMENTS.md)

## QEMU 4.0 hangs

QEMU 4.0 suffers from a regression, which causes both host and guest to hang on VM start (see [https://bugs.launchpad.net/qemu/+bug/1826422]).

In order to workaround this problem, add `kernel-irqchip=on` to the `-machine` option, for example:

```
-machine q35,accel=kvm,mem-merge=off,kernel-irqchip=on
```

QEMU 3.x/4.1+ users don't need this option.
[Previous: Monitors and audio](5_MONITORS_AND_AUDIO.md) | [Next: Possible improvements](7_POSSIBLE_IMPROVEMENTS.md)
