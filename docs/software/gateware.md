# Gateware

The FPGA gateware for GReX is built using the [CASPER
Toolflow](https://casper-toolflow.readthedocs.io/en/latest/index.html#a-note-on-operating-systems).
Because of the reliance on proprietary tools, it is quite difficult to set up.
And end-user of GReX shouldn't have to build the gateware themselves as we will
provide pre-built `FPG` files that one can directly upload. These steps here are
mostly for documentation of the gateware effort and completeness.

## Setting up the toolflow

The source of our repo will contain a git submodule that has a pinned version of
the toolflow (`mlib_devel`), as to eliminate any confusion. Kiran is building
the software on Arch Linux with:

- MATLAB R2018a
- Vivado 2019.1

Both of these were installed directly with their installers, as these old
versions don't work with the current AUR helpers (it also takes hundreds of
GB of HDD space and hours of installation). I had to tell MATLAB to _not_ use
their included version of libfreetype by following [this
guide](https://www.programmersought.com/article/56335887472/).
