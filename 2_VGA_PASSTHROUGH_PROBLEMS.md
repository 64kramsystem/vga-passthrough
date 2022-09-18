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

## Conclusion

VFIO is a relatively complicated topic, and one should healthily distrust typical "IT WORKS P3RF3CTLY" statements, as they are typically motivated by attention-seeking, without regard for technical accuracy.

I'm not implying by any means that VFIO *can't* work well, but I had the exact same experience with other technologies, and it's crucial to know very well what one is getting into, before spending a considerable amount of time.

[Previous: Introduction to VGA Passthrough](1_INTRODUCTION_TO_VGA_PASSTHROUGH.md) | [Next: Basic setup](3_BASIC_SETUP.md)
