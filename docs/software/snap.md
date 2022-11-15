# SNAP Configuration and Bringup

The digital backend to the GReX system is a
[SNAP](https://casper.astro.berkeley.edu/wiki/SNAP) board from the CASPER group
at Berkeley. This board contains the analog to digital converters and Xilinx
FPGA to perform the digitization and F-engine components of the system.

The setup and configuration of this board has seemingly never been well
documented, so we'll try to make it as painless as possible here.

The FPGA Simulink model is stored
[here](https://github.com/GReX-Telescope/gateware) with the latest releases
found [here](https://github.com/GReX-Telescope/gateware/releases). Grab the
latest fpg file, and you're good to go - no reason to recompile it.

## TAPCP and the Raspberry Pi

The gateware we've built for the SNAP board includes a small CPU called the MicroBlaze
that hosts a small webserver to deal with runtime interactions with FPGA memory. This server
gets an IP address from a DHCP server running on the main capture computer. This interface can also
be used to reprogram the SNAP if the gateware changes. By deafult, we'll ship SNAP boards that
have the GReX gateware preprogramed, but it's always possible to reprogram it.

This interface is over the UDP protocol TFTP, where the folks at CASPER have wrapped reading and
writing files as the interaction to both the flash and FPU memory. We've written a wrapper to the
so called "TAPCP" protocol [here](https://github.com/kiranshila/tapcp_rs). It is with this library
that the packet capture code informs the SNAP when to activate timing signals.

## FPGA Clocks, References, PPS, Synthesizers

The FPGA needs an external clock source, which is being provided via one of the
output of the Valon synthesizer in box. Additionally, the board needs "pulse per
second" (PPS) ticks, which come from the GPS reciever. If the user is in a place
which doesn't have GPS (maybe in a lab during testing), we can override that
check.

TODO! Somehow

TODO! Check clock is good and everything is locked? Does this happen after ADC configuration?
