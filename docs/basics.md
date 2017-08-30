
The basics of the protocol
==========================

Wrapped in I2C
--------------

The protocol is defined in the terms of messages and responses. However, I2C doesn't
know either of these concepts. As such, the real protocol is wrapped in I2C. This
first part outlines how this is done.

An I2C Master calls the board in one of the following ways:

    START Write (message) STOP
    START Write (message) START Read 1 byte START Read x bytes STOP

The first variant is for when the client doesn't expect an answer. The second is for
the case where the client does expect an answer. The first read retrieves the length of
the response, the second read retrieves the response itself and will be as long
as the value read earlier. There may be a way to designate a length that is higher
than 255 in a process that takes multiple reads. However, that hasn't been defined
yet, but we can state that for the moment `FF` isn't a valid length.

When a message doesn't have a response and the client does ask for one, the
board should simply respond with a length of zero. It's also valid for the client not
to request a response even though the call should generate one. This is all meant to
provide good backwards and forwards compatibility.

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
`[Recangular brackets]` mean that the value optional.

The length of a message is always dictated by the protocol. No operation can be the
same as the first part of another operation. In other words, if you've read a part
of the operation, the protocol will always be able to tell whether that's the entire
operation or you need one or more bytes to complete the operation. Whether or not 
a pin number is present is defined by the combination of mode and operation
(though currently you can tell just by looking at the mode). The amount of data
that goes with an operation is also defined by the combination of mode and operation.

The bytes EF and FF are not valid for use in the mode, the operation or the pin.
They are valid for use in data bytes.

A way of supporting variable amounts of data is still being looked at.

Modes: 00
---------

The mode 00 is the [admin mode](admin-operations.md) and represents calls that are done 
on the device instead of on one of its pins. All other currently planned modes are
[pin-based](pin-operations.md).