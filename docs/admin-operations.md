
Admin operations
================

The mode 00 contains admin operations, which aren't done on a single pin, but rather
on the whole device. 

Here's a quick overview of what the admin operations are (planned to be)

    00 00: reserved
    00 01: ping
    00 02: Protocol version
    00 03: list pins length
    00 04: list pins
    00 05: list capabilities length
    00 06: list capabilities 
    00 07: get device identifier
    00 08: get host version
    00 09: reboot device
    00 0A: <address>: set i2c address 
    00 0B: <address>: set address temporarily (lost on reboot, prevent unnecessary write) 
    00 0C: reset address (undo 00 0A)

For more information on `00 02` to `00 05` see the 
[list pins and capabilities page](list-pins-capabilities.md).