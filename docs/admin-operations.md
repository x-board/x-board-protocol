
Admin operations
================

The mode 00 contains admin operations, which aren't done on a single pin, but rather
on the whole device. 

Here's a quick overview of what the admin operations are (planned to be)

    00 00: reserved
    00 01: ping
    00 02: list pins
    00 03: list capabilities 
    00 04: get device identifier
    00 05: get host version
    00 06: reboot device
    00 07 <address>: set i2c address 
    00 08 <address>: set address temporarily (lost on reboot, prevent unnecessary write) 
    00 09: reset address (undo 00 05)