
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

The values `EF` and `FF` are not valid modes, aren't valid parts of operations and
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
- The ranges should always be as long as possible. So, a response of `00 33 44 77 FF`
  is not valid and should be replaced with `00 77 FF`.
- Ranges should be listed from low to high.

This should all be rather intuitive, but stating it explicitly means that there is
only a single canonical way to represent any layout of pins.