
The basics of the protocol
==========================

Message format
--------------

The format of a message is this:

    <mode> <operation*> [<pin>] [<data*>]

Here, a value `<in angular brackets>` represent a single byte value, while a similar
value with an `<added asterisk>` represents a values of one or more bytes. 
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