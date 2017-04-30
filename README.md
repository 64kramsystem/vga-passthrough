# Table of contents #

0. [General introduction](#general-introduction)
1. [Introduction to VGA Passthrough](01_INTRODUCTION_TO_VGA_PASSTHROUGH.md)
2. [Basic setup](02_BASIC_SETUP.md)
3. [Possible improvements](03_POSSIBLE_IMPROVEMENTS.md)
4. [Useful tools](04_USEFUL_TOOLS.md)
5. [References](05_REFERENCES.md)

# General introduction #

This repository includes my notes for setting up a VGA Passthrough on a Linux machine (it's targeted to Ubuntu; more advanced users can adapt it to other distributions).

VGA passthrough is a setup which allows virtualized environments (QEMU, in this case) to perform 3D acceleration at near native speed.

The rationale for this document is that the information on the subject is spread, confusing and outdated, so I've decided to create a single, consistent document - a reference.

## Help/Contributions ##

Contributions or any other form of help (improvements, extensions...) are very appreciated.

The main area to look at is [Possible improvements](); possibly, minor things can be improved in [Basic setup]().

The best workflow is to create a Pull request with the modifications, but you can also send me an email.

## Reference system ##

This guide has been executed on the following system:

- Ubuntu 16.04 x86-64
- ASRock Mini ITX FATAL1TY Z170 Gaming-ITX/AC
- Intel i7-6770K
- Gigabyte GTX 960 GV-N960IXOC-2GD

there are also references to my previous one:

- Gigabyte GA-F2A88XN-WIFI FM2+ Mini ITX
- AMD A10-7800
- Gigabyte GTX 960 GV-N960IXOC-2GD
