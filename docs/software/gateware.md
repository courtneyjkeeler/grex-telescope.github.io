# Gateware

The FPGA gateware for GReX is built using the [CASPER
Toolflow](https://casper-toolflow.readthedocs.io/en/latest/index.html#a-note-on-operating-systems).
Because of the reliance on proprietary tools, it is quite difficult to set up.
And end-user of GReX shouldn't have to build the gateware themselves as we will
provide pre-built bitstream `FPG` files that one can directly upload.
These steps here are mostly for documentation of the gateware effort and completeness.

## Design Overview

The gateware itself is composed of three major functional blocks:
1. Digitization
2. Channelization
3. Packetization

We will go over the design and implementation of each block here.

### Digitization

At the beginning of the signal path, the raw voltage present at the analog to digital converter's 
(ADC) input in converted into a signed 8 bit fixed point number with the fixed point at 7. This 
results in the full scale voltage data spanning from -1 to 1. As the ADCs present on the SNAP FPGA
board have several inputs, we have a collection of multiplexers to select the proper pairs.

The implementaion of the ADC interface in the gateware is setup such that each conversion cycle
yields two samples subsequent in time. This is an artifact of the FPGA clock running at 250 MHz while
the ADC is running at 500 MHz. We need the ADC to run at this speed to nyquist sample the input at 500 Msps
for our 250 MHz of bandwidth.

These pairs of subsequent samples need to be selected for the appropriate ADCs as well, which is the job of
the second multiplexer. Both of these multiplexers are set at runtime via the standard casper interface, which
will be covered later.

Finally, these two sets of inputs are fed into the next stage, the F-engine.

### Channelization

The F-Engine, or channelizer, takes both polarization's time sample pair time streams and converts them into
a stream of channelized voltage data. To do so we form the "typical" F-Engine by using a polyphase filterbank and
FFT. Together, these form a standard short-time fourier transform. Given the bandwidth of 250 MHz and our selected
channel discritization of 2048 channels, we get ~122 kHz per channel every 8.192 us.

The output of this F-Engine has quite high bit depth, to prevent overflows. To match our networking, we need to
requantize this to something manageable.

The network connection has 10 Gb/s of bandwith total, so if we resample both polarizations to 8+8 bit complex numbers,
that's 16 bits per channel per polarization per 8.192 us, which is exactly 8 Gb/s. This gives us plenty of headroom and
more than enough precision.

To accomplish this requantization, we multiply both complex components by a user-selectable gain term and then take
the 8 most significant bits.

### Packetization

The last part of the gateware is packaing the channelized voltage data to send to the server. The way the CASPER
10 GbE block works is by accepting 64 bit words and appending them to a first-in-first-out (FIFO) buffer when a
separate "tx_valid" signal is true. The issue here is that the clock for the 10 GbE core is external and fixed,
independent of the FPGA clock. To make matters worse, it's slower than our FPGA clock of 250 MHz. So, if we clocked
data in every cycle of the FPGA clock, this buffer would overflow. Not only that, but the data we have every FPGA clock
cycle isn't 64 bits, it's 32 bits as we have both polarization's 8+8 bit complex channel value.

So, a solution is to "transose" the data where every other clock cycle we create a 64 bit word of two channels worth of
data. Therefore, we are clocking data in effectivley at 125 MHz, which is under the 10 GbE core clock and won't overflow.

After we clock in 1024 words (as each word is 2 channels and we have 2048 channels), we must set the "end of frame" flag
high (coincident with the last valid word). This will then transmit a UDP packet to the configured server and repeat the cycle.

### Timing

A work in progress feature is the addition of timing information to the payloads. This will be critical eventually, but is
currently untested. The SNAP board has an additional input which accepts a "pulse per second" signal where rising edges are
coincident with a second from an accuracte UTC clock. For the case of GReX, this will come from a GPS timing reciever or external
MASER source. In the gateware, there is an "arm" register which is logical ANDed with this PPS signal which starts the timing system.

So, the user will activate this "arm" register and note the next UTC second (assuming the computer that is performing the arming has
a reasonably accurate clock). Then, the next true second will "flag" the very first packet which will reset a packet counter. Every
UDP packet will have a header which contains the packet count since the first PPS signal. As the clocks of the system are discriminated
by this timing source, and we expect the packets to occur every 8.192 us, and we know when the UTC second of the first channel was, we
can work out the timestamp of each packet.

## Setting up the toolflow

The source of our repo will contain a git submodule that has a pinned version of
the toolflow (`mlib_devel`), as to eliminate any confusion. Kiran is building
the software on Arch Linux with:

- MATLAB R2021a
- Vivado 2021.1

<<<<<<< HEAD
Both of these were installed directly with their installers, as these old
versions don't work with the current AUR helpers (it also takes hundreds of
GB of HDD space and hours of installation).
=======
I used AUR helpers for both of these and played around getting their horribly vendored shared libraries to work.
>>>>>>> 3c38fdd (Add more gateware documentation)
