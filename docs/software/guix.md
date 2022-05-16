# Pipeline Modules and Guix

[Guix](https://guix.gnu.org/) is a functional package manager and tool to
instantiate and manage Unnix-like operating systems. By functional, Guix defines
packages through a purely functional deployment model in which every build is
deterministic and is a pure function of the pacakge's "inputs" or dependenices.
This solves the problem of [dependency
hell](https://en.wikipedia.org/wiki/Dependency_hell) and reproducability.

For GReX, many of our software modules and components exist as forks of
preexisting software as well as some custom code. To ensure all of these
components work together in harmony, we'll host a guix channel that provides the
build recipes for our software
[here](https://github.com/GReX-Telescope/guix-grex). Most of this software,
however, relies on the non-free CUDA runtime. As such, the user of these modules
must add the [proprietary HPC
channel](https://gitlab.inria.fr/guix-hpc/guix-hpc-non-free) to their guix channels.

```scheme
(cons (channel
        (name 'guix-hpc-non-free)
        (url "https://gitlab.inria.fr/guix-hpc/guix-hpc-non-free.git"))
      %default-channels)
```

!!! note

    As a note, we are using non-free CUDA as to leverage high-performance preexisting code.
    CUDA, and non-free software in general, denies users the ability to study and modify it.
    This is detrimental to user freedom and to proper scientific review and experimentation.
    As such, we ask that you not share these modules widely as to encourage more open alternatives.
