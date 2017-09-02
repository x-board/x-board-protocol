
Listing Pins and Capabilities
=============================

Listing pins and capabilities are two of the most important parts of the x-board
protocol. They are the thing that enables the rest of the protocol and a starting
point for a client.

This document describes the two calls and the structure of the response that should
be given.

Aims and goals
--------------

The intention is to keep the responses as short as possible in typical cases. This
not only reduces the amount of data that will have to be sent without doing anything,
it also reduces the amount of space taken up in a microprocessor's memory.

However, at the same time, the responses should also be intuitive. This is meant to
make it easy to implement for both the software (or hardware) of a x-board device and
an x-board client. At the same time, this also makes it easier to follow along with 
the output when trying to debug a device without using a client.

Another important thing is the combination of forwards and backwards compatibility.
Basically, both of these are provided by these two calls and some rules about how to
treat changing (or not changing) of calls.

Forwards compatibility
----------------------

For forwards compatibility, the rule is that a client should ignore any parts of the
capabilities that it doesn't know about. It may of course let the user know that this
client doesn't support all capabilities of the board, but it should just treat the
given methods as if they aren't there.

EF and FF
---------

The values `DF`, `EF` and `FF` are not valid modes, aren't valid parts of operations and
aren't valid pin numbers. This is all so they can be used in these calls to represent
other things.

List Pins
---------

The list pins call is pretty simple. The client makes this call:

    00 02

The board responds with ranges of pin numbers it has. It sends pairs of bytes, where
each first byte of a pair designates the beginning of a range and the second byte
designates the end of the range. Single pins that aren't part of a range are
represented by having its pin number listed twice (as the begin and end of a range).
Finally, the response is terminated with `FF`.

For example, a board that has pins 0, 3, 4, 7, 9, 10, 11, 12 and 13 would respond with:

    00 00 03 04 07 07 09 0D FF

There are several additional rules for consistency:

- Ranges should always go from the lower pin number to a higher one. So, the first
  number of a pair should always be lower than the second or the same.
- There should be no overlap between two separate ranges.
- The ranges should always be as long as possible. So, a response of `00 03 04 05 07 FF`
  is not valid and should be replaced with `00 07 FF`.
- Ranges should be listed from low to high.

This should all be rather intuitive, but stating it explicitly means that there is
only a single canonical way to represent any layout of pins.

List Capabilities
-----------------

(Note: I use quite a few examples of operations here. I am writing this before
they are stabilized, so they shouldn't be taken to actually mean that. Check
the pin operation reference instead.)

### The call ###

The other call is to list the capabilities of a board. This call is a bit harder
just because there is so much more information in it.

The client's call is once again pretty simple:

    00 03

### The response ###

However, the response is more involved. It will basically just list capabilities
of the board, using `DF`, `EF` and `FF` as operators. The mode, operation and pin parts
of a call are treated mostly the same in this response, so I will refer all three
together as the *command*. The data part of a call is treated rather differently.

The idea is simple. During the command part, you just list the byte that would be 
used in a call to state that it's a capability of the board. Then you can use `EF`
to "jump into" that byte and specify which parts of the byte sequence so far are
supported, or just continue if the entire byte sequence is supported. `FF` can be
used to terminate "go back up a level". The response is also terminated with an `FF`.

If that sounded complicated, here's an example. It is for a board that supports
only `00 01`, `00 02`, `00 03` and `00 04`. (This is less than the mandatory calls.)

    00 EF 01 02 03 04 FF FF

Or, here's the same thing formatted a little differently:

    00 
        EF
        01
        02
        03
        04
        FF
    FF
    
It starts with claiming mode `00`. Then it jumps into that mode using `EF` and
lists the parts of that mode it supports. It supports `01` (ping), `02` (list pins),
`03` (list capabilities) and `04` (board identifier). Then, it 
says it's done with mode `00` by sending a `FF`. Finally, it says it's done by sending
another `FF`.

You can also more `EF`'s to go into deeper levels. On top of that, you can use
`EF FF` to use a range instead of just a single value. Let's look at a device that
supports the mandatory calls, `00 07` (set I2C address) `00 08` (set I2C address
temporarily) and `00 08` (reset I2C address). It also supports digital set
(`01 01`) on pins 1 through 5, pin 7 and pins 10 through 20.

    00
        EF
        01
            EF FF
        05
        07
        08
        FF
    01
        EF
        01
            EF
            01
                EF FF
            05
            07
            0A
                EF FF
            14
            FF
        FF
    FF

In this example, we start by defining what we can do in mode `00`. There we can
do the range from `01` to `05`, `07` and `08`. The `FF` signifies that we're done
with mode `00`.

Next up is mode `01`, here we look at operation `01` and enter it using `EF`. We
support this operation on the pin range 1 to 5, the pin 7 and the pin range 10 to
20 (in hexadecimal). Finally, there are three `FF`'s. The first is to exit `01 01`
and drops us back to `01`. The second drops out of that as well. The third
signifies the end of the answer.

For data, it works slightly different. If using `EF` to step in leaves you in the
"data part" of the call, the next thing in the message is a single byte containing
the number of bytes of data this call takes. This could have been left out, but it
prevents a problem with forwards compatibility. It can also help if we later decide
to support variable length data and helps debugging. Next, there are twice that many
bytes. These are pairs of bytes, each representing a byte of the data part. The first
byte is the minimum value for that byte, the second is the maximum value.
After that, you drop out of the data part and back to the command automatically. 
This way a range of valid values is defined. If multiple ranges are valid, you can 
immediately enter the range *again* to define the next range.

