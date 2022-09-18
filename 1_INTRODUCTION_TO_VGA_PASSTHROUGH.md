# Introduction to VGA Passthrough

## Advantages of VGA Passthrough

Although setting up a VGA Passthrough may arguably be a "convenience net negative" (switching system on a dual boot system takes less than a minute), there are very interesting use cases.

The main, significant advantage, is the rollback ability of virtual machines. Since with this setup it's trivial to rollback a system by deleting a single file (assuming a diff disk setup), malware essentially doesn't pose a threat.  
(To be pedantic, there is the risk of malware capable of crossing the virtualization boundaries, but this is a tiny risk; arguably, a disk failure is more likely.)

Another significant advantage is the ease of reinstallation (which is an unfortunate routine of Windows systems); again, with a diff disk setup, one can have a frozen snapshot of the installed system, and roll back to it by deleting one file.

Of course, VGA Passthrough is not appropriate for those seeking 100% of performance, but for those who are OK with "95%", there are significant advantages.

## Guide and audience orientation

This guide is intended to be a walkthrough, more than a troubleshooting guide.

Therefore, the audience is people who own hardware already known to work well with VFIO, and want an easy, hassle-free, way to setup the virtualization, rather than those who spend hours trying to make passthrough work on unsupported hardware, or to achieve impractical but ideal setups.

This is not as obvious as it sounds, because there is a surprising amount of users willing to spend many hours trying to solve the infamous audio crackling issue, when a 3$ USB sound card solves the issue permanently and perfectly.

## Philosophy

I think one common misunderstanding of VGA Passthrough is to conceptually put software before hardware, with the assumption that with enough tweaking and experimentation, a system will work well.

This is not the case in real world: VGA Passthrough is not a very mature/supported technology yet, both in the software and hardware, and fixing certain issues is impossible without low-level knowledge of the technologies involved.

Trying to hammer unsupported hardware to make it work generally leads down the rabbit hole (where there is *always* one more change worth trying... that doesn't ultimately work), wasting large amounts of time and hair.

Independently of the maturity though, the upside is that on systems where the VGA Passthrough works without tweaks, it's very solid, virtually with no issues.

For time-sensitive people, my advice is to briefly test if the system is compatible (by following this guide), and if it doesn't, to plan a system upgrade, without attempting any tweak.

[Previous: General introduction](README.md) | [Next: VGA Passthrough Problems](2_VGA_PASSTHROUGH_PROBLEMS.md)
