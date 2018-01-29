# General introduction

This repository includes my notes for setting up a VGA Passthrough on a Linux machine (it's targeted to Ubuntu; more advanced users can adapt it to other distributions).

VGA passthrough is a setup which allows virtualized environments (QEMU, in this case) to perform 3D acceleration at near native speed.

The rationale for this document is that the information on the subject is spread, confusing and outdated, so I've decided to create a single, consistent document - a reference.

## Table of contents

0. [General introduction](README.md)
1. [Introduction to VGA Passthrough](1_INTRODUCTION_TO_VGA_PASSTHROUGH.md)
2. [A word of caution](2_A_WORD_OF_CAUTION.md)
3. [Basic setup](3_BASIC_SETUP.md)
4. [Troubleshooting](4_TROUBLESHOOTING.md)
5. [Possible improvements](5_POSSIBLE_IMPROVEMENTS.md)
6. [Profiling KVM](6_PROFILING_KVM.md)
7. [QEMU Disk utils/LibGuestFS handy commands](7_USEFUL_TOOLS.md)
8. [References](8_REFERENCES.md)

## Help/Contributions

Contributions or any other form of help (improvements, extensions...) are very appreciated.

The main area to look at is [Possible improvements](5_POSSIBLE_IMPROVEMENTS.md); possibly, minor things can be improved in [Basic setup](3_BASIC_SETUP.md).

The best workflow is to create a Pull request with the modifications, but you can also send me an email.

## Reference system

This guide has been executed on the following system:

- Ubuntu 16.04 x86-64
- ASRock Mini ITX FATAL1TY Z170 Gaming-ITX/AC
- Intel i7-6770K
- Gigabyte GTX 960 GV-N960IXOC-2GD

there are also references to my previous one:

- Gigabyte GA-F2A88XN-WIFI FM2+ Mini ITX
- AMD A10-7800
- Gigabyte GTX 960 GV-N960IXOC-2GD

[Next: Introduction to VGA Passthrough](1_INTRODUCTION_TO_VGA_PASSTHROUGH.md)
