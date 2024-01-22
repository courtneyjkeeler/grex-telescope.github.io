# Operation

We'll flesh out all the details on operation soon, but in the meantime here are the critical details.

## Turning on the SNAP

To turn on the SNAP, SSH into the Pi (password is the same as the host machine) via

```sh
ssh pi@192.168.0.2
```

Then on the pi create (if it doesn't already exist) a bash script called `snap.sh` with the following:

!!! warning

    This is assuming you have a V2 power supply, ON and OFF are reversed otherwise

```bash
#!/bin/env bash
# Usage: ./snap.sh <on|off>
BASE_GPIO_PATH=/sys/class/gpio
PWN_PIN=20
if [ ! -e $BASE_GPIO_PATH/gpio$PWN_PIN ]; then
  echo "20" > $BASE_GPIO_PATH/export
fi
echo "out" > $BASE_GPIO_PATH/gpio$PWN_PIN/direction
if [[ -z $1 ]];
then 
    echo "Please pass `on` or `off` as an argument"
else
    case $1 in
    "on" | "ON")
    echo "0" > $BASE_GPIO_PATH/gpio$PWN_PIN/value
    ;;
    "off" | "OFF")
    echo "1" > $BASE_GPIO_PATH/gpio$PWN_PIN/value
    ;;
    *)
    echo "Please pass `on` or `off` as an argument"
    exit -1
    ;;
    esac
fi
exit 0
```

Make it executable is `chmod +x snap.sh`, and use `./snap.sh <on|off>` to control the power state of the SNAP.

## Running the Pipeline

In the `grex` folder, under `pipeline` there is the single bash script that runs the pipeline.
Simply calling it `./grex.sh` should start everything up. By default, it will run the normal detection pipeline.
If you want to just run T0 (packet capture and first stage processing), remove the final line that calls `psrdada` and replace it (the whole line) with `filterbank`.

## Triggering voltage dumps

Normally, T2 will send triggers to T0 to dump the voltage ringbuffer to disk. You can emulate this by sending a UDP packet to the trigger socket any other way. One simple way is with bash

```bash
echo " " > /dev/udp/localhost/65432
```

## SSH Port Tunneling

In some circumstances, it may be usefull to access ports on the GReX server on your local computer remotely. We can accomplish this using [SSH Tunneling](https://www.ssh.com/academy/ssh/tunneling-example).

One example of why we might want to do this is to access the 10 GbE switch configuration that is located in the
far-side GReX box. It runs a normal web page on a static ip of `192.168.88.1`. You can access this from a web
browser if you are sitting at the GReX server, but not remotely.

To access it using SSH tunneling, we can forward that IP's port 80 (standard HTTP) to our local computer at some
unused, non-privaleged port.

```shell
ssh -L 8080:192.168.88.1:80 username@grex-server-address
```

Another example is perhaps you want to run a [Jupypter Hub](https://jupyter.org/hub) instance on the GReX server. In that case, the website it is hosting is on the server itself, so you would run:

```shell
ssh -L 8080:localhost:80 username@grex-server-address
```

# Data Processing

## Voltage Dumps

The voltage dumps are stored in [NetCDF](https://www.unidata.ucar.edu/software/netcdf/), a machine-independent
data format intended for array-oriented scientific data such as ours. This format is a subset of HDF5 and is
supported by all major programming languages. Not only that, but it is self describing! Our files contain axis
variables on each dimension to remove any ambiguity as to which coordinate each data point refers to.

Using the NetCDF binaries, we can quickly introspect a voltage dump to see how it is laid out

```shell
user@grex:/hdd/data/voltages$ ncdump -h grex_dump-20240122T055935.nc
netcdf grex_dump-20240122T055935 {
dimensions:
        time = 524288 ;
        pol = 2 ;
        freq = 2048 ;
        reim = 2 ;
variables:
        double time(time) ;
                time:units = "Days" ;
                time:long_name = "Dynamic Barycentric Time (TDB) since J2000" ;
        string pol(pol) ;
                pol:long_name = "Polarization" ;
        double freq(freq) ;
                freq:units = "Megahertz" ;
                freq:long_name = "Frequency" ;
        string reim(reim) ;
                reim:long_name = "Complex" ;
        byte voltages(time, pol, freq, reim) ;
                voltages:long_name = "Channelized Voltages" ;
                voltages:units = "Volts" ;
}
```

Part of the trickiness here is HDF5 (and NetCDF by extension) doesn't support complex numbers, so the real and imaginary components are stored independently as another dimension.

### Python Reading Example

In Python, an excellent library for dealing with dimensional data is [xarray](https://docs.xarray.dev/en/stable/getting-started-guide/index.html). This
library supports a large number of file formats, which are optional dependencies.
For us, we need the netCDF4 python package, but it's probably best just to
use the complete installation.

It might also help if you read the xarray [page on NetCDF](https://docs.xarray.dev/en/stable/user-guide/io.html).

Reading this file with

```python
xarray.open_dataset("grex_dump.nc")
```

will associate all the axes to the appropriate dimensions, we just have to construct the complex numbers manually.

A one-liner to do this would be

```python
voltages = ds["voltages"].sel(reim="real") + ds["voltages"].sel(reim="imaginary")*1j
```

This may take a while for large dumps though, and you may want to slice it up into chunks (not sure if there is an elegant way to do that here).

But now you can do eveything xarray can do!

A few examples:

#### Stokes I

Calculated by taking the time-average of the sum of magnitude squared of the voltages

```python
stokesi = (abs(voltages)**2).sum(dim="pol").mean(dim="time")
```

#### H1 Line

Get the Stokes intensity around the H1 line (+/- 1 MHz) as a function a time

```python
h1 = (abs(voltages)**2).sum(dim="pol").sel(freq=slice(1421,1419)).sum(dim="freq")
```