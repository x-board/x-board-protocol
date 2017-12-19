
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

    00 02

The board should respond with a single byte containing the number of bytes that the answer
this board gives to the list pins call (`00 03`).

The list pins call  itself is also pretty simple. The client makes this call:

    00 03

The board responds with ranges of pin numbers it has. It sends pairs of bytes, where
each first byte of a pair designates the beginning of a range and the second byte
designates the end of the range. Single pins that aren't part of a range are
represented by having its pin number listed twice (as the begin and end of a range).

For example, a board that has pins 0, 3, 4, 7, 9, 10, 11, 12 and 13 would respond with:

    00 00 03 04 07 07 09 0D

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

    00 04

The client will respond with two bytes which combined in big-endian fashion represent
the length of the list capabilities call. It's not truly expected that many boards will
have responses longer than 255 bytes, but it sounds like it might be a possibility, so
protocol provides for it.

Nest, there is the call to list the capabilities of a board. The response to this board
is more complicated because it is pretty dense in information, but the call itself is
rather simple:

    00 03

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

    00 
        FE
        01
        02
        03
        04
        FF
    
It starts with claiming mode `00`. Then it jumps into that mode using `FE` and
lists the parts of that mode it supports. It supports `01` (ping), `02` (list pins length),
`03` (list pins) and `04` (list capabilities length). Then, it 
says it's done with mode `00` by sending a `FF`.

You can also use more `FE`'s to go into deeper levels. Even if an operation is multiple
bytes, you will always use `FE` to step into a single byte. On top of that, you can use
`FE FF` to use a range instead of just a single value. Let's look at a device that
supports the mandatory calls, `00 09` (set I2C address) `00 0A` (set I2C address
temporarily) and `00 0B` (reset I2C address). It also supports digital set
(`01 01`) on pins 1 through 5, pin 7 and pins 10 through 20.

    00
        FE
        01
            FE FF
        07
        09
        0A
        0B
        FF
    01
        FE
        01
            FE
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
do the range from `01` to `07`, `09`, `0A` and `0B`. The `FF` signifies that we're done
with mode `00`.

Next up is mode `01`, here we look at operation `01` and enter it using `FE`. We
support this operation on the pin range 1 to 5, the pin 7 and the pin range 10 to
20 (in hexadecimal). Finally, there are three `FF`'s. The first is to exit `01 01`
and drops us back to `01`. The second drops out of that as well.

