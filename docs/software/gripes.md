# Gripes

This is a list of things I've figured out, but not sure what to do with yet

- The image that was on the RPi we've been using runs an old version of
  TCPBorphServer, which is incompatible with `casperfpga`
- TCOBorphServer is running through init.d and not systemd, for some reason, and
  all of it's logs are discarded. Why?????
- PSRDADA is completley undocumented
- The dada interface to heimdall is undocumented
