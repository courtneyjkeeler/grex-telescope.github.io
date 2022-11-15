# Software Stack

There are several moving parts to this whole project, most of which are
organized in our [GitHub organization](https://github.com/GReX-Telescope).
The notable pieces of software are:

- [snapctl](https://github.com/GReX-Telescope/snapctl) - SNAP bringup and configuration
- [byte_slurper](https://github.com/GReX-Telescope/byte_slurper) - UDP Packet
  capture and exfil to heimdal
- [heimdall](https://github.com/GReX-Telescope/heimdall-astro) - Our fork of the
  pulse detection pipeline which removes clustering and RFI excision
- T2, T3, etc
- [fem_mnc](https://github.com/GReX-Telescope/fem_mnc) - Monitor and control of
  the FEM through the Pi's network connection

These are supported by some fundamental libraries

- [sigproc_filterbank](https://github.com/kiranshila/sigproc_filterbank) - A
  rust library for reading/writing SIGPROC filterbank files
- [psrdada-rs](https://github.com/kiranshila/psrdada-rs) - A rust library for
  interacting with PSRDADA buffers
- [casperfpga_rs](https://github.com/kiranshila/casperfpga_rs) - A rust library for
interacting with the SNAP board over TAPCP