For data, it works slightly different. When you step into the data part of an operation,
you do so with the sequence of `FD FE` instead of just `FE`. The next thing in the message
is the length of the data this call takes. This is usually a single byte, but see 
[Lengths](basics.md#lengths) for a full specification of how this works. All of this is 
done with the question of forwards compatibility in mind; this way, the client does not 
have to know about details of the protocol that may be added after the client was created. 
Next, there are twice that many bytes. These are pairs of bytes, each representing a byte of
the data part. The first byte is the minimum value for that byte, the second is the maximum 
value. After that, you drop out of the data part and back to the command automatically. 
This way a range of valid values is defined. If multiple ranges are valid, you can 
immediately enter the range *again* to define the next range.

Before I show that in another example, there's one more trick I want to introduce.
Normally, you would use `FE` to enter the thing you just defined. This can be a 
single or a range. However, you can also use it to mean "everything". This is done
by using `FE` to step in without having defined what to step into.

Alright, let's put it into practice. Here, we have a board with pins 0-5 and 10-15
that supports `01 02 01`(PWM which has one byte of data) on pin 1-3 but only values
of 128 or higher and supports PWM fully on pins 5 and 10-15. It also supports
`01 02 02` (SoftPWM which has two bytes of data, a value and a frequency) with any value
on frequency 100-200 on all pins and with values 50-100 on frequency 50 also on all
pins.

    00
        FE
        01
            FE
            FF
        07
        FF
    01
        FE
        02
            FE
            01
                FE
                01
                    FE FF
                03
                    FD FE
                    01
                    80 FF
                05
                    FE FF
                0F
                    FD FE
                    01
                    80 FF
                FF
            
            02
                FE
                    FD FE
                    02
                    00 FF
                    64 C8
                    
                    FD FE
                    02 
                    32 64
                    32 32
                FF
            FF
        FF

As always, we start off with the obligatory calls. After we exit mode `00`, we enter
mode `01` and immediately continue to operation `01 01`. There, we define pin range 1-3
and jump into the data part with `FD FE`. Next is `01` which is the number of bytes of 
data this operation takes. Then we have `80` (hexadecimal for 128) as the minimum and 
`FF` (hexadecimal for 255) as the maximum. Do note that this `FF` has nothing to do with 
dropping out of the data part, as that happens automatically. It's just the maximum value
of the range. Then, we define the same range of values for pins 5-15. We can create a
range over pins 6-9 because they don't exist. Finally, we drop out of `01 02 01` and back 
into `01 02`.

Next, we enter `01 02 02`. Immediately, we enter again, which means we enter "all pins".
For all pins, we start with `02` which is the number of bytes of data for the 
operation we define give `00` as the minimum value for value and `64` (hexadecimal
for 100) as the minimum value for frequency. Next, we define `FF` as the maximum for
value and `C8` (hexadecimal for 200) as the maximum for frequency. In order to say
"anything for value" we just gave it the range of 0-255. We drop out of the data
block automatically and enter it again immediately to define another range. This time
we define a single value for the frequency by giving it the range of 32-32. Finally, we
drop out of everything we're still in.

### Identical parts of the protocol ###

There may be different parts of the protocol which have similar hardware requirements
and take the same arguments. In such cases, they will be listed as part of the protocol 
as "identical". Instead of stepping into such a part of the protocol, you can instead 
use `FD` to declare that any of its content is the same last element this was related to.
The `FD` is then followed by the part of the protocol for which you are using the fact
they are identical, which is then closed off with a `FF`.

An example of where such a relationship may be defined is modes `01` and `02`. The one
sets the value of a pin and the other sets a default for that pin when the board is
powered up. If the hardware support is there for saving a default state for a pin,
it is likely the two modes will have the same contents. It's also pretty obvious how
this works, so in the protocol definition, mode `03` won't be defined separately, but
just have a reference to being identical to `02`.

When something is defined as being identical to another part of the protocol, you
can use this technique both ways. In the definition, you can either define the original
and declare the "identical" one to have the same definition, or you can define the
identical one and state that the original has the same definition. You might even not
have the original and declare one thing that's identical to it to have the same
definition as another thing that also has the same definition. However, see
[The canonical response](#the-canonical-response) for which of the items should be
defined and which should be declared as the same.

Being identical is transitive. This means that if a mode is identical to another mode,
all operations and partial operations in that mode are also identical to the respective
operations and partial operation in the other mode. 

You can also use this technique on a pin or pin range. This makes sense if the original
is defined for multiple pins or pin ranges and you only want to copy the values for one
of these pins or ranges. It can also be used for pin ranges within a single operation if
both ranges share the same data. For a pin range, define it as identical to the first pin
of the range. In case multiple ranges start at the same pin, this shouldn't be used.
You can't use this after part of the data has been defined, nor when the entire data part 
has been defined.

Let's show this in some examples.

    00
        FE
        01
            FE
            FF
        07
        FF
    01
        FE
        02
            FE
            02
                FE
                01
                    FE FF
                05
                    FD FE
                    02
                    00 FF
                    80 FF
                0A
                    FE FF
                0F
                    FD 01 02 02 01 FF
                FF
        FF
    03
        FD 02 FF

The above example starts off with defining the mandatory operations. Then, it declares that it can
use SoftPWM on pins 1 to 5, but only with frequencies above 128 (`80` in hexadecimal). Next, it
uses the copying mechanic to state that it supports SoftPWM in the same way on pins 10-15 (`0A`
and `0F` in hexadecimal respectively). Finally, it uses the copying mechanic again on the mode
`03` to state it supports setting a default in exactly the same way as it supports setting pins
right now.

    00
        FE
        01
            FE
            FF
        07
        FF
    01
        FE
        02
            FE
            01
                FE
                01
                    FE FF
                03
                FF
            02
                FE
                01
                    FE FF
                05
                FF
            FF
    03
        FE
        02
            FD 01 02 01 FF
        FF

This example defines the mandatory operations, as well as PWM on pins 1 through 3 (with any data)
as well as SoftPWM on pins 1 through 5 (with any data as well). For the set default, it uses the
copying mechanic to state that it supports PWM in the same way, without stating it supports other
things (here: SoftPWM) for set default.

### Similar parts of the protocol ###

When the hardware requirements are similar, but the call takes different arguments, it can be
declared as similar. This has the same goal as things being marked as identical, but it works
slightly differently.

Just like for identical parts, it is started by `FD`, followed by the part it is similar to.
However, the next thing is `FE`. That is followed by a data definition, just like when providing
a normal definition of what data is allowed. So, it starts off with the [length](basics.md#lengths)
of the data, followed by that many pairs of minimum and maximum values for each of those bytes.
The protocol will define how the similar base is combined with new data, as well as how many bytes
the new data should be.

Let's make that clear by providing an example. We'll use `01 02 01` which is analog set using PWM
and `01 03` which is fade to value. `01 03` is defined to be similar to `01 02`. It expects one byte
of data when combining: time. In the combination process, it uses the allowed values for "value"
from `01 02` and uses them as the allowed values for both "from-value" and "to-value". It then adds
the new time value to get all defined data values.

    00
        FE
        01
            FE
            FF
        07
        FF
    01
        FE
        02
            FE
            01
                FE
                01
                    FE FF
                03
                FF
            FF
        03
            FD 01 01 FE
            01
            01 0A

Here, we define the standard operations and PWM with any value on pins 1 through 3. Then, it uses the
similarity to define fading from any value to any value with time within 1 and 10.

Instead of the `FD .. FE` sequence, the `FD .. FD FF` can be used. In this case, you aren't stepping
into the data part. This means that all possible values are allowed for the values that would be defined
for on top of the similarity.

    00
        FE
        01
            FE
            FF
        07
        FF
    01
        FE
        02
            FE
            01
                FE
                01
                    FE FF
                03
                    FD FE
                    01
                    00 80
                FF
            FF
        03
            FD 01 01 FD FF

Here, we define the standard operations and PWM with values up to 128 on pins 1 through 3. 
Then, it uses the similarity to define fading from any value up to 128 to any value up to 
128 with any value for time.

Similarity is transitive, like being identical is. This means that it can be used on
full (or larger partial) operation when a partial operation is defined as similar. It can
also be used for different pins or pin ranges like is the case identical parts, in the
case of pin ranges, you use the first pin here as well. However, pin ranges within the
same operation aren't automatically similar like how they are automatically the same.

Unlike is the case when copying identical parts, similarities are directional. This
means that they should always be used from the similar operation to the original.
If possible, it should point to the original operation, not to something that is
identical to it, even if the original one is simply defined as a copy of the identical
property. 

If this is not possible, you can use a different syntax instead. This starts with a `FD` as
always, but is then followed by the operation or partial operation that the current one
is similar to. This is always the (partial) operation on which the similarity is defined,
even if you are at a deeper level. That is then followed by another `FD` and the
(partial) operation it is identical to (on which being identical is defined), a third
`FD` and finally the part you want to copy. If these last two parts are the same, one of
the two may be left out together with a `FD`. This is all closed by either `FE` or `FD FF`
same as above.

In summary, all of the following are possible:

    # This is similar to 01 01 with the following additional data
    FD 01 01 FE
    
    # This is similar to 01 01 with any additional data
    FD 01 01 FD FF
    
    # 01 01 is identical 02 02 and this is similar to 02 02 with the following additional data
    FD 01 01 
    FD 02 02 FE
    
    # 01 01 is identical 02 02 and this is similar to 02 02 with any additional data
    FD 01 01 
    FD 02 02 FD FF
    
    # 01 01 is identical 02 02 and this is similar to 02 02 01 with the following additional data
    FD 01 01 
    FD 02 02 
    FD 02 02 01 FE
    
    # 01 01 is identical 02 02 and this is similar to 02 02 01 with any additional data
    FD 01 01
    FD 02 02
    FD 02 02 01 FD FF

For the later options, it's worth checking if using this actually saves data over just repeating
the definition. If it doesn't, using it won't give you the [the canonical response](the-canonical-response)

### The canonical response ###

There are usually multiple ways to write a response for the same set of functionality.
There are rules for picking the canonical response. A board should always (try to)
give the canonical response. These rules are:

- Always pick the response that takes the lowest number of bytes

- In case you can use `FD .. FF` to declare a part as identical, do so only if this
  decreases the length of the response (not if it is the same).

- In case a part of the response uses `FD .. FF` to declare it as the same as another
  part, make sure the (lexicographically) lower part is defined normally, while the
  higher part is declared identical.

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