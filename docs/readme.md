
x-board-protocol
================

This is the protocol for the x-board project, which defines a way to communicate
with x-board extension boards over I2C. The x-boars project itself aims to
implement this protocol small development boards, letting you use them as extension
boards to complement single-board computers. However, others are free to implement
this protocol as well.

Any board that supports this protocol will automatically be supported by all clients
that implement the protocol. The protocol is designed to allow partial functionality
and to have different versions of clients and boards (and board software) working
together, keeping the two as decoupled as possible.

For now, here's a list of the pages that make up the protocol definition:

- [Design goals](goals.md) that drive the protocol development
- [The basics](basics.md)
- [The admin operations](admin-operations.md)
  - [Listing pins and capabilities](list-pins-capabilities.md)
- [The pin operations](pin-operations.md)
- [Ideas](ideas.md) that haven't been put into the protocol yet