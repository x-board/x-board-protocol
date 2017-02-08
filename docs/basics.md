
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

Mandatory operations
--------------------

Most of the operations are optional, meaning that the board can either support them or not
support them. There are a couple of exceptions to this which must be supported. These are 
listed below.

    00 01: Ping
    
This operation can be used to check if the board is still alive or to see if the I2C address
is used by an x-board board.

    00 02: List pins

This operation requests for the board to identify which pins are available. This only includes
available pins, so not ones that are - for example - in use by I2C. The response to this call
is specified on the [list pins and capabilities page](list-pins-capabilities.md).

    00 03: List capabilities
    
This operation is the crux of having a protocol based on optional functionality. By calling
this function, the device can ask the board what it supports. The board must respond with a 
listing of what functionality it supports on which pins. There is an extensive specification
of the response to this call on the [list pins and capabilities page](list-pins-capabilities.md).

    00 04: Board identifier
    
The board should respond with an identifier that represents the hardware of the board. What
this should exactly be should be worked out and written down.

    00 05: Board host version

The board should respond with its version. This version should be different if the capabilities
of this same board have changed. This most commonly happen for a change in software, but should
the hardware change on a new revision of a board, the version should also be different. The exact
scheme used is up to the specific board, but a guideline will be written to create some 
consistency. It should be noted that while it is not expected to be a common occurrence, a version
number may be reused after a large number of versions to prevent running out of version numbers.