# Monitors and speakers

## Introduction

Using a guest with the VGA passthrough presents a minor inconvenience - handling separate outputs.

Since the outputs - at least the video signal - are separated, and one expects to fully use peripherals at least on the host, it's convenient to find a practical way to switch outputs.

With modern peripherals, one can take advantage of the multiple inputs, and avoid hardware or software switches.

For the setups, we assume the following hardware ports:

- integrated GPU, used by the host, with HDMI output
- discrete GPU, used by the guest, with Display Port output
- on-board SPDIF (digital) output
- monitor(s) with HDMI and Display Port inputs, and jack (analog) audio output
- speakers with SPDIF (digital) and jack (analog) inputs

The strategies below can be applied independently of using one or two monitors (of course, conditionally to the ports availability).

## No switch, one monitor

Without switch, one can take advantage of the multiple inputs of the peripherals.

In this case, the setup will be:

- integrated GPU HDMI + discrete GPU Display Port outputs -> monitor inputs
- on-board SPDIF + monitor jack outputs -> speakers inputs

When the host starts, the default is the monitor set to use the HDMI input, and the speakers to use the SPDIF input.

When switching to the guest, one sets the monitor to use the Display Port input, and the speakers to use the jack input.  

The audio works because the guest uses the discrete GPU audio device; the signal is carried via Display Port, and from the monitor, it goes to the speakers.

## Now switch, two monitors

The advantage of using a switch is that it avoids using the monitor switch - the first is generally a single button push, while the second involves navigating through menus; additionally, some monitors go into sleep shortly after losing the signal, which introduces an annoying delay while switching.

In this case, the setup will be:

- integrated GPU Display Port + discrete GPU Display Port outputs -> switch -> gaming monitor input
- integrated GPU HDMI -> secondary monitor input
- on-board SPDIF + monitor jack outputs -> speakers inputs

The strategy is essentially the same. The difference is that when using the guest, the input is switched via switch.
[Previous: Input handling](4_INPUT_HANDLING.md) | [Next: Troubleshooting](6_TROUBLESHOOTING.md)
