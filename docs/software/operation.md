# Operation Details

We'll flesh out all the details on operation soon, but in the meantime here are the critical details.

## Turning on the SNAP

To turn on the SNAP, SSH into the Pi (password is the same as the host machine) via

```sh
ssh pi@192.168.0.2
```

Then on the pi create (if it doesn't already exist) a bash script called `snap.sh` with the following:

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
    echo "1" > $BASE_GPIO_PATH/gpio$PWN_PIN/value
    ;;
    "off" | "OFF")
    echo "0" > $BASE_GPIO_PATH/gpio$PWN_PIN/value
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

### Voltage Dump Format

The voltage dumps you get are written as HDF5 files. It contains a single tensor of complex voltage data from both polarizations. Additionally, there is a timestamp attribute on the data, as MJD Days (UTC). This timestamp is for the first sample in the time axis.

