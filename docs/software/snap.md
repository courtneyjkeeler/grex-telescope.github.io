# SNAP Configuration and Bringup

The digital backend to the GReX system is a
[SNAP](https://casper.astro.berkeley.edu/wiki/SNAP) board from the CASPER group
at Berkeley. This board contains the analog to digital converters and Xilinx
FPGA to perform the digitization and F-engine components of the system.

The setup and configuration of this board has seemingly never been well
documented, so we'll try to make it as painless as possible here.

## KATCP and the Raspberry Pi

The SNAP board itself has no nonvolatile memory, so every time it is power-cycled, it must be reconfigured.
Rather than require the user to bring along (and know how to use) a JTAG programmer, the folks at CASPER
have added a Raspsberry Pi header such that the GPIO from a Pi can bit-bang JTAG. To expose this functionality
from remote devices, saving you the trouble from SSHing, they've implemented a
KATCP server on the Pi known as
[tcpborphserver](https://casper.astro.berkeley.edu/wiki/Tcpborphserver). [KATCP](https://katcp-python.readthedocs.io/en/latest/_downloads/361189acb383a294be20d6c10c257cb4/NRF-KAT7-6.0-IFCE-002-Rev5-1.pdf)
is a monitor and control protocol developed by the folks at SARAO that purports easy usage and extension. [Here](https://casper.astro.berkeley.edu/wiki/KATCP) is the list of commands they've added, which has not been updated since 2012.

TODO! Where do we get the magic image? Do we have to configure it?

### KATCP Networking

The Pi is (hopefully) configured to speak KATCP on `10.10.1.3:7147`. You can check this by opening a telnet connection at that address
and running the `?watchdog` command. You should receive a `!watchdog ok` response.

## Bringup

TODO Cleanup This

katcp=0.5.4
corr=0.7.3
construct=2.5.0

```python
from corr import katcp_wrapper
f = katcp_wrapper.FpgaClient('10.10.1.3')
f.listbof()
```
