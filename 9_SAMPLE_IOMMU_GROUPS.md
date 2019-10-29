# Sample IOMMU groups

## ASRock Mini ITX FATAL1TY Z170 Gaming-ITX/AC

The discrete GPU devices are isolated in the group 1:

```
IOMMU group 0
  00:00.0 Host bridge [0600]: Intel Corporation Xeon E3-1200 v5/E3-1500 v5/6th Gen Core Processor Host Bridge/DRAM Registers [8086:191f] (rev 07)
IOMMU group 1
  00:01.0 PCI bridge [0604]: Intel Corporation Xeon E3-1200 v5/E3-1500 v5/6th Gen Core Processor PCIe Controller (x16) [8086:1901] (rev 07)
  01:00.0 VGA compatible controller [0300]: NVIDIA Corporation GM206 [GeForce GTX 960] [10de:1401] (rev a1)
  01:00.1 Audio device [0403]: NVIDIA Corporation Device [10de:0fba] (rev a1)
IOMMU group 2
  00:02.0 VGA compatible controller [0300]: Intel Corporation HD Graphics 530 [8086:1912] (rev 06)
IOMMU group 3
  00:14.0 USB controller [0c03]: Intel Corporation 100 Series/C230 Series Chipset Family USB 3.0 xHCI Controller [8086:a12f] (rev 31)
  00:14.2 Signal processing controller [1180]: Intel Corporation 100 Series/C230 Series Chipset Family Thermal Subsystem [8086:a131] (rev 31)
IOMMU group 4
  00:16.0 Communication controller [0780]: Intel Corporation 100 Series/C230 Series Chipset Family MEI Controller #1 [8086:a13a] (rev 31)
IOMMU group 5
  00:17.0 SATA controller [0106]: Intel Corporation Q170/Q150/B150/H170/H110/Z170/CM236 Chipset SATA Controller [AHCI Mode] [8086:a102] (rev 31)
IOMMU group 6
  00:1c.0 PCI bridge [0604]: Intel Corporation 100 Series/C230 Series Chipset Family PCI Express Root Port #1 [8086:a110] (rev f1)
IOMMU group 7
  00:1c.4 PCI bridge [0604]: Intel Corporation 100 Series/C230 Series Chipset Family PCI Express Root Port #5 [8086:a114] (rev f1)
IOMMU group 8
  00:1c.6 PCI bridge [0604]: Intel Corporation 100 Series/C230 Series Chipset Family PCI Express Root Port #7 [8086:a116] (rev f1)
IOMMU group 9
  00:1d.0 PCI bridge [0604]: Intel Corporation 100 Series/C230 Series Chipset Family PCI Express Root Port #9 [8086:a118] (rev f1)
IOMMU group 10
  00:1f.0 ISA bridge [0601]: Intel Corporation Z170 Chipset LPC/eSPI Controller [8086:a145] (rev 31)
  00:1f.2 Memory controller [0580]: Intel Corporation 100 Series/C230 Series Chipset Family Power Management Controller [8086:a121] (rev 31)
  00:1f.3 Audio device [0403]: Intel Corporation 100 Series/C230 Series Chipset Family HD Audio Controller [8086:a170] (rev 31)
  00:1f.4 SMBus [0c05]: Intel Corporation 100 Series/C230 Series Chipset Family SMBus [8086:a123] (rev 31)
IOMMU group 11
  00:1f.6 Ethernet controller [0200]: Intel Corporation Ethernet Connection (2) I219-V [8086:15b8] (rev 31)
IOMMU group 12
  03:00.0 USB controller [0c03]: ASMedia Technology Inc. ASM1142 USB 3.1 Host Controller [1b21:1242]
IOMMU group 13
  04:00.0 Network controller [0280]: Intel Corporation Wireless 7260 [8086:08b1] (rev bb)
```

## Gigabyte GA-F2A88XN-WIFI FM2+ Mini ITX

I don't own this motherboard anymore, however, the discrete GPU devices were isolated in a single group.

## MSI B450M Mortar

The discrete GPU device is isolated in the group 8:

