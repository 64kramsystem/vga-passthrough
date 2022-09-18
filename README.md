# General introduction

This repository includes my notes for setting up a VGA Passthrough on a Linux machine (it's targeted to Ubuntu; more advanced users can adapt it to other distributions).

VGA passthrough is a setup that allows virtualized environments (QEMU, in this case) to perform 3D acceleration at near native speed.

The rationale for this document is that the information on the subject is spread, confusing and outdated, so I've decided to create a single, consistent document - a reference.

## Table of contents

0. [General introduction](README.md)
1. [Introduction to VGA Passthrough](1_INTRODUCTION_TO_VGA_PASSTHROUGH.md)
2. [VGA Passthrough Problems](2_VGA_PASSTHROUGH_PROBLEMS.md)
3. [Basic setup](3_BASIC_SETUP.md)
4. [Input handling](4_INPUT_HANDLING.md)
5. [Monitors and audio](5_MONITORS_AND_AUDIO.md)
6. [Troubleshooting](6_TROUBLESHOOTING.md)
7. [Possible improvements](7_POSSIBLE_IMPROVEMENTS.md)
8. [Profiling KVM](8_PROFILING_KVM.md)
9. [Sample IOMMU groups](9_SAMPLE_IOMMU_GROUPS.md)
10. [QEMU Disk utils/LibGuestFS handy commands](10_USEFUL_TOOLS.md)
11. [References](11_REFERENCES.md)

## Help/Contributions

Contributions or any other form of help (improvements, extensions...) are very appreciated.

The main area to look at is [Possible improvements](6_POSSIBLE_IMPROVEMENTS.md); possibly, minor things can be improved in [Basic setup](3_BASIC_SETUP.md).

The best workflow is to create a Pull request with the modifications, but you can also send me an email.

## Reference systems

This guide has been tested on several systems (see the [Sample IOMMU groups](9_SAMPLE_IOMMU_GROUPS.md) chapter), using the LTS Ubuntu versions from 16.04 onwards.

[Next: Introduction to VGA Passthrough](1_INTRODUCTION_TO_VGA_PASSTHROUGH.md)
