
Admin operations
================

The mode 00 contains admin operations, which aren't done on a single pin, but rather
on the whole device. 

Here's a quick overview of what the admin operations are (planned to be)

    00 00: reserved
    00 01: ping
    00 02: list pins length
    00 03: list pins
    00 04: list capabilities length
    00 05: list capabilities 
    00 06: get device identifier
    00 07: get host version
    00 08: reboot device
    00 09: <address>: set i2c address 
    00 0A: <address>: set address temporarily (lost on reboot, prevent unnecessary write) 
    00 0B: reset address (undo 00 0A)

For more information on `00 02` to `00 05` see the 
[list pins and capabilities page](list-pins-capabilities.md).