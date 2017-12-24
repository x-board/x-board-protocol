
Mandatory operations
--------------------

Most of the operations are optional, meaning that the board can either support them or not
support them. There are a couple of exceptions to this which must be supported. These are 
listed below.

    00 01: Ping
    
This operation can be used to check if the board is still alive or to see if the I2C address
is used by an x-board board.

    00 02: Protocol version

This operation is used to see what version of the protocol the board supports. While the aim is to
create forwards and backwards compatibility, this will let you know whether there might be things
that are broken or things that the board may support that the client doesn't. The protocol version
uses [semver](http://semver.org), so some hard comparisons can actually be derived from the version 
numbers.

    00 03: List pins length
    00 04: List pins

These two operations request for the board to identify which pins are available. This only includes
available pins, so not ones that are - for example - in use by I2C. The response to these calls
is specified on the [list pins and capabilities page](list-pins-capabilities.md).

    00 05: List capabilities length
    00 06: List capabilities
    
These operations are the crux of having a protocol based on optional functionality. By calling
these functions, the client can ask the board what it supports. The board must respond with a 
listing of what functionality it supports on which pins. There is an extensive specification
of the response to these calls on the [list pins and capabilities page](list-pins-capabilities.md).

    00 07: Board identifier
    
The board should respond with an identifier that represents the hardware of the board. What
this should exactly be should be worked out and written down.

    00 08: Board host version

The board should respond with its version. This version should be different if the capabilities
of this same board have changed. This most commonly happen for a change in software, but should
the hardware change on a new revision of a board, the version should also be different. The exact
scheme used is up to the specific board, but a guideline will be written to create some 
consistency. It should be noted that while it is not expected to be a common occurrence, a version
number may be reused after a large number of versions to prevent running out of version numbers.