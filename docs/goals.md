
Design goals
============

Supporting different capabilities and pins
------------------------------------------

By having the device expose what pins it uses and what its capabilities it has,
devices with different capabilities can be supported with the exact same protocol.
Looking to the future, this means that support can easily be expanded to different
microcontrollers. It would even be possible for more specialized boards to support
the same protocol, letting them benefit from the client (and any alternative
clients) without having to write it themselves.

This is implemented by having a call that specifies which pins are available on 
the device as well as having one that lets a client know which functionality is
available on those pins.

Backwards compatibility
-----------------------

The aim is to have newer versions of the client work well with older versions of
the host software. This is achieved through the same methods above for supporting
different devices. Basically, it is just like communicating with a device that
simply supportsl less functionality.

Forwards compatibility
----------------------

Another aim is to have older client software work properly with newer host images.
The idea is that you can use older clients just fine (if they implement the
protocol properly) but that you just won't be able to use functionality that wasn't
in the protocol yet when the client was made.

Idea- and requirement-driven design
-----------------------------------

The design will be driven both by requirements for different projects and by
just the ideas of what one could do with such a board. This means that there may
very well be gaps in what is implemented that are caused by ideas that are just
not implemented yet.

Implementation-based  stabilization
-----------------------------------

While ideas are central to the design of the protocol, the existence of an
actual implementation is central to the stabilization of the protocol. This
can either be one of the host images of this project or a piece of software
or hardware developed externally. If it is an external project, it will of
course require cooperation in getting the protocol stable.

Guidelines promoting intuitiveness
----------------------------------

Alongside the protocol itself, there will be some guidelines on its implementation.
These are guidelines rather than hard requirements. These guidelines are meant
to make the use of the device as easy as possible and make it easy for a user to 
switch from one x-board supporting device to another as well as to make it as 
intuitive as possible to get started with x-board.