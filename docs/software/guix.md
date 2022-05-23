# Pipeline Modules and Guix

[Guix](https://guix.gnu.org/) is a functional package manager and tool to
instantiate and manage Unix-like operating systems. By functional, Guix defines
packages through a purely functional deployment model in which every build is
deterministic and is a pure function of the package's "inputs" or dependencies.
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

## Installation

To start from a bare server, we need a few prerequisites. First, to actually
build the install image, you need guix the guix binary installed. Installer isos
will be provided, eventually.

Clone the [GReX Guix](https://github.com/GReX-Telescope/guix-grex) repo and run,
this may take a while.

```bash
$ guix system image --image-type=iso9660 system/install.scm
```

After that, make an installation media, either CD or USB and boot into it. If
you want to make a USB, simply

```bash
$ sudo dd if=<the iso that was created> of=/dev/<your USB> status=progress bs=32M && sync
```

Once you boot into the installation media, select "Install using the shell based
process"

### Partitions

We'll partition the servers with UEFI, you can use any tool you like for this, I
like `cfdisk`. You'll need to make a 512M vfat partition for EFI and then ext4
the rest.

Then, format and mount the partitions

```bash
mkfs.ext4 /dev/root_partition
mkfs.fat -F 32 /dev/efi_system_partition
mount /dev/root_partition /mnt
mount --mkdir /dev/efi_system_partition /mnt/boot/efi
```

Now we can setup the installation environment with

```bash
herd start cow-store /mnt
```

### Initial Installation

First, we need to grab the system configuration for _this_ machine (assuming it
exists). GReX servers we control will have their own configuration file.

```bash
git clone https://github.com/GReX-Telescope/guix-grex
```

First, we'll copy over the channels and update (this may take a while)

```bash
mkdir -p ~/.config/guix
cp guix-grex/channels.scm ~/.config/guix
guix pull
hash guix  # This is necessary to ensure the updated profile path is active!
```

Then initialize the system with

```bash
cd guix-grex
guix system -L . init system/grex-01.scm /mnt
```

### Initial System Setup

Now you can reboot and setup the users!

First, we need to change the password. Login as root and

```bash
passwd      # For root
passwd grex # For the GReX user
```

Again, we need to pull down our channels list

```bash
git clone https://github.com/GReX-Telescope/guix-grex
ln -sf guix-grex/channels.scm ~/.config/guix/channels.scm
```

And then one final pull

```bash
guix pull
```

We're ready to go!
