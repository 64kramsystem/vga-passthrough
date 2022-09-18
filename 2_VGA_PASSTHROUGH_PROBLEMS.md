# VGA Passthrough Problems

It's quite common to hear that "VGA passthrough works perfectly". This section explains some important pitfalls, and gives food for thought.

## A word of caution

VGA passthrough is associated with "near-native" or "95%" performance.

This is an example benchmark (with a little editing) of Metro Last Light Redux, on my previous system:

![Native](metro_benchmark/native_amd.png?raw=true "Native")
![VFIO](metro_benchmark/vfio_amd.png?raw=true "VFIO")

... guess what? It's almost exactly 95% of native performance!

But let's have a more detailed look at the graphs:

![Native](https://rawgithub.com/saveriomiroddi/vga-passthrough/master/metro_benchmark/native_amd.svg)
![VFIO](https://rawgithub.com/saveriomiroddi/vga-passthrough/master/metro_benchmark/vfio_amd.svg)

... ooops!

This setup had very good average framerate (truly ~95% across the board, as commonly), but severe issues with latency, causing wild stuttering.

"Near native performance"? Not so much.

## USB ports are emulated

USB ports, in the default configuration, are emulated by QEMU; they're not passed through as one would think.

This can cause some problems. For example, as of around 2020 (and possibly, even as of 2022), there is a problem across the USB stack (either the Linux driver, or a bug in QEMU) that prevents the Xbox 360 controller to work properly when connected via USB.

In some cases, it's possible to pass through the whole USB controller, which typically includes multiple ports, but in some cases, it isn't (for example, with my latest motherboard, it wasn't possible), or it's undesirable.

If one is not aware of this, it can be very confusing.

There are two workarounds to this problem:

1. Buy a USB card and pass it through;
2. Use [hotplugger](https://github.com/darkguy2008/hotplugger) - a program that does some trickery in order to allow passing single USB ports.

## Conclusion

VFIO is a complex topic, and one should healthily distrust typical "IT WORKS P3RF3CTLY" statements, as they are typically motivated by attention-seeking, without regard for technical accuracy.

At least until the time I had it set up (2020) VFIO could not work without significant downsides (the very least being reliability). It seems that the situation is still similar (again, be skeptical towards "it works perfectly" statements), but certainly, it's possible that support will improve.

I'm personally a very big fan of it, but until it will be a reliable technology (if it will ever be), matter of factually, dual booting is the only reliable solution.

[Previous: Introduction to VGA Passthrough](1_INTRODUCTION_TO_VGA_PASSTHROUGH.md) | [Next: Basic setup](3_BASIC_SETUP.md)
