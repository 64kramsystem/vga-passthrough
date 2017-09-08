# Introduction to VGA Passthrough

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

With this I want to highlight that VFIO is a relatively complicated matter, and you should healthily distrust typical "IT WORKS P3RF3CTLY" statements, as they are significantly subject to narcissism and/or plain incompetence (I had the very same experience when building a Raspberry Pi access point, and with the Surface Pro 3... but they're other stories).

I stress that this example is not representative of the success rate of VFIO setups; it's a suggestion that information[s] must be very carefully analyzed.

[Previous: General introduction](README.md)
[Next: Basic setup](2_BASIC_SETUP.md)
