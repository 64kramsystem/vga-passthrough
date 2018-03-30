# Troubleshooting

## Introduction to troubleshooting

Currently, this section only has one suggestion; others may be added.

### Using a more recent firmware

Sometimes, issues are due to bugs in the OVMF firmware. They typically manifest themselves as hangs on boot, and are sometimes triggered by QEMU updates.

In such cases, a good first debug attempt is to build the latest firmware from source.

Prepare the machine:

    sudo apt-get install build-essential uuid-dev iasl git gcc-5 nasm

Clone the repository and change path:

    git clone https://github.com/tianocore/edk2.git
    cd edk2

Compile and build the tools:

    make -C BaseTools
    export EDK_TOOLS_PATH=$(pwd)/BaseTools
    . edksetup.sh BaseTools

Configure the build, in this case for an X64 target:

    export COMPILATION_MAX_THREADS=$((1 + $(lscpu --all -p=CPU | grep -v ^# | sort | uniq | wc -l)))
    
    perl -i -pe 's/^(ACTIVE_PLATFORM).*              /$1 = OvmfPkg\/OvmfPkgX64.dsc/x'  Conf/target.txt
    perl -i -pe 's/^(TOOL_CHAIN_TAG).*               /$1 = GCC5/x'                     Conf/target.txt
    perl -i -pe 's/^(TARGET_ARCH).*                  /$1 = X64/x'                      Conf/target.txt
    perl -i -pe "s/^(MAX_CONCURRENT_THREAD_NUMBER).*/\$1 = $COMPILATION_MAX_THREADS/x" Conf/target.txt

Build:

    build

Enjoy!:

    $ ls -1 Build/OvmfX64/DEBUG_GCC5/FV/OVMF_*.fd
    Build/OvmfX64/DEBUG_GCC5/FV/OVMF_CODE.fd
    Build/OvmfX64/DEBUG_GCC5/FV/OVMF_VARS.fd
[Previous: Input handling](4_INPUT_HANDLING.md) | [Next: Possible improvements](6_POSSIBLE_IMPROVEMENTS.md)
