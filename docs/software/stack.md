# Software Stack

There are several moving parts to this whole project, most of which are
organized in our [GitHub organization](https://github.com/GReX-Telescope).
The notable pieces of software are:

- [snap_bringup](https://github.com/GReX-Telescope/snap_bringup) - SNAP bringup and configuration
- [T0](https://github.com/GReX-Telescope/GReX-T0) - UDP Packet
  capture and exfil to heimdal
- [heimdall (T1)](https://github.com/GReX-Telescope/heimdall-astro) - Our fork of the
  pulse detection pipeline which removes clustering and RFI excision
- [T2](https://github.com/GReX-Telescope/GReX-T2)
- [FrontendModule](https://github.com/GReX-Telescope/FrontendModule) - Hardware and software design for the Frontend Module (FEM)

These are supported by some fundamental libraries

- [sigproc_filterbank](https://github.com/kiranshila/sigproc_filterbank) - A
  rust library for reading/writing SIGPROC filterbank files
- [psrdada-rs](https://github.com/kiranshila/psrdada-rs) - A rust library for
  interacting with PSRDADA buffers
- [casperfpga_rs](https://github.com/kiranshila/casperfpga_rs) - A rust library for
interacting with the SNAP board over TAPCP