# SNAP Configuration and Bringup

The digital backend to the GReX system is a
[SNAP](https://casper.astro.berkeley.edu/wiki/SNAP) board from the CASPER group
at Berkeley. This board contains the analog to digital converters and Xilinx
FPGA to perform the digitization and F-engine components of the system.

The setup and configuration of this board has seemingly never been well
documented, so we'll try to make it as painless as possible here.

## Raspberry Pi

TODO! Where do we get the magic image? Do we have to configure it?

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
