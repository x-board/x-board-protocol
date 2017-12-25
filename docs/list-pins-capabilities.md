
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

FE and FF
---------

The values `FD`, `FE` and `FF` are not valid modes, aren't valid parts of operations and
aren't valid pin numbers. This is all so they can be used in these calls to represent
other things.

List Pins
---------

The list pins call will have different lengths for different boards. That's why a client
should first request the length of the list pins call. This is done through a simple call:

    00 03

The board should respond with a single byte containing the number of bytes that the answer
this board gives to the list pins call (`00 03`).

The list pins call  itself is also pretty simple. The client makes this call:

    00 04

The board responds with ranges of pin numbers it has. It sends pairs of bytes, where
each first byte of a pair designates the beginning of a range and the second byte
designates the end of the range. Single pins that aren't part of a range are
represented by having its pin number listed twice (as the begin and end of a range).

For example, a board that has pins 0, 3, 4, 7, 9, 10, 11, 12 and 13 would respond with:

    00 00 03 04 07 07 09 0D

Or formatted slightly differently:

    00 00
    03 04
    07 07
    09 0D

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

### The calls ###

Like with the list pins call, the length of the list pins will be different between
different boards and board versions. That's why there's once again a separate call
to get the length of the response for this board. This is done through the following
call:

    00 05

The client will respond with two bytes which combined in big-endian fashion represent
the length of the list capabilities call. It's not truly expected that many boards will
have responses longer than 255 bytes, but it sounds like it might be a possibility, so
protocol provides for it.

Next, there is the call to list the capabilities of a board. The response to this board
is more complicated because it is pretty dense in information, but the call itself is
rather simple:

    00 06

### The response ###

As mentioned, the response is more involved. It will basically just list capabilities
of the board, using `FD`, `FE` and `FF` as operators. The mode, operation and pin parts
of a call are treated mostly the same in this response, so I will refer all three
together as the *command*. The data part of a call is treated rather differently.

The idea is simple. During the command part, you just list the byte that would be 
used in a call to state that it's a capability of the board. Then you can use `FE`
to "jump into" that byte and specify which parts of the byte sequence so far are
supported, or just continue if the entire byte sequence is supported. `FF` can be
used to terminate "go back up a level".

If that sounded complicated, here's an example. It is for a board that supports
only `00 01`, `00 02`, `00 03` and `00 04`. (This is less than the mandatory calls.)

    00 FE 01 02 03 04 FF

Or, here's the same thing formatted a little differently:

    00 FE
        01
        02
        03
        04
    FF
    
It starts with claiming mode `00`. Then it jumps into that mode using `FE` and
lists the parts of that mode it supports. It supports `01` (ping), `02` (protocol version)
`03` (list pins length) and `04` (list pins). Then, it says it's done with mode `00` by 
sending a `FF`.

You can also use more `FE`'s to go into deeper levels. Even if an operation is multiple
bytes, you will always use `FE` to step into a single byte. On top of that, you can use
`FE FF` to use a range instead of just a single value. Let's look at a device that
supports the mandatory calls, `00 0A` (set I2C address) `00 0B` (set I2C address
temporarily) and `00 0C` (reset I2C address). It also supports digital set
(`01 01`) on pins 1 through 5, pin 7 and pins 10 through 20.

    00 FE
        01
            FE FF
        08
        0A
        0B
        0C
    FF
    01 FE
        01 FE
            01
                FE FF
            05
            07
            0A
                FE FF
            14
        FF
    FF

In this example, we start by defining what we can do in mode `00`. There we can
do the range from `01` to `08`, `0A`, `0B` and `0C`. The `FF` signifies that we're done
with mode `00`.

Next up is mode `01`, here we look at operation `01` and enter it using `FE`. We
support this operation on the pin range 1 to 5, the pin 7 and the pin range 10 to
20 (in hexadecimal). Finally, there are three `FF`'s. The first is to exit `01 01`
and drops us back to `01`. The second drops out of that as well.

For any operation that the board signals that it supports, it is implied that any
value can be given for the data part of the call. There is no way to specify in the
list capabilities response that only some values are allowed for the data part.

### The canonical response ###

There are usually multiple ways to write a response for the same set of functionality.
There are rules for picking the canonical response. A board should always (try to)
give the canonical response. These rules are:

- Always pick the response that takes the lowest number of bytes

- If the number of bytes is the same, use the one that has the lower values
  (this translate into "prefer enumerations over ranges in such cases")

- If parts of the content can change places (without increasing the total size). Sort
  them by their values. When two ranges start at the same value, do use the shorter
  range first. Otherwise, just sort them lexicographically (i.e. first byte vs first
  byte, move to next byte in case of a tie, no more bytes is lower than any byte value)

A client should actually support any non-canonical responses as well. This is because
it can be easy to accidentally provide a non-canonical response. Moreover, the canonical
answer can sometimes even change based on outside factors (when finalizing a partial
operation without giving it more children making a shortcut possible). On top of that,
this also mitigates the risk of missing a corner case that resulted in a shorter
response.