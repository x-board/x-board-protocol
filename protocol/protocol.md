
Payload: 00 <operation> [<data>] 
Payload: <operation> <pin> [<data>]

First byte is zero: admin operations which do not have a pin. 
First byte is not zero: set something on a pin 

00 00: reserved
00 01: ping
00 02: list pins
00 03: list capabilities 
00 04: get host version
00 05: reboot device
00 06 <address>: set i2c address 
00 07 <address>: set address temporarily (lost on reboot, prevent unnecessary write) 
00 08: reset address (undo 0 5)

01 <pin> <value>: set pin to high (value == 01) or low (value == 00)
02 <pin> <value>: PWM
03 <pin> <value>: SoftPWM (set frequency?) 
04 <pin> <value> <time>: Fade to value in time, using PWM (min / max is digital at end?) 
05 <pin> <value> <time>: Fade to value in time, using SoftPWM (min / max is digital at end?)
06 <pin> <type> <on-time> <off-time> [<action-time>]: blink with time as frequency and type as blink type
    Blink type 00: on/off (no action time) 
    Blink type 01: fade using PWM (action time is length of fade) 
    Blink type 02: fade using SoftPWM (action time is length of fade)
07 <pin> <action> <data>: set any of actions 01-06 as automatic on startup 
08 <pin> <value> <time> <expect>: set pin to high or low after time has passed, expect no more i2c input until that time if expect is zero  