(Note that the above will need a redesign. The current version needs to know the length
of every operation, which is slated not to be constant but a part of the protocol.
This means that under the current design, the client can't be forward compatible, which
is not the intended result! I am considering using `DF` as a third special value.)

Before I show that in another example, there's one more trick I want to introduce.
Normally, you would use `EF` to enter the thing you just defined. This can be a 
single or a range. However, you can also use it to mean "everything". This is done
by using `EF` to step in without having defined what to step into.

Alright, let's put it into practice. Here, we have a board with pins 0-5 and 10-15
that supports `01 02`(PWM which has one byte of data) on pin 1-3 but only values
of 128 or higher and supports PWM fully on ports 5 and 10-15. It also supports
`01 03` (SoftPWM which has two bytes of data, a value and a frequency) with any value
on frequency 100-200 on all pins and with values 50-100 on frequency 50 also on all
pins.

    00
        EF
        01
            EF
            FF
        05
        FF
    01
        EF
        02
            EF
            01
                EF FF
            03
                EF
                01
                80 FF
            05
                EF FF
            0F
                EF
                01
                80 FF
            FF
        03
            EF
                EF
                02
                00 FF
                64 C8
                
                EF
                02 
                32 64
                32 32
            FF
        FF
    FF

As always, we start off with the obligatory calls. After we exit mode `00`, we enter
mode `01` and immediately continue to operation `01`. There, we define pin range 1-3
and jump into it. Next is `01` which is the number of bytes of data this operation
takes. Then we have `80` (hexadecimal for 128) as the minimum and `FF` (hexadecimal 
for 255) as the maximum. Do note that this `FF` has nothing to do with dropping out 
of the data part, as that happens automatically. It's just the maximum value of the 
range. Then, we define the same range of values for pins 5-15. We can create a range 
over pins 6-9 because they don't exist. Finally, we drop out of `01 02` and back 
into `01`.

Next, we enter `01 03`. Immediately, we enter again, which means we enter "all pins".
For all pins, we start with `02` which is the number of bytes of data for the 
operation we define give `00` as the minimum value for value and `64` (hexadecimal
for 100) as the minimum value for frequency. Next, we define `FF` as the maximum for
value and `C8` (hexadecimal for 200) as the maximum for frequency. In order to say
"anything for value" we just gave it the range of 0-255. We drop out of the data
block automatically and enter it again immediately to define another range. This time
we define a single value for the frequency by giving it the range of 32-32. Then, we
drop out of everything we're still in, and then close off the response with `FF`.

### Rules about skipping and jumping ###

Before we can wrap this up, there are some rules that should be mentioned.

If there are multiple modes that have the same operations, (e.g. `02` SET and
`03` SET DEFAULT) you can skip any of them beyond the first by `EF`-stepping into
them. This means that for that mode, you support exactly the same for this mode 
as for the last mode with the same possibilities. This can greatly reduce the
size of your response. You're not allowed to not do `EF` on a mode in any other
situation, and you may not use double `EF` (to define things for all operations
in a mode) on a mode in any situation. This is because later versions of the
protocol may always support more operations for that mode. On top of that, stepping
into everything of a mode just doesn't make much sense.

(I am aware that the above piece isn't very clear. I don't even know what it says,
despite the fact that I was the one who wrote it. I plan to redesign and then
rewrite this functionality. The functionality should still be present and I actually
also want it to apply to (partial) operations.)

For a partial operation, you may both skip jumping in to state you support all its 
child operations and jump into all if all children have the same amount of remaining
operations and the same number of bytes of data. For both, though, you may only do
this if the partial state is officially stable. Such a stabilization is a promise
that the partial state won't get any new children. It is separate from all its
children being stabilized (because even then it could get another child) but can
of course only happen if all children are stable.

For a complete command, you can always skip the data (no `EF`) to allow anything the
protocol allows, but never jump in for everything (double `EF`) because the
data is always the last step. Commands that don't take data should of course
never be stepped into. For a command that does have a complete operation and still
need a pins you can either skip (no `EF`: "any data on any pin") or jump in for
everything (double `EF`: "this data on any pin").

### The canonical response ###

There are usually multiple ways to write a response for the same set of functionality.
There are rules for picking the canonical response. A board should always (try to)
give the canonical response. These rules are:

- Always pick the response that takes the lowest number of bytes

- If the number of bytes is the same, use the one that has the lower values
  (this translate into "prefer enumerations over ranges in such cases")

- If parts of the content can change places (without increasing the total size). Sort
  them by their values. When two ranges start at the same value, use do the shorter
  range first. Otherwise, just sort them lexicographically (i.e. first byte vs first
  byte, move to next byte in case of a tie, no more bytes is lower than any byte value)

A client should actually support any non-canonical responses as well. This is because
it can be easy to accidentally provide a non-canonical response. Moreover, the canonical
answer can sometimes even change based on outside factors (when finalizing a partial
operation without giving it more children makes a shortcut possible). On top of that,
this also reduces the chance of missing a corner case that resulted in a shorter
response.