```
IOMMU group 0
        00:01.0 Host bridge [0600]: Advanced Micro Devices, Inc. [AMD] Family 17h (Models 00h-0fh) PCIe Dummy Host Bridge [1022:1452]
IOMMU group 1
        00:01.1 PCI bridge [0604]: Advanced Micro Devices, Inc. [AMD] Device [1022:15d3]
IOMMU group 2
        00:01.2 PCI bridge [0604]: Advanced Micro Devices, Inc. [AMD] Device [1022:15d3]
IOMMU group 3
        00:01.6 PCI bridge [0604]: Advanced Micro Devices, Inc. [AMD] Device [1022:15d3]
IOMMU group 4
        00:08.0 Host bridge [0600]: Advanced Micro Devices, Inc. [AMD] Family 17h (Models 00h-0fh) PCIe Dummy Host Bridge [1022:1452]
        00:08.2 PCI bridge [0604]: Advanced Micro Devices, Inc. [AMD] Device [1022:15dc]
        39:00.0 SATA controller [0106]: Advanced Micro Devices, Inc. [AMD] FCH SATA Controller [AHCI mode] [1022:7901] (rev 61)
IOMMU group 5
        00:08.1 PCI bridge [0604]: Advanced Micro Devices, Inc. [AMD] Device [1022:15db]
IOMMU group 6
        00:14.0 SMBus [0c05]: Advanced Micro Devices, Inc. [AMD] FCH SMBus Controller [1022:790b] (rev 61)
        00:14.3 ISA bridge [0601]: Advanced Micro Devices, Inc. [AMD] FCH LPC Bridge [1022:790e] (rev 51)
IOMMU group 7
        00:18.0 Host bridge [0600]: Advanced Micro Devices, Inc. [AMD] Device [1022:15e8]
        00:18.1 Host bridge [0600]: Advanced Micro Devices, Inc. [AMD] Device [1022:15e9]
        00:18.2 Host bridge [0600]: Advanced Micro Devices, Inc. [AMD] Device [1022:15ea]
        00:18.3 Host bridge [0600]: Advanced Micro Devices, Inc. [AMD] Device [1022:15eb]
        00:18.4 Host bridge [0600]: Advanced Micro Devices, Inc. [AMD] Device [1022:15ec]
        00:18.5 Host bridge [0600]: Advanced Micro Devices, Inc. [AMD] Device [1022:15ed]
        00:18.6 Host bridge [0600]: Advanced Micro Devices, Inc. [AMD] Device [1022:15ee]
        00:18.7 Host bridge [0600]: Advanced Micro Devices, Inc. [AMD] Device [1022:15ef]
IOMMU group 8
        10:00.0 VGA compatible controller [0300]: Advanced Micro Devices, Inc. [AMD/ATI] Ellesmere [Radeon RX 470/480/570/570X/580/580X] [1002:67df] (rev e7)
IOMMU group 9
        15:00.0 USB controller [0c03]: Advanced Micro Devices, Inc. [AMD] Device [1022:43d5] (rev 01)
        15:00.1 SATA controller [0106]: Advanced Micro Devices, Inc. [AMD] Device [1022:43c8] (rev 01)
        15:00.2 PCI bridge [0604]: Advanced Micro Devices, Inc. [AMD] Device [1022:43c6] (rev 01)
        16:00.0 PCI bridge [0604]: Advanced Micro Devices, Inc. [AMD] Device [1022:43c7] (rev 01)
        16:01.0 PCI bridge [0604]: Advanced Micro Devices, Inc. [AMD] Device [1022:43c7] (rev 01)
        16:04.0 PCI bridge [0604]: Advanced Micro Devices, Inc. [AMD] Device [1022:43c7] (rev 01)
        18:00.0 Ethernet controller [0200]: Realtek Semiconductor Co., Ltd. RTL8111/8168/8411 PCI Express Gigabit Ethernet Controller [10ec:8168] (rev 15)
IOMMU group 10
        2e:00.0 Non-Volatile memory controller [0108]: Samsung Electronics Co Ltd NVMe SSD Controller SM981/PM981 [144d:a808]
IOMMU group 11
        38:00.0 VGA compatible controller [0300]: Advanced Micro Devices, Inc. [AMD/ATI] Raven Ridge [Radeon Vega Series / Radeon Vega Mobile Series] [1002:15dd] (rev c6)
IOMMU group 12
        38:00.1 Audio device [0403]: Advanced Micro Devices, Inc. [AMD/ATI] Device [1002:15de]
        38:00.2 Encryption controller [1080]: Advanced Micro Devices, Inc. [AMD] Device [1022:15df]
        38:00.3 USB controller [0c03]: Advanced Micro Devices, Inc. [AMD] Device [1022:15e0]
        38:00.4 USB controller [0c03]: Advanced Micro Devices, Inc. [AMD] Device [1022:15e1]
        38:00.6 Audio device [0403]: Advanced Micro Devices, Inc. [AMD] Device [1022:15e3]
```

