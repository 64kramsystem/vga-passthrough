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

## Guest-exclusive cards must still be managed by the host

In theory, permanently assigning a card to the `vfio` driver (therefore, using it exclusively in the guest O/S) is a sensible choice: it minimizes instability, and avoids having to restart X when starting a VM.

However, it turns out that this strategy has a serious downside, that is very easy to overlook: when not used by the VM, the card will consume a significant amount of electricity, and get hot.

This happens because the `vfio` driver doesn't manage the power state of the card, so the card doesn't go into the power-saving mode.

The only way to solve this problem is to have the video driver manage it. There are different approaches to accomplish this.

The first is to actually use the card also in the host - it will be managed by X, and indirectly by the driver. The downside is that one needs to terminate the X session before starting the VM, and vice versa.

The second is to make the video driver handle the card while preventing X from taking ownership of the card. It goes like this:

1. make the `vfio` driver control the card on startup (X has an option not to take ownership of a video card, but it's not reliable)
2. after X starts, unbind the card from the `vfio` driver, and bind it to the video card driver.

Now, when one needs to start the VM, unbind the card from the video driver, and bind it to the `vfio` driver, and when the VM session stops, vice versa.

This is an interesting approach, but unfortunately, it's not completely reliable - once in a while, when binding to the video driver, the card does *not* go into power saving mode.

## Some features may not be supported when passing a card through

Not all the hardware features may be supported when passing a card through. The last time I've tried, in 2020, the [Resizeable BAR](https://www.pcmag.com/how-to/boost-gaming-speed-with-a-click-what-is-resizable-bar-and-can-it-juice) functionality was not supported, and caused the VM to hang, unless it was deactivated.

## Card drivers may not support VGA passthrough reliably

Again, due to driver limitations, VGA passthrough may not work reliably. At least some AMD cards/drivers (e.g. RX 5000/6000 series) are not able to consistently perform a card reset when terminating a VM session, causing the host to hang.

This is the "AMD Reset bug", which seems to be caused by Windows not resetting the card on powerdown, and the Linux driver not being able to do it when taking control back.

Regardless of the cause, this is [very](https://www.reddit.com/r/VFIO/comments/qboqvl/dreaded_amd_reset_bug_on_5600xt/) [problematic](https://forum.level1techs.com/t/6700xt-reset-bug/181814/35).

## Conclusion

VFIO is a complex topic, and one should healthily distrust typical "IT WORKS P3RF3CTLY" statements, as they are typically motivated by attention-seeking, without regard for technical accuracy.

At least until the time I had it set up (2020) VFIO could not work without significant downsides (the very least being reliability). It seems that the situation is still similar (again, be skeptical towards "it works perfectly" statements), but certainly, it's possible that support will improve.

I'm personally a very big fan of it, but until it will be a reliable technology (if it will ever be), matter of factually, dual booting is the only reliable solution.

[Previous: Introduction to VGA Passthrough](1_INTRODUCTION_TO_VGA_PASSTHROUGH.md) | [Next: Basic setup](3_BASIC_SETUP.md)
