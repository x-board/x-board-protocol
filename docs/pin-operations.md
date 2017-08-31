
Pin operations
==============

The modes beyond 00 contain pin operations, which are done on a single pin.

Mode 01 is "SET" and sets pins to a certain state. This is planned to be something like this:

    01 00 <pin> <value>: digital; set pin to high (value == 01) or low (value == 00)
    01 01 <pin> <value>: PWM
    01 02 <pin> <value>: SoftPWM (set frequency?)
    01 03 <pin> <value> <time>: Fade to value in time, using PWM (min / max is digital at end?) 
    01 04 <pin> <value> <time>: Fade to value in time, using SoftPWM (min / max is digital at end?)
    01 05 <pin> <type> <on-time> <off-time> [<action-time>]: blink with time as frequency and type as blink type
        Blink type 00: on/off (no action time) 
        Blink type 01: fade using PWM (action time is length of fade) 
        Blink type 02: fade using SoftPWM (action time is length of fade)

There are some changes I have planned here, but not worked out yet. The idea is to move the 
blink type to be the second byte of the operation. Likewise, PWM and SoftPWM will share the first
byte of their operation, only being differentiated by the second byte. Fade to value will sport a
similar system where they share a byte of the operation. Additionally, I want to define a lot of
"equivalences" here that allow parts of the capabilities response to be skipped because they will
be equal to another part. I am also considering adding a "your best PWM" operation, which defaults
to PWM for PWM capable pins, but falls back to SoftPWM when PWM is not available.

Mode 02 is "SET DEFAULT". Its usage is the same as 01, but instead of setting the pin to a state
right now, sets what the default is, which is triggered when the board is powered up.

Mode 03 is "DELAYED SIGNAL". A definite use case is to connect it to the power-on of your system,
in which case you can power down your system and rely on the board to wake it up again later on.
It may include ways to have the microcontroller go into sleep mode as well, saving even more power.
The format for delayed signal operations has not yet been decided.