(taken from https://www.reddit.com/r/VFIO/comments/byw3wp/where_have_the_msi_b450m_mortar_board_and_can/eqn99f2)

## MSI B450M Mortar Max

The discrete GPU devices are isolated in the group 12:

```
IOMMU group 0
  00:01.0 Host bridge [0600]: Advanced Micro Devices, Inc. [AMD] Device [1022:1482]
IOMMU group 1
  00:01.3 PCI bridge [0604]: Advanced Micro Devices, Inc. [AMD] Device [1022:1483]
IOMMU group 2
  00:02.0 Host bridge [0600]: Advanced Micro Devices, Inc. [AMD] Device [1022:1482]
IOMMU group 3
  00:03.0 Host bridge [0600]: Advanced Micro Devices, Inc. [AMD] Device [1022:1482]
IOMMU group 4
  00:03.1 PCI bridge [0604]: Advanced Micro Devices, Inc. [AMD] Device [1022:1483]
IOMMU group 5
  00:04.0 Host bridge [0600]: Advanced Micro Devices, Inc. [AMD] Device [1022:1482]
IOMMU group 6
  00:05.0 Host bridge [0600]: Advanced Micro Devices, Inc. [AMD] Device [1022:1482]
IOMMU group 7
  00:07.0 Host bridge [0600]: Advanced Micro Devices, Inc. [AMD] Device [1022:1482]
  00:07.1 PCI bridge [0604]: Advanced Micro Devices, Inc. [AMD] Device [1022:1484]
  27:00.0 Non-Essential Instrumentation [1300]: Advanced Micro Devices, Inc. [AMD] Device [1022:148a]
IOMMU group 8
  00:08.0 Host bridge [0600]: Advanced Micro Devices, Inc. [AMD] Device [1022:1482]
  00:08.1 PCI bridge [0604]: Advanced Micro Devices, Inc. [AMD] Device [1022:1484]
  00:08.2 PCI bridge [0604]: Advanced Micro Devices, Inc. [AMD] Device [1022:1484]
  00:08.3 PCI bridge [0604]: Advanced Micro Devices, Inc. [AMD] Device [1022:1484]
  28:00.0 Non-Essential Instrumentation [1300]: Advanced Micro Devices, Inc. [AMD] Device [1022:1485]
  28:00.1 Encryption controller [1080]: Advanced Micro Devices, Inc. [AMD] Device [1022:1486]
  28:00.3 USB controller [0c03]: Advanced Micro Devices, Inc. [AMD] Device [1022:149c]
  28:00.4 Audio device [0403]: Advanced Micro Devices, Inc. [AMD] Device [1022:1487]
  30:00.0 SATA controller [0106]: Advanced Micro Devices, Inc. [AMD] FCH SATA Controller [AHCI mode] [1022:7901] (rev 51)
  31:00.0 SATA controller [0106]: Advanced Micro Devices, Inc. [AMD] FCH SATA Controller [AHCI mode] [1022:7901] (rev 51)
IOMMU group 9
  00:14.0 SMBus [0c05]: Advanced Micro Devices, Inc. [AMD] FCH SMBus Controller [1022:790b] (rev 61)
  00:14.3 ISA bridge [0601]: Advanced Micro Devices, Inc. [AMD] FCH LPC Bridge [1022:790e] (rev 51)
IOMMU group 10
  00:18.0 Host bridge [0600]: Advanced Micro Devices, Inc. [AMD] Device [1022:1440]
  00:18.1 Host bridge [0600]: Advanced Micro Devices, Inc. [AMD] Device [1022:1441]
  00:18.2 Host bridge [0600]: Advanced Micro Devices, Inc. [AMD] Device [1022:1442]
  00:18.3 Host bridge [0600]: Advanced Micro Devices, Inc. [AMD] Device [1022:1443]
  00:18.4 Host bridge [0600]: Advanced Micro Devices, Inc. [AMD] Device [1022:1444]
  00:18.5 Host bridge [0600]: Advanced Micro Devices, Inc. [AMD] Device [1022:1445]
  00:18.6 Host bridge [0600]: Advanced Micro Devices, Inc. [AMD] Device [1022:1446]
  00:18.7 Host bridge [0600]: Advanced Micro Devices, Inc. [AMD] Device [1022:1447]
IOMMU group 11
  03:00.0 USB controller [0c03]: Advanced Micro Devices, Inc. [AMD] Device [1022:43d5] (rev 01)
  03:00.1 SATA controller [0106]: Advanced Micro Devices, Inc. [AMD] Device [1022:43c8] (rev 01)
  03:00.2 PCI bridge [0604]: Advanced Micro Devices, Inc. [AMD] Device [1022:43c6] (rev 01)
  20:00.0 PCI bridge [0604]: Advanced Micro Devices, Inc. [AMD] Device [1022:43c7] (rev 01)
  20:01.0 PCI bridge [0604]: Advanced Micro Devices, Inc. [AMD] Device [1022:43c7] (rev 01)
  20:04.0 PCI bridge [0604]: Advanced Micro Devices, Inc. [AMD] Device [1022:43c7] (rev 01)
  22:00.0 Ethernet controller [0200]: Realtek Semiconductor Co., Ltd. RTL8111/8168/8411 PCI Express Gigabit Ethernet Controller [10ec:8168] (rev 15)
  25:00.0 VGA compatible controller [0300]: Advanced Micro Devices, Inc. [AMD/ATI] Caicos PRO [Radeon HD 7450] [1002:677b]
  25:00.1 Audio device [0403]: Advanced Micro Devices, Inc. [AMD/ATI] Caicos HDMI Audio [Radeon HD 6450 / 7450/8450/8490 OEM / R5 230/235/235X OEM] [1002:a...
IOMMU group 12
  26:00.0 VGA compatible controller [0300]: NVIDIA Corporation GM206 [GeForce GTX 960] [10de:1401] (rev a1)
  26:00.1 Audio device [0403]: NVIDIA Corporation Device [10de:0fba] (rev a1)
```

[Previous: Profiling KVM](8_PROFILING_KVM.md) | [Next: QEMU Disk utils/LibGuestFS handy commands](10_USEFUL_TOOLS.md)
