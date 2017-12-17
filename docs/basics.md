
The basics of the protocol
==========================

Wrapped in I2C
--------------

The protocol is defined in the terms of messages and responses. However, I2C doesn't
know either of these concepts. As such, the real protocol is wrapped in I2C. This
first part outlines how this is done.

An I2C Master calls the board in one of the following ways:

    START Write (message) STOP
    START Write (message) START Read length bytes STOP

The first variant is for when the client doesn't expect an answer. The second is for
the case where the client does expect an answer. The client is supposed to know the
length of the response for any call it does. For most calls that have a response,
this is defined as part of the protocol. There are two cases in which the length isn't
defined as part of the protocol because it can differ. These are the list pins
(`00 03`) and the list capabilities (`00 05`) calls. Both of these calls have a matching
call that will return the length of the response for this board (`00 02` and `00 04`
respectively) and the client is supposed to use these to find out the length of the
response before doing the calls themselves.

The protocol is defined in terms of the happy flow only. When the client software does
not follow the protocol (e.g. does a read without a preceding write), it doesn't 
matter what the board responds with. The board should, though, follow the protocol
again as soon as the client follows the protocol again. Of course, it's nice if the
behaviour is consistent and just always responding with zeroes is the recommended
strategy. However, efficient implementation of the protocol comes first.

Message format
--------------

The format of a message is this:

    <mode> <operation*> [<pin>] [<data*>]

Here, a value `<in angular brackets>` represent a single byte value, while a similar
value with an `<added asterisk*>` represents a value of one or more bytes. 
`[Rectangular brackets]` mean that the value optional.

The length of a message is always dictated by the protocol. No operation can be the
same as the first part of another operation. In other words, if you've read a part
of the operation, the protocol will always be able to tell whether that's the entire
operation or you need one or more bytes to complete the operation. Whether or not 
a pin number is present is defined by the combination of mode and operation
(though currently you can tell just by looking at the mode). The amount of data
that goes with an operation is also defined by the combination of mode and operation.

The bytes `FD`, `FE` and `FF` are not valid for use in the mode, the operation or the pin.
They are valid for use in data bytes.

A way of supporting variable amounts of data is still being looked at.

Lengths
-------

(Note: I'm planning to get rid of this part of the protocol all together.)

Initially, only the first byte is examined. If that byte isn't `FD`, `FE` or `FF`, the value is
taken as the length. `FD` and `FE` are reserved values and simply shouldn't occur here. If this
byte is `FF`, the next byte is read and it's taken as the number of bytes that will be used to
define the length. Then, that number of bytes are read and these bytes are combined Big-Endian
and their combined value is used as the length.


Modes: 00
---------

The mode 00 is the [admin mode](admin-operations.md) and represents calls that are done 
on the device instead of on one of its pins. All other currently planned modes are
[pin-based](pin-operations.md).