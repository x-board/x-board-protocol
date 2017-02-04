
Payload: 0 <operation> [<data>] 
Payload: <operation> <pin> [<data>]

First byte is zero: admin operations which do not have a pin. 
First byte is not zero: set something on a pin 

0 0: reserved
0 1: ping
0 2: list pins
0 3: list capabilities 
0 4: get host version
0 5: reboot device
0 6 <address>: set i2c address 
0 7 <address>: set address temporarily (lost on reboot, prevent unnecessary write) 
0 8: reset address (undo 0 5)

1 <pin> <value>: set pin to high (value == 1) or low (value == 0)
2 <pin> <value>: PWM
3 <pin> <value>: SoftPWM (set frequency?) 
4 <pin> <value> <time>: Fade to value in time, using PWM (min / max is digital at end?) 
5 <pin> <value> <time>: Fade to value in time, using SoftPWM (min / max is digital at end?)
6 <pin> <type> <on-time> <off-time> [<action-time>]: blink with time as frequency and type as blink type
    Blink type 0: on/off (no action time) 
    Blink type 1: fade using PWM (action time is length of fade) 
    Blink type 2: fade using SoftPWM (action time is length of fade)
7 <pin> <action> <data>: set any of actions 1-6 as automatic on startup 
8 <pin> <value> <time> <expect>: set pin to high or low after time has passed, expect no more i2c input until that time if expect is zero 

Watchdog? 